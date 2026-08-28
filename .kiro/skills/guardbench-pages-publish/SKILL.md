---
name: guardbench-pages-publish
description: 워크스페이스 루트(개인 레포)에 문서·포탈 페이지를 추가/수정하고 GitHub Pages에 배포할 때 사용한다. 팀 커밋 컨벤션을 따라 커밋하고, push 전 검증 보고를 거친 뒤 배포 주소를 안내한다. 개인 레포에만 적용하며 기관 레포에는 사용하지 않는다.
---

# GuardBench Pages 배포 스킬

워크스페이스 루트 개인 레포(`gdone9009/guardbench-dashboard`)의 정적 페이지를 만들고 GitHub Pages에 배포하는 루프다.

## 사전 확인

- `git remote -v`로 개인 레포(`gdone9009/*`)인지 확인한다. 기관 레포(`GuardBench/*`)면 이 스킬을 쓰지 않는다.
- 루트는 `main` push 시 `.github/workflows/pages.yml`이 전체를 GitHub Pages로 배포한다.

## 페이지 작성 규칙

- 스타일은 기존 `index.html`, `project-status.html`, `easy-guide.html`과 일관되게 맞춘다 (Tailwind CDN + Pretendard 폰트).
- 새 페이지는 메인 `index.html`에서 도달 가능하도록 링크를 연결한다.
- 상대 경로를 사용한다 (GitHub Pages 하위 경로 배포 대응).

## 커밋 컨벤션 (팀 규칙)

형식: `<type>(scope): <설명>`. 문서·페이지는 주로 `docs`, 구조 개편은 `refactor`.

- 한 커밋에는 독립적으로 검토 가능한 한 목적만 담는다.
- 특정 파일만 스테이징한다 (`git add <파일>`). `git add .` / `-A`는 피한다.
- 무관한 변경(`.DS_Store`, 기관 레포 서브모듈 수정 등)은 스테이징하지 않는다.

## push 전 검증 보고 (팀 규칙)

push 전 다음을 확인·보고한다:
- `git status` — 의도한 파일만 staged 됐는지
- `git diff --cached --stat` — 변경 규모
- 무관한 파일이 포함되지 않았는지

## 배포 후

- push 성공 후 배포 주소를 안내한다: `https://gdone9009.github.io/guardbench-dashboard/<파일명>`
- Pages 워크플로는 1~2분 내 반영된다.

## 안전

- 파괴적 Git 작업(force push, reset --hard)은 명시적 승인 없이 하지 않는다.
- 기관 레포 서브모듈 변경은 이 스킬 범위 밖이므로 건드리지 않는다.
