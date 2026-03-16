---
name: team-dispatcher
description: Analyzes user request to select the right Agent Team (bugfix/review/refactor/performance/feature), dispatches to correct team skill. Triggers on "team dispatch", "팀 디스패치", "팀 선택", "어떤 팀".
---

# Team Dispatcher

사용자의 지시를 분석 → 작업 유형과 도메인 판별 → 적절한 팀 스킬 자동 호출.

---

## Phase 1: Request Analysis

사용자의 지시에서 두 가지를 판별합니다:

### 1.1 작업 유형 판별

| Signal | Type | Team Pattern |
|--------|------|-------------|
| "버그", "고쳐", "fix", "에러", "안됨", "오류", "깨짐" | **bugfix** | 조사 → 수정 → 검증 |
| "리뷰", "review", "검토", "점검", "확인해줘" | **review** | 보안 + 품질 + 성능 병렬 리뷰 |
| "리팩토링", "refactor", "정리", "개선", "기술부채" | **refactor** | 행위 보존 리팩토링 |
| "느려", "성능", "최적화", "performance", "속도" | **performance** | 분석 → 최적화 → 벤치마크 |
| "추가", "만들어", "구현", "개발", "기능", "feat", "새로" | **feature** | 도메인별 개발팀 |

### 1.2 도메인 판별 (feature 유형일 때)

> **이 테이블은 `team-setup` 스킬이 프로젝트 분석 후 자동 생성합니다.**
> 아래는 참고용 예시입니다. 실제 프로젝트에서는 team-setup이 생성한 테이블로 대체됩니다.

| Signal | Domain | Team Skill |
|--------|--------|-----------|
| {domain_1_keywords} | {domain_1} | team-{domain_1} |
| {domain_2_keywords} | {domain_2} | team-{domain_2} |
| ... | ... | ... |

---

## Phase 2: Team Selection

### Decision Tree

```
사용자 지시
  ├── 버그/에러 키워드? → team-bugfix
  ├── 리뷰/검토 키워드? → team-review
  ├── 리팩토링/정리 키워드? → team-refactor
  ├── 성능/최적화 키워드? → team-performance
  ├── 기능 개발 키워드?
  │     ├── 도메인 식별 가능? → 해당 도메인 team-{domain}
  │     └── 도메인 불명확? → Phase 3으로 (사용자에게 질문)
  └── 키워드 불명확? → Phase 3으로 (사용자에게 질문)
```

### Multi-Domain Detection

하나의 지시가 여러 도메인에 걸칠 때:
- 도메인 2개 이하: 더 넓은 범위의 팀 선택 (예: admin + backend → team-admin)
- 도메인 3개 이상: 사용자에게 분할 제안

### Complexity Check

팀이 필요 없는 단순 작업인지 판단:
- 파일 1-2개, 변경 50줄 이하 예상 → 팀 없이 직접 작업 제안
- 파일 3개+, 다중 레이어 → 팀 소집

---

## Phase 3: Confirmation & Dispatch

### 3.1 선택 결과 요약

사용자에게 간단히 보여줍니다:

```
📋 분석 결과:
- 작업 유형: {type}
- 도메인: {domain}
- 추천 팀: {team_skill} ({member_count}명)
  - {role_1} (opus/sonnet)
  - {role_2} (sonnet)
  - ...

이 팀으로 진행할까요?
```

### 3.2 사용자 확인 후 호출

사용자가 확인하면:
```
Skill(skill="{selected_team_skill}")
```

사용자가 조정 요청하면 반영 후 재확인.

---

## Phase 4: Task Handoff

팀 스킬 호출 시 사용자의 원래 지시를 `{user_task}`로 전달합니다.
팀 스킬의 Step 4에서 팀장에게 SendMessage로 전달됩니다.

---

## Rules

- 항상 사용자에게 팀 선택 결과를 보여주고 확인을 받습니다
- 단순 작업에 팀을 소집하지 않습니다 (오버킬 방지)
- 도메인이 불명확하면 추측하지 말고 질문합니다
- 프로젝트에 해당 팀 스킬이 없으면 안내합니다
- 한 번에 하나의 팀만 소집합니다
