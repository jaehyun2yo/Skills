# E2E Testing Pipeline Upgrade Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade declarative TDD rules into a procedural E2E testing pipeline that writes, runs, and auto-verifies E2E tests — reducing manual user QA.

**Architecture:** Edit 4 existing files across 2 skills (project-setup, team-setup). No new files. project-setup handles E2E infra scaffolding + scoring; team-setup handles procedural pipeline + hook enforcement.

**Tech Stack:** Markdown skill files (prompt templates for Claude Code agents)

**Spec:** `docs/superpowers/specs/2026-03-17-e2e-pipeline-upgrade-design.md`

---

## Chunk 1: project-setup Skill Updates

### Task 1: Update project-setup/SKILL.md — E2E Detection & Scaffolding

**Files:**
- Modify: `project-setup/SKILL.md:14-47` (Phase 1) and `SKILL.md:66-73` (Phase 3) and `SKILL.md:77-80` (Phase 4)

- [ ] **Step 1: Add E2E detection to Phase 1**

After the existing `1.3 Development Methodology Check` section (line 43), add a new `1.4 E2E Infrastructure Check` section. Change `**1.4 CLAUDE.md Analysis**` (line 45) to `**1.5 CLAUDE.md Analysis**`.

**Note:** All line numbers in this plan reference the ORIGINAL file state. After edits, line numbers will shift — use content matching for subsequent steps.

```markdown
**1.4 E2E Infrastructure Check**
```bash
# E2E infrastructure detection — config files first, then dependencies
ls playwright.config.* cypress.config.* wdio.conf.* 2>/dev/null
ls e2e/ tests/e2e/ test/e2e/ 2>/dev/null
# Only check devDependencies, not full file
grep -A 50 '"devDependencies"' package.json 2>/dev/null | grep -E "playwright|cypress|webdriverio"
grep -E "playwright|cypress|pytest" pyproject.toml 2>/dev/null
```
Record: E2E framework, E2E directory, E2E config file presence.
```

- [ ] **Step 2: Add E2E recommendations to Phase 3**

In Phase 3 (Report), after line 71's recommendation rankings, add E2E-specific recommendations:

```markdown
- E2E infrastructure gaps:
  - Missing E2E framework → `[🔴 Critical]`
  - Missing E2E directory structure (`e2e/`) → `[🟠 High]`
  - Missing E2E command in CLAUDE.md → `[🟠 High]`
```

- [ ] **Step 3: Add E2E scaffolding reference to Phase 4**

In Phase 4 (Generate), after line 80's tech-profiles reference, add:

```markdown
For E2E infrastructure, use qa-tester auto-detection logic for platform/framework selection. Generate E2E scaffolding using the E2E Infrastructure Template in → [setup-templates.md](setup-templates.md) Section K.
project-setup defers to qa-tester SKILL.md conventions for framework-specific config defaults (e.g. screenshot, trace settings).
```

- [ ] **Step 4: Update supporting files reference**

Find the `## Supporting Files` section (originally at line 110) and replace its content:

```markdown
## Supporting Files
- [checklist.md](checklist.md) — Scoring criteria (A–K, 48 items, includes E2E items J7-J8)
- [setup-templates.md](setup-templates.md) — File templates for all configurable items (includes Agent Team templates + E2E Infrastructure)
- [tech-profiles.md](tech-profiles.md) — Tech stack-specific placeholder values
```

- [ ] **Step 5: Verify SKILL.md edit**

Read the modified file and confirm:
- 1.4 E2E Infrastructure Check exists after 1.3
- Old 1.4 CLAUDE.md Analysis is now 1.5
- Phase 3 has E2E-specific recommendation severities
- Phase 4 references E2E scaffolding template and qa-tester deference

- [ ] **Step 6: Commit**

```bash
git add project-setup/SKILL.md
git commit -m "feat(project-setup): add E2E detection, recommendations, and scaffolding reference"
```

---

### Task 2: Update project-setup/checklist.md — E2E Scoring Items

**Files:**
- Modify: `project-setup/checklist.md:113-126` (Section J)

- [ ] **Step 1: Redistribute J5/J6 and add J7/J8**

Replace lines 113-126 (the entire J section) with:

