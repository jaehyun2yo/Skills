---
name: skill-eval
description: Run evals, benchmarks, A/B comparisons, auto-generate scenarios, and batch-evaluate all skills — measures pass rate, token usage, and execution time per scenario. Triggers on "스킬 평가", "eval skill", "benchmark skill", "스킬 테스트", "skill eval", "스킬 벤치마크", "eval all", "전체 평가", "스킬 대시보드", "skill dashboard".
---

# Skill Eval — 스킬 품질 측정 · 벤치마크 · A/B 비교 · 일괄 평가

스킬의 품질을 정량적으로 측정하고, 모델 업데이트나 스킬 수정 후 성능 변화를 추적한다.

---

## Mode Detection

사용자 입력에서 모드를 판별한다:

| Signal | Mode |
|--------|------|
| "eval", "평가", "테스트" + 스킬명 | Mode 1: Eval |
| "benchmark", "벤치마크" + 스킬명 | Mode 2: Benchmark |
| "compare", "비교", "A/B" | Mode 3: Compare |
| "auto-gen", "시나리오 생성", "시나리오 만들어" | Mode 4: Auto-Generate |
| "eval all", "전체 평가", "일괄", "batch" | Mode 5: Batch Eval |
| "dashboard", "대시보드", "현황" | Mode 6: Dashboard |

---

## Mode 1: Eval (단일 스킬 테스트 실행)

### Step 1: 대상 스킬 확인

```bash
# 사용자가 지정한 스킬 또는 전체 스킬 목록
ls -d */SKILL.md 2>/dev/null | sed 's|/SKILL.md||'
```

사용자에게 확인:
- 어떤 스킬을 평가할지
- 테스트 시나리오가 이미 있는지, 새로 만들지

### Step 2: 테스트 시나리오 로드

시나리오 파일 검색 순서:
1. `evals/{skill-name}/scenarios.json` (Skills 레포 표준 위치)
2. `.claude/evals/{skill-name}/scenarios.json` (프로젝트별 위치)

```json
{
  "skill": "code-polish",
  "scenarios": [
    {
      "id": "unused-imports",
      "name": "미사용 import가 있는 파일에서 정리 실행",
      "description": "...",
      "setup": { "requires_git": true, "files_to_create": {...} },
      "input": "코드정리",
      "expected": {
        "behavior": "미사용 import 제거",
        "must_contain": ["미사용", "제거"],
        "files_changed": ["src/test-fixtures/unused-imports.ts"],
        "validation": "lint/test 통과"
      }
    }
  ]
}
```

시나리오가 없으면 → Mode 4 (Auto-Generate)로 전환 제안.

### Step 3: 시나리오별 독립 실행

각 시나리오를 **독립 에이전트**로 실행:

1. 에이전트 생성 (Agent 도구, `isolation: "worktree"`)
2. setup 단계 수행 (파일 생성, 명령 실행)
3. 스킬 호출 시뮬레이션 (input 메시지 전달)
4. 결과 수집:
   - **동작 일치**: `expected.behavior`와 실제 동작 비교
   - **필수 포함**: `expected.must_contain` 키워드 검증
   - **금지 포함**: `expected.must_not_contain` 키워드 부재 검증
   - **파일 변경**: `expected.files_changed` 파일이 실제 변경되었는지
   - **토큰 사용량**: 에이전트의 총 토큰 소비량
   - **실행 시간**: 시작~완료 시간

**병렬 실행**: 독립 시나리오는 동시에 실행하여 시간 단축.

### Step 4: 결과 리포트

