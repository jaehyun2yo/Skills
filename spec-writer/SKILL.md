---
name: spec-writer
description: Generates comprehensive project specification documents (PRD, Project Constitution, SRS, SDS, functional spec, screen design, API spec, DB spec, test spec, implementation plan) step by step with full traceability between documents. Use this skill whenever a user wants to plan a new project, write planning docs, create specifications, document an existing codebase, or mentions any of these keywords - even if they only ask for one document type. Triggers on "기획", "기획문서", "명세서", "PRD", "SRS", "SDS", "spec", "specification", "문서작성", "프로젝트 기획", "project planning", "요구사항", "설계문서", "API 명세", "DB 명세", "기능명세", "화면설계", "테스트 명세", "구현 계획", "constitution", "wireframe".
---

# Spec Writer — 프로젝트 기획문서 생성기

프로젝트 기획에 필요한 핵심 문서를 단계별로 생성한다.
각 문서는 이전 문서에서 파생되며, 요구사항 ID를 통해 문서 간 추적성(traceability)을 유지한다.

## 문서 파이프라인

```
PRD → 프로젝트 원칙 → SRS → SDS
                              ↓
              기능명세서 → 화면설계서 → API 명세서 → DB 명세서
                                                        ↓
                                          테스트 명세서 → 구현 계획서
                                                        ↓
                                                  추적성 매트릭스
```

각 단계에서 유저 확인을 받은 후 다음 문서로 진행한다.
프로젝트 성격에 따라 불필요한 문서는 건너뛸 수 있다 (예: API가 없는 CLI 도구는 API 명세서 불필요).

---

## Phase 0: 모드 감지 및 프로젝트 파악

두 가지 모드를 지원한다. 유저의 상황에 맞게 자동 판단한다.

### 모드 A: 신규 프로젝트 (인터뷰)

유저가 아이디어만 있고 코드가 없는 경우. 구조화된 인터뷰로 요구사항을 도출한다.

**핵심 질문** (한 번에 다 묻지 않는다 — 대화하며 자연스럽게 파악):
1. 어떤 문제를 해결하려는 프로젝트인가?
2. 주요 타겟 사용자는 누구인가?
3. 핵심 기능 3-5개는 무엇인가?
4. 기술 스택 선호가 있는가? (없으면 추천)
5. 외부 연동이 필요한 서비스가 있는가?
6. 예상 규모 (사용자 수, 데이터량)?

인터뷰 중에 파악한 내용을 중간중간 요약해서 유저에게 확인받는다.
유저가 모호하게 답하면 구체적인 예시를 들어 선택지를 제시한다.

### 모드 B: 기존 코드베이스 분석

이미 코드가 존재하는 프로젝트의 문서를 역으로 생성하는 경우.

```bash
# 프로젝트 구조 파악
ls -la
find . -name "*.md" -not -path "*/node_modules/*" | head -20

# 기술 스택 감지
ls package.json Cargo.toml pyproject.toml go.mod build.gradle 2>/dev/null

# 코드 구조 분석
find src -type f -name "*.ts" -o -name "*.py" -o -name "*.go" 2>/dev/null | head -30

# API 엔드포인트 탐색
grep -r "router\.\|@app\.\|@Get\|@Post\|@Controller" src/ --include="*.ts" --include="*.py" -l 2>/dev/null

# DB 스키마 탐색
find . -name "*.prisma" -o -name "*.sql" -o -name "migration*" 2>/dev/null | head -10
```

분석 결과를 유저에게 요약 보고한 후 PRD 작성을 시작한다.

---

## Phase 1: 문서 생성 파이프라인

모든 문서는 `docs/specs/` 폴더에 저장한다. 폴더가 없으면 생성한다.

```bash
mkdir -p docs/specs
```

### 추적성 ID 체계

