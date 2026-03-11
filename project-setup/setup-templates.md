# Setup Templates

Ready-to-use templates for all configurable items.
Placeholders use `{{PLACEHOLDER}}` syntax — replace during generation.

---

## A. CLAUDE.md Sections to Add

When CLAUDE.md exists, merge these sections. When it doesn't, create full file.

### Session Protocol Section
```markdown
## Session Protocol

**Start:**
1. Read `docs/progress.txt` — what happened last session
2. Read `docs/features-list.md` — identify next task
3. Read relevant spec (if exists) before coding
4. Plan before code — never start without a plan

**End:**
1. Commit all changes
2. Update `docs/progress.txt`
3. Update `docs/features-list.md` status
4. Update `docs/changelog/CHANGELOG.md`
5. Update specs if code changed behavior

**Stuck (2+ failed fix attempts):**
1. STOP — do not attempt a third time
2. Commit current state
3. Record in progress.txt: what tried, why failed, which files
4. /clear → new session → different approach
```

### Context Management Section
```markdown
## Context Management

- 0-50%: work freely
- 50-70%: use Grep, avoid full file reads
- 70-90%: /compact
- 90%+: commit → progress.txt → /clear
- Never read files > 500 lines in full — use Grep for targeted search
{{LARGE_FILE_WARNINGS}}
```

### Skill Triggers Section
```markdown
## Skill Triggers

| Keyword | Skill | Action |
|---------|-------|--------|
| plan, 계획, design | plan-feature | Create implementation plan, no coding |
| wrap up, 마무리, handoff | progress-update | Record session progress |
| context, 컨텍스트, slow | context-check | Context threshold response |
{{ADDITIONAL_TRIGGERS}}
```

---

## B. Hooks

### settings.json (with hooks)
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '=== Git ===' && git status --short 2>/dev/null && echo '' && echo '=== Recent ===' && git log --oneline -5 2>/dev/null && echo '' && echo '=== Progress ===' && head -25 docs/progress.txt 2>/dev/null && echo '' && echo '=== Pending ===' && grep -c 'FAILING\\|❌\\|⚠️' docs/features-list.md 2>/dev/null && echo ' items remaining'"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Before finishing, verify: 1) All changes committed? 2) docs/progress.txt updated? 3) docs/features-list.md status accurate? 4) Specs updated if code changed? If all done, respond 'complete'. Otherwise, handle remaining items."
          }
        ]
      }
    ]
  },
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### Alternative: Shell-script based hooks (for complex logic)

session-start.sh:
```bash
#!/bin/bash
PROGRESS_FILE="docs/progress.txt"
if [ -f "$PROGRESS_FILE" ]; then
  echo "=== Progress ==="
  head -25 "$PROGRESS_FILE"
  echo ""
  echo "=== Recent Commits ==="
  git log --oneline -5 2>/dev/null
  echo ""
  echo "=== Pending ==="
  grep -c 'FAILING\|❌' docs/features-list.md 2>/dev/null && echo " items remaining"
else
  echo "[WARN] docs/progress.txt not found. Create it to enable session handoff."
fi
```

session-stop.sh:
```bash
#!/bin/bash
echo "=== Session End Check ==="
PROGRESS_FILE="docs/progress.txt"
if [ -f "$PROGRESS_FILE" ]; then
  FILE_TIME=$(stat -c %Y "$PROGRESS_FILE" 2>/dev/null || stat -f %m "$PROGRESS_FILE" 2>/dev/null)
  NOW=$(date +%s)
  DIFF=$((NOW - FILE_TIME))
  if [ "$DIFF" -gt 1800 ]; then
    echo "[!] progress.txt not updated in 30+ minutes."
  else
    echo "[OK] progress.txt recently updated."
  fi
fi
CHANGES=$(git status --porcelain 2>/dev/null | wc -l)
if [ "$CHANGES" -gt 0 ]; then
  echo "[!] Uncommitted changes: ${CHANGES} files"
  git status --short 2>/dev/null
else
  echo "[OK] All changes committed."
fi
```

---

## C. Skills

