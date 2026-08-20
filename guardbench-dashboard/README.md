# 🚀 GuardBench 3-Depth 시연 대시보드 목업 (Mockup)

Amazon Bedrock Guardrails 자동화 검증/평가 플랫폼 **GuardBench**의 3-Depth 시연 웹 대시보드입니다.

---

## 🌟 3-Depth 시연 구조

1. **Depth 1: Executive Dashboard (대시보드)**
   - 전체 Test Runs 통계 및 보안 결함(`SECURITY_REGRESSION`) 개요
   - Baseline Target(`v1.2`) vs Candidate Target(`v2.0-MAT`) 버전 릴리즈 현황
   - 2축 분리 평가 분류(Change Classification) 도넛 차트 및 최근 Runs 목록
2. **Depth 2: Test Run Detail & Quality Gate (테스트 런 상세)**
   - 특정 Test Run (`TR-20260820-001`)의 Quality Gate 최종 판정 상태
   - Execution Reliability 카터 (`Execution Error ≠ Assertion Failure`)
   - 100개 테스트 케이스 집계 및 4가지 분류 스탯 (`NO_CHANGE`, `IMPROVEMENT`, `USABILITY_REGRESSION`, `SECURITY_REGRESSION`)
   - 인터랙티브 필터링 테이블 (보안 결함만 보기, 개선건만 보기 등)
3. **Depth 3: Test Case Deep Dive & Inspector (개별 케이스 백드롭)**
   - 테스트 케이스 `TC-SEC-042` (System Prompt Override Attempt) 프롬프트 원문
   - Baseline vs Candidate Side-by-Side Bedrock Guardrail 응답 결과 비교
   - Bedrock Trace Log JSON 원문 데이터 확인 및 2축 비교 차이점(Diff) 감지

---

## 🚀 실행 방법

### 방법 1: Python HTTP 서버 (권장)
```bash
cd /Users/gdone/dev/aws/guardbench-dashboard
python3 -m http.server 8080
```
브라우저에서 `http://localhost:8080` 접속

### 방법 2: 브라우저 직접 열기
```bash
open /Users/gdone/dev/aws/guardbench-dashboard/index.html
```

---

> 💡 **KOSA AWS 3팀 - GuardBench Project**
