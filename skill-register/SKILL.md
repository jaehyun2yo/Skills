---
name: skill-register
description: Discovers created skills, registers to CLAUDE.md trigger table, creates symlinks to ~/.claude/skills/, verifies resolution. Triggers on "스킬 등록", "register skill", "스킬 연결", "skill register", "등록해줘".
---

# Skill Register

Scan created skills → Register to project → Update CLAUDE.md → Verify

Runs after skills are built. Ensures they are properly wired into the target project.

---

## Phase 1: Discover Skills

**1.1 Find skill sources**
```bash
# Check common skill locations
ls ~/.claude/skills/*/SKILL.md 2>/dev/null
ls .claude/skills/*/SKILL.md 2>/dev/null
```

**1.2 Parse each SKILL.md frontmatter**
Extract from each skill:
- `name` — skill identifier
- `description` — includes trigger keywords
- `disable-model-invocation` — if true, manual-only

**1.3 Detect existing registrations**
```bash
# Check what's already registered in CLAUDE.md
grep -i "skill\|trigger\|keyword" CLAUDE.md 2>/dev/null
```

Build a table: skill name | triggers | registered? | symlinked?

---

## Phase 2: Present Registration Plan

Show the user:

```
## 스킬 등록 현황

| 스킬 | 트리거 키워드 | 등록 상태 | 심링크 |
|------|--------------|----------|--------|
| plan-feature | plan, 계획, design | ❌ 미등록 | ❌ |
| progress-update | 마무리, wrap up | ❌ 미등록 | ✅ |
| ... | | | |

### 등록할 항목
1. CLAUDE.md 스킬 트리거 테이블 업데이트
2. ~/.claude/skills/ 심링크 생성 (없는 경우)
3. settings.json 스킬 참조 확인
```

Ask: **"위 스킬들을 등록할까요? 특정 스킬만 선택하려면 번호를 알려주세요."**

---

## Phase 3: Register

### 3.1 Update CLAUDE.md Skill Triggers Table

If CLAUDE.md has a `## Skill Triggers` section, **merge** new entries. If none exists, **add** the section.

Format:
```markdown
## Skill Triggers

| Keyword | Skill | Action |
|---------|-------|--------|
| plan, 계획, design | plan-feature | Create implementation plan, no coding |
| 마무리, wrap up, session end | progress-update | Record session progress |
| ... | ... | ... |
```

Rules:
- Preserve existing entries — only add new ones
- Sort by skill name for consistency
- Korean + English triggers for bilingual usage

### 3.2 Create Symlinks

```bash
# For each unlinked skill
ln -sf /absolute/path/to/skill-folder ~/.claude/skills/skill-name
```

- Use absolute paths for symlinks
- Skip if symlink already exists and points to correct target
- If target is wrong, ask user before overwriting

### 3.3 Project-local skills (.claude/skills/)

If skills are project-specific (not global):
```bash
# Copy to project .claude/skills/ instead of symlink to global
cp -r /path/to/skill .claude/skills/skill-name
```

Ask user: **"글로벌(~/.claude/skills)로 등록할까요, 프로젝트 로컬(.claude/skills)로 등록할까요?"**

---

## Phase 4: Verify Registration

```bash
# Verify symlinks resolve
ls -la ~/.claude/skills/ 2>/dev/null
ls -la .claude/skills/ 2>/dev/null

# Verify CLAUDE.md has trigger table
grep -A 20 "Skill Triggers" CLAUDE.md 2>/dev/null

# Verify each skill's SKILL.md is readable from registered path
for skill in ~/.claude/skills/*/SKILL.md; do head -5 "$skill" 2>/dev/null; done
```

### Verification Report

```
## 등록 완료

### 등록된 스킬
| 스킬 | 위치 | 트리거 등록 | 상태 |
|------|------|-----------|------|
| plan-feature | ~/.claude/skills/ | ✅ | 정상 |
| progress-update | .claude/skills/ | ✅ | 정상 |

### CLAUDE.md 변경 사항
- Skill Triggers 테이블: N개 항목 추가

### 다음 단계
- 스킬 트리거 키워드로 테스트해보세요 (예: "계획 세워줘")
- 필요시 /usage-guide 로 사용법 문서를 생성하세요
```

---

## Rules
1. **Never overwrite** existing CLAUDE.md content — merge only
2. **Ask before** creating symlinks or copying files
3. **Absolute paths** for all symlinks
4. **Skip duplicates** — don't re-register already registered skills
5. **Bilingual triggers** — Korean + English for each skill

---

## Context Rules
- Read only frontmatter of SKILL.md files (head -10)
- Never read supporting files during registration
- Total context < 15% of window