### plan-feature/SKILL.md
```markdown
---
name: plan-feature
description: Creates implementation plan before coding. Triggers on "plan", "계획", "설계", "design", "analyze". Prevents premature coding.
---

# Feature Planning

## Critical Rule
**Do NOT write any code during this phase.**
Planning must be approved before implementation begins.

## Procedure
1. Read `docs/features-list.md` — find target item
2. Check for existing spec at `docs/specs/features/` (if specs directory exists)
3. Use Grep to find related code — do NOT read entire files
4. Write plan:

## Plan Format
```
## Plan: [Feature/Bug ID] — [Name]
**Goal**: One sentence
**Files to modify**: [list with what changes]
**Files to create**: [list with purpose]
**Steps** (each = one committable unit):
1. [ ] ...
2. [ ] ...
**Tests**: How to verify
**Risks**: What could break
```

5. Ask user for approval before proceeding

## Context Rules
- Use Grep, not full file reads
- {{LARGE_FILE_RULES}}
```

### progress-update/SKILL.md
```markdown
---
name: progress-update
description: Records session progress for handoff. Triggers on "마무리", "wrap up", "session end", "progress". Manual invocation only.
disable-model-invocation: true
---

# Progress Update

## Steps
1. `git log --oneline` — identify session commits
2. Update docs/progress.txt — add new entry at top
3. Update docs/features-list.md — change statuses
4. Update docs/changelog/CHANGELOG.md if applicable
5. Commit documentation updates
```

### context-check/SKILL.md
```markdown
---
name: context-check
description: Monitors context usage and takes corrective action. Triggers on "context", "컨텍스트", "tokens", "slow", "느려졌다".
---

# Context Management

## Thresholds
| Usage | Action |
|-------|--------|
| 0-50% | Work freely |
| 50-70% | Grep only, no full file reads |
| 70-90% | /compact |
| 90%+ | Commit → progress.txt → /clear |

## "Stuck Loop" Detection
If same fix attempted twice with same result:
1. STOP immediately
2. Commit current state
3. Write analysis in progress.txt
4. /clear → new session → different approach

## Never Read in Full
{{LARGE_FILE_LIST}}
```

### test-and-refactor/SKILL.md
```markdown
---
name: test-and-refactor
description: Test-driven development cycle with refactoring. Triggers on "test", "테스트", "refactor", "TDD", "quality".
---

# Test & Refactor Cycle

## Phase 1: Test First
1. Read feature spec/completion criteria
2. Write tests ({{TEST_FRAMEWORK}})
3. Run: `{{TEST_COMMAND}}`

## Phase 2: Implement
4. Write code to pass failing tests
5. Run tests — max 2 attempts, then STOP and analyze

## Phase 3: Refactor
6. Check project conventions:
{{CONVENTIONS_CHECKLIST}}
7. Check function size (< 50 lines)
8. Remove duplicates
9. Re-run tests

## Phase 4: Commit
10. `{{TYPECHECK_COMMAND}}`
11. `{{LINT_COMMAND}}`
12. Commit with test files

## Failure Protocol
2 failures → STOP → record in progress.txt → /clear → different approach
```

### spec-update/SKILL.md
```markdown
---
name: spec-update
description: Updates spec documents after code changes. Triggers on "spec update", "명세 업데이트", "sync docs". Manual only.
disable-model-invocation: true
---

# Spec Document Update

## Steps
1. `git diff --name-only` — identify changed files
2. Match changed files to spec documents
3. Update relevant specs
4. Update CHANGELOG.md
5. Report updated files to user
```

---

## D. Agents

### code-reviewer.md
```markdown
---
name: code-reviewer
description: Reviews code quality against project conventions. Triggers on "review", "리뷰", "check code".
tools: Read, Grep, Glob, Bash
model: sonnet
memory: project
---

You are a code reviewer for this project.

## Conventions to Verify
{{PROJECT_CONVENTIONS}}

## Review Checklist
- [ ] Convention violations
- [ ] Error handling completeness
- [ ] {{FRAMEWORK_SPECIFIC_CHECKS}}
- [ ] Security issues (hardcoded secrets, injection)
- [ ] Performance concerns

## Output: Critical / Warning / Info classification

## Memory
Record recurring patterns in MEMORY.md. Consult before each review.
```