문서 간 연결을 위해 일관된 ID 체계를 사용한다.
이 체계가 중요한 이유는, 나중에 요구사항이 변경되었을 때 영향받는 설계/기능/API/DB를 빠르게 추적할 수 있기 때문이다.

| 문서 | ID 접두사 | 형식 | 예시 |
|------|----------|------|------|
| PRD | BIZ | BIZ-001 | BIZ-001: 사용자 인증 |
| 프로젝트 원칙 | CONST | CONST-001 | CONST-001: REST API 컨벤션 |
| SRS | REQ-F / REQ-NF | REQ-F-001 | REQ-F-001: 이메일 로그인 |
| SDS | DES | DES-001 | DES-001: Auth 모듈 설계 |
| 기능명세서 | FUNC | FUNC-001 | FUNC-001: 로그인 플로우 |
| 화면설계서 | UI | UI-001 | UI-001: 로그인 화면 |
| API 명세서 | API | API-001 | API-001: POST /auth/login |
| DB 명세서 | DB | DB-001 | DB-001: users 테이블 |
| 테스트 명세서 | TC | TC-001 | TC-001: 로그인 성공 테스트 |
| 구현 계획서 | TASK | TASK-001 | TASK-001: Auth 모듈 구현 |

각 ID는 상위 문서의 ID를 참조한다:
- SRS의 REQ-F-001 → PRD의 BIZ-001에서 파생
- SDS의 DES-001 → SRS의 REQ-F-001을 구현
- 테스트의 TC-001 → REQ-F-001을 검증
- 구현 계획의 TASK-001 → DES-001, FUNC-001을 구현 단위로 분해
- 이런 식으로 모든 문서가 연결된다

---

### Step 1: PRD (Product Requirements Document)

프로젝트의 **왜**와 **무엇**을 정의하는 최상위 문서.
비즈니스 관점에서 프로젝트의 목적, 대상, 범위를 명확히 한다.

**템플릿**: `references/prd-template.md`를 읽고 프로젝트에 맞게 작성한다.

**작성 후 유저에게 확인**:
- "PRD를 작성했습니다. 검토 후 수정할 부분이 있으면 알려주세요."
- 유저가 승인하면 다음 단계로 진행
- 수정 요청이 있으면 반영 후 재확인

**저장**: `docs/specs/PRD.md`

### Step 2: 프로젝트 원칙 (Project Constitution)

프로젝트 전반에 적용되는 **불변 규칙과 제약 조건**을 정의한다.
코딩 표준, 보안 정책, 아키텍처 원칙, 기술적 제약 등 모든 후속 문서와 구현에 영향을 미치는 가드레일 역할을 한다.

이 문서가 PRD 직후에 오는 이유는, SRS부터 구현 계획까지 모든 문서가 이 원칙을 준수해야 하기 때문이다.
AI 기반 개발에서 특히 중요 — AI가 코드를 생성할 때 이 문서를 참조하여 일관된 품질을 유지한다.

**템플릿**: `references/constitution-template.md`를 읽고 작성한다.

**저장**: `docs/specs/CONSTITUTION.md`

### Step 3: SRS (Software Requirements Specification)

PRD에서 정의한 비즈니스 요구사항을 **구체적이고 측정 가능한 소프트웨어 요구사항**으로 변환한다.
개발자가 이 문서만 보고 "무엇을 구현해야 하는지" 명확히 알 수 있어야 한다.
프로젝트 원칙(CONSTITUTION)의 제약 조건을 모든 요구사항에 반영한다.

**템플릿**: `references/srs-template.md`를 읽고 작성한다.

**핵심 원칙**:
- 모든 요구사항은 검증 가능해야 한다 (모호한 표현 금지)
- 기능 요구사항(REQ-F)과 비기능 요구사항(REQ-NF)을 분리한다
- 각 요구사항에 우선순위를 부여한다 (P0: 필수, P1: 중요, P2: 선택)

**저장**: `docs/specs/SRS.md`

