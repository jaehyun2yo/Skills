# Skills

Personal skill collection for Claude Code — with Agent Team orchestration + auto session handoff.

## Quick Start

```bash
# 1. Clone
git clone https://github.com/jaehyun2yo/Skills.git

# 2. Symlink all skills at once
for skill in Skills/*/; do
  name=$(basename "$skill")
  ln -sf "$(pwd)/$skill" ~/.claude/skills/"$name"
done

# 3. Enable Agent Teams (optional)
# Add to .claude/settings.json:
# { "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

Done. Skills auto-trigger by keyword — no manual setup needed.

## Skills

| Skill | Triggers | What It Does |
|-------|----------|--------------|
| [dev-start](dev-start/) | "시작하자", "start coding", "이어서 개발" | Session init — loads handoff, shows state, suggests next work. Complex tasks → Agent Team suggestion |
| [dev-wrap](dev-wrap/) | "마무리", "wrap up", "커밋하고 끝" | Session end — generates handoff.md, cleans up teams, commits, updates docs |
| [project-setup](project-setup/) | "셋업", "setup", "프로젝트 설정" | Scans project → scores (A-K, 48pts) → generates missing config with approval |
| [skill-register](skill-register/) | "스킬 등록", "register skill" | Scans skills → registers to CLAUDE.md triggers table → creates symlinks |
| [code-polish](code-polish/) | "코드정리", "리팩토링", "code polish" | Auto-cleans changed files — dead code removal, optimization, refactoring. Auto-triggers via Stop hook |
| [usage-guide](usage-guide/) | "사용법", "usage guide", "매뉴얼" | Generates unified usage doc covering all skills, hooks, plugins, agents |
| [team-setup](team-setup/) | "팀 셋업", "team setup", "팀 시스템 설정" | Scans project → identifies domains → generates tailored team skills + hooks + rules |
| [team-dispatcher](team-dispatcher/) | "팀 불러와", "팀 소집", "team dispatch" | Analyzes user request → auto-selects right team (bugfix/review/feature/etc) |
| [skill-eval](skill-eval/) | "스킬 평가", "eval skill", "benchmark skill", "eval all", "대시보드" | Eval/benchmark/A/B compare/auto-generate scenarios/batch eval/dashboard — measures pass rate, tokens, time |
| [security-guard](security-guard/) | "보안 검사", "security check", "security scan" | OWASP Top 10 auto-scan for changed code, blocks dangerous commands via PreToolUse |
| [qa-tester](qa-tester/) | "QA", "E2E 테스트", "qa test", "실사용 테스트" | E2E QA — auto-detects platform (Web/Electron/Tauri/Flutter), runs app, tests user flows, reports bugs |

## Auto Session Handoff

The killer feature. Context gets full? No problem — everything is automatic via hooks.

### How It Works

```
[You're coding...]
  ↓ Context reaches threshold
  ↓ PreCompact hook fires automatically
  ↓ → Saves session state to .claude/handoff.md
  ↓     (completed work, pending tasks, decisions, failed approaches, next steps)
  ↓
  ↓ You type /clear
  ↓
  ↓ SessionStart hook fires automatically
  ↓ → Loads .claude/handoff.md into new session
  ↓ → Seamless continuation
```

**You only type `/clear`. Everything else is automatic.**

### What Gets Saved in Handoff

```markdown
# Session Handoff — 2026-03-12

## Completed
- Implemented auth module (src/auth/)

## In Progress
- [ ] Login validation — src/auth/login.ts:42 has error

## Decisions
- JWT stored in httpOnly cookies (security)

## Lore (Failed Approaches)        ← This is the key!
- ❌ passport.js — conflicts with custom middleware
- ❌ bcrypt rounds=15 — too slow, using rounds=12

## Next Steps
1. Fix login.ts:42 error
2. Write tests (tests/auth/)

## Team Re-spawn (if team was used)
Spawn 2 teammates:
- Frontend (Sonnet): src/components/ ONLY
- Tests (Sonnet): tests/ ONLY
```

The **Lore section** prevents the next session from repeating failed approaches.

### Setup (One-Time)

Add hooks to your project's `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [{
          "type": "command",
          "command": "echo '=== Git ===' && git branch --show-current && git status --short && echo '=== Progress ===' && head -25 docs/progress.txt 2>/dev/null || echo 'no progress.txt'"
        }]
      },
      {
        "matcher": "compact",
        "hooks": [{
          "type": "command",
          "command": "echo '=== Handoff ===' && cat .claude/handoff.md 2>/dev/null || echo 'No handoff'"
        }]
      },
      {
        "matcher": "clear",
        "hooks": [{
          "type": "command",
          "command": "echo '=== Handoff ===' && cat .claude/handoff.md 2>/dev/null || echo 'No handoff' && echo '=== Git ===' && git branch --show-current && git status --short"
        }]
      }
    ],
    "PreCompact": [
      {
        "hooks": [{
          "type": "prompt",
          "prompt": "Context is about to be compacted. Write a handoff file to .claude/handoff.md with: 1) Completed work, 2) In-progress items (file paths, line numbers), 3) Decisions and rationale, 4) Failed approaches and why (Lore), 5) Next steps, 6) Team re-spawn prompts if applicable."
        }]
      }
    ],
    "Stop": [
      {
        "hooks": [{
          "type": "prompt",
          "prompt": "Before finishing: 1) All changes committed? 2) .claude/handoff.md updated? 3) docs/progress.txt updated? 4) Active teammates shut down? Handle remaining items."
        }]
      }
    ]
  }
}
```

Add to `.gitignore`:
```
.claude/handoff.md
```

Or just run `"셋업해줘"` — the **project-setup** skill generates all of this automatically.

## Agent Team Integration

All skills are Agent Team-aware. When a task is complex (3+ directories, multi-layer changes), **dev-start** automatically suggests a team:

### Team Patterns

**Parallel Implementation** — feature dev, refactoring:
```
Create a team with 3 teammates:
- Backend (Sonnet): src/api/ ONLY
- Frontend (Sonnet): src/components/ ONLY
- Tests (Sonnet): tests/ ONLY
Use delegate mode. Require plan approval.
```

**Multi-Perspective Review** — code review, security audit:
```
Create a team with 3 reviewers:
- Security: auth, injection, data validation
- Performance: N+1 queries, memory, latency
- Test coverage: edge cases, missing tests
```

**Adversarial Debug** — complex bugs:
```
Spawn 5 teammates to investigate different hypotheses.
Have them challenge each other's theories like a scientific debate.
```

### Team Best Practices (Built Into Skills)

| Practice | How Skills Apply It |
|----------|-------------------|
| Lead = Manager only | `"Use delegate mode"` in spawn prompts |
| File conflict prevention | Directory ownership per teammate |
| Cost optimization | Sonnet for teammates, Opus for lead |
| Plan approval gate | Required before code changes |
| Idle cleanup | dev-wrap handles graceful shutdown |
| Session continuity | Handoff includes team re-spawn prompts |

### Team + Session Handoff

When lead's context fills during team work:
1. PreCompact saves team state (composition, task status, spawn prompts)
2. `/clear` → new session loads handoff
3. New lead session re-creates team from saved prompts

## Development Workflow

### Daily Flow

```
"시작하자"                      → dev-start loads handoff + state
  ↓ coding...
  ↓ context full → /clear       → auto handoff + restore
  ↓ more coding...