```markdown
## J. Testing & Development Methodology (6 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| J1 | Test framework configured (incl. E2E) | Config file + E2E framework (Playwright/Cypress/etc.) | Test framework only, no E2E | None |
| J2 | Test files exist | > 10 test files with actual cases | 1-10 files | None |
| J3 | Test commands documented | In CLAUDE.md with single-file pattern + E2E command | Basic command only | Not documented |
| J4 | TDD/SDD rules file | `.claude/rules/tdd-sdd.md` with spec-first + test-first + regression + bug-fix workflow | Partial rules (TDD or SDD only) | None |
| J5 | TDD enforcement + E2E-first | Red-Green-Refactor mandatory, E2E tests satisfy test-first mandate, regression required | TDD mentioned but no E2E-first or enforcement detail | None |
| J6 | SDD enforcement | Spec-first mandatory, spec-code sync rules, acceptance criteria required | Spec mentioned but no enforcement | None |
| J7 | E2E directory structure | `e2e/` + `pages/` + `flows/` + `fixtures/` exist | `e2e/` exists but incomplete structure | None |
| J8 | E2E run command documented | CLAUDE.md contains E2E execution command + single-file E2E command | E2E command exists but incomplete | Not documented |

---

**Note:** J4-J6 are checked by scanning `.claude/rules/tdd-sdd.md` or equivalent methodology rules. J7-J8 are checked by scanning the project directory structure and CLAUDE.md. For existing projects, check all files under `.claude/rules/` for TDD/SDD keywords.

**Scoring note:** J category was redistributed from the original 6 points to 8 items. J1-J4 = 1pt each, J5-J6 = 0.5pt each (split from original 1pt), J7-J8 = 0.5pt each (new). Total remains 6 points.
```

- [ ] **Step 2: Verify checklist edit**

Read `project-setup/checklist.md` and confirm:
- J1 mentions E2E framework
- J3 mentions E2E command
- J5 mentions E2E-first
- J7 (E2E directory) and J8 (E2E command) exist
- Total score unchanged (6 points for J category)

- [ ] **Step 3: Commit**

```bash
git add project-setup/checklist.md
git commit -m "feat(project-setup): add E2E scoring items J7-J8, update J1/J3/J5 for E2E"
```

---

### Task 3: Update project-setup/setup-templates.md — E2E Template + Hook Update

**Files:**
- Modify: `project-setup/setup-templates.md` — add Section K (after line 1090) and update TaskCompleted hook in Section B

- [ ] **Step 1: Add E2E Infrastructure Template as Section K**

After the last line (1090), append:

```markdown

---

## K. E2E Infrastructure Template

Scaffolding for E2E test infrastructure. Use qa-tester SKILL.md auto-detection for platform/framework selection. project-setup defers to qa-tester conventions for framework-specific config defaults.

### Directory Structure
```
e2e/
├── fixtures/          # Test data, seeds
├── pages/             # Page Object Model
├── flows/             # User flow tests
├── helpers/           # Utilities
└── {{CONFIG_FILE}}    # playwright.config.ts / wdio.conf.ts etc.
```

### Config File Placeholder
(Defer to qa-tester SKILL.md defaults for framework-specific settings)
```
{{E2E_CONFIG_CONTENT}}
```

Replace `{{E2E_CONFIG_CONTENT}}` based on detected platform:
| Platform | Config File | Key |
|----------|-----------|-----|
| Web (React/Next/Vue/Svelte) | `playwright.config.ts` | `baseURL`, `retries: 1` |
| Electron | `playwright.config.ts` | `use: { _electron }` |
| Tauri | `wdio.conf.ts` | `capabilities: [{ 'tauri:options' }]` |
| Flutter | `integration_test/` dir | No config file |
| API (Node) | `playwright.config.ts` | `use: { baseURL: api }` |
| API (Python) | `conftest.py` | `httpx.AsyncClient` fixture |
| API (Go) | `*_test.go` | `httptest.NewServer` |

### Base Fixture Templates

#### Authenticated User Fixture (Playwright example)
```typescript
// e2e/fixtures/auth.ts
import { test as base } from '@playwright/test';

export const test = base.extend({
  authenticatedPage: async ({ page }, use) => {
    // Login flow — customize per project
    await page.goto('/login');
    await page.fill('[name="email"]', '{{TEST_USER_EMAIL}}');
    await page.fill('[name="password"]', '{{TEST_USER_PASSWORD}}');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
    await use(page);
  },
});
```

#### Empty State Fixture
```typescript
// e2e/fixtures/empty-state.ts
import { test as base } from '@playwright/test';

