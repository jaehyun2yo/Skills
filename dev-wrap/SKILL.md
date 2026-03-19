---
name: dev-wrap
description: Session wrap-up — commits changes, shuts down Agent Teams, updates progress/features/changelog, generates handoff, pushes to GitHub. Triggers on "마무리", "끝", "done", "wrap up", "세션 정리", "오늘 여기까지", "커밋하고 끝", "정리해줘", "세션 끝", "마무리하자".
---

# Dev Wrap — 개발 세션 마무리

세션 종료 전에 모든 변경을 정리하고 다음 세션을 위한 핸드오프를 준비한다.
활성 Agent Team이 있으면 안전하게 정리하고, 모든 커밋을 GitHub에 push한다.

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

## Step 2: Agent Team 완료 판단 + 정리

현재 세션에 활성 팀이 있으면, 전체 완료 여부를 판단하고 정리한다.

### 2.1 전체 작업 완료 판단 (팀 리더 역할)

팀 리더는 다음 기준으로 **전체 완료**를 판단한다:

```bash
# 1. 전체 task 상태 확인
# TaskList로 모든 task의 status를 조회
# → 모든 task가 completed 상태인지 확인

# 2. 활성 팀원 확인
# → idle 상태가 아닌 팀원이 남아있는지 확인
```

**판단 기준:**
- 모든 task가 `completed` → **전체 완료 확정** → Step 2.2로 진행
- 미완료 task 존재 → 사용자에게 보고: **"[N]개 task가 미완료입니다. 마무리할까요, 계속 작업할까요?"**
  - 마무리 선택 → 미완료 task를 progress.txt에 "미완료" 기록 후 진행
  - 계속 선택 → dev-wrap 중단

### 2.2 팀원 순차 종료 (Graceful Shutdown)
각 팀원에게 종료 요청:
> SendMessage로 각 팀원에게 "작업이 모두 완료되었습니다. 종료해주세요." 전송

- 모든 팀원이 종료될 때까지 대기
- 팀원이 거부하면 사유 확인 후 사용자에게 보고

### 2.3 팀 삭제
TeamDelete로 팀 정리.

### 2.4 팀 세션 기록
팀으로 작업한 경우 progress.txt에 팀 정보 추가 기록:
```
- **팀 구성**: [팀원 수]명 ([역할1], [역할2], ...)
- **팀 작업 결과**: [완료된 task 수]/[전체 task 수] 완료
- **미완료 task**: [있으면 목록, 없으면 "없음"]
```

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

## Step 7: GitHub Push

모든 커밋 완료 후 원격 저장소에 push한다.

### 7.1 Push 전 확인

```bash
# 현재 브랜치와 리모트 상태 확인
git branch --show-current
git remote -v
git log --oneline @{upstream}..HEAD 2>/dev/null || echo "NO_UPSTREAM"
```

### 7.2 Push 실행

```bash
# upstream이 설정된 경우
git push

# upstream이 없는 경우 (새 브랜치)
git push -u origin $(git branch --show-current)
```

- push 실패 시 → 에러 내용을 사용자에게 보고 (force push 절대 금지)
- conflict 발생 시 → 사용자에게 보고하고 해결 방법 안내

### 7.3 Push 결과 확인

```bash
# push 성공 확인
git log --oneline -3
echo "✓ GitHub push 완료: $(git remote get-url origin) / $(git branch --show-current)"
```

---

## Step 8: 세션 종료 리포트

```
## 세션 마무리 완료

### 이번 세션 요약
- 커밋: N개
- 변경 파일: N개
- 완료 항목: [항목들]
- 팀 작업: [사용했으면 — N명 팀, M/T tasks 완료 | 미사용 시 생략]
- GitHub: [push 완료 — 브랜치명, 커밋 수]

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
13. **GitHub push 필수** — 모든 커밋 후 반드시 push (팀 작업 여부 무관)
14. **force push 금지** — push 실패 시 사용자에게 보고, 절대 --force 사용 금지
15. **팀 완료 자동 판단** — 팀 리더는 TaskList로 전체 task 상태를 확인하여 완료 여부 자동 판단