```
## Eval Report — code-polish

| # | Scenario | Result | Tokens | Time |
|---|----------|--------|--------|------|
| 1 | unused-imports | PASS | 2,340 | 8.2s |
| 2 | magic-numbers | PASS | 3,120 | 11.5s |
| 3 | deep-nesting | FAIL | 4,890 | 15.3s |

**Pass Rate: 66.7% (2/3)**
**Avg Tokens: 3,450**
**Avg Time: 11.7s**

### Failures
- `deep-nesting`: guard clause 적용했으나 테스트 실패로 롤백됨.
  Expected: 중첩 3단계->1단계. Actual: 변경 없음 (롤백).

### Assertion Details
| Scenario | Assertion | Result | Evidence |
|----------|-----------|--------|----------|
| unused-imports | must_contain: "제거" | PASS | "미사용 import 5개 제거" |
| deep-nesting | files_changed: deep.ts | FAIL | 파일 미변경 (롤백) |
```

---

## Mode 2: Benchmark (성능 측정 + 이력 비교)

### Step 1: 기존 시나리오 로드

`evals/{skill-name}/scenarios.json` 필수. 없으면 Mode 4로 먼저 시나리오 생성.

### Step 2: 전체 시나리오 실행

Mode 1과 동일하게 실행하되, 결과를 파일로 저장:

```json
// evals/{skill-name}/benchmark-2026-03-19.json
{
  "skill": "code-polish",
  "date": "2026-03-19",
  "model": "claude-sonnet-4-6",
  "results": {
    "passRate": 0.667,
    "avgTokens": 3450,
    "avgTime": 11.7,
    "scenarios": [...]
  }
}
```

### Step 3: 이전 벤치마크 비교

이전 결과 파일이 있으면 자동 비교:

```
## Benchmark Comparison — code-polish

| Metric | Previous (03-10) | Current (03-19) | Change |
|--------|-----------------|-----------------|--------|
| Pass Rate | 50.0% | 66.7% | +16.7% |
| Avg Tokens | 4,200 | 3,450 | -17.9% |
| Avg Time | 14.2s | 11.7s | -17.6% |

### Trend (최근 5회)
| Date | Pass Rate | Tokens | Time |
|------|-----------|--------|------|
| 03-01 | 33.3% | 5,200 | 18.3s |
| 03-06 | 50.0% | 4,200 | 14.2s |
| 03-10 | 50.0% | 4,200 | 14.2s |
| 03-19 | 66.7% | 3,450 | 11.7s |
```

---

## Mode 3: Compare (A/B 비교)

### Step 1: 두 버전 준비

사용자가 비교할 두 스킬 버전을 지정:
- 현재 버전 vs git의 이전 커밋 버전
- 또는 두 개의 다른 SKILL.md 파일

### Step 2: 동일 시나리오로 양쪽 실행

각 버전별 독립 에이전트로 동일 시나리오 실행 (worktree 격리).

### Step 3: 비교 리포트

```
## A/B Comparison — code-polish

| Metric | Version A (current) | Version B (previous) | Winner |
|--------|-------------------|---------------------|--------|
| Pass Rate | 66.7% | 50.0% | A |
| Avg Tokens | 3,450 | 4,200 | A |
| Avg Time | 11.7s | 14.2s | A |

### Per-Scenario Breakdown
| Scenario | A Result | B Result | Delta Tokens |
|----------|----------|----------|-------------|
| unused-imports | PASS | PASS | -200 |
| magic-numbers | PASS | FAIL | +120 |
| deep-nesting | FAIL | FAIL | -800 |

**Recommendation: Version A (current) is better across all metrics.**
```

---

## Mode 4: Auto-Generate (시나리오 자동 생성)

SKILL.md를 분석하여 테스트 시나리오를 자동으로 생성한다.

### Step 1: 대상 스킬 선택

```bash
ls -d */SKILL.md 2>/dev/null | sed 's|/SKILL.md||'
```

사용자에게 확인: 어떤 스킬의 시나리오를 생성할지.

### Step 2: SKILL.md 분석

대상 스킬의 SKILL.md를 읽고 다음을 추출:

1. **핵심 기능**: Step/Phase별 주요 동작
2. **트리거 키워드**: description에서 추출
3. **입력 조건**: 어떤 상태에서 실행되는지 (git 필요, 파일 필요 등)
4. **기대 출력**: 리포트 형식, 생성 파일, 행동 변화
5. **엣지 케이스**: Rules에서 예외 상황 추출
6. **분기 조건**: 조건문 (if/else, 있으면/없으면)

