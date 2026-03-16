---
name: skill-eval
description: Run evals, benchmarks, and A/B comparisons on skills — measures pass rate, token usage, and execution time per scenario. Triggers on "스킬 평가", "eval skill", "benchmark skill", "스킬 테스트", "skill eval", "스킬 벤치마크".
---

# Skill Eval — 스킬 품질 측정 · 벤치마크 · A/B 비교

스킬의 품질을 정량적으로 측정하고, 모델 업데이트나 스킬 수정 후 성능 변화를 추적한다.

---

## Mode 1: Eval (테스트 실행)

### Step 1: 대상 스킬 확인

```bash
# 사용자가 지정한 스킬 또는 전체 스킬 목록
ls -d */SKILL.md 2>/dev/null | sed 's|/SKILL.md||'
```

사용자에게 확인:
- 어떤 스킬을 평가할지
- 테스트 시나리오가 이미 있는지, 새로 만들지

### Step 2: 테스트 시나리오 정의

시나리오 파일 위치: `.claude/evals/{skill-name}/scenarios.json`

```json
{
  "skill": "code-polish",
  "scenarios": [
    {
      "id": "unused-imports",
      "description": "미사용 import가 있는 파일에서 정리 실행",
      "input": "src/test-fixtures/unused-imports.ts 파일을 정리해줘",
      "expected": {
        "behavior": "미사용 import 제거",
        "files_changed": ["src/test-fixtures/unused-imports.ts"],
        "validation": "lint/test 통과"
      }
    }
  ]
}
```

시나리오가 없으면 사용자와 함께 작성. 최소 3개 시나리오 권장.

### Step 3: 시나리오별 독립 실행

각 시나리오를 **독립 에이전트**로 실행:

1. 에이전트 생성 (Agent 도구, `isolation: "worktree"`)
2. 스킬 호출 시뮬레이션
3. 결과 수집: 동작 일치 여부, 토큰 사용량, 실행 시간

**병렬 실행**: 독립 시나리오는 동시에 실행하여 시간 단축.

### Step 4: 결과 리포트

```
## Eval Report — code-polish

| # | Scenario | Result | Tokens | Time |
|---|----------|--------|--------|------|
| 1 | unused-imports | ✅ PASS | 2,340 | 8.2s |
| 2 | magic-numbers | ✅ PASS | 3,120 | 11.5s |
| 3 | deep-nesting | ❌ FAIL | 4,890 | 15.3s |

**Pass Rate: 66.7% (2/3)**
**Avg Tokens: 3,450**
**Avg Time: 11.7s**

### Failures
- `deep-nesting`: guard clause 적용했으나 테스트 실패로 롤백됨.
  Expected: 중첩 3단계→1단계. Actual: 변경 없음 (롤백).
```

---

## Mode 2: Benchmark (성능 측정)

### Step 1: 기존 시나리오 로드

`.claude/evals/{skill-name}/scenarios.json` 필수. 없으면 Eval 모드로 먼저 시나리오 생성.

### Step 2: 전체 시나리오 실행

Mode 1과 동일하게 실행하되, 결과를 파일로 저장:

```json
// .claude/evals/{skill-name}/benchmark-2026-03-16.json
{
  "skill": "code-polish",
  "date": "2026-03-16",
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

| Metric | Previous (03-10) | Current (03-16) | Change |
|--------|-----------------|-----------------|--------|
| Pass Rate | 50.0% | 66.7% | +16.7% ✅ |
| Avg Tokens | 4,200 | 3,450 | -17.9% ✅ |
| Avg Time | 14.2s | 11.7s | -17.6% ✅ |
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
| Pass Rate | 66.7% | 50.0% | A ✅ |
| Avg Tokens | 3,450 | 4,200 | A ✅ |
| Avg Time | 11.7s | 14.2s | A ✅ |

**Recommendation: Version A (current) is better across all metrics.**
```

---

## Rules

1. **독립 실행** — 각 시나리오는 독립 에이전트 (worktree 격리)로 실행. 시나리오 간 상태 공유 없음
2. **결과 보존** — 벤치마크 결과는 `.claude/evals/` 에 JSON으로 저장. git 커밋하지 않음
3. **비용 인식** — 시나리오 수 × 에이전트 = 토큰 비용. 5개 이상 시나리오는 사용자 확인
4. **모델 기록** — 벤치마크에 사용된 모델 ID를 항상 기록 (모델 간 비교 가능)
5. **실패 분석** — FAIL 시나리오는 원인까지 분석하여 리포트
6. **시나리오 버전 관리** — 시나리오 파일 변경 시 이전 벤치마크와 비교 불가 경고
