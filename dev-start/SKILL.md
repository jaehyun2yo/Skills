---
name: dev-start
description: Development session initializer — reads progress, checks pending tasks, shows git state, detects active teams, suggests next work. Auto-triggers on session start or coding start keywords. Triggers on "개발 시작", "코딩 시작", "시작하자", "start coding", "start dev", "작업 시작", "뭐하고 있었지", "이어서 개발", "어디까지 했지".
---

# Dev Start — 개발 세션 시작

세션 시작 시 프로젝트 상태를 파악하고 다음 작업을 제안한다.
복잡한 작업이면 Agent Team 구성도 제안한다.

---

## Step 1: 프로젝트 상태 수집

```bash
# Git 상태
git status --short 2>/dev/null
git log --oneline -5 2>/dev/null
git branch --show-current 2>/dev/null

# 핸드오프 파일 (이전 세션에서 자동/수동 생성)
cat .claude/handoff.md 2>/dev/null || echo "NO_HANDOFF"

# 진행 기록 (있으면)
head -30 docs/progress.txt 2>/dev/null || echo "NO_PROGRESS"

# 미해결 항목 (있으면)
grep -A 1 "FAILING\|IN_PROGRESS\|미해결\|진행 중" docs/features-list.md 2>/dev/null || echo "NO_FEATURES_LIST"

# 미커밋 변경
git diff --stat 2>/dev/null

# 활성 팀 확인 (OMC 사용 시)
cat .omc/state/team-state.json 2>/dev/null | head -5 || echo "NO_ACTIVE_TEAM"
```

---

## Step 2: 핸드오프 기반 세션 복원

`.claude/handoff.md`가 존재하면 — 이전 세션이 자동 핸드오프를 남긴 것:

1. 핸드오프 파일 내용을 읽고 상태 리포트에 통합
2. **"이전 세션에서 핸드오프를 남겼습니다. 이어서 작업합니다."** 안내
3. 핸드오프의 "다음 단계" 항목을 추천 작업으로 사용
4. **Lore 섹션**(실패한 접근법)이 있으면 반드시 표시 — 같은 삽질 방지
5. 팀 재생성 프롬프트가 있으면 팀 구성 제안

핸드오프 파일이 없으면 → progress.txt 기반으로 폴백.

---

## Step 3: 상태 리포트

수집한 정보를 간결하게 정리해서 보여준다:

```
## 세션 시작 — [프로젝트명]

**브랜치**: main
**미커밋 변경**: 3 files
**최근 커밋**: [최근 3개 요약]
**핸드오프**: [있으면 "이전 세션 핸드오프 발견" + 요약, 없으면 생략]

### 이전 세션에서 이어받은 것 (핸드오프 또는 progress.txt)
[핸드오프 파일 요약 또는 progress.txt 최신 항목]

### 주의: 실패한 접근법 (Lore)
[핸드오프에 Lore 섹션이 있으면 표시 — "이 방법은 이미 시도했고 실패함"]

### 미해결 작업
- [🔴 우선순위 높음] ...
- [🟡 진행 중] ...

### 추천 다음 작업
→ [핸드오프의 "다음 단계" 또는 가장 우선순위 높은 항목]
```

---

## Step 4: 미커밋 변경 처리

미커밋 변경이 있으면 먼저 처리:

1. `git diff --stat` 으로 변경 파일 확인
2. 사용자에게 물어보기: **"미커밋 변경이 있습니다. 커밋할까요, 스태시할까요, 그대로 이어서 할까요?"**
3. 사용자 선택에 따라 처리

---

## Step 5: 작업 진입

사용자가 작업을 선택하면:
1. 관련 spec이 있으면 읽기 (있는 경우에만)
2. 관련 코드 위치 Grep으로 파악
3. 바로 코딩 시작하지 말고 간단한 계획 확인

사용자가 바로 코딩하겠다고 하면 즉시 시작.

---

## Step 6: 복잡한 작업 감지 → Agent Team 제안

다음 조건 중 2개 이상 해당하면 Agent Team 구성을 제안:
- 3개 이상 디렉토리/레이어에 걸친 변경 필요
- features-list.md에 미해결 항목이 5개 이상
- 사용자가 "리팩토링", "마이그레이션", "전체", "대규모" 등 키워드 사용
- PR 리뷰, 코드 리뷰, 보안 점검 등 다관점 분석 필요

### 제안 형식

```
### Agent Team 추천

이 작업은 여러 영역에 걸쳐있어 **Agent Team**으로 병렬 처리하면 효과적입니다.

**추천 구성**: [작업 유형에 따라 아래 패턴 중 선택]

#### 옵션 A: 병렬 구현팀 (기능 개발/리팩토링)
- Teammate 1 (Sonnet): [영역1] 담당 — src/[dir1]/ ONLY
- Teammate 2 (Sonnet): [영역2] 담당 — src/[dir2]/ ONLY
- Teammate 3 (Sonnet): 테스트 전담 — tests/ ONLY
- Lead (Opus): 조정 전용, delegate mode

#### 옵션 B: 다관점 리뷰팀 (코드 리뷰/점검)
- Teammate 1 (Sonnet): 보안 리뷰
- Teammate 2 (Sonnet): 성능 리뷰
- Teammate 3 (Sonnet): 테스트 커버리지 리뷰

#### 옵션 C: 가설 경쟁팀 (디버깅/조사)
- Teammate 1~N (Sonnet): 각각 다른 가설 조사
- 서로 반박하며 합의 도출

**"팀을 구성할까요? 아니면 단독으로 진행할까요?"**
```

### Team 구성 시 자동 적용되는 베스트 프랙티스

팀 생성 프롬프트에 다음을 항상 포함:
1. **Delegate mode**: `"Use delegate mode — only coordinate, don't implement yourself"`
2. **디렉토리 소유권**: 각 teammate에 `"ONLY touch files in [assigned dirs]"` 명시
3. **Plan approval**: `"Require plan approval before making any changes"`
4. **Sonnet 기본**: `"Use Sonnet for each teammate"` (비용 최적화)
5. **CLAUDE.md 참조**: teammate는 자동으로 CLAUDE.md를 로드하므로 spawn prompt에 중복 포함 금지

---

## Rules
- docs/progress.txt, docs/features-list.md가 없어도 동작 — git 상태만으로도 충분
- 파일이 없으면 "없음"으로 표시하고 넘어감, 생성을 강요하지 않음
- 전체 리포트는 20줄 이내로 압축
- 사용자가 이미 할 일을 말했으면 리포트 생략하고 바로 진행
- 활성 팀이 있으면 리포트에 포함하되, in-process teammate는 resume 불가함을 안내
- Agent Team 제안은 강제가 아님 — 사용자가 거부하면 즉시 단독 작업 진입
- 단순한 작업(1~2개 파일, 단일 기능)에는 팀 제안하지 않음