### Step 3: 시나리오 초안 생성

추출한 정보를 바탕으로 최소 3개 시나리오 생성:

1. **Happy Path**: 모든 조건이 갖춰진 정상 실행
2. **Edge Case**: 파일 없음, 빈 프로젝트 등 예외 상황
3. **Branch Coverage**: 주요 분기의 다른 경로

각 시나리오는 다음 필드를 포함:
```json
{
  "id": "kebab-case-id",
  "name": "한글 시나리오 이름",
  "description": "시나리오 상세 설명",
  "setup": {
    "requires_git": true,
    "files_to_create": {},
    "files_to_delete": [],
    "pre_commands": []
  },
  "input": "트리거 키워드를 포함한 자연어 입력",
  "expected": {
    "behavior": "기대 동작 서술",
    "must_contain": ["출력에 반드시 포함될 키워드"],
    "must_not_contain": ["출력에 포함되면 안 되는 키워드"],
    "files_changed": [],
    "files_created": [],
    "output_format": "기대 출력 형식"
  }
}
```

### Step 4: 사용자 검토 및 저장

시나리오 초안을 사용자에게 보여주고 확인:

```
## 자동 생성된 시나리오 — {skill-name}

### Scenario 1: {name}
- Input: "{input}"
- Expected: {behavior}
- Assertions: {count}개

### Scenario 2: {name}
...

시나리오를 저장할까요? 수정이 필요하면 알려주세요.
```

승인 시 `evals/{skill-name}/scenarios.json`에 저장.

---

## Mode 5: Batch Eval (전체 스킬 일괄 평가)

### Step 1: 전체 스킬 + 시나리오 현황 파악

```bash
# 전체 스킬 목록
ls -d */SKILL.md 2>/dev/null | sed 's|/SKILL.md||'

# 시나리오 보유 현황
ls evals/*/scenarios.json 2>/dev/null
```

시나리오가 없는 스킬은 건너뛰거나, Mode 4로 자동 생성 후 실행.

### Step 2: 비용 예측

```
## Batch Eval 계획

| Skill | Scenarios | Est. Tokens | Est. Time |
|-------|-----------|-------------|-----------|
| dev-start | 3 | ~9,000 | ~30s |
| code-polish | 3 | ~10,000 | ~35s |
| security-guard | 3 | ~8,000 | ~25s |
| ... | ... | ... | ... |
| **Total** | **30** | **~90,000** | **~5min** |

진행할까요? 특정 스킬만 선택하려면 번호를 알려주세요.
```

사용자 확인 후 실행.

### Step 3: 병렬 실행

- 스킬 간 독립이므로 **스킬 단위 병렬** 실행
- 각 스킬 내 시나리오도 **시나리오 단위 병렬** 실행
- 최대 동시 에이전트 수: 5 (비용/리소스 제한)

### Step 4: 대시보드 리포트

Mode 6 형식으로 출력.

---

## Mode 6: Dashboard (전체 현황 대시보드)

### Step 1: 벤치마크 데이터 수집

```bash
# 모든 벤치마크 결과 파일 수집
ls evals/*/benchmark-*.json 2>/dev/null

# 시나리오 보유 현황
ls evals/*/scenarios.json 2>/dev/null
```

### Step 2: 대시보드 출력