export const test = base.extend({
  cleanPage: async ({ page }, use) => {
    // Navigate to app with clean state
    await page.goto('/');
    await use(page);
  },
});
```

### CLAUDE.md Addition Template
```markdown
## E2E Testing

- E2E run: `{{E2E_RUN_COMMAND}}`
- E2E single file: `{{E2E_SINGLE_COMMAND}}`
- E2E UI mode: `{{E2E_UI_COMMAND}}`
```

Replace commands based on detected platform:
| Platform | Run All | Single File | UI Mode |
|----------|---------|------------|---------|
| Playwright | `npx playwright test` | `npx playwright test e2e/flows/{file}.spec.ts` | `npx playwright test --ui` |
| Cypress | `npx cypress run` | `npx cypress run --spec e2e/flows/{file}.cy.ts` | `npx cypress open` |
| WebdriverIO | `npx wdio run wdio.conf.ts` | `npx wdio run wdio.conf.ts --spec e2e/flows/{file}.ts` | N/A |
| pytest | `pytest e2e/` | `pytest e2e/flows/test_{file}.py` | N/A |

### Generation Flow
1. qa-tester auto-detection logic determines platform
2. Present E2E framework install command to user for approval
3. Generate config file + directory structure (`e2e/fixtures/`, `e2e/pages/`, `e2e/flows/`, `e2e/helpers/`)
4. Generate base fixtures (authenticated user, empty state)
5. Add E2E commands to CLAUDE.md
```

- [ ] **Step 2: Add TaskCompleted hook to setup-templates.md Section B**

The current Section B settings.json template (lines 162-231) has SessionStart, PreCompact, PostToolUse, and Stop hooks but NO TaskCompleted hook. Add a TaskCompleted hook entry to the settings.json template.

