---
name: team-setup
description: Analyzes project structure to identify domains, generates tailored Agent Team skills with hooks and rules per domain. Triggers on "팀 셋업", "team setup", "팀 생성 셋업", "팀 시스템 설정", "에이전트 팀 설정".
---

# Team Setup Skill

프로젝트 분석 → 도메인 식별 → 팀 스킬 자동 생성 → CLAUDE.md 등록 → hooks 설정.

새 프로젝트에서 에이전트 팀 시스템을 한번에 구축합니다.

---

## Phase 1: Project Analysis

### 1.1 Tech Stack Detection

```bash
# Package manifest
ls package.json Cargo.toml pyproject.toml go.mod build.gradle pom.xml 2>/dev/null
head -80 package.json 2>/dev/null

# Framework detection
grep -l "next\|react\|vue\|svelte\|angular" package.json 2>/dev/null
grep -l "nestjs\|express\|fastify\|django\|flask\|gin\|actix" package.json pyproject.toml go.mod 2>/dev/null

# Database
grep -l "prisma\|typeorm\|sequelize\|drizzle\|supabase\|mongoose" package.json 2>/dev/null
ls prisma/schema.prisma 2>/dev/null
```

Record: language, frontend framework, backend framework, ORM, database, test framework.

### 1.2 Directory Structure Analysis

```bash
# Top-level structure
ls -d */ 2>/dev/null

# Source directories (depth 2)
find src -maxdepth 2 -type d 2>/dev/null || find . -maxdepth 2 -type d -not -path '*/node_modules/*' -not -path '*/.git/*' 2>/dev/null

# Route/page structure (for web apps)
find . -path "*/app/*" -name "page.*" -o -path "*/pages/*" -name "*.tsx" -o -path "*/routes/*" -name "*.ts" 2>/dev/null | head -30
```

### 1.3 Existing Setup Check

```bash
# Already has team skills?
find .claude/skills -name "team-*" -type d 2>/dev/null
# Has CLAUDE.md?
wc -l CLAUDE.md 2>/dev/null
# Has team rules?
cat .claude/rules/team-rules.md 2>/dev/null
# Has agent teams enabled?
grep "AGENT_TEAMS" .claude/settings.json 2>/dev/null
```

---

## Phase 2: Domain Identification

프로젝트 구조에서 자연스러운 도메인 경계를 식별합니다.

### 2.1 Web Application Patterns

| Pattern | Domain Signal |
|---------|--------------|
| `src/app/(admin)/` or `src/pages/admin/` | Admin domain |
| `src/app/api/` or `src/routes/api/` | API/Backend domain |
| `src/app/(public)/` or `src/pages/` (root) | Public domain |
| `src/components/` (shared) | Shared/Common domain |
| Separate backend dir (`api/`, `server/`, `backend/`) | Backend domain |
| `tests/`, `__tests__/` | Test domain |

### 2.2 Backend Application Patterns

| Pattern | Domain Signal |
|---------|--------------|
| `src/modules/` or `src/features/` | Module-per-domain |
| `src/controllers/` + `src/services/` | Layer-based |
| `cmd/` + `internal/` | Go-style domain |
| `apps/` (monorepo) | App-per-domain |

### 2.3 General Rules

- **Monorepo**: Each app/package = 1 domain
- **MVC/Layer**: Group by feature, not by layer
- **Microservices**: Each service = 1 domain
- **Minimum 2 domains**, maximum 7 (beyond 7 = consolidate)

---

## Phase 3: Team Pattern Selection

도메인별로 적절한 팀 패턴을 매칭합니다.

### 3.1 Available Patterns

#### Pattern A: Full-Stack Team (5 members)
```
Lead (opus) — coordinator + reviewer
├── Frontend (sonnet) — UI/UX implementation
├── Backend (sonnet) — API/DB implementation
├── E2E QA Tester (sonnet) — runs app, tests user flows, captures screenshots
└── Unit Tester (sonnet) — TDD unit/integration tests (read-only)
```
**When:** Domain spans frontend + backend + DB

#### Pattern B: Feature Dev Team (4 members)
```
Lead (opus) — coordinator + reviewer
├── Implementer (sonnet) — coding
├── E2E QA Tester (sonnet) — runs app, tests user flows
└── Validator (sonnet) — review + regression check (read-only)
```
**When:** Domain is primarily one layer (frontend-heavy or backend-heavy)