```
## Skill Evaluation Dashboard — 2026-03-19

### Overall Summary
| Metric | Value |
|--------|-------|
| Total Skills | 11 |
| Evaluated | 8/11 (72.7%) |
| Avg Pass Rate | 78.3% |
| Total Scenarios | 30 |

### Per-Skill Status
| # | Skill | Scenarios | Last Eval | Pass Rate | Tokens | Time | Trend |
|---|-------|-----------|-----------|-----------|--------|------|-------|
| 1 | spec-writer | 3 | 03-16 | 100% | 107K | 742s | -- |
| 2 | code-polish | 3 | 03-19 | 66.7% | 3.4K | 12s | UP |
| 3 | security-guard | 3 | 03-19 | 100% | 2.8K | 10s | NEW |
| 4 | dev-start | 3 | 03-19 | 100% | 4.2K | 15s | NEW |
| 5 | dev-wrap | 3 | -- | -- | -- | -- | -- |
| 6 | qa-tester | 3 | -- | -- | -- | -- | -- |
| 7 | project-setup | 3 | -- | -- | -- | -- | -- |
| 8 | team-setup | 3 | -- | -- | -- | -- | -- |
| 9 | team-dispatcher | 4 | -- | -- | -- | -- | -- |
| 10| skill-register | 3 | -- | -- | -- | -- | -- |
| 11| usage-guide | 2 | -- | -- | -- | -- | -- |

### Pass Rate Distribution
100%   ████████████████  3 skills
66-99% ████████          1 skill
33-65% ████              0 skills
0-32%  ██                0 skills
N/A    ██████████████    7 skills (미평가)

### Token Efficiency (evaluated skills only)
| Skill | Tokens/Scenario | Rank |
|-------|----------------|------|
| security-guard | 933 | 1 (most efficient) |
| code-polish | 1,150 | 2 |
| dev-start | 1,400 | 3 |
| spec-writer | 35,678 | 4 (heaviest) |

### Recommendations
1. [Action] 미평가 스킬 7개 — `eval all`로 일괄 평가 추천
2. [Improve] code-polish — deep-nesting 시나리오 실패. SKILL.md 개선 필요
3. [Cost] spec-writer — 시나리오당 35K 토큰. 최적화 여지 있음
```

### Step 3: 벤치마크 비교 (이전 대시보드 대비)

이전 대시보드 스냅샷이 있으면 변화 표시:

```
### Changes Since Last Dashboard (03-10)
| Skill | Pass Rate | Change | Note |
|-------|-----------|--------|------|
| code-polish | 50% → 66.7% | +16.7% | SKILL.md 개선 효과 |
| dev-start | N/A → 100% | NEW | 첫 평가 |
```

### Step 4: 대시보드 저장

결과를 파일로 저장:

```json
// evals/dashboard-2026-03-19.json
{
  "date": "2026-03-19",
  "model": "claude-sonnet-4-6",
  "total_skills": 11,
  "evaluated": 4,
  "avg_pass_rate": 0.917,
  "skills": [...]
}
```

---

## Rules

1. **독립 실행** — 각 시나리오는 독립 에이전트 (worktree 격리)로 실행. 시나리오 간 상태 공유 없음
2. **결과 보존** — 벤치마크 결과는 `evals/` 에 JSON으로 저장. git 커밋하지 않음 (사용자 요청 시만)
3. **비용 인식** — 시나리오 수 x 에이전트 = 토큰 비용. 5개 이상 시나리오는 사용자 확인. Batch 실행 시 총 비용 예측 표시
4. **모델 기록** — 벤치마크에 사용된 모델 ID를 항상 기록 (모델 간 비교 가능)
5. **실패 분석** — FAIL 시나리오는 원인까지 분석하여 리포트
6. **시나리오 버전 관리** — 시나리오 파일 변경 시 이전 벤치마크와 비교 불가 경고
7. **시나리오 위치 표준화** — 이 Skills 레포에서는 `evals/{skill-name}/scenarios.json` 사용. 대상 프로젝트에서는 `.claude/evals/{skill-name}/scenarios.json` 사용
8. **Auto-Generate 품질** — 자동 생성 시나리오는 반드시 사용자 검토 후 저장. 자동 저장 금지
9. **Batch 제한** — 동시 에이전트 최대 5개. 대기열 방식으로 처리
10. **대시보드 갱신** — Batch Eval 완료 시 자동으로 대시보드도 갱신
11. **Trend 표시** — 이전 결과가 있으면 변화 방향 (UP/DOWN/STABLE/NEW) 표시
12. **skill-eval 자기 평가 금지** — skill-eval 스킬 자체는 평가 대상에서 제외 (메타 순환 방지)
