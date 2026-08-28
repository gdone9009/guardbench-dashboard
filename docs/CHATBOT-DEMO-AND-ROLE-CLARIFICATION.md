# 챗봇 데모 작업 정리 및 GuardBench 역할 정립

> 작성일: 2026-08-28
> 관련 배포: GitHub Pages (`gdone9009.github.io/guardbench-dashboard/`)

## 1. 이번 작업 요약

| 산출물 | 경로 | 배포 주소 |
| --- | --- | --- |
| Guardrail 챗봇 필터 데모 | `chatbot-demo/index.html` | `.../chatbot-demo/index.html` |
| GuardBench vs Guardrail 역할 설명 | `chatbot-demo/about-roles.html` | `.../chatbot-demo/about-roles.html` |
| v2 3단계 작동 목업 (기존) | `frontend-v2/index.html` | `.../frontend-v2/index.html` |
| 메인 포탈 챗봇 카드 연결 | `index.html` | `.../` |

모든 작업은 **개인 레포(`gdone9009/guardbench-dashboard`)** 에만 커밋했습니다. 기관 레포(`GuardBench/*`)는 조사만 하고 수정하지 않았습니다.

## 2. GuardBench 역할 정립 (질문에 대한 답)

계약 문서(`guardbench-backend/docs`)를 근거로 확인했습니다.

### 핵심: Guardrail과 GuardBench는 다른 것

| 구분 | Guardrail (Amazon Bedrock) | GuardBench (우리 프로젝트) |
| --- | --- | --- |
| 역할 | 런타임 필터 (실시간 ALLOW/BLOCK) | 정책 변경 검증 도구 |
| 작동 시점 | 평상시 매 요청마다 | 정책 버전업 시에만 |
| API | `ApplyGuardrail` (모델과 분리된 독립 평가) | Guardrail을 테스트로 호출 |
| 비유 | 문 앞 경비원 | 경비원 채용 시험 |

### 질문별 답변

**Q. 챗봇 입력을 GuardBench로 1차 필터링한다?**
→ 아니요. 실시간 필터링은 **Guardrail**의 역할입니다. GuardBench는 런타임 트래픽에 개입하지 않습니다. (역할이 뒤바뀐 오해)

**Q. 이건 시스템 프롬프트/컨텍스트인가?**
→ 아니요. 근거: `bedrock-guardrails-adapter.md` — `ApplyGuardrail`은 **foundation model 호출과 분리된 독립 평가 API**이며, 모델 ID나 모델 응답을 요청하지 않고 입력 텍스트만 받아 ALLOW/BLOCK을 판정합니다. 시스템 프롬프트(LLM 답변 방식 지시)와는 완전히 다른 계층입니다.

**Q. 가드레일 정책 변경을 미리 테스트하는 역할이 맞나?**
→ **맞습니다.** 근거: `mvp-scope.md`, ADR 0007. 현재 정책(Baseline)과 변경 예정 정책(Candidate DRAFT)을 같은 테스트 자산으로 비교해 보안 회귀를 찾습니다.

**Q. 평상시엔 작동 안 하고 버전업할 때만 쓰나?**
→ **맞습니다.** CI 게이트처럼 정책 배포 전에만 실행합니다. 평상시 실시간 처리는 Guardrail이 담당합니다.

### 전체 흐름

```
① 평상시:   사용자 → Guardrail(v7) 필터 → 챗봇 응답
② 정책 개선: v8 초안(DRAFT) 작성
③ 배포 전:   GuardBench가 v7 vs v8 테스트 → 보안 회귀 발견 → 배포 차단
④ 수정·재검증: 통과하면 → 런타임 Guardrail을 v8로 교체
```

## 3. 챗봇 데모가 보여주는 것

- 입력이 **Guardrail 정책(v7/v8)** 으로 1차 필터링되는 모습 (ALLOW/BLOCK)
- 우측 제어판에서 정책 버전을 v7↔v8로 전환 가능
- **v8은 의도적으로 보안 회귀를 심음**: "주민등록번호" 요청을 실수로 통과시킴
- 이를 통해 "왜 배포 전에 GuardBench 검증이 필요한가"를 체험

## 4. v2 "실제 작동 코드"에 대한 설명과 질문

### 현재 한 것

`frontend-v2/index.html`을 **클라이언트에서 실제로 작동하는 3단계 목업**으로 만들었습니다 (실행 이력 → 결과 상세 → 테스트별 비교). 챗봇 데모도 JS로 실제 판정이 동작합니다.

### 확인이 필요한 질문 (사용자 결정 필요)

GitHub Pages는 **정적 호스팅**이라 서버·DB·AWS 호출이 불가능합니다. "실제 작동하는 코드"를 어디까지 원하시는지에 따라 방향이 갈립니다:

1. **정적 데모 수준 (현재)**: 프론트에서 규칙·데이터를 시뮬레이션. GitHub Pages에서 바로 작동. → **이미 완료**
2. **실제 백엔드 연동**: `guardbench-frontend`(기관 레포)를 실제 `guardbench-backend` API에 연결.
   - 이건 **기관 레포 수정**이 필요합니다. 팀 워크플로(Issue → worktree → 리뷰 → 승인)를 따라야 하며, 저는 기관 레포에 커밋하지 않습니다.
   - 또한 백엔드가 실제 배포(ECS+RDS+SQS)되어 있어야 합니다. 현재는 IaC 정의만 있고 미배포 상태입니다.
3. **로컬 풀스택 실행**: 백엔드를 로컬에서 띄우고(`./gradlew bootRun`) 프론트를 붙여 실제 API로 작동. GitHub Pages 배포와는 별개.

**→ 어떤 수준을 원하시나요?** 2번(실제 API 연동)이라면 기관 레포 작업 승인과 백엔드 배포가 선행되어야 합니다.

## 5. 접속 주소 모음

- 챗봇 데모: https://gdone9009.github.io/guardbench-dashboard/chatbot-demo/index.html
- 역할 설명: https://gdone9009.github.io/guardbench-dashboard/chatbot-demo/about-roles.html
- v2 대시보드: https://gdone9009.github.io/guardbench-dashboard/frontend-v2/index.html
- 포탈: https://gdone9009.github.io/guardbench-dashboard/