#### Pattern C: Specialist Pair (3 members)
```
Lead (opus) — coordinator + main implementer
├── E2E QA Tester (sonnet) — runs app, tests user flows
└── Tester (sonnet) — TDD + validation
```
**When:** Focused domain, small scope

### 3.2 Maintenance Teams (Always Created)

These 4 teams are project-agnostic and always generated:

| Team | Members | Purpose |
|------|---------|---------|
| team-bugfix | 5 (lead + investigator + fixer + e2e qa + tester) | Bug investigation + fix + E2E regression |
| team-review | 4 (lead + security + quality + perf reviewer) | Multi-perspective code review |
| team-refactor | 4 (lead + implementer + e2e qa + validator) | Behavior-preserving refactoring + E2E verification |
| team-performance | 4 (lead + analyzer + optimizer + e2e qa) | Measurement-based optimization + E2E perf check |

### 3.3 Pattern Selection Logic

```
Domain size analysis:
  - Frontend + Backend + DB → Pattern A (Full-Stack)
  - Primarily frontend OR backend → Pattern B (Feature Dev)
  - Small/focused scope → Pattern C (Specialist Pair)
```

---

## Phase 4: Present Plan

사용자에게 분석 결과를 보여줍니다:

```markdown
## 프로젝트 분석 결과

**Tech Stack:** {language} / {frontend} / {backend} / {db}

### 식별된 도메인 & 추천 팀

| Domain | Scope | Pattern | Members |
|--------|-------|---------|---------|
| {domain_1} | {directories} | {pattern} | {count} |
| {domain_2} | {directories} | {pattern} | {count} |
| ... | ... | ... | ... |

### 유지보수 팀 (공통)
- team-bugfix (4명)
- team-review (4명)
- team-refactor (3명)
- team-performance (3명)

### 추가 생성 파일
- .claude/skills/team-dispatcher/SKILL.md (프로젝트 전용 디스패처)
- .claude/rules/team-rules.md (공통 규칙)
- .claude/settings.json (TaskCompleted + TeammateIdle hooks)
- CLAUDE.md 업데이트 (팀 규칙 + 스킬 트리거)

진행할까요? 도메인이나 팀 구성을 조정하시겠습니까?
```

사용자 확인/조정 후 Phase 5로 진행.

---

## Phase 5: Generate Team Skills

### 5.1 Common Rules File

`.claude/rules/team-rules.md` 생성:

