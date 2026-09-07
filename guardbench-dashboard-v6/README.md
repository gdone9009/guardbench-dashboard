# 🛡️ GuardBench Dashboard Version 6 (v6) - Final Release Enterprise Demonstration

> **GuardBench (Amazon Bedrock Guardrail 회귀 검증 & Quality Gate 오케스트레이터)**  
> **버전**: `v6.0 Official Final Release Enterprise Demonstration Page`  
> **라이브 접속 주소**: [https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v6/](https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v6/)

---

## 📌 1. v6.0 최종 시연 페이지 개요

기관 레포지토리(`GuardBench/guardbench-frontend`, `guardbench-backend`, `guardbench-iac`)의 최종 서비스 규격을 기반으로 새롭게 구축한 **3-Depth 완벽 대화형 가상 시연 페이지**입니다.

### 🌟 주요 추가 반영 기능 (v6.0 Final Features)
1. **🚥 Quality Gate Evidence Banner & Interactive Chart.js**:
   - 통과율, 보안 결함 수, 사용성 저하 수, 실행 신뢰도를 임계값 라인과 비교하는 대화형 차트 Canvas.
2. **🏃 Run Progress Stepper (`RunProgressStepper`)**:
   - `QUEUED` (접수 멱등성 보장) $\rightarrow$ `PREPARING` (Target materialization) $\rightarrow$ `RUNNING` (SQS 팬아웃) $\rightarrow$ `FINISHED` (최종 판정).
3. **📁 CSV / JSON Bulk Test Case Import Panel (`BulkTestCaseCreatePanel`)**:
   - 드래그 앤 드롭 파일 업로드를 지원하는 대량 스냅샷 등록 패널.
4. **⚙️ 실시간 Quality Gate Policy Configurator**:
   - 슬라이더 조절 시 1,000건 데이터가 즉시 재계산되고 차트와 Quality Gate 승인 상태가 동적으로 변환.
5. **🔍 Side-by-Side Raw JSON Trace Code Diff Inspector**:
   - Bedrock `ApplyGuardrail` 요청/응답 Raw Payload 원문 JSON 줄 단위 코드 diff.
6. **📄 Executive Quality Audit Report Generator**:
   - C-Level 감사 보고서 작성 및 `window.print()` PDF 원클릭 출력 모달.
7. **➕ 커스텀 테스트 케이스 실시간 추가**:
   - UI에서 직접 프롬프트를 입력하여 1,000건 세트에 추가 및 즉시 재평가.

---

## 🌐 2. 접속 및 실행 방법

- **라이브 웹 주소**: [https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v6/](https://gdone9009.github.io/guardbench-dashboard/guardbench-dashboard-v6/)
- **로컬 실행 명령어**:
  ```bash
  cd /Users/gdone/dev/aws/guardbench-dashboard-v6
  python3 -m http.server 8086
  ```
  브라우저에서 `http://localhost:8086` 접속