### Step 4: SDS (Software Design Specification)

SRS의 요구사항을 **어떻게 구현할 것인지** 설계하는 문서.
시스템 아키텍처, 컴포넌트 구조, 기술적 의사결정을 담는다.
프로젝트 원칙에 정의된 아키텍처 원칙과 기술적 제약을 준수한다.

**템플릿**: `references/sds-template.md`를 읽고 작성한다.

**작성 시 고려사항**:
- 아키텍처 다이어그램을 Mermaid로 포함한다
- 기술 스택 선정의 근거를 명시한다
- 컴포넌트 간 의존성과 데이터 흐름을 설명한다

**저장**: `docs/specs/SDS.md`

### Step 5: 기능명세서 (Functional Specification)

각 기능의 **상세 동작**을 정의한다.
SRS가 "무엇"이라면, 기능명세서는 "구체적으로 어떻게 동작하는가"를 설명한다.
UI 플로우, 입출력, 예외 처리, 비즈니스 규칙을 포함한다.

**템플릿**: `references/func-spec-template.md`를 읽고 작성한다.

**저장**: `docs/specs/FUNC-SPEC.md`

### Step 6: 화면설계서 (Screen Design / Wireframe)

기능명세서의 사용자 플로우를 **시각적 UI 구조**로 변환한다.
각 화면의 레이아웃, 컴포넌트 배치, 상태별 화면(로딩/빈 상태/에러), 화면 간 네비게이션을 정의한다.

UI가 있는 프로젝트(웹, 모바일, 데스크톱)에서 기능명세와 실제 프론트엔드 구현 사이의 갭을 줄여주는 핵심 문서.

**템플릿**: `references/screen-design-template.md`를 읽고 작성한다.

**건너뛰기 조건**: UI가 없는 프로젝트(API 서비스, 라이브러리, 데이터 파이프라인)는 건너뛴다.

**저장**: `docs/specs/SCREEN-DESIGN.md`

### Step 7: API 명세서 (API Specification)

시스템의 외부 인터페이스를 정의한다.
REST, GraphQL, gRPC 등 프로젝트에 맞는 형식으로 작성한다.

**템플릿**: `references/api-spec-template.md`를 읽고 작성한다.

**건너뛰기 조건**: API가 없는 프로젝트 (CLI 도구, 데스크톱 앱 등)는 이 단계를 건너뛴다.
유저에게 "이 프로젝트에 API 명세서가 필요한가요?" 확인한다.

**저장**: `docs/specs/API-SPEC.md`

### Step 8: DB 명세서 (Database Specification)

데이터 모델, 스키마, 관계, 인덱스 전략을 정의한다.

**템플릿**: `references/db-spec-template.md`를 읽고 작성한다.

**건너뛰기 조건**: DB를 사용하지 않는 프로젝트는 건너뛴다.

**저장**: `docs/specs/DB-SPEC.md`

### Step 9: 테스트 명세서 (Test Specification)

SRS의 요구사항과 기능명세서의 동작 정의를 **검증 가능한 테스트 케이스**로 변환한다.
어떤 요구사항이 어떤 테스트로 검증되는지 매핑하여, 테스트 커버리지의 누락을 방지한다.

테스트 명세서가 중요한 이유는, 명세서가 아무리 잘 작성되어도 검증 계획이 없으면 "구현했다 = 완성됐다"라는 착각에 빠지기 때문이다.

**템플릿**: `references/test-spec-template.md`를 읽고 작성한다.

**저장**: `docs/specs/TEST-SPEC.md`

### Step 10: 구현 계획서 (Implementation Plan)

모든 설계와 명세를 **실제 개발 태스크**로 분해한다.
개발 순서, 의존성, 예상 소요 시간, 마일스톤을 정의하여 설계 문서와 코딩 사이의 갭을 메운다.

이 문서가 마지막에 오는 이유는, 모든 명세가 확정되어야 정확한 태스크 분해가 가능하기 때문이다.

