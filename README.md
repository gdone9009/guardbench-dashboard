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

## 🌐 5. 배포 및 시연 접속 안내 (Live Demonstration)

- **통합 포털 웹 주소**: [https://gdone9009.github.io/guardbench-dashboard/](https://gdone9009.github.io/guardbench-dashboard/)
- **v5.0 최종 버전 접속**: [https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v5/](https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v5/)

#### 로컬 실행 방법
```bash
cd /Users/gdone/dev/aws/guardbench-dashboard-v5
python3 -m http.server 8085
```
브라우저에서 `http://localhost:8085` 접속 (또는 [`index.html`](file:///Users/gdone/dev/aws/guardbench-dashboard-v5/index.html) 직접 열기)

---

> 💡 **KOSA AWS 3팀 — GuardBench Project**  
> 팀원: 전공자/실력자 2인, 중급자 2인, 입문자 1인 (총 5인)  
> 기술 스택: Java 21, Spring Boot, Amazon Bedrock, Amazon SQS, AWS ECS Fargate, PostgreSQL, Terraform, Tailwind CSS
