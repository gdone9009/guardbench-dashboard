---
inclusion: fileMatch
fileMatchPattern: 'guardbench-backend/**'
---

# GuardBench Backend 컨벤션 (조건부 하네스)

`guardbench-backend/` 파일을 다룰 때 적용한다. 이 레포는 GuardBench 기관 소유이므로 **커밋·push를 하지 않는다.** 조사·분석·리뷰만 수행한다.

## DDD 패키지 구조 (package-by-domain)

최상위는 기술 계층이 아니라 도메인으로 나눈다.

```
com.guardbench/
├─ testdefinition/   (TestSuite, TestCase)
├─ testrun/          (TestRun 수명주기, 비동기 실행, Snapshot)
├─ evaluation/       (Assertion, Change, Quality Gate)
├─ guardrail/        (Bedrock Adapter)
└─ common/           (실제 횡단 관심사만)
```

각 도메인 내부는 `domain`, `application`, `presentation`, `infrastructure`로 나눈다.
전역 `controller/service/repository/entity/dto` 구조는 사용하지 않는다.

## Bounded Context 경계 (ADR 0006)

- 다른 Context의 Domain Java 타입·ID VO·Enum·Repository를 직접 import하지 않는다.
- 소비 Context가 `application/port` 아래 Port와 값 계약을 소유하고, `infrastructure/integration` Adapter가 명시적으로 mapping한다.
- Domain에 JPA·Spring·AWS·HTTP 타입을 노출하지 않는다.
- 같은 Context 내 Aggregate 간에는 객체 참조 대신 전용 ID VO를 사용한다.

## 핵심 상태 분리

`HTTP 오류 ≠ Execution ERROR ≠ Assertion FAIL ≠ NOT_COMPARABLE ≠ Quality Gate FAIL`

이들은 독립적인 상태다. TestRun 수명주기는 `QUEUED → PREPARING → RUNNING → FINISHED`이며 `FINISHED`만 터미널이다.

## 코드 변경 전 확인

작업 유형별로 `guardbench-backend/docs/`의 관련 계약을 먼저 읽는다:
- 도메인: `domain/core-model.md`, `domain/evaluation-contract.md`, `decisions/0006-*.md`
- API: `api/README.md`, `api/openapi.yaml`
- 영속성/인프라: 관련 ADR, `architecture/infrastructure.md`
