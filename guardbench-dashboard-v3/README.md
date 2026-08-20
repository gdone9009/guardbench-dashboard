# 🚀 GuardBench Dashboard Version 3 (v3) - 1,000 Cases Enterprise Scale Edition

Amazon Bedrock Guardrails 자동화 검증/평가 플랫폼 **GuardBench**의 **버전 3 (v3)** 대용량(1,000 Cases) 엔터프라이즈 에디션 대시보드 웹 애플리케이션입니다.

---

## 🏢 1,000건 테스트 케이스 생성 및 부서별 구성

대규모 엔터프라이즈 운영 환경을 완벽하게 시뮬레이션하기 위해 **8대 부서별 TestSuite 하에 1,000개의 테스트 케이스**를 정교하게 동적 생성했습니다:

| TestSuite ID | 부서 / 담당 영역 | 케이스 수 | 주요 검증 내용 |
| :--- | :--- | :---: | :--- |
| **`SUITE-FIN-01`** | 자산관리부 (Financial Risk) | 200 건 | PII 마스킹 (주민등록번호/계좌번호/신용카드), 종목 추천 거부 |
| **`SUITE-SEC-02`** | 정보보안팀 (Information Security) | 200 건 | System Prompt 비밀키 유출, Base64 바이패스, Jailbreak |
| **`SUITE-HR-03`** | 인사총무팀 (Human Resources) | 150 건 | 임직원 연봉/인사평가 기밀 유출 방지 및 규정 안내 |
| **`SUITE-CS-04`** | 고객지원팀 (Customer Support) | 150 건 | 악성 비속어 차단, 경쟁사 비방 및 환불 문의 오탐 검증 |
| **`SUITE-MED-05`** | 의료헬스케어팀 (Medical Compliance) | 100 건 | 무단 처방전 작성 방지 및 환자 진료 차트 보안 |
| **`SUITE-DEV-06`** | 개발DevOps팀 (DevOps Security) | 100 건 | API Key / SSH Key / DB 비밀번호 주입 유출 검증 |
| **`SUITE-LEGAL-07`** | 법무준법팀 (Legal Compliance) | 50 건 | 무단 법률 자문 제공 및 계약서 기밀 조항 유출 |
| **`SUITE-MKT-08`** | 마케팅팀 (Marketing Content) | 50 건 | 허위 과장 광고 표현 및 저작권 침해 우회 방지 |

---

## 📊 1,000건 2축 평가 분포 통계 (2-Axis Distribution)

- **`NO_CHANGE` (동일)**: 850건 (85.0%)
- **`IMPROVEMENT` (기능 개선 ✨)**: 80건 (8.0%)
- **`USABILITY_REGRESSION` (사용성 저하 ⚠️)**: 50건 (5.0%)
- **`SECURITY_REGRESSION` (보안 결함 🚨)**: 20건 (2.0% - 배포 차단 원인)

---

## 🖥️ v3 엔터프라이즈 UI/UX 대용량 처리 기능

1. **고성능 페이지네이션 (Pagination System)**:
   * 1,000개 케이스를 20 / 50 / 100개 단위로 나누어 지연 없이 즉각 랜더링합니다.
2. **다중 입체 필터링 (Multi-Faceted Filter Bar)**:
   * 부서(TestSuite) + 심각도(Severity: CRITICAL, HIGH, MEDIUM, LOW) + 2축 평가 결과 + 실시간 프롬프트 검색어를 조합하여 1,000건 중 원하는 케이스를 0.1초 만에 추출합니다.
3. **데이터 엑스포트 (CSV & JSON Export)**:
   * 1,000건 전체 또는 필터링된 케이스 결과를 CSV/JSON 파일로 원클릭 다운로드합니다.

---

## 🔄 버전 1, 버전 2 대비 버전 3 주요 변경 사항 (v1 vs v2 vs v3)

| 비교 항목 | 버전 1 (`v1`) | 버전 2 (`v2`) | 버전 3 (`v3 1,000 Cases Scale`) |
| :--- | :--- | :--- | :--- |
| **테스트 케이스 수** | 5 건 (개념검증용) | 16 건 (부서별 4개 슈트) | **1,000 건 (엔터프라이즈 스케일)** |
| **조직 범위** | 단일 통합 | 4개 부서 | **8개 주요 전사 부서 확장** |
| **UI 대용량 처리** | 단일 스크롤 표 | 기본 탭 필터링 | **페이지네이션 (20/50/100) + 다중 입체 필터** |
| **데이터 Export** | 없음 | 없음 | **CSV & JSON 원클릭 Export 지원** |

---

## 🚀 로컬 실행 방법

```bash
cd /Users/gdone/dev/aws/guardbench-dashboard-v3
python3 -m http.server 8083
```
브라우저에서 `http://localhost:8083` 접속 (또는 `open index.html`)

---

> 💡 **GuardBench v3**: **1,000건 대용량 검증과 고성능 페이지네이션/필터링/엑스포트**가 완비된 엔터프라이즈 에디션입니다.
