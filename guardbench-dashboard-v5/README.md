# 🛡️ GuardBench Dashboard Version 5 (v5) - 상세 기술 설명서 & 설계 해설

> **GuardBench (Amazon Bedrock Guardrail 회귀 검증 & Quality Gate 오케스트레이터)**  
> **버전**: `v5.0 Ultimate Enterprise Evaluation & Policy Configurator Edition`  
> **라이브 접속 주소**: [https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v5/](https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v5/)

---

## 📌 1. 왜 이렇게 설계했는가? (Architecture Rationale & Design Decisions)

GuardBench v5 대시보드 및 회귀 평가 엔진은 단순히 화면을 구성한 것이 아니라, **실제 기업 환경에서 AI 가드레일을 배포할 때 발생하는 5가지 결정적 문제**를 해결하기 위한 엄격한 백엔드 도메인 원칙을 바탕으로 설계되었습니다:

### 1. **"Test What You Deploy" 원칙 — Target Materialization**
- **이유**: 가변 상태인 Working Draft(DRAFT)를 직접 테스트하면, 테스트 도중 누군가 설정을 수정할 경우 "테스트한 결과"와 "실제 프로덕션 배포 버전"이 달라지는 심각한 불일치가 생깁니다.
- **해결 방식**: GuardBench는 배포 승인용 TestRun 생성 시 Draft 설정을 **numbered Version(`v2.0-MAT`)으로 불변 고정(Materialization)**한 후에만 테스트를 수행하여 **100% 재현성(Reproducibility)**을 보장합니다.

### 2. **2축 분리 평가 체계 (2-Axis Verdict Framework)**
- **이유**: 단일 `PASS/FAIL` 결과만으로는 "이전 버전 대비 가드레일 성능이 개선되었는지, 신규 보안 구멍이 뚫렸는지"를 구분할 수 없습니다.
- **해결 방식**: 
  - **1축 (`Candidate Assertion`)**: 신규 버전이 기대 동작을 만족하는가? (`PASS` / `FAIL`)
  - **2축 (`Change Classification`)**: 이전 버전 대비 어떤 영향이 발생하는가? (`NO_CHANGE`, `IMPROVEMENT ✨`, `USABILITY_REGRESSION ⚠️`, `SECURITY_REGRESSION 🚨`)

### 3. **인프라 오류(`Execution Error`)와 정책 결함(`Assertion Failure`)의 엄격한 분리**
- **이유**: 네트워크 타임아웃, AWS API 일시 오류 등 인프라 장애를 AI 정책 필터 실패로 오판하면 억울하게 배포가 차단됩니다.
- **해결 방식**: 인프라 오류는 **`Execution Reliability 100%`** 카드로 별도 관측하고, 정책 품질 평가(`Quality Gate Status`)와 철저히 분리하여 실행 신뢰도가 부족할 경우 정책 평가를 진행하지 않고 `NOT_EVALUATED`로 안전하게 격리합니다.

### 4. **실시간 배포 정책 임계값 조절기 (Quality Gate Policy Configurator)**
- **이유**: 부서나 서비스 성격에 따라 요구되는 보안 수준이 다릅니다. (자산관리부는 보안 결함 0건 필수 vs 고객 CS부는 정상 질문 오탐 방지가 우선일 수 있음).
- **해결 방식**: 사용자가 `최대 허용 보안 결함 수`, `최대 허용 사용성 저하 수`, `최소 통과율`을 슬라이더로 직접 조정하면 1,000건 평가 결과가 실시간 재계산되어 Quality Gate 승인 여부(`PASSED 🟢` / `FAILED 🔴` / `WARNING ⚠️`)가 동적으로 변환되도록 구현했습니다.

### 5. **OWASP Top 10 for LLM & 8개 부서별 1,000개 고유 테스트 케이스**
- **이유**: 동일한 단순 프롬프트의 반복 테스트는 실제 해커의 정교한 탈옥(Jailbreak)이나 사내 인사/연봉 기밀 유출 공격을 선제 감지하지 못합니다.
- **해결 방식**: OWASP Top 10 for LLM(System Prompt Leak, Base64/Unicode Bypass, DAN, Crescendo 등) 및 사내 인사/연봉/의료/금융/DevOps 8개 부서별 시나리오를 조합하여 **100% 고유한 1,000개 공격 프롬프트 세트**를 작성했습니다.

---