### bug-analyzer.md
```markdown
---
name: bug-analyzer
description: Analyzes bug root causes with structured hypotheses. Triggers on "bug", "error", "버그", "에러".
tools: Read, Grep, Glob, Bash
model: sonnet
memory: project
---

You analyze bugs in this project.

## Data Flow
{{PROJECT_DATA_FLOW}}

## Common Bug Patterns
{{COMMON_BUG_PATTERNS}}

## Procedure
1. Extract error keywords → Grep related code
2. Trace data flow
3. Form 3 hypotheses (ranked)
4. Suggest verification + fix for top hypothesis

## Memory
Record bug patterns in MEMORY.md.
```

---

## E. Commands

### status.md
```markdown
Check project status:
1. `git status --short` — uncommitted changes
2. `git log --oneline -5` — recent commits
3. Read top of `docs/progress.txt` — last session
4. Count items in `docs/features-list.md` by status
5. Output concise Korean summary
```

### plan.md
```markdown
Plan next feature:
1. Read `docs/features-list.md` — highest priority FAILING item
2. Check if spec exists
3. If no spec: create one first
4. Create implementation plan (no coding)
5. Ask user for approval
```

### quality-check.md
```markdown
Run full quality pipeline:
1. `{{TYPECHECK_COMMAND}}` — type checking
2. `{{LINT_COMMAND}}` — lint checking
3. `{{TEST_COMMAND}}` — run tests
4. Report pass/fail for each step
```

---

## F. Rules

### context-management.md
```markdown
# Context Management Rules

## Thresholds
| Usage | Action |
|-------|--------|
| 0-50% | Work freely |
| 50-70% | Grep only |
| 70-90% | /compact |
| 90%+ | Commit → progress → /clear |

## Large Files — Never Read in Full
{{LARGE_FILE_LIST}}

## Session Switch Protocol
Before /clear: commit → progress.txt → features-list.md → changelog
```

### spec-code-sync.md
```markdown
# Spec-Code Sync Rules

## Rule 1: No code without a plan
New features → create plan/spec first → get approval → then code

## Rule 2: Code changes → spec updates
After modifying code, check if matching spec needs updating.

## Rule 3: Changes → CHANGELOG
All changes recorded in docs/changelog/CHANGELOG.md

## Rule 4: Work logs
{{WORK_LOG_RULES}}
```

---

## G. Operational Documents

### docs/progress.txt
```
# Progress Log — {{PROJECT_NAME}}

## 최신 상태 (가장 위가 최신)

### 세션 #001 — {{DATE}}
- **작업 내용**: 프로젝트 셋업 구성
- **완료한 것**: .claude/ 디렉토리, 운영 문서 생성
- **다음에 할 것**: [첫 번째 작업 항목]
- **이슈**: 없음
- **관련 커밋**: (초기 셋업)
```

### docs/features-list.md
```markdown
# 기능 & 버그 추적 — {{PROJECT_NAME}}

## FAILING (미해결)
| # | 항목 | 우선순위 | 설명 |
|---|------|----------|------|
| | | | |

## IN_PROGRESS (진행 중)
| # | 항목 | 설명 |
|---|------|------|
| | | |

## PASSING (완료)
| # | 항목 | 완료일 |
|---|------|--------|
| | | |
```

### docs/changelog/CHANGELOG.md
```markdown
# Changelog — {{PROJECT_NAME}}

## [미출시]

### 추가
- 프로젝트 셋업 구성 (.claude/, 운영 문서)

### 수정

### 변경
```

### docs/team-briefing.md
```markdown
# {{PROJECT_NAME}} Team Briefing

## Project
{{PROJECT_DESCRIPTION}}

## Tech Stack
{{TECH_STACK}}

## Key Conventions
{{KEY_CONVENTIONS}}

## Directory Map
{{DIRECTORY_MAP}}

## Before Working
1. Read docs/progress.txt
2. Read docs/features-list.md
3. Read relevant spec if exists

## Commands
{{BUILD_COMMANDS}}
```