```markdown
# Agent Team Rules (All Teammates)

## Development Methodology: SDD + TDD + E2E-First (Mandatory)
1. Spec first (SDD): Read feature spec before coding. If none exists, create one first.
2. E2E first: E2E tests are the PRIMARY pre-implementation tests and satisfy the TDD "test-first" mandate.
3. Unit tests are complementary, not required before implementation.
4. Full rules: `.claude/rules/tdd-sdd.md` (if exists, MUST follow — note: E2E tests fulfill the "write failing test" step)

## E2E Testing (Mandatory for All Implementation Agents)
Use the correct E2E tool based on platform:
- Web apps → Playwright
- Electron apps → Playwright Electron API (`_electron.launch`)
- Tauri apps → WebdriverIO
- Flutter apps → integration_test
- API only → Playwright API testing / supertest
Platform is auto-detected. See `qa-tester` skill for details.

## Task Execution Pipeline (Mandatory)

### Precondition: E2E Infrastructure
- If no E2E framework is detected (no config file, no `e2e/` directory):
  Set up E2E infrastructure first (per project-setup E2E scaffolding template).
- If E2E framework exists: proceed to Phase 1.

### Phase 1: Spec Confirmation
- Read spec/PRD. If none exists, create one first.
- Identify core user flows (based on acceptance criteria).

### Phase 2: E2E Test — Core Flows (Pre-Implementation)
- Write E2E tests for identified core user flows.
- Run tests → MUST confirm FAIL (RED).
- Commit E2E test files (creates timestamp evidence for verification gate).
- Report test file paths and failure results to team lead.
  Example: "e2e/flows/login.spec.ts — 3 tests, 3 failed (expected)"

### Phase 3: Implementation (GREEN)
- Implement code to pass the E2E tests.
- Unit tests may be added freely during implementation.

### Phase 4: E2E Verification
- Run FULL E2E test suite.
- Generate concise report:
  ✅ passed: 12  ❌ failed: 1  ⏱ duration: 8.3s
  FAIL: e2e/flows/checkout.spec.ts > "order confirmation after payment"
  Error: Expected status 200, got 500
- ALL PASS → Phase 6
- ANY FAIL → Phase 5

### Phase 5: Fix Loop (max 3 rounds)
Round N:
  1. Debugger: Analyze failure root cause + generate report
  2. Debugger: Verify whether the TEST ITSELF is correct
     - If test is wrong → propose test fix (not code fix)
     - If code is wrong → proceed to step 3
  3. Implementation agent: Write fix plan
  4. Team lead: Review fix plan → approve/reject
     - If rejected: implementation agent revises plan and resubmits
       (max 2 plan revisions per round, does NOT count toward 3-round limit)
     - If approved: proceed to step 5
  5. Implementation agent: Execute approved fix
  6. Re-run E2E suite → PASS → Phase 6, FAIL → Round N+1

  After 3 failures:
  - Generate escalation report with failure history
  - Set task status → BLOCKED
  - Escalate to user with:
    ⚠️ E2E Fix Loop Failed 3 Times — User Intervention Required
    Task: {task name}
    Failed test: {test file}:{test name}
    Attempts:
      Round 1: {root cause} → {fix applied} → test validity: {VALID/INVALID/N/A} → FAIL
      Round 2: {root cause} → {fix applied} → test validity: {VALID/INVALID/N/A} → FAIL
      Round 3: {root cause} → {fix applied} → test validity: {VALID/INVALID/N/A} → FAIL

### Phase 6: Edge Cases + Polish
- Add edge case / error scenario E2E tests.
- Refactor code (keep tests green).
- Run full test suite (final).
- Report to team lead with E2E results.

### Fix Loop Report Format
## Fix Loop Report — Round {N}
- Failed test: {test file}:{test name}
- Error: {error message}
- Root cause: {debugger analysis}
- Test validity: VALID / INVALID / N/A (if invalid: proposed test fix)
- Fix plan: {description}
- Lead approved: yes/no (if no: revision count)
- Result after fix: PASS/FAIL

## Bug Fix Workflow (Critical)
1. Write E2E test that reproduces the bug → verify it FAILS
2. Implement fix → verify reproduction test PASSES
3. Run FULL E2E test suite → no regressions allowed
4. Verify related functionality still works
5. Report to lead: bug reproduction test + full suite results

## Forbidden Patterns
- ❌ Code without spec
- ❌ Code before E2E tests (for core flows)
- ❌ Marking complete with failing tests
- ❌ Running only new tests (skipping regression suite)
- ❌ Deleting failing tests instead of fixing code
- ❌ Tautological tests (tests that always pass)
- ❌ Skipping fix loop (submitting with known failures)

## Verification Gate (MUST Pass Before Completion)
1. ✅ Spec exists and is current
2. ✅ Core flow E2E tests committed BEFORE implementation (evidence: E2E test commit precedes impl commit)
3. ✅ Full E2E suite PASS (report attached)
4. ✅ Edge case E2E tests added
5. ✅ Full regression suite PASS
6. ✅ Type check passes
7. ✅ Lint passes
8. ✅ Code polish complete
If ANY check fails → fix it. Do NOT submit incomplete work to lead.

## Cost Guidelines
- broadcast 금지 — always use targeted SendMessage
- 완료 후 즉시 종료
- 반복 3회 제한 — same fix fails 3 times → STOP and report to lead
- 불필요한 파일 읽기 금지

## Plan Approval (Mandatory)
- 코드 변경 전 반드시 plan → 팀장 승인 후 구현

## Communication
- 팀장에게만 SendMessage (팀원 간 직접 소통 금지)
- 보고 포맷: [역할] 상태: 내용
```

Adapt type check and lint commands to the project's tech stack:
- **Node/TS:** `npx tsc --noEmit`, `pnpm lint` (or `npm run lint`)
- **Python:** `mypy .`, `ruff check .` (or `flake8`)
- **Go:** `go vet ./...`, `golangci-lint run`
- **Rust:** `cargo check`, `cargo clippy`

Also add project-specific Hard Rules from CLAUDE.md if they exist.

### 5.2 Domain Team Skills

For each domain, generate `.claude/skills/team-{domain}/SKILL.md`:

