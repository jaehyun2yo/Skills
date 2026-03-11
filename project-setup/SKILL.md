---
name: project-setup
description: Evaluates current Claude Code project setup, scores it, recommends improvements tailored to the project's tech stack and needs, and creates missing files with user approval. Use when starting a new project, onboarding to existing code, or when the setup feels incomplete. Triggers on "setup", "셋업", "셋팅", "configure", "설정", "프로젝트 설정", "project setup", "초기 설정", "init setup".
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

**1.3 CLAUDE.md Analysis** (if exists)
Check: build/test commands, architecture, conventions, session protocol, context management, skill triggers.

---

## Phase 2: Score

Use scoring criteria from → [checklist.md](checklist.md)

10 categories (A–J), 39 points total. Grade:

| Score | Grade |
|-------|-------|
| 33-39 | A — Production-ready |
| 26-32 | B — Good, minor gaps |
| 19-25 | C — Significant gaps |
| 12-18 | D — Many missing pieces |
| 0-11  | F — Minimal or no setup |

---

## Phase 3: Report

Present concise evaluation in Korean:
- Project name, detected tech stack, score/grade
- Category scores table (category | score | status emoji)
- Ranked recommendations: `[🔴 Critical]` → `[🟠 High]` → `[🟡 Medium]`

Then ask: **"어떤 항목을 셋업할까요? 번호를 선택하거나 '전부'라고 해주세요."**

---

## Phase 4: Generate

For selected items, generate files using → [setup-templates.md](setup-templates.md)
Customize placeholders per detected stack using → [tech-profiles.md](tech-profiles.md)

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
- [checklist.md](checklist.md) — Scoring criteria (A–J, 39 items)
- [setup-templates.md](setup-templates.md) — File templates for all configurable items
- [tech-profiles.md](tech-profiles.md) — Tech stack-specific placeholder values
