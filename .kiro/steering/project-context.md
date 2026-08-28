---
inclusion: always
---

# GuardBench 프로젝트 컨텍스트 (하네스)

이 워크스페이스는 GuardBench — Amazon Bedrock Guardrails 정책 변경의 보안 회귀를 검증하는 AI Security Regression Test Platform이다. 여러 독립 Git 리포지토리로 구성된 멀티 루트 워크스페이스다.

## 리포지토리 지도

| 디렉토리 | 소유 | 역할 | 브랜치 | 커밋 정책 |
| --- | --- | --- | --- | --- |
| `guardbench-backend/` | GuardBench 기관 | Spring Boot API + 비동기 Worker | `dev` | 기관 레포 — 커밋 금지 |
| `guardbench-frontend/` | GuardBench 기관 | React 19 SPA | `main` | 기관 레포 — 커밋 금지 |
| `guardbench-iac/` | GuardBench 기관 | Terraform 인프라 | `dev` | 기관 레포 — 커밋 금지 |
| 워크스페이스 루트 (`.`) | 개인 (`gdone9009`) | GitHub Pages 문서/포탈 | `main` | 개인 레포 — 자유 커밋 |
| `guardbench-dashboard-v1~v5/`, `official-frontend/` | 개인 | 시연·목업 자산 | `main` | 개인 레포 |

## 커밋 경계 (중요)

- **기관 레포(`GuardBench/*`)에는 절대 커밋·push하지 않는다.** pull/조회/분석만 허용한다.
- **개인 레포(워크스페이스 루트)에는 자유롭게 커밋·push한다.**
- 어느 레포에 작업하는지 불분명하면 `git remote -v`로 확인한 뒤 진행한다.

## 판단 우선순위 (팀 규칙)

현재 지시 → `APPROVED` GitHub 계약 → 테스트·공개 코드 → `DRAFT` 문서 → Notion 순이다.
`guardbench-backend/docs/`가 구현 계약의 정본이며, 문서 상단 `Status`가 효력을 결정한다.

## 안전 규칙

- 기관 레포의 미커밋 변경(예: `guardbench-frontend`의 로컬 수정)은 출처가 불명확하면 되돌리지 않고 보존한다.
- GitHub와 Notion이 충돌하면 GitHub를 우선하고 차이를 보고한다.
- 공개 API, DB schema, 의존성, 아키텍처 변경은 사전 확인한다.

## 배포

워크스페이스 루트는 `main` push 시 GitHub Pages(`gdone9009.github.io/guardbench-dashboard/`)로 자동 배포된다. `.github/workflows/pages.yml`이 루트 전체를 정적 배포한다.