Find the Stop hook closing bracket in the settings.json template (after line 225's `]`), and add before the closing `}` of the hooks object:

```json
    "TaskCompleted": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "A teammate reported task completion. Perform these checks IN ORDER:\n\nSTEP 1: E2E Evidence Check\n- Does the completion message include an E2E report?\n- Expected format: ✅ passed: N  ❌ failed: N  ⏱ duration: Ns\n- If missing → REJECT: 'Attach E2E test execution results'\n\nSTEP 2: E2E Result Validation\n- If failed > 0 → REJECT: 'E2E test failures exist. Enter Fix Loop'\n- If passed == 0 → REJECT: 'No E2E tests found'\n\nSTEP 3: Core Flow Coverage\n- Do E2E tests cover the spec's acceptance criteria?\n- If missing flows → REJECT: 'Core flow {flow_name} E2E missing'\n\nSTEP 4: Regression Check\n- Full test suite results (existing tests not broken)\n- If regression failures → REJECT: 'Regression test failures'\n\nSTEP 5: Standard Checks\n- Type check PASS, Lint PASS, Code polish complete\n\nALL PASS → ACCEPT the task\nANY REJECT → Send rejection with reason + trigger Fix Loop (max 3 rounds, then escalate to user)"
          }
        ]
      }
    ]
```

- [ ] **Step 3: Verify Section K and TaskCompleted hook added**

Read `setup-templates.md` and confirm:
- Section K exists at the end with E2E directory structure, config placeholders, fixture templates
- Section B settings.json template now includes a TaskCompleted hook with 5-step E2E verification

- [ ] **Step 4: Commit**

```bash
git add project-setup/setup-templates.md
git commit -m "feat(project-setup): add E2E infrastructure template (Section K) + TaskCompleted E2E hook"
```

---

## Chunk 2: team-setup Skill Updates

### Task 4: Update team-setup/SKILL.md — Procedural E2E Pipeline

**Files:**
- Modify: `team-setup/SKILL.md:192-256` (team-rules.md template — Workflow + Verification Gate sections)
- Modify: `team-setup/SKILL.md:396-414` (TaskCompleted hook)

- [ ] **Step 1: Replace team-rules.md template content**

In the team-rules.md template (inside the markdown code fence starting at line 192), replace all content from `## Development Methodology: SDD + TDD (Mandatory)` (line 195) through `- 보고 포맷: [역할] 상태: 내용` (line 255), ending just before the closing code fence at line 256. Preserve the opening `# Agent Team Rules (All Teammates)` heading (lines 193-194) and the closing code fence.

Replace with:

```markdown
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
```
## Fix Loop Report — Round {N}
- Failed test: {test file}:{test name}
- Error: {error message}
- Root cause: {debugger analysis}
- Test validity: VALID / INVALID / N/A (if invalid: proposed test fix)
- Fix plan: {description}
- Lead approved: yes/no (if no: revision count)
- Result after fix: PASS/FAIL
```

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

- [ ] **Step 2: Update TaskCompleted hook in Phase 6.2**

In the Phase 6.2 hooks section, replace the entire `"TaskCompleted": [...]` block (lines 400-405 in the original file — from `"TaskCompleted"` through its closing `}]`) while preserving the `"TeammateIdle": [...]` block (lines 406-413) intact.

Replace the TaskCompleted block with:

```json
"TaskCompleted": [{
  "hooks": [{
    "type": "prompt",
    "prompt": "A teammate reported task completion. Perform these checks IN ORDER:\n\nSTEP 1: E2E Evidence Check\n- Does the completion message include an E2E report?\n- Expected format: ✅ passed: N  ❌ failed: N  ⏱ duration: Ns\n- If missing → REJECT: 'Attach E2E test execution results'\n\nSTEP 2: E2E Result Validation\n- If failed > 0 → REJECT: 'E2E test failures exist. Enter Fix Loop'\n- If passed == 0 → REJECT: 'No E2E tests found'\n\nSTEP 3: Core Flow Coverage\n- Do E2E tests cover the spec's acceptance criteria?\n- If missing flows → REJECT: 'Core flow {flow_name} E2E missing'\n\nSTEP 4: Regression Check\n- Full test suite results (existing tests not broken)\n- If regression failures → REJECT: 'Regression test failures'\n\nSTEP 5: Standard Checks\n- Type check PASS\n- Lint PASS\n- Code polish complete\n\nALL PASS → ACCEPT the task\nANY REJECT → Send rejection with reason and trigger Fix Loop:\n\nFix Loop Round {N}/3 triggered.\nReason: {rejection reason}\nAction required:\n1. Request failure analysis from debugger\n2. Debugger verifies: is the TEST correct or is the CODE wrong?\n3. Write fix plan based on analysis\n4. Report fix plan to team lead (me) for approval\n   - If plan is rejected, revise and resubmit (max 2 revisions)\n5. After approval, execute fix\n6. Re-run full E2E suite and report completion again\n\nFailure count: {N}/3. Escalation to user after 3 failures."
  }]
}]
```

- [ ] **Step 3: Verify team-setup edits**

Read `team-setup/SKILL.md` and confirm:
- `## Development Methodology` mentions "E2E-First"
- `## Task Execution Pipeline` has 6 phases with Precondition
- Phase 2 includes "Commit E2E test files"
- Phase 5 has fix loop with test validity check and plan rejection handling
- Verification Gate has 8 items (not 7)
- TaskCompleted hook has 5-step E2E verification + fix loop trigger
- Bug Fix Workflow mentions E2E test

- [ ] **Step 4: Commit**

```bash
git add team-setup/SKILL.md
git commit -m "feat(team-setup): replace declarative TDD with procedural E2E pipeline + fix loop"
```

---

## Chunk 3: Final Verification

### Task 5: Cross-File Consistency Check

**Files:**
- Read: all 4 modified files

- [ ] **Step 1: Verify cross-references**

Check:
1. `project-setup/SKILL.md` Phase 4 references "setup-templates.md Section K" → Section K exists
2. `project-setup/checklist.md` J7/J8 criteria match the E2E template structure in Section K
3. `team-setup/SKILL.md` team-rules.md template references `tdd-sdd.md` with E2E clarification
4. TaskCompleted hook prompt in `team-setup/SKILL.md` aligns with Verification Gate items
5. TaskCompleted hook in `setup-templates.md` Section B matches the one in `team-setup/SKILL.md`

- [ ] **Step 2: Verify no contradictions**

Check:
1. J5 says "E2E tests satisfy test-first mandate" — matches team-setup's "E2E-First" methodology
2. project-setup defers to qa-tester conventions — no conflicting config defaults
3. Fix loop max 3 rounds + max 2 plan revisions — consistent between pipeline and hook
4. Verification Gate has 8 items in team-setup — hook checks align

- [ ] **Step 3: Final commit (if any fixes needed)**

```bash
git add -A
git commit -m "fix: resolve cross-file consistency issues in E2E pipeline upgrade"
```

- [ ] **Step 4: Summary commit**

If all individual commits passed, no summary commit needed. Otherwise:

```bash
git log --oneline -5
```

Verify commit history shows clean progression of changes.