**템플릿**: `references/implementation-plan-template.md`를 읽고 작성한다.

**저장**: `docs/specs/IMPLEMENTATION-PLAN.md`

---

## Phase 2: 추적성 매트릭스 생성

모든 문서 작성이 완료되면, 문서 간 연결 관계를 한눈에 볼 수 있는 추적성 매트릭스를 생성한다.
이 매트릭스는 나중에 요구사항 변경 시 영향 분석에 핵심적인 역할을 한다.

**형식**:

```markdown
# 추적성 매트릭스

| PRD (BIZ) | SRS (REQ) | SDS (DES) | 기능명세 (FUNC) | 화면설계 (UI) | API (API) | DB (DB) | 테스트 (TC) | 구현 (TASK) |
|-----------|-----------|-----------|----------------|-------------|-----------|---------|------------|------------|
| BIZ-001   | REQ-F-001, REQ-F-002 | DES-001 | FUNC-001 | UI-001 | API-001 | DB-001 | TC-001, TC-002 | TASK-001 |
| BIZ-002   | REQ-F-003 | DES-002 | FUNC-002 | UI-002, UI-003 | API-002 | DB-002 | TC-003 | TASK-002, TASK-003 |
```

**저장**: `docs/specs/TRACEABILITY.md`

---

## Phase 3: 최종 보고

모든 문서 생성 완료 후 유저에게 요약을 제공한다:

1. 생성된 문서 목록과 경로
2. 각 문서의 핵심 요약 (1-2줄)
3. 문서 간 커버리지 — 모든 BIZ 요구사항이 하위 문서에서 추적되는지 확인
4. 테스트 커버리지 — 모든 REQ가 TC에 매핑되었는지 확인
5. 구현 커버리지 — 모든 기능이 TASK에 배정되었는지 확인
6. 누락된 영역이 있으면 알림

---

## 유의사항

### 문서 품질 기준
- **구체성**: "빠르게 응답해야 한다" ✗ → "API 응답 시간 95%ile 200ms 이내" ✓
- **추적성**: 모든 하위 문서 항목은 상위 문서 ID를 참조해야 한다
- **일관성**: 용어, ID 형식, 문서 구조를 전체 문서에서 통일한다
- **원칙 준수**: 프로젝트 원칙(CONSTITUTION)에 위배되는 내용이 없는지 매 문서에서 확인

### 프로젝트 유형별 문서 추천

| 프로젝트 유형 | PRD | 원칙 | SRS | SDS | 기능명세 | 화면설계 | API 명세 | DB 명세 | 테스트 | 구현계획 |
|-------------|-----|------|-----|-----|---------|---------|---------|---------|--------|---------|
| 웹 앱 (풀스택) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| API 서비스 | ✓ | ✓ | ✓ | ✓ | △ | ✗ | ✓ | ✓ | ✓ | ✓ |
| CLI 도구 | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | △ | ✓ | ✓ |
| 모바일 앱 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 라이브러리/SDK | ✓ | ✓ | ✓ | ✓ | △ | ✗ | ✓ | ✗ | ✓ | ✓ |
| 데이터 파이프라인 | ✓ | ✓ | ✓ | ✓ | △ | ✗ | △ | ✓ | ✓ | ✓ |

✓ = 필수, △ = 선택 (프로젝트에 따라), ✗ = 불필요

PRD 작성 직후에 이 표를 참고하여 유저에게 어떤 문서가 필요한지 확인한다.

### 기존 문서가 있는 경우
`docs/specs/` 에 이미 문서가 존재하면:
- 기존 문서를 먼저 읽고 내용을 파악한다
- 유저에게 "기존 문서를 업데이트할까요, 새로 작성할까요?" 확인한다
- 업데이트 시 기존 ID 체계를 유지하고, 변경 이력을 문서 상단에 기록한다
