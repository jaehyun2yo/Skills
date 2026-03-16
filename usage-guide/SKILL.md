---
name: usage-guide
description: Generates project-specific unified usage guide covering all skills, MCP plugins, hooks, agents, and commands as a single reference document. Triggers on "사용법", "usage guide", "가이드 작성", "매뉴얼", "how to use", "사용 문서".
---

# Usage Guide Generator

Scan project setup → Detect plugins/skills/hooks → Generate unified usage document

Creates a practical reference document so team members (and future sessions) know how to use everything configured in the project.

---

## Phase 1: Inventory

Scan all configured components. Minimize context — read only metadata.

**1.1 Skills**
```bash
# Global skills
ls ~/.claude/skills/*/SKILL.md 2>/dev/null | while read f; do head -5 "$f"; echo "---"; done

# Project skills
ls .claude/skills/*/SKILL.md 2>/dev/null | while read f; do head -5 "$f"; echo "---"; done
```
Record: name, trigger keywords, manual-only flag.

**1.2 MCP Plugins**
```bash
# Check Claude Code MCP config
cat ~/.claude/mcp_servers.json 2>/dev/null || cat ~/.claude.json 2>/dev/null
# Project-level MCP
cat .claude/mcp_servers.json 2>/dev/null
# Check settings for MCP references
cat .claude/settings.json 2>/dev/null
```
Record: server name, tools provided, purpose.

**1.3 Hooks**
```bash
# From settings.json — check all hook types including PreCompact
grep -A 5 "SessionStart\|Stop\|PostToolUse\|PreToolUse\|PreCompact" .claude/settings.json 2>/dev/null
```
Record: hook type, what it does, matchers (startup/compact/clear).

**1.4 Agents**
```bash
ls .claude/agents/*.md 2>/dev/null | while read f; do head -5 "$f"; echo "---"; done
```
Record: name, trigger, purpose.

**1.5 Commands**
```bash
ls .claude/commands/*.md 2>/dev/null | while read f; do echo "== $f =="; head -3 "$f"; done
```
Record: name, what it does.

**1.6 Agent Team setup**
```bash
# Check if teams are enabled
grep -l "AGENT_TEAMS" .claude/settings.json ~/.claude/settings.json 2>/dev/null
# Check for team patterns doc
ls docs/team-patterns.md 2>/dev/null
# Check for directory ownership in CLAUDE.md
grep -i "ownership\|directory.*owner\|team.*composition\|팀.*구성" CLAUDE.md 2>/dev/null
```
Record: teams enabled?, patterns documented?, directory ownership defined?

**1.7 Existing docs**
```bash
ls docs/*.md docs/*.txt 2>/dev/null
head -5 CLAUDE.md 2>/dev/null
```

---

## Phase 2: Analyze Interactions

Identify how components work together. Key questions:

1. **Plugin ↔ Skill overlap**: Does an MCP plugin provide functionality that a skill also covers? Document which to prefer and when.
2. **Hook → Skill chain**: Do hooks trigger skill-like behavior? (e.g., SessionStart hook reads progress.txt, which progress-update skill also manages)
3. **Agent ↔ Command relation**: Which agents are invoked by commands?
4. **Plugin-specific workflows**: How do MCP tools enhance the development flow? (e.g., Context7 for docs lookup, Serena for symbolic editing)
5. **Team ↔ Skill integration**: How do dev-start/dev-wrap skills interact with Agent Team lifecycle? (team detection → creation → monitoring → cleanup)

