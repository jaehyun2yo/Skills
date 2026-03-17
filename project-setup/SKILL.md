---
name: project-setup
description: Scores project setup (48-point checklist across 11 categories), recommends improvements tailored to tech stack, generates missing configs with user approval. Triggers on "setup", "셋업", "셋팅", "configure", "설정", "프로젝트 설정", "project setup", "초기 설정", "init setup".
---

# Project Setup Skill

Scan → Score → Report → User selects → Generate → Verify

Unlike `auditing-project-readiness` (diagnosis only), this skill **takes action** after approval.

---

## Phase 1: Project Scan

Gather metadata efficiently. Never read large files in full.

**1.1 Tech Stack Detection**
```bash
ls package.json Cargo.toml pyproject.toml go.mod build.gradle pom.xml Gemfile pubspec.yaml 2>/dev/null
head -80 package.json 2>/dev/null  # or equivalent manifest
```
Record: language, framework, test framework, build tool, package manager, database, deployment target.

**1.2 Setup Scan**
```bash
wc -l CLAUDE.md 2>/dev/null || echo "NO_CLAUDE_MD"
find .claude -type f 2>/dev/null | sort || echo "NO_CLAUDE_DIR"
find docs -type f -name "*.md" 2>/dev/null | wc -l
find . \( -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" \) | grep -v node_modules | wc -l
git log --oneline -5 2>/dev/null
```

**1.3 Development Methodology Check**
```bash
# Check for TDD/SDD rules (any naming convention)
cat .claude/rules/tdd-sdd.md 2>/dev/null || \
  grep -rl "TDD\|SDD\|test-first\|spec-first" .claude/rules/ 2>/dev/null || \
  echo "NO_TDD_SDD_RULES"
# Check CLAUDE.md for methodology section
grep -i "TDD\|SDD\|test-driven\|spec-driven" CLAUDE.md 2>/dev/null || echo "NO_METHODOLOGY_IN_CLAUDE_MD"
```
If TDD/SDD rules exist, score J4-J6 accordingly. If missing, flag as `[🔴 Critical]` recommendation.

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

**1.5 CLAUDE.md Analysis** (if exists)
Check: build/test commands, architecture, conventions, session protocol, context management, skill triggers, development methodology (TDD/SDD).

---

## Phase 2: Score

Use scoring criteria from → [checklist.md](checklist.md)

11 categories (A–K), 50 points total. Grade:

| Score | Grade |
|-------|-------|
| 43-50 | A — Production-ready |
| 34-42 | B — Good, minor gaps |
| 25-33 | C — Significant gaps |
| 15-24 | D — Many missing pieces |
| 0-14  | F — Minimal or no setup |

---

## Phase 3: Report

Present concise evaluation in Korean:
- Project name, detected tech stack, score/grade
- Category scores table (category | score | status emoji)
- Ranked recommendations: `[🔴 Critical]` → `[🟠 High]` → `[🟡 Medium]`
- E2E infrastructure gaps:
  - Missing E2E framework → `[🔴 Critical]`
  - Missing E2E directory structure (`e2e/`) → `[🟠 High]`
  - Missing E2E command in CLAUDE.md → `[🟠 High]`

Then ask: **"어떤 항목을 셋업할까요? 번호를 선택하거나 '전부'라고 해주세요."**

---

## Phase 4: Generate

For selected items, generate files using → [setup-templates.md](setup-templates.md)
Customize placeholders per detected stack using → [tech-profiles.md](tech-profiles.md)
For E2E infrastructure, use qa-tester auto-detection logic for platform/framework selection. Generate E2E scaffolding using the E2E Infrastructure Template in → [setup-templates.md](setup-templates.md) Section K.
project-setup defers to qa-tester SKILL.md conventions for framework-specific config defaults (e.g. screenshot, trace settings).

### Rules
1. **Language**: skills/agents/hooks/rules/commands → English. Business docs (progress, changelog, PRD) → Korean
2. **CLAUDE.md**: merge missing sections only, never overwrite existing content
3. **Size limits**: skills < 300 lines, CLAUDE.md < 200 lines, team-briefing < 150 lines, rules < 100 lines
4. **Approval**: show file path + summary before each creation, skip if declined
5. **Generate one category at a time**, confirm between categories

---

## Phase 5: Verify

```bash
find .claude -type f | sort
wc -l CLAUDE.md
```

Re-score, show before/after comparison, list created files, suggest next steps.

---

## Context Rules
- Never read files > 500 lines in full — use Grep
- Never read README.md in full — use `head -50`
- Load tech-profiles.md for detected stack section only
- Total context < 25% of window

---

## Supporting Files
- [checklist.md](checklist.md) — Scoring criteria (A–K, 48 items, includes E2E items J7-J8)
- [setup-templates.md](setup-templates.md) — File templates for all configurable items (includes Agent Team templates + E2E Infrastructure)
- [tech-profiles.md](tech-profiles.md) — Tech stack-specific placeholder values