**Template structure:**
```markdown
---
name: team-{domain}
description: {domain_korean} 파트 에이전트 팀 생성. Triggers on "{trigger_keywords}"
---

# {Domain} Part Agent Team

{description}

## Scope
- {directory_list}

## Step 1: TeamCreate
TeamCreate("{domain}-team")

## Step 2: TaskCreate ({count} roles)
{role_tasks based on pattern}

## Step 3: Agent Spawn ({count} parallel)
{agent_definitions with:
  - Lead: model="opus"
  - Members: model="sonnet", mode="plan"
  - E2E QA Tester: model="sonnet", scope="e2e/ only, read-only src/"
  - Each with: role, scope, constraints, workflow, rules}

### E2E QA Tester Spawn Template
Always include the QA Tester with this prompt:
```
You are the E2E QA Tester. Test from the user's perspective.

RULES:
1. Write E2E tests, NOT unit tests
2. Start the actual app and test against it
3. Auto-detect platform and use correct tool (Playwright/WebdriverIO/integration_test)
4. Capture screenshots at major steps
5. Test user FLOWS, not functions
6. Report bugs with: steps + screenshot + fix suggestion
7. Run FULL E2E suite after any code change (regression)

SCOPE: e2e/ directory ONLY. Read-only access to src/ for understanding.
```

## Step 4: SendMessage to Lead
SendMessage(to="{domain}-lead", message="팀이 생성되었습니다. 작업 요청: {user_task}")
```

**Customization per tech stack:**

| Tech | Lead Prompt Additions | Member Prompt Additions |
|------|----------------------|------------------------|
| Next.js | App Router, Server Components | queryKeys, styles.ts, Zustand |
| NestJS | Module architecture, Prisma | DTO class-validator, ValidationPipe |
| React | Component patterns | hooks, state management |
| Django | App structure, ORM | serializers, views, migrations |
| Go | Package structure | interfaces, error handling |
| Rust | Module/crate structure | ownership, lifetimes |

### 5.3 Maintenance Team Skills

Generate 4 maintenance team skills. These are mostly project-agnostic but adapt:
- Type check / lint commands to tech stack
- Test commands to test framework
- Project-specific conventions from CLAUDE.md

### 5.4 Dispatcher Skill

`.claude/skills/team-dispatcher/SKILL.md`을 생성합니다.

Phase 2에서 식별한 도메인과 Phase 5.2~5.3에서 생성한 팀 스킬 목록을 기반으로 **프로젝트 전용 dispatcher**를 생성합니다.

**생성 방법:**

1. `team-dispatcher/SKILL.md` (이 Skills 레포의 범용 템플릿)을 기반으로 복사
2. **도메인 판별 테이블**(Phase 1.2)을 프로젝트의 실제 도메인으로 교체:

```markdown
### 1.2 도메인 판별 (feature 유형일 때)

| Signal | Domain | Team Skill |
|--------|--------|-----------|
| "{domain_1_keyword_1}", "{domain_1_keyword_2}", ... | {domain_1} | team-{domain_1} |
| "{domain_2_keyword_1}", "{domain_2_keyword_2}", ... | {domain_2} | team-{domain_2} |
```

각 도메인의 Signal은 다음에서 추출:
- 디렉토리명 (예: `src/app/(admin)/` → "관리자", "admin")
- 프레임워크/라이브러리명 (예: NestJS → "NestJS", "API")
- 비즈니스 용어 (예: 주문관리 → "주문", "order")
- 한국어 + 영어 키워드를 모두 포함

3. **Decision Tree**(Phase 2)도 동일하게 업데이트:

```
사용자 지시
  ├── 버그/에러 키워드? → team-bugfix
  ├── 리뷰/검토 키워드? → team-review
  ├── 리팩토링/정리 키워드? → team-refactor
  ├── 성능/최적화 키워드? → team-performance
  ├── 기능 개발 키워드?
  │     ├── {domain_1} 키워드? → team-{domain_1}
  │     ├── {domain_2} 키워드? → team-{domain_2}
  │     └── 도메인 불명확? → 사용자에게 질문
  └── 키워드 불명확? → 사용자에게 질문
```

4. **description 필드**의 트리거 키워드도 프로젝트에 맞게 갱신

---

## Phase 6: Configure Hooks & CLAUDE.md

### 6.1 Enable Agent Teams

Ensure `.claude/settings.json` has:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 6.2 Add Team Hooks

