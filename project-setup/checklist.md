# Setup Scoring Checklist

## Scoring: ✅ = full point, ⚠️ = half point, ❌ = 0 points

---

## A. CLAUDE.md Quality (7 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| A1 | Exists, < 200 lines | File exists, under 200 lines | Exists but > 200 lines | No CLAUDE.md |
| A2 | Build/test commands | All commands (dev, build, test, lint, typecheck) | Some commands | No commands |
| A3 | Architecture overview | Tech stack + directory structure + key patterns | Only tech stack | None |
| A4 | Coding conventions | Project-specific rules (3+) | Generic rules only | None |
| A5 | Session protocol | Start + end + stuck protocol | Partial (1-2 of 3) | None |
| A6 | Context management | Thresholds + large file warnings | Mentioned but vague | None |
| A7 | Skill triggers table | Table mapping keywords to skills | Skills mentioned, no table | None |

---

## B. Hooks (4 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| B1 | SessionStart hook | Shows progress + git + pending items | Shows minimal info | None |
| B2 | Stop hook | Checks progress + commits + spec sync | Checks 1-2 items | None |
| B3 | PostToolUse hook (optional) | Typecheck or format after edits | Partial | None (0.5 max) |
| B4 | Hook quality | English, script-based for complex logic | Inline + Korean | Broken or missing |

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

## J. Testing (3 points)

| # | Item | ✅ | ⚠️ | ❌ |
|---|------|----|----|-----|
| J1 | Test framework configured | Config file + project-specific settings | Default config | None |
| J2 | Test files exist | > 10 test files with actual cases | 1-10 files | None |
| J3 | Test commands documented | In CLAUDE.md with single-file pattern | Basic command only | Not documented |
