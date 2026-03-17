# Setup Scoring Checklist

## Scoring: ✅ = full point, ⚠️ = half point, ❌ = 0 points

---

## A. CLAUDE.md Quality (8 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| A1 | Exists, < 200 lines | File exists, under 200 lines | Exists but > 200 lines | No CLAUDE.md |
| A2 | Build/test commands | All commands (dev, build, test, lint, typecheck) | Some commands | No commands |
| A3 | Architecture overview | Tech stack + directory structure + key patterns | Only tech stack | None |
| A4 | Coding conventions | Project-specific rules (3+) | Generic rules only | None |
| A5 | Session protocol | Start + end + stuck + Plan Mode + /insights | Partial (1-2 of 3) | None |
| A6 | Context management | Proactive /compact at 60-70% + compact instructions + large file warnings | Thresholds but no compact instructions | None |
| A7 | Skill triggers table | Table mapping keywords to skills | Skills mentioned, no table | None |
| A8 | Extended thinking guide | think/ultrathink usage table with cost awareness | Mentioned but no guide | None |

---

## B. Hooks (8 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| B1 | SessionStart hook | Shows progress + git + pending items | Shows minimal info | None |
| B2 | Stop hook | Checks progress + commits + handoff + spec sync | Checks 1-2 items | None |
| B3 | PostToolUse hook | Auto-format after edits (Prettier/ruff/gofmt) | Partial (format only) | None |
| B4 | Hook quality | English, script-based for complex logic | Inline + Korean | Broken or missing |
| B5 | PreCompact hook | Auto-generates .claude/handoff.md before compaction | Mentioned but no implementation | None |
| B6 | SessionStart matchers | Separate matchers for startup/compact/clear with handoff auto-load | Single matcher, no handoff load | None |
| B7 | Async Hook usage | `async: true` on heavy analysis hooks (Stop/PostToolUse) | Mentioned but not applied | None |
| B8 | HTTP Hook configuration | Remote validation/policy server connected | URL configured but untested | None |
| B9 | MCP Tool Matching Hook | `mcp__*` pattern in PreToolUse/PostToolUse matchers | Pattern exists but no logic | None |
| B10 | PreToolUse security gate | Blocks dangerous commands (rm -rf, DROP TABLE, force push) | Partial pattern list | None |

---

## C. Skills (5 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| C1 | Planning skill | "No code" rule + plan format + approval gate | Plan exists, no enforcement | None |
| C2 | Progress/handoff skill | Updates progress + features-list + changelog | Partial update | None |
| C3 | Context management skill | Thresholds + stuck detection + recovery | Mentioned, no protocol | None |
| C4 | Test/quality skill | TDD cycle + refactor + conventions check | Testing only, no refactor | None |
| C5 | Skills in English + match tech stack | All English, correct framework | Mixed language or generic | Wrong framework or Korean |

---

## D. Agents (3 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| D1 | Code reviewer agent | Project conventions + memory + specific checks | Generic reviewer | None |
| D2 | Bug analyzer agent | Hypotheses + data flow + project bug patterns | Generic analyzer | None |
| D3 | Domain specialist agent | Project-domain expertise + architecture knowledge | Partially relevant | None |

---

## E. Commands (3 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| E1 | Status command | Shows git + progress + pending count | Partial info | None |
| E2 | Plan command | Reads features-list + creates plan | Partial workflow | None |
| E3 | Quality check command | Typecheck + lint + test pipeline | 1-2 checks only | None |

---

## F. Rules (2 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| F1 | Context management rules | Thresholds + large file list + session switch protocol | Partial | None |
| F2 | Spec-code sync rules | Code→spec update + plan-first + changelog | Partial | None |

---

## G. Operational Documents (5 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| G1 | progress.txt | Structured session log (date, done, next, issues) | Unstructured notes | None |
| G2 | features-list.md | Items by status (FAILING/IN_PROGRESS/PASSING) | TODO list without status | None |
| G3 | CHANGELOG.md | Dated entries with Added/Fixed/Changed | Sparse or outdated | None |
| G4 | team-briefing.md | Compact (< 150 lines), English, has conventions | Exists but too long or missing key info | None |
| G5 | Work logs or ADR | Change records with decisions documented | Some records | None |

---

## H. Spec Documents (4 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| H1 | PRD or requirements | Feature list with status tracking | Requirements without status | None |
| H2 | Feature specs | Per-feature with completion criteria + code refs | Some docs, missing criteria | None |
| H3 | API specs | Endpoints with request/response/auth/errors | Partial documentation | None |
| H4 | DB specs | Tables/columns matching actual schema | Exists but outdated | None |

---

## I. Context Efficiency (3 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| I1 | No oversized required docs (> 50KB) | None, or properly split | 1-2 oversized | Multiple oversized |
| I2 | Feature-level docs (not monolithic) | Per-feature specs exist | Mix of monolithic + feature | Only monolithic |
| I3 | Large file warnings in CLAUDE.md | Explicit list of files to avoid | General advice | None |

---

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

---

## K. Agent Team Readiness (5 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| K1 | Team env enabled | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS: "1"` in settings.json | Mentioned but not configured | Not set |
| K2 | CLAUDE.md team-optimized | <= 500 lines, has build/test/conventions (teammates auto-load this — brevity saves tokens per teammate) | 501-1000 lines or missing key info | No CLAUDE.md or > 1000 lines |
| K3 | Directory ownership rules | Explicit directory-to-role mapping documented | Partial or vague file boundaries | None |
| K4 | Team composition patterns | Documented patterns for common scenarios (review, debug, implement) | Some patterns mentioned | None |
| K5 | Team cost strategy | Model routing documented (Opus lead, Sonnet teammates) + cleanup protocol | Partial (model mentioned, no protocol) | None |