"마무리" (or session end)       → code-polish auto-cleans → dev-wrap commits + handoff
```

### Project Onboarding

```
"셋업해줘"                      → project-setup scores + generates config
"스킬 등록해줘"                  → skill-register wires skills to CLAUDE.md
"사용법 만들어줘"                → usage-guide generates docs/usage-guide.md
```

## Scoring System (project-setup)

| Category | Points | What It Checks |
|----------|--------|---------------|
| A. CLAUDE.md Quality | 7 | Commands, architecture, conventions, session protocol |
| B. Hooks | 6 | SessionStart, Stop, PreCompact, matchers, handoff auto-load |
| C. Skills | 5 | Planning, progress, context, testing, language |
| D. Agents | 3 | Code reviewer, bug analyzer, domain specialist |
| E. Commands | 3 | Status, plan, quality check |
| F. Rules | 2 | Context management, spec-code sync |
| G. Operational Docs | 5 | progress.txt, features-list, changelog, team-briefing |
| H. Spec Documents | 4 | PRD, feature specs, API specs, DB specs |
| I. Context Efficiency | 3 | No oversized docs, feature-level splitting |
| J. Testing | 3 | Framework, test files, documented commands |
| K. Agent Team | 5 | Team env, CLAUDE.md optimized, directory ownership, patterns, cost strategy |
| **Total** | **46** | **A(39+) B(31-38) C(23-30) D(14-22) F(0-13)** |

## Eval System

All skills come with pre-built test scenarios for benchmarking and quality measurement.

```
evals/
  {skill-name}/
    scenarios.json        # Test scenarios (3+ per skill)
    benchmark-*.json      # Benchmark results (auto-generated)
  dashboard-*.json        # Overall dashboard snapshots
```

### Quick Commands

| Command | What It Does |
|---------|-------------|
| "스킬 평가 code-polish" | Run eval on a single skill |
| "benchmark skill security-guard" | Benchmark with history tracking |
| "eval all" / "전체 평가" | Batch evaluate all skills |
| "스킬 대시보드" / "skill dashboard" | Show overall dashboard |
| "시나리오 생성 dev-start" | Auto-generate scenarios from SKILL.md |
| "A/B 비교 code-polish" | Compare two versions of a skill |

### Dashboard Example

```
## Skill Evaluation Dashboard

| Skill           | Scenarios | Pass Rate | Tokens | Trend |
|-----------------|-----------|-----------|--------|-------|
| spec-writer     | 3         | 100%      | 107K   | --    |
| code-polish     | 3         | 66.7%     | 3.4K   | UP    |
| security-guard  | 3         | 100%      | 2.8K   | NEW   |
| dev-start       | 3         | --        | --     | --    |
| ...             | ...       | ...       | ...    | ...   |
```

## Structure

```
skill-name/
  SKILL.md              # Main skill file (required)
  supporting-file.*     # Only if referenced from SKILL.md

evals/
  {skill-name}/
    scenarios.json      # Test scenarios
    benchmark-*.json    # Results history
```
