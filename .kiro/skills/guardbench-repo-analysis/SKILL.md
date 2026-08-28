---
name: guardbench-repo-analysis
description: GuardBench 기관 레포(backend/frontend/iac)의 구조·도메인·진행 상황을 조사하거나 프로젝트 현황 문서를 작성할 때 사용한다. APPROVED 계약 문서를 우선 근거로 삼고, 코드와 커밋 이력을 교차 확인하여 검증된 사실만 보고한다.
---

# GuardBench 레포 분석 스킬

기관 레포를 조사·분석해 검증된 현황을 산출하는 루프다. **조사·분석만 하며 커밋하지 않는다.**

## 근거 우선순위 (팀 규칙)

`APPROVED` GitHub 계약 → 테스트·공개 코드 → `DRAFT` 문서 → Notion 순. `guardbench-backend/docs/`가 정본이며 문서 상단 `Status`가 효력을 결정한다.

## 문서 진입점

`guardbench-backend/docs/README.md`가 문서 지도다. 작업별로 다음을 읽는다:
- 제품/범위: `product/mvp-scope.md`
- 도메인: `domain/core-model.md`, `domain/evaluation-contract.md`
- 아키텍처: `architecture/system-overview.md`, `architecture/infrastructure.md`
- API: `api/openapi.yaml`
- 결정: `decisions/*.md` (ADR)

## 조사 절차

1. 각 레포에서 `git remote -v`, `git branch --show-current`, `git log --oneline`으로 소유·브랜치·최근 작업을 확인한다.
2. 도메인/아키텍처는 코드보다 APPROVED 문서를 먼저 읽는다.
3. 문서 주장과 실제 코드·커밋을 교차 확인한다. 불일치는 그대로 보고한다.
4. 코드 규모는 실제 파일 카운트로 측정한다 (`find ... | wc -l`).
5. 배포 상태는 IaC 정의 존재와 실제 배포 여부를 구분한다.

## 보고 원칙

- 검증한 사실과 확인 못 한 항목을 명확히 구분한다.
- "구현 완료"는 코드·테스트 근거가 있을 때만 표기한다.
- Terraform "정의됨"과 "실제 배포됨"을 혼동하지 않는다.
- 큰 조사는 context-gatherer 서브에이전트에 위임해 메인 컨텍스트를 보존한다.

## 산출물

현황 문서는 워크스페이스 루트 `docs/`에 markdown으로 저장하고, 필요하면 `guardbench-pages-publish` 스킬로 HTML 페이지화하여 배포한다.