## 🖥️ 2. 화면별 구성 요소 및 상세 사용 가이드

GuardBench v5는 상단 네비게이션을 통해 **5가지 핵심 뷰(View)**로 구성되어 있습니다:

### 📍 ① 📊 메인 대시보드 (Executive Dashboard)
- **동적 Quality Gate 상태 배너**: Configurator 슬라이더 설정에 따라 `PASSED 🟢`, `FAILED 🔴`, `WARNING ⚠️` 상태와 판정 사유가 실시간 갱신됩니다.
- **4대 핵심 KPI 카드**: Total Cases (1,000건), Security Regressions (30건 🚨), Usability Regressions (70건 ⚠️), Execution Reliability (100%).
- **Target Comparison Overview**: Baseline (`v1.2 Published`) vs Candidate (`v2.0-MAT Materialized`).
- **OWASP LLM 10대 카테고리 그리드**: 카테고리별 보안 결함 발생 건수 및 클릭 원터치 필터링.

### 📍 ② ⚙️ 배포 정책 설정 (Quality Gate Policy Configurator) - NEW!
- **최대 허용 보안 결함 (Security Regression)**: 0 ~ 50건 슬라이더 (기본 0건).
- **최대 허용 사용성 저하 (Usability Regression)**: 0 ~ 100건 슬라이더 (기본 50건).
- **최소 요구 통과율 (Pass Rate)**: 50.0% ~ 100.0% 슬라이더 (기본 90.0%).
- **동적 재산출**: 슬라이더를 움직이면 1,000건 결과가 실시간 재계산되어 배포 판정이 동적으로 변환됩니다.

### 📍 ③ 🛡️ 1,000개 Test Cases & 다중 입체 필터링 (Cases View)
- **다중 입체 필터 바**: OWASP 카테고리 + 담당 부서 + 심각도(CRITICAL, HIGH, MEDIUM, LOW) + 2축 평가 결과 + 실시간 프롬프트 검색.
- **고성능 페이지네이션**: 20 / 50 / 100개 단위 분할 랜더링.
- **➕ 커스텀 케이스 추가**: 신규 프롬프트 및 카테고리를 UI에서 직접 입력하여 1,000건 데이터셋에 실시간 추가 및 재검증.

### 📍 ④ 🔍 Side-by-Side Raw JSON Trace Code Diff Inspector - NEW!
- **코드 Diff 가시성**: Baseline v1.2와 Candidate v2.0-MAT의 Bedrock `ApplyGuardrail` Raw JSON Payload를 줄 단위 Red/Green 코드 하이라이트로 비교 점검.

### 📍 ⑤ 📄 Executive Quality Audit Report Generator - NEW!
- **C-Level & 감사용 인쇄 모달**: 경영진 및 보안 감사팀 제출용 표준 **GuardBench Executive Quality Audit Report** 모달 작성 및 `window.print()` 원클릭 PDF 출력 지원.

---

## 🔄 3. 버전 1부터 버전 5까지의 전체 진화 과정 (v1 ~ v5 Diff)

| 비교 항목 | v1 (기술목업) | v2 (사용자 UI) | v3 (1,000 Scale) | v4 (OWASP 1,000건) | **v5 (Ultimate Edition)** |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **케이스 구성** | 5건 개념검증 | 16건 부서별 | 1,000건 단순생성 | 1,000건 OWASP 고유 | **1,000건 + 사용자 커스텀 추가** |
| **배포 정책** | 고정 | 고정 | 고정 | 고정 | ⚙️ **실시간 Policy Configurator 슬라이더** |
| **감사 리포트** | 없음 | 없음 | CSV/JSON | CSV/JSON | 📄 **Executive Audit Report 인쇄/PDF 모달** |
| **JSON Diff** | 텍스트 | 텍스트 | 텍스트 | 텍스트 | 🔍 **Side-by-Side Code Diff Inspector** |

---

## 🚀 4. 로컬 실행 및 라이브 접속 방법

* **라이브 웹 주소**: [https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v5/](https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v5/)
* **로컬 실행 명령어**:
  ```bash
  cd /Users/gdone/dev/aws/guardbench-dashboard-v5
  python3 -m http.server 8085
  ```
  브라우저에서 `http://localhost:8085` 접속 (또는 [`index.html`](file:///Users/gdone/dev/aws/guardbench-dashboard-v5/index.html) 열기)

---

> 💡 **KOSA AWS 3팀 — GuardBench Project**
