# 🌟 GuardBench Dashboard Version 5 (v5) - Ultimate Policy Configurator & Audit Report Edition

Amazon Bedrock Guardrails 자동화 검증/평가 플랫폼 **GuardBench**의 **버전 5 (v5)** 최종 고도화 엔터프라이즈 에디션 대시보드 웹 애플리케이션입니다.

---

## 📊 1. 이전 버전 (v4) 5대 평가 기준 자체 진단 리포트

GuardBench v4를 5가지 도메인 및 기술 평가 항목으로 측정한 자체 진단 결과입니다:

| 평가 항목 | 충실도 점수 | v4 상태 | 미흡점 및 v5 개선 조치 내역 |
| :--- | :---: | :--- | :--- |
| **① 도메인 충실도 & 2축 판정** | **90 / 100** | 2축 분류 우수 | Quality Gate 배포 기준을 실시간으로 조절하는 기능 미흡 → **v5 Policy Configurator 구축** |
| **② 대용량 시각화 & 트렌드** | **85 / 100** | OWASP/부서별 그리드 제공 | 슬라이더 조절 시 실시간 배포 상태 반향 → **v5 동적 Quality Gate 배너 구축** |
| **③ Trace Inspector & Diff** | **80 / 100** | 텍스트 결과 비교 지원 | Raw Bedrock JSON Trace의 코드 Diff 가시성 미흡 → **v5 JSON Trace Code Diff Inspector 구축** |
| **④ 배포 정책 설정** | **60 / 100** | 배포 기준이 고정됨 | 사용자가 허용 결함 개수를 직접 지정하는 기능 부재 → **v5 실시간 임계값 슬라이더 구축** |
| **⑤ 리포팅 및 감사** | **75 / 100** | CSV/JSON Export 지원 | C-Level/감사팀 제출용 서식 부족 → **v5 Executive Audit Report 인쇄/PDF 모달 구축** |

---

## ⚙️ 2. v5 핵심 신규 개발 기능

### 1. ⚙️ Quality Gate Policy Configurator (배포 정책 실시간 조절기)
- **기능**: `최대 허용 보안 결함 수` (0~50건 슬라이더), `최대 허용 사용성 저하 수` (0~100건 슬라이더), `최소 요구 통과율` (50~100% 슬라이더) 제공.
- **실시간 반응**: 슬라이더 변경 시 1,000개 케이스 결과가 즉시 재산출되어 **Quality Gate Status (`PASSED 🟢` / `FAILED 🔴` / `WARNING ⚠️`)가 동적으로 업데이트**됩니다.

### 2. 🔍 Side-by-Side Raw JSON Trace Diff Inspector
- Baseline v1.2와 Candidate v2.0-MAT의 Bedrock `ApplyGuardrail` Raw JSON Payload에 대해 줄 단위 하이라이트 Diff를 제공합니다.

### 3. 📄 Executive Quality Audit Report Generator (인쇄 / PDF 저장 지원)
- 경영진, 보안 감사팀, C-Level 보고용 표준 **GuardBench Executive Quality Audit Report** 모달 작성을 지원하며 `window.print()`로 즉시 PDF 출력이 가능합니다.

### 4. ➕ 실시간 사용자 커스텀 Test Case 추가 기능
- 신규 프롬프트 문구와 부서, OWASP 카테고리를 UI에서 직접 입력하여 1,000건 데이터셋에 실시간 추가 및 재검증이 가능합니다.

---

## 🔄 버전 1 ~ 버전 5 전체 버전 발전사 (v1 vs v2 vs v3 vs v4 vs v5)

| 비교 항목 | v1 | v2 | v3 | v4 | **v5 (Ultimate Edition)** |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **케이스 수** | 5건 | 16건 | 1,000건 단순생성 | 1,000건 OWASP 고유 | **1,000건 + 사용자 커스텀 추가** |
| **배포 정책** | 고정 | 고정 | 고정 | 고정 | ⚙️ **실시간 Policy Configurator 슬라이더** |
| **감사 리포트** | 없음 | 없음 | CSV/JSON | CSV/JSON | 📄 **Executive Audit Report 인쇄/PDF 모달** |
| **JSON Diff** | 텍스트 | 텍스트 | 텍스트 | 텍스트 | 🔍 **Side-by-Side Code Diff Inspector** |

---

## 🚀 로컬 실행 방법

```bash
cd /Users/gdone/dev/aws/guardbench-dashboard-v5
python3 -m http.server 8085
```
브라우저에서 `http://localhost:8085` 접속 (또는 `open index.html`)

---

> 💡 **GuardBench v5**: **Quality Gate 배포 정책 실시간 조절기, JSON Trace Diff, Executive Audit Report**가 완성된 최종 엔터프라이즈 에디션입니다.
