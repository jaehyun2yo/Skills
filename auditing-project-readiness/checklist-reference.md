# Audit Checklist Reference

Detailed scoring criteria for each checklist item.

## Scoring

- **Present**: Fully satisfies the requirement
- **Partial**: Exists but incomplete or misconfigured
- **Missing**: Does not exist

---

## A. Claude Code Infrastructure

| ID | Item | Present | Partial | Missing |
|----|------|---------|---------|---------|
| A1 | CLAUDE.md exists, < 200 lines | File exists and under limit | File exists but over 200 lines | No CLAUDE.md |
| A2 | Build/test commands in CLAUDE.md | Commands section with working commands | Commands exist but incomplete | No commands |
| A3 | Architecture overview | Clear architecture section | Mentioned but vague | Not documented |
| A4 | Coding conventions | Explicit style rules | Some conventions implied | Not documented |
| A5 | Session start/end protocol | Both protocols defined | Only one defined | Neither defined |
| A6 | Context management rules | Explicit rules for large files | Partial guidance | Not documented |
| A7 | Skill trigger table | Table with keywords mapped to skills | Some triggers mentioned | No trigger table |
| A8 | settings.json with hooks | File exists with configured hooks | File exists, no hooks | No settings.json |
| A9 | SessionStart hook | Hook configured and working | Hook exists but broken | No hook |
| A10 | Stop hook | Hook configured and working | Hook exists but broken | No hook |
| A11 | settings.local.json clean | < 50 entries or doesn't exist | 50-100 entries | > 100 entries |

## B. Skills & Agents

| ID | Item | Present | Partial | Missing |
|----|------|---------|---------|---------|
| B1 | Planning skill | Skill with clear planning workflow | Generic planning notes | No planning skill |
| B2 | Progress/handoff skill | Skill for session continuity | Manual handoff notes | No handoff skill |
| B3 | Context management skill | Skill for managing context window | Ad-hoc rules only | No skill |
| B4 | Stack-matched skills | Skills match actual tech stack | Some mismatch | Wrong framework skills |
| B5 | Code review agent | Agent with review instructions | Generic review prompt | No review agent |
| B6 | Bug analysis agent | Agent for debugging workflow | Generic debug prompt | No bug agent |
| B7 | Project-specific agents | Agents reference project conventions | Generic agents only | No agents |
| B8 | Slash commands | Commands for common workflows | Few commands | No commands |

## C. Spec Documents

| ID | Item | Present | Partial | Missing |
|----|------|---------|---------|---------|
| C1 | PRD | Requirements doc with scope | Informal requirements | No PRD |
| C2 | Feature specs | Per-feature spec files | Monolithic spec | No feature specs |
| C3 | API specs | API endpoints documented | Partial API docs | No API specs |
| C4 | DB schema docs | Schema with relationships | Partial schema | No DB docs |
| C5 | Completion criteria | Test scenarios per spec | Some criteria | No criteria |
| C6 | Code file references | Specs link to source files | Some references | No references |
| C7 | Spec size < 500 lines | All specs under limit | Some over limit | Most over limit |

## D. Operational Documents

| ID | Item | Present | Partial | Missing |
|----|------|---------|---------|---------|
| D1 | Progress tracking | Active progress file | Stale progress file | No tracking |
| D2 | Feature/bug tracking | Active tracking file | Stale tracking | No tracking |
| D3 | Changelog | Up-to-date changelog | Outdated changelog | No changelog |
| D4 | ADRs | Decision records exist | Informal decisions logged | No ADRs |
| D5 | Team briefing doc | < 200 line team brief | Oversized or stale | No briefing |

## E. Context Efficiency

| ID | Item | Present | Partial | Missing |
|----|------|---------|---------|---------|
| E1 | No oversized docs (> 50KB) | All docs under limit | 1-2 files over | Multiple files over |
| E2 | Multiple context sources | CLAUDE.md + docs | README + minimal docs | README only |
| E3 | Feature-level docs | Per-feature documentation | Some feature docs | Monolithic only |
| E4 | Oversize file warnings | CLAUDE.md warns about large files | Implicit guidance | No warnings |
| E5 | Compact strategy | Documented compact approach | Ad-hoc approach | No strategy |

## F. Testing & Quality

| ID | Item | Present | Partial | Missing |
|----|------|---------|---------|---------|
| F1 | Test framework | Configured and working | Configured but issues | No framework |
| F2 | Test files | Meaningful test coverage | Few test files | No tests |
| F3 | E2E tests | E2E setup with scenarios | E2E configured, no tests | No E2E |
| F4 | Test commands in CLAUDE.md | All test commands documented | Some commands | No commands |
| F5 | CI/CD pipeline | Working pipeline | Pipeline with issues | No pipeline |