Add to `.claude/settings.json` hooks (merge, don't overwrite):

```json
{
  "TaskCompleted": [{
    "hooks": [{
      "type": "prompt",
      "prompt": "A teammate reported task completion. Perform these checks IN ORDER:\n\nSTEP 1: E2E Evidence Check\n- Does the completion message include an E2E report?\n- Expected format: ✅ passed: N  ❌ failed: N  ⏱ duration: Ns\n- If missing → REJECT: 'Attach E2E test execution results'\n\nSTEP 2: E2E Result Validation\n- If failed > 0 → REJECT: 'E2E test failures exist. Enter Fix Loop'\n- If passed == 0 → REJECT: 'No E2E tests found'\n\nSTEP 3: Core Flow Coverage\n- Do E2E tests cover the spec's acceptance criteria?\n- If missing flows → REJECT: 'Core flow {flow_name} E2E missing'\n\nSTEP 4: Regression Check\n- Full test suite results (existing tests not broken)\n- If regression failures → REJECT: 'Regression test failures'\n\nSTEP 5: Standard Checks\n- Type check PASS\n- Lint PASS\n- Code polish complete\n\nALL PASS → ACCEPT the task\nANY REJECT → Send rejection with reason and trigger Fix Loop:\n\nFix Loop Round {N}/3 triggered.\nReason: {rejection reason}\nAction required:\n1. Request failure analysis from debugger\n2. Debugger verifies: is the TEST correct or is the CODE wrong?\n3. Write fix plan based on analysis\n4. Report fix plan to team lead (me) for approval\n   - If plan is rejected, revise and resubmit (max 2 revisions)\n5. After approval, execute fix\n6. Re-run full E2E suite and report completion again\n\nFailure count: {N}/3. Escalation to user after 3 failures."
    }]
  }],
  "TeammateIdle": [{
    "hooks": [{
      "type": "command",
      "command": "echo 'TeammateIdle: 팀장에게 요약 보고서를 SendMessage로 전달하세요:' && echo '[역할] 요약 보고서' && echo '- 완료한 것' && echo '- 테스트 결과 (새 테스트 + 전체 스위트)' && echo '- 이슈' && echo '- 다음 작업 제안' && echo '보고서 전달 후 즉시 종료하세요.'",
      "timeout": 10
    }]
  }]
}
```

### 6.3 Update CLAUDE.md

Add/update these sections in CLAUDE.md:

1. **Agent Team Rules** section
2. **Development Methodology: SDD + TDD** section
3. **Skill Triggers** table with all team skills

---

## Phase 7: Verify

### 7.1 File Check
```bash
# All team skills created?
ls .claude/skills/team-*/SKILL.md

# Dispatcher created with project domains?
grep "Signal.*Domain.*Team Skill" .claude/skills/team-dispatcher/SKILL.md
grep "team-" .claude/skills/team-dispatcher/SKILL.md | head -10

# Common rules?
cat .claude/rules/team-rules.md | head -5

# Hooks configured?
grep "TaskCompleted\|TeammateIdle" .claude/settings.json

# CLAUDE.md updated?
grep "team-" CLAUDE.md
```

### 7.2 Summary Report

```markdown
## Team Setup Complete

### Created
- {N} domain team skills: {list}
- 4 maintenance team skills: bugfix, review, refactor, performance
- 1 dispatcher skill (프로젝트 도메인 {N}개 매핑 완료)
- Common rules: .claude/rules/team-rules.md
- Hooks: TaskCompleted, TeammateIdle

### How to Use
- "{domain}팀" — 해당 도메인 팀 소집
- "버그수정팀/리뷰팀/리팩토링팀/성능팀" — 유지보수 팀 소집
- "팀 불러와" + 작업 지시 — 디스패처가 자동 판단

### Configuration
- Lead: opus, Members: sonnet
- All members: plan approval required
- SDD + TDD workflow enforced
- broadcast 금지, 3회 반복 제한, 완료 즉시 종료
```

---

## Rules

- 기존 `.claude/settings.json` hooks가 있으면 merge합니다 (덮어쓰지 않음)
- 기존 team skills가 있으면 덮어쓸지 사용자에게 확인합니다
- CLAUDE.md가 없으면 최소한의 CLAUDE.md를 생성합니다
- 도메인 7개 초과 시 통합을 제안합니다
- 모든 팀 스킬은 프로젝트의 실제 디렉토리 구조를 반영합니다
- 한국어/영어 응답은 프로젝트 CLAUDE.md의 언어 설정을 따릅니다
