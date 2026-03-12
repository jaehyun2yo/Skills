---
name: dev-wrap
description: Development session wrap-up — commits changes, cleans up agent teams, updates progress/features/changelog, prepares for next session handoff. Also auto-runs via Stop hook. Triggers on "마무리", "wrap up", "세션 정리", "오늘 여기까지", "커밋하고 끝", "세션 끝", "마무리하자".
---

# Dev Wrap — 개발 세션 마무리

세션 종료 전에 모든 변경을 정리하고 다음 세션을 위한 핸드오프를 준비한다.
활성 Agent Team이 있으면 안전하게 정리한다.

---

## Step 1: 변경 사항 + 팀 상태 파악

```bash
# 미커밋 변경 확인
git status --short 2>/dev/null
git diff --stat 2>/dev/null

# 이번 세션 커밋 확인
git log --oneline --since="4 hours ago" 2>/dev/null || git log --oneline -10 2>/dev/null

# 활성 팀 확인 (OMC 사용 시)
cat .omc/state/team-state.json 2>/dev/null | head -5 || echo "NO_ACTIVE_TEAM"
```

---

## Step 2: Agent Team 정리 (활성 팀이 있는 경우)

현재 세션에 활성 팀이 있으면 세션 종료 전에 정리:

### 2.1 팀원 작업 완료 확인
`Ctrl+T`(task list 토글)로 미완료 task가 있는지 체크.
(단축키는 Claude Code 버전에 따라 변경될 수 있음)

### 2.2 팀원 순차 종료 (Graceful Shutdown)
리드 세션에서 자연어로 각 팀원에게 종료 요청:
> "Ask [teammate-name] to shut down"

- 모든 팀원이 종료될 때까지 대기
- 팀원이 거부하면 사유 확인 후 사용자에게 보고

### 2.3 팀 삭제
리드 세션에서 자연어로 팀 정리 요청:
> "Clean up the team"

### 2.4 팀 세션 기록
팀으로 작업한 경우 progress.txt에 팀 정보 추가 기록:
```
- **팀 구성**: [팀원 수]명 ([역할1], [역할2], ...)
- **팀 작업 결과**: [완료된 task 수]/[전체 task 수] 완료
```

**사용자에게 확인**: **"활성 팀이 있습니다. 팀을 정리하고 마무리할까요?"**
- 사용자가 팀 유지를 원하면 팀 정리 건너뜀 (다음 세션에서 새로 spawn 필요 안내)

---

## Step 3: 미커밋 변경 커밋

미커밋 변경이 있으면:

1. `git diff` 로 변경 내용 파악
2. 변경 내용에 맞는 커밋 메시지 작성
3. 사용자에게 확인: **"이 내용으로 커밋할까요?"**
4. 승인 시 커밋

여러 주제의 변경이 섞여있으면 분리 커밋 제안.

---

## Step 4: 문서 업데이트

docs/ 구조가 있으면 자동 업데이트. 없으면 건너뜀.

### 4.1 progress.txt

```
### 세션 #NNN — YYYY-MM-DD
- **작업 내용**: [이번 세션에서 한 것 요약]
- **완료한 것**: [완료된 항목들]
- **다음에 할 것**: [이어서 해야 할 것]
- **이슈**: [발견된 문제, 없으면 "없음"]
- **관련 커밋**: [커밋 해시 나열]
- **팀 작업**: [팀 사용 시 — 팀원 수, 역할, 완료 task 수] (팀 미사용 시 생략)
```

기존 내용 위에 새 항목을 추가 (최신이 위로).

### 4.2 features-list.md

이번 세션에서 완료된 항목:
- `FAILING` → `PASSING` (완료 시)
- `FAILING` → `IN_PROGRESS` (작업 시작했지만 미완료)

### 4.3 CHANGELOG.md

변경이 사용자에게 의미 있는 수준이면 추가:
- **추가**: 새 기능
- **수정**: 버그 수정
- **변경**: 기존 동작 변경

---

## Step 5: 핸드오프 파일 생성

세션 종료 시 `.claude/handoff.md`를 자동 생성/갱신한다.
이 파일은 PreCompact 훅에 의해 자동으로도 생성되지만, 세션 마무리 시 최종 버전을 작성한다.

```markdown
# Session Handoff — YYYY-MM-DD

## 완료한 것
- [이번 세션에서 완료한 항목]

## 진행 중 (미완료)
- [ ] [미완료 작업 — 파일 경로, 줄 번호 등 구체적으로]

## 결정사항
- [이번 세션에서 내린 기술 결정과 이유]

## 실패한 접근법 (Lore)
- [시도했지만 실패한 방법과 이유 — 다음 세션이 같은 삽질을 반복하지 않도록]

## 다음 단계
1. [가장 중요한 다음 작업]
2. [그 다음]

## 팀 재생성 (Agent Team 사용 시)
[팀으로 작업했으면 재생성 프롬프트 포함]
```

핸드오프 파일은 **git에 커밋하지 않는다** — `.gitignore`에 `.claude/handoff.md` 추가 권장.
이 파일은 세션 간 로컬 상태 전달 용도.

---

## Step 6: 문서 커밋

문서 업데이트를 별도 커밋:
```
docs: update progress and features-list for session #NNN
```

---

## Step 7: 세션 종료 리포트

```
## 세션 마무리 완료

### 이번 세션 요약
- 커밋: N개
- 변경 파일: N개
- 완료 항목: [항목들]
- 팀 작업: [사용했으면 — N명 팀, M/T tasks 완료 | 미사용 시 생략]

### 다음 세션에서 할 것
1. [가장 중요한 다음 작업]
2. [그 다음]

### 다음 세션 팀 구성 추천 (해당 시)
→ [남은 작업이 복잡하면 추천 팀 구성 제안]
→ [예: "3명 병렬 구현팀 — frontend/backend/tests 분리"]

### 미해결 이슈
- [있으면 나열, 없으면 "없음"]
```

---

## Rules
1. **docs/ 없어도 동작** — 커밋만이라도 정리
2. **사용자 확인 후 커밋** — 자동 커밋 금지
3. **세션 번호 자동 증가** — progress.txt에서 마지막 번호 읽어서 +1
4. **날짜는 절대 날짜** (YYYY-MM-DD) 사용
5. **리포트는 간결하게** — 10줄 이내
6. **민감 파일 제외** — .env, credentials 등은 커밋 목록에서 경고
7. **팀 정리 순서** — 반드시 팀원 shutdown → 팀 cleanup 순서 (역순 시 오류)
8. **팀 상태 기록** — 팀으로 작업했으면 progress.txt에 팀 정보 포함
9. **다음 세션 팀 추천** — 남은 작업이 복잡하면 추천 구성 제안 (강제 아님)
10. **핸드오프 항상 생성** — 세션 종료 시 .claude/handoff.md를 반드시 생성/갱신
11. **Lore 필수 기록** — 실패한 접근법이 있으면 핸드오프에 반드시 포함
12. **핸드오프 ≠ 커밋** — .claude/handoff.md는 로컬 상태 파일, git에 커밋하지 않음
