# GuardBench 프로젝트 현황

> 작성일: 2026-08-28
> 기준: 각 리포지토리 최신 커밋 기준

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [리포지토리 구성](#2-리포지토리-구성)
3. [Backend 현황](#3-backend-현황)
4. [Frontend 현황](#4-frontend-현황)
5. [Infrastructure 현황](#5-infrastructure-현황)
6. [아키텍처 요약](#6-아키텍처-요약)
7. [개발 진행 타임라인](#7-개발-진행-타임라인)

---

## 1. 프로젝트 개요

**GuardBench**는 Amazon Bedrock Guardrails의 정책 변경 전후를 동일한 테스트 자산으로 실행하고, 기대 동작 위반과 보안 회귀를 자동 구분하는 **AI Security Regression Test Platform**이다.

### 핵심 가치

- 정책 변경 시 보안 회귀(Security Regression) 자동 탐지
- 과잉 차단(Usability Regression) 식별
- Quality Gate 기반 배포 판정 자동화

### 팀 구성

- KOSA AWS 3팀 (5인)
- 개발 기간: 2026-08-14 ~ 현재 (약 2주)

---

## 2. 리포지토리 구성

| 리포지토리 | 역할 | 브랜치 | 커밋 수 | 기간 |
|-----------|------|--------|---------|------|
| [GuardBench/guardbench-backend](https://github.com/GuardBench/guardbench-backend) | Spring Boot API + Worker | `dev` | 227 | 08-14 ~ 08-27 |
| [GuardBench/guardbench-frontend](https://github.com/GuardBench/guardbench-frontend) | React SPA | `main` | 20 | 08-25 ~ 08-27 |
| [GuardBench/guardbench-iac](https://github.com/GuardBench/guardbench-iac) | Terraform 인프라 | `dev` | 16 | 08-21 ~ 08-27 |

### 코드 규모

| 구분 | 파일 수 |
|------|---------|
| Backend 본체 (Java) | 269개 |
| Backend 테스트 (Java) | 99개 |
| Frontend (TS/TSX) | 21개 |
| IaC (Terraform .tf) | 13개 |

---

## 3. Backend 현황

### 기술 스택

| 항목 | 기술 |
|------|------|
| 언어 | Java 21 |
| 프레임워크 | Spring Boot 4.1.1 |
| 빌드 | Gradle (Wrapper) |
| DB | PostgreSQL 16.14 + Flyway |
| 메시징 | Amazon SQS (AWS SDK 2.54.3) |
| AI 서비스 | Amazon Bedrock Guardrails |
| 테스트 | JUnit 5, ArchUnit 1.5.0, Testcontainers |

### 패키지 구조 (DDD Bounded Context)

```
com.guardbench/
├── common/              ← 공통 설정, 에러, 응답 포맷
│   ├── config/
│   ├── error/
│   └── presentation/
├── testdefinition/      ← TestSuite & TestCase 관리
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── presentation/
├── testrun/             ← TestRun 수명주기, 비동기 실행
│   ├── domain/
│   ├── application/port/out/
│   ├── infrastructure/
│   │   ├── persistence/
│   │   └── integration/testdefinition/
│   └── presentation/
├── evaluation/          ← Assertion, Change, Quality Gate
│   ├── domain/
│   ├── application/
│   └── infrastructure/
└── guardrail/           ← AWS Bedrock Adapter
    └── infrastructure/
```

### 구현 완료 기능

- **TestSuite/TestCase CRUD API** — 생성, 조회, 수정, 논리 삭제
- **TestRun 비동기 실행 파이프라인**
  - 접수 → Outbox → SQS 발행 → Resolution → Execution → Finalization
  - 3개 SQS Queue: `gb-run-resolve`, `gb-workitems`, `gb-run-finalize`
- **Bedrock Guardrails Adapter** — DRAFT → numbered version materialization, ApplyGuardrail 호출
- **평가 Core** — Assertion(PASS/FAIL), Change Classification, Quality Gate 판정
- **조회 API** — TestRun 목록, 상세, 결과 목록 (Offset Pagination, Multi-sort, 필터링)
- **통합 테스트** — Testcontainers + LocalStack 기반 MVP 시나리오 검증
- **CI** — GitHub Actions backend-ci.yml (build + test merge gate)

### API 엔드포인트 (OpenAPI 3.0.3)

| Method | Path | 설명 |
|--------|------|------|
| POST | `/api/v1/test-suites` | TestSuite + 초기 TestCase 생성 |
| GET | `/api/v1/test-suites` | 목록 조회 (페이지네이션, 필터) |
| GET | `/api/v1/test-suites/{id}` | 상세 조회 |
| PATCH | `/api/v1/test-suites/{id}` | 수정 |
| GET | `/api/v1/test-suites/{id}/test-cases` | Suite 내 TestCase 목록 |
| POST | `/api/v1/test-suites/{id}/test-cases` | TestCase 추가 |
| GET | `/api/v1/test-cases/{id}` | TestCase 상세 |
| PATCH | `/api/v1/test-cases/{id}` | TestCase 수정 |
| DELETE | `/api/v1/test-cases/{id}` | TestCase 논리 삭제 |
| POST | `/api/v1/test-runs` | TestRun 비동기 접수 (202) |
| GET | `/api/v1/test-runs` | TestRun 목록 |
| GET | `/api/v1/test-runs/{id}` | 진행률 폴링 (경량) |
| GET | `/api/v1/test-runs/{id}/results` | 최종 결과 (FINISHED만) |

### DB 마이그레이션

- `V1__create_guardbench_schema.sql` — 핵심 도메인 스키마
- `V2__create_async_testrun_technical_tables.sql` — 비동기 실행 기술 테이블

### 수명주기 상태머신

```
TestRun: QUEUED → PREPARING → RUNNING → FINISHED
                                          ├── executionOutcome: COMPLETED | INCOMPLETE | ERROR
                                          └── qualityGate: PASS | FAIL | NOT_EVALUATED
```

---

## 4. Frontend 현황

### 기술 스택

| 항목 | 기술 |
|------|------|
| 프레임워크 | React 19 + TypeScript 6.0 |
| 빌드 | Vite 8.2 |
| 스타일링 | Tailwind CSS 4.3 |
| 아이콘 | Lucide React |
| 린트 | oxlint |
| E2E | Playwright |

### 프로젝트 구조

```
src/
├── assets/          ← 정적 리소스
├── components/
│   ├── common/      ← 재사용 컴포넌트
│   ├── layout/      ← 레이아웃 컴포넌트
│   └── views/       ← 뷰별 컴포넌트
├── hooks/           ← useLiveRunProgress 등 커스텀 훅
├── mocks/           ← 목 데이터
├── services/        ← API 서비스 레이어 (apiClient, testSuiteService 등)
└── types/           ← TypeScript 타입 정의
```

### 구현 완료 기능

- **Dashboard** — 전체 현황 뷰
- **Suites** — TestSuite 목록 + 생성 모달
- **Runs** — TestRun 목록 + 진행률 실시간 폴링 (2초 간격)
- **NewRun** — TestRun 생성 뷰
- **ResultDetail** — 결과 상세 뷰
- **Architecture** — 시스템 아키텍처 뷰
- **API 연동** — OpenAPI 기반 전체 서비스 연동 완료 (TestSuite, TestCase, TestRun)
- **CreateSuiteModal** — 신규 스위트 생성 모달 UI + POST API 연동

### 배포 상태

- GitHub Actions로 `main` push 시 S3 → CloudFront 자동 배포
- CloudFront Distribution ID: `E1PVL0Z78B1HMR`
- 현재 화면 데이터는 일부 mock, API 연동 레이어는 준비 완료

---

## 5. Infrastructure 현황

### 기술 스택

| 항목 | 기술 |
|------|------|
| IaC 도구 | Terraform >= 1.5.0 |
| Provider | AWS ~> 5.0 |
| State | S3 backend + S3 native locking |
| Region | ap-northeast-2 (서울) |

### Terraform 파일 구성

| 파일 | 리소스 |
|------|--------|
| `vpc.tf` | VPC 10.1.0.0/16, public/private subnet 각 2개 (2 AZ) |
| `security-groups.tf` | ALB, API, RDS, VPC Endpoint용 SG |
| `alb.tf` | Internet-facing ALB, HTTP listener, Target Group (8080) |
| `cloudfront-s3.tf` | CloudFront + S3 OAC (SPA), `/api/*` behavior → ALB |
| `ecr.tf` | ECR 리포지토리 (백엔드 이미지) |
| `ecs.tf` | ECS Cluster + Fargate Task (512 CPU / 1024 MiB) |
| `rds.tf` | PostgreSQL 16.14, db.t4g.micro, gp3 20GB, Single-AZ |
| `sqs.tf` | 3 source queues + 3 DLQ |
| `vpc-endpoints.tf` | SQS, Bedrock, ECR, Logs, Secrets Manager |
| `observability.tf` | CloudWatch alarms, log metric filter, SNS |
| `variables.tf` | 환경 변수 정의 |
| `provider.tf` | AWS provider, S3 backend 설정 |
| `outputs.tf` | 출력 값 정의 |

### 배포 아키텍처

```
Browser (HTTPS)
    │
    ▼
CloudFront Distribution
    ├── /* → S3 (정적 SPA, OAC)
    └── /api/* → HTTP :80
                    ▼
               Public ALB
                    │ HTTP :8080
                    ▼
               ECS Fargate Service (guardbench-dev-app, desired: 1)
               ├── Spring MVC HTTP API
               ├── Outbox Publisher
               ├── gb-run-resolve consumer
               ├── gb-workitems consumer
               ├── gb-run-finalize consumer
               └── Bedrock adapter
                    │
                    ├── PostgreSQL :5432 → Private RDS
                    └── HTTPS :443 → VPC Endpoints
                         ├── SQS
                         ├── Bedrock / Bedrock Runtime
                         ├── ECR API / ECR DKR / S3 Gateway
                         ├── CloudWatch Logs
                         └── Secrets Manager
```

### 배포 현황

| 리소스 | 상태 |
|--------|------|
| VPC, Subnet, VPC Endpoints | 배포 완료 |
| ALB, Security Groups | 배포 완료 |
| CloudFront + S3 (Frontend) | 배포 완료, 운영 중 |
| ECR | 배포 완료 |
| ECS Cluster + Service | Terraform 정의 완료, 최초 배포 대기 |
| RDS PostgreSQL | Terraform 정의 완료, 최초 배포 대기 |
| SQS Queues (3 + 3 DLQ) | Terraform 정의 완료, 최초 배포 대기 |

---

## 6. 아키텍처 요약

### 설계 원칙

- **DDD Hexagonal Architecture** — Domain은 Spring, JPA, AWS SDK에 의존하지 않음
- **Bounded Context 독립성** — testdefinition, testrun, evaluation은 독립 도메인 경계
- **Consumer-Owned Port** — Context 간 통신은 소비자 소유 Port + Integration Adapter
- **Outbox Pattern** — 안정적 SQS 메시지 발행
- **Test What You Deploy** — DRAFT guardrail을 numbered version으로 materialize 후 테스트

### 평가 계약 (Quality Gate)

| 지표 | 기준 |
|------|------|
| candidateAssertionPassRate | >= 0.95 |
| securityRegressionCount | == 0 |
| usabilityRegressionRate | <= 0.05 |
| testExecutionSuccessRate | >= 0.95 |

모든 조건 충족 시 PASS, 하나라도 미충족 시 FAIL.

### Change Classification

| Expected | Baseline | Candidate | Assertion | ChangeType |
|----------|----------|-----------|-----------|------------|
| ALLOW | ALLOW | BLOCK | FAIL | USABILITY_REGRESSION |
| BLOCK | BLOCK | ALLOW | FAIL | SECURITY_REGRESSION |
| ALLOW | BLOCK | ALLOW | PASS | IMPROVEMENT |
| BLOCK | ALLOW | BLOCK | PASS | IMPROVEMENT |

---

## 7. 개발 진행 타임라인

### Week 1 (08-14 ~ 08-20)

- 프로젝트 초기화 (Spring Boot, Gradle, .gitignore)
- 구현 계약 문서 정리 (도메인 모델, API, 평가 계약)
- ADR 작성 (도메인 경계, 영속성, 비동기 실행)
- ArchUnit 아키텍처 테스트 도입
- TestDefinition 도메인 및 Persistence 구현
- TestRun 도메인 구현
- Bedrock Adapter 설계 문서 확정

### Week 2 (08-21 ~ 08-27)

- Evaluation Core 구현 (Assertion, Change, Quality Gate)
- Bedrock SDK Adapter 구현
- TestSuite/TestCase API 전체 구현
- TestRun 접수 및 조회 API 구현
- **비동기 Worker 파이프라인 전체 구현**
  - Outbox Publisher, Resolution Worker, Execution Worker, Finalization
  - SQS 메시지 + DLQ + LocalStack 테스트
- MVP 통합 테스트 추가
- CI merge gate 적용
- **IaC**: VPC → ALB → CloudFront → ECR → ECS → RDS → SQS 전체 스택 구현
- **Frontend**: React SPA 초기화, API 연동, TestRun 폴링, Suite 생성 모달
- 인프라 배포 계약 문서 확정 (최초 배포 대기)

### 현재 상태 (08-28)

- **Backend**: MVP 기능 구현 완료, 통합 테스트 통과
- **Frontend**: API 연동 완료, 주요 뷰 구축 완료
- **IaC**: Terraform 정의 완료, 백엔드 최초 배포 대기 중
- **다음 단계**: ECS + RDS + SQS 실 배포 → 프론트엔드-백엔드 통합 검증

---

## 관련 문서

- [Backend 문서 지도](../guardbench-backend/docs/README.md)
- [핵심 도메인 모델](../guardbench-backend/docs/domain/core-model.md)
- [시스템 아키텍처 개요](../guardbench-backend/docs/architecture/system-overview.md)
- [인프라 배포 계약](../guardbench-backend/docs/architecture/infrastructure.md)
- [MVP 범위](../guardbench-backend/docs/product/mvp-scope.md)
- [평가 계약](../guardbench-backend/docs/domain/evaluation-contract.md)
- [OpenAPI 스펙](../guardbench-backend/docs/api/openapi.yaml)

---

## 업데이트 이력 (2026-08-28 재동기화)

기관 레포를 재동기화한 결과, **핵심 도메인 모델이 근본적으로 재정의**되었습니다. 이전 문서 내용 중 아래 항목이 변경되었으니 함께 참고하세요.

### 최신 커밋 (재동기화 후)

| 리포지토리 | 브랜치 | 최신 커밋 |
| --- | --- | --- |
| guardbench-backend | dev | `c53afe8` (PR #137, MVP 완료 문서 동기화) |
| guardbench-frontend | main | `695b04b` (PR #24, secret role ARN) |
| guardbench-iac | dev | `20fcc37` (PR #8, custom OIDC subject) |

### 가장 큰 변화: SUT(테스트 대상) 개념 전환 (ADR 0010 → 0011)

**이전 모델**: Bedrock Guardrail 자체를 테스트 대상으로 호출하고, Baseline(v7) vs Candidate(v8)를 동시 비교했다.

**현재 모델**:
- **Target = AI Application** (OpenAI 호환 HTTP 엔드포인트). 하나의 TestRun은 단일 Application Target만 실행한다.
- **Evaluator = 판정기** (provider 독립). Application의 자연어 응답을 `EvaluationResult(ALLOW|BLOCK)`로 정규화한다. **Bedrock Guardrail은 이제 "첫 번째 Evaluator 구현"** 으로 위치가 바뀌었다.
- 사용자는 provider/Guardrail id를 직접 넣지 않고, inline `evaluationProfile`(checks: PROMPT_INJECTION / PII_LEAKAGE / HARMFUL_CONTENT, strictness: RELAXED / STANDARD / STRICT)만 제출한다.
- **Baseline/Candidate 동시 비교는 폐기**되었고, 대신 이미 완료된 두 Run의 저장 결과를 비교하는 **Regression API**가 신설되었다.

새 실행 흐름:
```
TestCaseSnapshot → AI Application Target → 자연어 응답 → Evaluator → EvaluationResult(ALLOW/BLOCK) → ExpectedResult 비교 → Assertion → Quality Gate
```

### 신규 Regression API

- `GET /api/v1/test-runs/{id}/comparable-runs` — 비교 가능한 다른 Run 목록
- `GET /api/v1/test-runs/{currentId}/comparisons/{comparisonId}` — 두 Run 비교 (Application/Evaluator 재호출 없이 저장 결과만 비교)
- 비교 불가 시 `409 TEST_RUNS_NOT_COMPARABLE`

### Quality Gate 재정의 (현재 Run만 집계)

- `assertionPassRate >= 0.95` **그리고** `executionSuccessRate >= 0.95` → PASS
- 하나라도 미달 → FAIL / 평가 가능 Assertion 없음 → NOT_EVALUATED
- 이전의 securityRegressionCount, usabilityRegressionRate 지표는 Regression API 쪽으로 분리됨

### 패키지 구조 변화

- `guardrail` 패키지 **삭제** → **`target`**(OpenAI 호환 HTTP 어댑터) + **`evaluator`**(Bedrock 판정 구현)로 분리
- 현재: `testdefinition / testrun / evaluation / evaluator / target / common`

### 코드 규모 변화

| 구분 | 이전 | 현재 |
| --- | --- | --- |
| Backend main (Java) | 269 | 308 |
| Backend test (Java) | 99 | 105 |
| Flyway 마이그레이션 | V1~V2 | V1~V11 |

### Frontend / IaC

- **Frontend**: 여전히 mock 중심 SPA. 신규 문서 4종(screen-spec, user-flows, ui-guidelines, frontend-deployment) 추가. UI는 아직 구모델(Baseline/Candidate)이 남아 있어 백엔드 신모델과 불일치(문서에 12건 정리됨).
- **IaC**: GitHub OIDC 배포 역할 추가(`github-oidc.tf`). 프론트 CI가 정적 키 없이 배포하도록 현대화.

### 데모에 미치는 영향 (참고)

이 워크스페이스의 챗봇 데모/역할 설명 페이지는 "Guardrail이 런타임 필터"라는 관점으로 만들어졌는데, 백엔드 신모델에서는 **테스트 대상이 AI Application이고 Guardrail은 판정기(Evaluator)** 로 역할이 재정의되었습니다. 데모 문구를 신모델에 맞춰 갱신할지는 별도 결정이 필요합니다.
