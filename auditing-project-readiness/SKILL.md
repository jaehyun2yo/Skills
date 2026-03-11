---
name: auditing-project-readiness
description: Use when starting a new project, onboarding to an existing codebase, or when development workflow feels stuck or inefficient. Triggers on "audit", "진단", "프로젝트 분석", "project health", "what's missing", "뭐가 부족", "개선", "improve setup".
---

# Auditing Project Readiness

## Purpose
Scan an existing project and produce a structured diagnostic report identifying:
- Missing or misconfigured Claude Code infrastructure (.claude/, CLAUDE.md, hooks, skills, agents)
- Missing spec documents (PRD, feature specs, API specs, DB specs)
- Missing operational documents (progress tracking, session handoff, changelog)
- Context efficiency issues (oversized files, missing compact strategies)
- Test and quality gaps
- Agent team readiness

Output: A markdown report at `docs/PROJECT_AUDIT.md` with findings + actionable fix templates.

---

## Execution Procedure

### Phase 1: Project Discovery (Read-only, minimal context)

Gather project metadata efficiently. Do NOT read large files.

```
Step 1.1 — Project root scan
- List top-level files and directories (depth 2)
- Identify: package.json, CLAUDE.md, README.md, .claude/, docs/, tests/, e2e/
- Record tech stack from package.json (dependencies, devDependencies)

Step 1.2 — .claude/ directory scan
- List all files in .claude/ recursively
- Categorize: settings.json, settings.local.json, agents/, skills/, commands/, rules/
- Count items in each category

Step 1.3 — Documentation scan
- List all .md files in docs/ (depth 2)
- Calculate total size of each doc file
- Flag files > 30KB as "context-heavy"
- Check for: PRD, SRS, SDS, feature specs, API specs, DB specs, progress, changelog

Step 1.4 — Git metadata
- `git log --oneline -1` — last commit
- `git log --oneline | wc -l` — total commits
- `git log --format='%H' --diff-filter=A -- '*.test.*' '*.spec.*' | wc -l` — test file commits

Step 1.5 — CLAUDE.md analysis (if exists)
- Read CLAUDE.md (should be < 200 lines)
- Check for: commands, architecture, conventions, workflow protocol, session protocol, skill triggers
```

### Phase 2: Diagnostic Checklist

Score each item: ✅ Present, ⚠️ Partial, ❌ Missing

#### Category A: Claude Code Infrastructure
```
A1. CLAUDE.md exists and is < 200 lines
A2. CLAUDE.md contains build/test commands
A3. CLAUDE.md contains architecture overview
A4. CLAUDE.md contains coding conventions
A5. CLAUDE.md contains session start/end protocol
A6. CLAUDE.md contains context management rules
A7. CLAUDE.md contains skill trigger table
A8. .claude/settings.json exists with hooks
A9. SessionStart hook configured
A10. Stop hook configured
A11. settings.local.json is clean (no one-off commands > 50 entries)
```

#### Category B: Skills & Agents
```
B1. At least 1 planning skill exists
B2. At least 1 progress/handoff skill exists
B3. At least 1 context management skill exists
B4. Skills match project tech stack (not generic/wrong framework)
B5. At least 1 code review agent defined
B6. At least 1 bug analysis agent defined
B7. Agents reference project-specific conventions
B8. Slash commands exist for common workflows
```

#### Category C: Spec Documents
```
C1. PRD or product requirements document exists
C2. Feature specs exist (docs/specs/features/ or equivalent)
C3. API specs exist (docs/specs/api/ or equivalent)
C4. DB schema docs exist (docs/specs/db/ or equivalent)
C5. Specs include completion criteria / test scenarios
C6. Specs include related code file references
C7. Specs are < 500 lines each (context-friendly)
```

#### Category D: Operational Documents
```
D1. Progress tracking file exists (progress.txt or equivalent)
D2. Feature/bug tracking file exists (features-list.md or equivalent)
D3. Changelog exists (CHANGELOG.md)
D4. Architecture Decision Records exist (ADR)
D5. Agent team briefing document exists (< 200 lines)
```

#### Category E: Context Efficiency
```
E1. No single doc file > 50KB that agents routinely need
E2. README.md is not the only project context source
E3. Feature-level docs exist (not just monolithic architecture docs)
E4. CLAUDE.md warns against reading oversized files
E5. Compact/clear strategy documented
```

#### Category F: Testing & Quality
```
F1. Test framework configured (Jest, Vitest, Playwright, etc.)
F2. Test directory exists with actual test files
F3. E2E test setup exists
F4. Test commands documented in CLAUDE.md
F5. CI/CD pipeline exists (.github/workflows/ or equivalent)
```

### Phase 3: Report Generation

Generate `docs/PROJECT_AUDIT.md` using the template in `audit-report-template.md`.

For each ❌ item:
- Explain WHY it matters (1-2 sentences)
- Provide a READY-TO-USE file template or code snippet
- Specify exact file path where it should be created

For each ⚠️ item:
- Explain what's partial
- Suggest specific improvement

### Phase 4: Priority Ranking

Rank all ❌ items by impact:

**Priority 1 (Session handoff — fixes "getting stuck mid-project")**
- D1: progress.txt
- D2: features-list.md
- A8/A9/A10: Hooks

**Priority 2 (Spec-driven workflow — fixes "code gets messy")**
- C1: PRD
- C2: Feature specs
- B1: Planning skill

**Priority 3 (Agent effectiveness — fixes "tokens wasted, no result")**
- B5/B6: Review and bug agents
- D5: Team briefing
- E1-E4: Context efficiency

**Priority 4 (Quality automation)**
- F1-F5: Testing setup
- B3: Context management skill

### Phase 5: Execution Plan

Generate a phased execution plan:
- Phase A (1 hour): Critical operational docs + hooks
- Phase B (1 hour): Skills + agents + commands
- Phase C (ongoing): Spec documents, one at a time per feature worked on

---

## Output Requirements

1. Report language: **한국어** (Korean) for findings and explanations — this is a human-review document
2. File templates inside the report: **English** for skills/agents/hooks/rules, **Korean** for business docs
3. Report location: `docs/PROJECT_AUDIT.md`
4. Report should be self-contained — reader can follow it without other documents
5. Include a summary score: "X/Y items passing" per category
6. Include copy-paste-ready file contents for all missing items

---

## Context Efficiency Rules

- Do NOT read README.md in full (use head -50 for overview)
- Do NOT read any file > 500 lines (use wc -l to check first)
- Use `find`, `grep`, `wc` to gather metadata — not `cat` on large files
- Total context used by this skill should be < 30% of window
- If project is very large, scan only the most critical directories first

---

## Supporting Files
- [audit-report-template.md](audit-report-template.md) — Report output template
- [checklist-reference.md](checklist-reference.md) — Detailed checklist with scoring criteria
