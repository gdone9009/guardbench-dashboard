# 🛡️ GuardBench (Amazon Bedrock Guardrail 회귀 검증 & Quality Gate 오케스트레이터)

> **KOSA AWS 3팀 (GuardBench Project)**  
> **라이브 웹사이트 포털**: [https://gdone9009.github.io/guardbench-dashboard/](https://gdone9009.github.io/guardbench-dashboard/)  
> **메인 백엔드 저장소**: [`GuardBench/guardbench-backend`](https://github.com/GuardBench/guardbench-backend)  
> **로컬 개발 경로**: [`/Users/gdone/dev/aws`](file:///Users/gdone/dev/aws)

---

## 📌 1. 프로젝트 개요 (Project Overview)

**GuardBench**는 Amazon Bedrock Guardrail(AI 보안 울타리)의 정책을 수정·업데이트할 때 발생하는 **보안 결함(Security Regression), 사용성 저하(Usability Regression), PII 유출 위험**을 자동화된 1,000건의 테스트 스냅샷과 비동기 SQS 파이프라인을 통해 정밀 검증하고, 배포 승인 여부(**Quality Gate Verdict**)를 자동으로 판정하는 **엔터프라이즈 AI 회귀 평가 및 오케스트레이션 플랫폼**입니다.

---

## 💡 2. 왜 이렇게 설계했는가? (Architecture Rationale & Design Decisions)

GuardBench 아키텍처는 단순한 LLM 가드레일 호출 툴이 아닙니다. 실제 기업 운영 환경에서 직면하는 **5가지 핵심 엔지니어링 문제**를 해결하기 위해 다음과 같은 엄격한 도메인 원칙을 수립하고 설계했습니다:

### 1. **"Test What You Deploy" — 가변 DRAFT 직접 테스트 금지 및 Target Materialization**
- **문제점**: Bedrock 콘솔에서 편집 중인 가변 상태(Working Draft)를 그대로 테스트하면, 테스트 실행 중에 누군가 설정을 수정할 경우 테스트 결과와 실제 프로덕션 배포 버전 간의 불일치(Inconsistency)가 발생합니다.
- **해결책**: GuardBench는 배포 승인용 TestRun 생성 시 Draft 설정을 **numbered Version(`v2.0-MAT`)으로 불변 고정(Materialization)**한 후에만 테스트를 수행합니다. 이를 통해 **"테스트된 버전과 실제 배포되는 버전의 100% 동일성"**을 보장합니다.

### 2. **2축 분리 평가 프롬프트 (2-Axis Verdict Framework)**
- **문제점**: 단일 `PASS/FAIL` 지표만으로는 "이전 버전 대비 가드레일 성능이 좋아졌는지, 약해졌는지"를 입체적으로 알 수 없습니다.
- **해결책**: GuardBench는 평가 결과를 2개의 독립된 축으로 분리합니다:
  - **1축 (`Candidate Assertion`)**: 신규 버전이 기대 동작을 만족하는가? (`PASS` / `FAIL`)
  - **2축 (`Change Classification`)**: 이전 버전 대비 어떤 영향이 발생하는가? (`NO_CHANGE`, `IMPROVEMENT ✨`, `USABILITY_REGRESSION ⚠️`, `SECURITY_REGRESSION 🚨`)

### 3. **인프라 오류(`Execution Error`)와 정책 결함(`Assertion Failure`)의 엄격한 분리**
- **문제점**: AWS Bedrock API 네트워크 타임아웃이나 인프라 일시 장애가 발생했을 때, 이를 "가드레일 보안 필터 실패"로 오판하면 억울하게 배포가 차단되는 심각한 오류가 생깁니다.
- **해결책**: 인프라 실행 오류는 **`Execution Reliability 100%`** 인프라 축으로 관리하고, 정책 품질 평가는 **`Quality Gate Status`** 축으로 완전히 격리합니다. 실행 신뢰도가 부족할 경우 정책 평가를 진행하지 않고 `NOT_EVALUATED`로 안전하게 보존합니다.

### 4. **실시간 배포 정책 임계값 조절기 (Quality Gate Policy Configurator)**
- **문제점**: 모든 조직이 동일한 배포 기준을 가질 수 없습니다. (보안이 최우선인 자산관리부는 오탐을 감수하더라도 보안 결함 0건이 필수인 반면, 일반 고객 CS부는 사용성 저하 방지가 우선일 수 있음).
- **해결책**: 사용자가 `최대 허용 보안 결함 수`, `최대 허용 사용성 저하 수`, `최소 통과율`을 슬라이더로 직접 조정하면, 1,000건 평가 결과가 실시간 재산출되어 Quality Gate 승인 여부(`PASSED 🟢` / `FAILED 🔴` / `WARNING ⚠️`)가 동적으로 반영되도록 구현했습니다.

### 5. **OWASP Top 10 for LLM & 8개 부서별 1,000개 고유 테스트 케이스**
- **문제점**: 동일한 단순 프롬프트의 반복 테스트는 실제 해커의 정교한 탈옥(Jailbreak) 및 사내 기밀 유출 공격을 밝혀내지 못합니다.
- **해결책**: OWASP Top 10 for LLM(System Prompt Leak, Base64/Unicode Bypass, DAN, Crescendo 등) 및 사내 인사/연봉/의료/금융/DevOps 8개 부서별 시나리오를 조합하여 **100% 고유한 1,000개 공격 프롬프트 세트**를 구축했습니다.

---

## 🏗️ 3. 시스템 아키텍처 및 AWS 인프라 구성 (AWS Infrastructure)

GuardBench 인프라는 백엔드 오케스트레이션과 Bedrock 연동 워커를 **비동기 SQS 4대 큐 메커니즘**으로 분리하여 고성능 및 확장성을 확보했습니다.

```text
① API Request ──> SQS gb-run-resolve (FIFO) ──> ② Spring Boot Orchestrator (Materialization)
                                                         │
⑧ Quality Gate Verdict <── ⑦ SQS gb-run-finalize <── ⑥ RDS <── ⑤ ECS Fargate <── ④ SQS gb-workitems
 (Final Evaluation)         (Finalize Signal)    (attempt)   (ApplyGuardrail)    (Fan-out WorkItems)
```

- **Spring Boot Orchestrator**: Run 생성 접수, Target Materialization, TestCaseSnapshot 고정, SQS 팬아웃, 최종 Quality Gate 판정.
- **ECS Fargate Executor Workers**: SQS `gb-workitems` 큐에서 케이스를 컨슘하여 Bedrock `ApplyGuardrail` API 호출 후 Append-Only로 RDS에 기록.
- **SQS 4대 비동기 큐**:
  1. 📮 `gb-run-resolve` (FIFO): Run 생성 멱등성 보장
  2. 📤 `gb-workitems` (Standard): 엑세큐터 워커 대상 팬아웃
  3. 📥 `gb-run-finalize` (Standard): 실행 완료 시그널 전송
  4. 🚨 `gb-workitems-dlq` (DLQ): 실패 케이스 격리 및 Sweeper 메커니즘
- **Terraform IaC**: 전 인프라 자원을 테라폼 코드로 모듈화하여 `terraform apply` 단 한 줄로 개인 계정에 5분 만에 복제 배포 지원.

---

## 🖥️ 4. 대시보드 버전별 진화 과정 (v1 ~ v5 Evolution)

| 버전 (Version) | 핵심 특징 및 개발 목표 | 주요 기능 및 화면 |
| :--- | :--- | :--- |
| **v1.0 (기술 목업)** | 엔지니어링 3-Depth 구조 검증 | • Depth 1: 대시보드<br>• Depth 2: Test Run 상세<br>• Depth 3: Evaluation Inspector |
| **v2.0 (사용자 UI)** | 기획자/보안담당자 친화적 Modern UI | • 🚥 품질 게이트 상태 신호등 경고 배너<br>• 🧪 실시간 대화형 가드레일 샌드박스(Sandbox)<br>• 💡 용어집(Glossary) 탭 |
| **v3.0 (1,000 Scale)** | 1,000건 대용량 엔터프라이즈 에디션 | • 8개 부서 1,000건 동적 케이스 로딩<br>• 20/50/100개 단위 고성능 페이지네이션<br>• CSV / JSON 원클릭 Export |
| **v4.0 (OWASP 공격)** | 실제 알려진 LLM 프롬프트 공격 세트 | • OWASP Top 10 for LLM 1,000개 100% 고유 프롬프트<br>• DAN, Crescendo, Base64/Unicode 난독화 검증<br>• OWASP 10대 카테고리 필터링 |
| **v5.0 (Ultimate)** | **배포 정책 실시간 조절 & 감사 리포트** | • ⚙️ **Quality Gate Policy Configurator (임계값 슬라이더)**<br>• 🔍 **Side-by-Side Raw JSON Trace Code Diff**<br>• 📄 **Executive Audit Report 인쇄/PDF 모달**<br>• ➕ **실시간 사용자 커스텀 Case 추가** |

---

## 🚀 5. 최신 추가 개발 및 고도화 내역 (2026년 9월 최신 기준)

GuardBench 플랫폼은 초기 MVP 및 1차 부하 테스트 구축을 넘어 **SageMaker 머신러닝 평가기 연동, 대량 파일 임포트, 동시성 제어, 다차원 결과 분석 및 비주얼 테스팅** 영역으로 크게 고도화되었습니다.

| 구분 | 주요 개발 변경사항 | 💡 쉬운 비유 및 설명 |
| :--- | :--- | :--- |
| **백엔드 (Backend)** | • **SageMaker Classifier Evaluator 어댑터 추가** (`SageMakerClassifierEvaluatorAdapter`) <br>• **테스트 케이스 대량 일괄 등록 & 핑거프린트 멱등성** (`TestCaseBulkCreateCommand`) <br>• **TestRun별 개별 Quality Gate 임계값 정책** (`QualityGatePolicy`) <br>• **Attention Facets 차원별 정밀 결과 서비스** (`GetTestRunResultDetailService`) <br>• **WorkItem 동시성 한계 제어기** (`WorkItemConcurrencyController`) | • **외부 전문 진단 기기 연동**: Bedrock 가드레일뿐만 아니라 Amazon SageMaker 기반 커스텀 분류 모델도 평가기로 직접 연결할 수 있습니다.<br>• **CSV/JSON 서류 일괄 접수**: 1,000개 테스트 케이스를 중복 없이 안전하게 한 번에 일괄 등록합니다.<br>• **맞춤형 통과 기준표**: Run별로 보안 결함 허용치나 최소 통과율을 자유롭게 설정하여 판정합니다.<br>• **동시 작업 차량 수 제어**: SQS 메시지가 과도하게 쏟아져도 시스템이 멈추지 않도록 최대 동시 처리량을 일정하게 조절합니다. |
| **프론트엔드 (Frontend)** | • **CSV/JSON 대량 파일 업로드 패널** (`BulkTestCaseCreatePanel`) <br>• **Quality Gate 증거 시각화 & 차트** (`QualityGateEvidence`, `QualityGateMetricsChart`) <br>• **스텝별 대화형 진행률 표시기** (`RunProgressStepper`) <br>• **Vitest & Playwright 브라우저 테스팅 자동화** (`tests/browser/`) | • **엑셀/JSON 파일 원클릭 업로드**: 드래그 앤 드롭으로 1,000개 프롬프트 세트를 즉시 등록합니다.<br>• **품질 성적표 차트 시각화**: 통과율, 보안 결함, 사용성 저하 비율을 한눈에 파악할 수 있는 차트 배너를 제공합니다.<br>• **브라우저 로봇 검수기**: 화면 클릭과 UI 레이아웃이 깨지지 않도록 Vitest 브라우저 자동 테스트 슈트를 정립했습니다. |
| **인프라 (IaC & CI/CD)** | • **SageMaker Classifier 모듈화** (`sagemaker-classifier.tf`) <br>• **DB 접근 모듈화** (`db-access.tf`) <br>• **성능 모니터링 분석 권한 분리** (`performance-analysis-access.tf`) | • **AI 분류기 자동 인프라**: SageMaker 모델 엔드포인트와 전용 IAM 스코프를 테라폼 한 줄로 자동 구성합니다.<br>• **최소 권한 데이터베이스 보안**: DB 접근 권한 및 분석 전용 IAM 권한을 엄격히 계층화하여 보안성을 높였습니다. |

---

## 🔮 6. 더 고도화된 엔터프라이즈 서비스로 발전하기 위한 5가지 개선 로드맵

현재 GuardBench는 엔터프라이즈급 사전 회귀 평가 및 Quality Gate 기능을 갖추었으나, 향후 상용 수준의 완벽한 AI Safety 플랫폼으로 진화하기 위해 다음 5가지 고도화 방향을 제안합니다:

### 1. 🤖 **Multi-Evaluator Ensemble & Weighted Scoring (다중 평가기 앙상블 및 가중치 채점)**
- **개념**: 단일 Bedrock Guardrail 응답에만 의존하지 않고, **Bedrock + SageMaker Classifier + LLM-as-a-Judge(GPT-4o/Claude 3.5 Sonnet)**의 판정 결과를 가중치 앙상블(Ensemble)하여 종합 판정.
- **기대 효과**: 단일 평가기의 오탐(False Positive)과 미탐(False Negative) 비율을 99% 이상 보정.

### 2. ⚡ **Automated Red Teaming & Jailbreak Generation (자동 프롬프트 공격 생성기)**
- **개념**: 사용자가 1,000개의 프롬프트를 일일이 수동 작성하지 않더라도, **Red Teaming LLM Agent가 최신 탈옥(Jailbreak) 기법(Crescendo, Base64, Multilingual, Roleplay)의 변종 프롬프트 10,000개를 자동으로 생성 및 자동 확장**.
- **기대 효과**: 신종 공격에 대한 제로데이(Zero-day) 프롬프트 우회 방지.

### 3. 🛠️ **Guardrail Policy Auto-Tuning & Self-Healing (가드레일 자가 치유 및 자동 정책 튜닝)**
- **개념**: `SECURITY_REGRESSION` 발생 시, **"Bedrock Guardrail의 Sensitive Information Policy에서 PII 마스킹 레벨을 HIGH로 올리고, Content Filter의 Hate/Violence Threshold를 BLOCK(HIGH)로 상향해야 함"**을 코드로 제시해 주고 **[원클릭 자동 적용(Self-Healing)]** 버튼 제공.
- **기대 효과**: 보안 담당자의 수동 설정 조작 시간 90% 단축.

### 4. 👥 **Production Shadow Traffic Replay (프로덕션 섀도우 트래픽 리플레이)**
- **개념**: 실제로 챗봇 사용자가 입력하는 운영 환경 트래픽을 섀도우(Shadow) 복제하여 GuardBench 검증 파이프라인으로 백그라운드 재실행.
- **기대 효과**: 실제 사용자 질의 패턴을 바탕으로 프로덕션에 미칠 영향을 100% 정밀 예측.

### 5. 🔔 **CI/CD Release Gatekeeper & Collaboration Bot (자동 배포 승인 및 슬랙/팀즈 알림)**
- **개념**: GitHub PR 생성 시 자동으로 GuardBench TestRun을 실행하고, Quality Gate `PASSED` 시 PR 자동 승인/병합, `FAILED` 시 블록 및 상세 사유를 **Slack/Microsoft Teams 차트로 즉시 알림**.
- **기대 효과**: 개발자/보안팀 간 커뮤니케이션 비용 제거 및 완전 자동화된 DevSecOps 달성.

---

## 🚘 7. GuardBench vs Amazon Bedrock Guardrail의 역할 구분 (쉬운 비유)

```text
[실제 도로 주행] ──> Amazon Bedrock Guardrail (실시간 차선 이탈 방지 장치)
                      사용자가 챗봇에 질문할 때마다 위험한 질문(욕설, 개인정보)을 0.1초 만에 즉시 차단!

[자동차 안전 검사소] ──> GuardBench 플랫폼 (사전 회귀 평가 및 Quality Gate)
                         새 가드레일 정책을 도로에 내보내기 전, 1,000가지 충돌 테스트를 돌려
                         "이 정책을 배포해도 안전한가?" (PASSED / FAILED) 최종 승인을 내려주는 검수 센터!
```

---

## 🌐 8. 배포 및 시연 접속 안내 (Live Demonstration)

- **공식 팀 프론트엔드 포털 (최신 프로덕션 빌드)**: [http://localhost:3000/](http://localhost:3000/)
- **개인 멀티버전 포털 웹 주소**: [https://gdone9009.github.io/guardbench-dashboard/](https://gdone9009.github.io/guardbench-dashboard/)
- **v5.0 최종 버전 접속**: [https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v5/](https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v5/)

#### 로컬 공식 팀 프론트엔드 시연 구동 방법
```bash
cd /Users/gdone/dev/aws/guardbench-frontend
npm run build
python3 -m http.server 3000 --directory dist
```
브라우저에서 [`http://localhost:3000`](http://localhost:3000) 접속

---

> 💡 **KOSA AWS 3팀 — GuardBench Project**  
> 팀원: 전공자/실력자 2인, 중급자 2인, 입문자 1인 (총 5인)  
> 기술 스택: Java 21, Spring Boot, Amazon Bedrock, Amazon SageMaker, Amazon SQS, AWS ECS Fargate, PostgreSQL, Terraform, React 19, TypeScript, Vitest, Tailwind CSS