Build an interaction map (don't output yet — use for guide structure).

---

## Phase 3: Generate Usage Guide

Create `docs/usage-guide.md` in Korean. Structure:

```markdown
# {{PROJECT_NAME}} 사용 가이드

## 개요
이 문서는 프로젝트에 구성된 스킬, 플러그인, 훅, 에이전트, 커맨드의 사용법을 정리합니다.

---

## 빠른 시작
| 하고 싶은 것 | 방법 |
|-------------|------|
| 기능 계획 세우기 | "계획 세워줘" 또는 /plan-feature |
| 코드 리뷰 | "리뷰해줘" 또는 /code-reviewer |
| 진행 상태 확인 | /status |
| ... | ... |

---

## 스킬 (Skills)

### [skill-name]
- **트리거**: keyword1, keyword2, ...
- **용도**: 한 줄 설명
- **사용 예시**: "keyword를 포함한 자연어 요청"
- **주의사항**: (있으면)

(각 스킬별 반복)

---

## MCP 플러그인 (Plugins)

### [plugin-name]
- **제공 도구**: tool1, tool2, ...
- **용도**: 한 줄 설명
- **사용 시점**: 언제 이 플러그인이 자동/수동으로 사용되는지
- **스킬과의 관계**: (겹치는 기능이 있으면 어떤 걸 쓸지 가이드)

(각 플러그인별 반복)

---

## 훅 (Hooks)

### SessionStart
- **동작**: 세션 시작 시 자동 실행되는 내용
- **출력 정보**: 어떤 정보를 보여주는지

### Stop
- **동작**: 세션 종료 시 체크하는 항목

(각 훅별 반복)

---

## 에이전트 (Agents)

### [agent-name]
- **트리거**: keyword1, keyword2
- **용도**: 한 줄 설명
- **사용 예시**: 어떤 상황에서 호출하면 좋은지

---

## 커맨드 (Commands)

### /command-name
- **용도**: 한 줄 설명
- **사용법**: 슬래시 커맨드 형태

---

## 워크플로우 가이드

### 일반 개발 흐름 (단독)
(Phase 1에서 발견된 실제 스킬/에이전트/커맨드 기반으로 작성)
1. 세션 시작 → SessionStart 훅 자동 실행 (있으면)
2. 작업 계획 → 계획 스킬 사용 (등록되어 있으면)
3. 구현 → 코드 작성
4. 테스트/리뷰 → 등록된 테스트/리뷰 스킬 사용
5. 마무리 → Stop 훅 자동 실행 또는 "마무리" 키워드

### Agent Team 개발 흐름 (복잡한 작업)
(Agent Team이 활성화된 경우 — settings.json에 CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS: "1")

**언제 팀을 쓸까?**
| 상황 | 추천 방식 |
|------|---------|
| 1-2개 파일, 단순 변경 | 단독 작업 |
| 독립적 빠른 작업 | Subagent (Agent 도구) |
| 3+ 영역, 에이전트 간 소통 필요 | **Agent Team** |
| 다관점 분석 (리뷰, 디버깅) | **Agent Team** |

**팀 워크플로우**:
1. 세션 시작 → dev-start가 복잡도 감지 → 팀 구성 제안
2. 팀 생성 → 자연어로 요청 (예: "3명으로 팀 만들어줘")
3. 작업 분배 → Lead가 task 생성, 디렉토리 소유권 배정
4. 병렬 작업 → `Shift+Down`으로 팀원 모니터링, `Ctrl+T`로 task 확인
5. 마무리 → dev-wrap이 팀 정리 (shutdown → cleanup)

**팀 구성 패턴**:
- **병렬 구현**: Lead(조정) + N명(각 영역 담당, Sonnet)
- **다관점 리뷰**: Lead(종합) + 보안/성능/테스트 리뷰어(Sonnet)
- **경쟁 가설**: Lead(중재) + N명(각 가설 조사, 서로 반박)

**비용 인식**:
- 3명 팀 ≈ 3-4x, 5명 팀 ≈ 7x (단독 대비)
- Sonnet teammate 기본 사용으로 비용 절감
- 유휴 teammate 즉시 shutdown

### 버그 수정 흐름
(Phase 1에서 발견된 디버깅/분석 도구 기반으로 작성)
- 단순 버그: 단독 디버깅
- 복잡한 버그: Agent Team — 경쟁 가설 패턴 활용
  - "각 팀원이 다른 가설 조사, 서로 반박하며 합의 도출"

### 세션 관리 & 컨텍스트 관리

**자동 핸드오프 시스템** (훅 기반 — 키워드 입력 불필요):
```
[코딩 중...]
  ↓ 컨텍스트 임계점 도달
  ↓ PreCompact 훅 자동 발동 → .claude/handoff.md 저장
  ↓ /clear 입력 (이것만 직접)
  ↓ SessionStart 훅 자동 발동 → handoff.md 읽어서 주입
  ↓ 끊김 없이 이어서 작업
```

**컨텍스트 용량별 대응**:
- 0-50%: 자유롭게 작업
- 50-70%: Grep만 사용, 전체 파일 읽기 금지
- 70-80%: `/compact` 수동 실행 (95% 자동 압축보다 효과적)
- 90%+: `/clear` → 핸드오프 자동 복원

**핸드오프 파일에 포함되는 것**:
- 완료한 것 / 진행 중인 것 (파일 경로, 줄 번호 포함)
- 결정사항과 이유
- **Lore (실패한 접근법)** — 같은 삽질 방지의 핵심
- 다음 단계 (우선순위 순)
- Agent Team 사용 시: 팀 구성 + 재생성 프롬프트

**팀 작업 중 세션 관리**:
- 팀원도 개별 컨텍스트를 가지므로 리드 컨텍스트가 꽉 차도 팀원은 계속 작업 가능
- 리드의 핸드오프에 팀 상태 포함 → /clear 후 팀 재구성 가능

---

## Agent Team 활용 팁

(Agent Team이 설정되어 있는 경우에만 이 섹션 포함)

### 팀 구성 핵심 원칙
1. **리드 = 매니저**: `"Use delegate mode"` — 리드가 직접 코딩하면 조정 품질 저하
2. **디렉토리 소유권**: 각 teammate에 `"ONLY touch files in [dirs]"` — 파일 충돌 원천 차단
3. **Plan approval**: 코드 작성 전 계획 승인 — 잘못된 방향 조기 차단
4. **CLAUDE.md 최적화**: 500줄 이내 유지 — "3명이 좋은 CLAUDE.md 읽는 게 3명이 코드베이스 탐색하는 것보다 훨씬 저렴"
5. **spawn prompt에 context 포함**: 팀원은 리드 대화 이력 미상속 — 필요한 배경 모두 포함

### 팀 상호작용 (현재 알려진 단축키 — 버전에 따라 변경될 수 있음)
- `Shift+Down`: 다음 팀원으로 전환
- `Ctrl+T`: Task list 토글
- `Escape`: 팀원 세션 → 리드 복귀
- Split pane 모드: tmux/iTerm2 필요 (각 팀원 별도 pane)

### 세션 관리
- 팀원은 `/resume` 후 복원 불가 → 새로 spawn 필요
- 세션 종료 전 반드시 팀 정리 (dev-wrap이 자동 처리)
- progress.txt에 팀 작업 결과 기록 → 다음 세션 핸드오프
- PreCompact 훅이 팀 상태를 포함한 handoff.md 자동 생성
- /clear 후 SessionStart 훅이 handoff.md를 자동 로드 → 팀 재구성 프롬프트 포함

---

## 플러그인 활용 팁

(MCP 플러그인별 실전 사용 팁 — Phase 2에서 분석한 내용 기반)

---

## 참고
- CLAUDE.md: 프로젝트 규칙 및 컨벤션
- docs/progress.txt: 세션 간 핸드오프
- docs/features-list.md: 기능/버그 추적
- docs/team-patterns.md: Agent Team 구성 패턴 및 프롬프트 (있는 경우)
```

---

## Phase 4: Review & Finalize

1. Show generated guide summary to user (section titles + item counts)
2. Ask: **"가이드를 생성할까요? 추가하거나 빼고 싶은 섹션이 있으면 알려주세요."**
3. Write `docs/usage-guide.md` after approval
4. Update CLAUDE.md if needed — add reference to usage guide

---

## Rules
1. **Korean** for the guide document — it's for human consumption
2. **Detect, don't assume** — only document what actually exists in the project
3. **Plugin-aware** — check MCP configs and document actual plugin tools, not generic ones
4. **No duplication** — if CLAUDE.md already documents something well, reference it instead of repeating
5. **Practical examples** — each item gets a concrete usage example
6. **Keep it scannable** — tables for quick reference, details below

---

## Context Rules
- Read only frontmatter/headers from skill files
- Read MCP config files in full (they're small)
- Never read implementation code
- Total context < 20% of window
