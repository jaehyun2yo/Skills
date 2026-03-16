---
name: code-polish
description: Auto-cleanup changed source files — remove dead code, improve naming, optimize loops, strengthen types. Validates with lint/test, rolls back on failure. Auto-triggers via Stop hook. Triggers on "코드정리", "코드 최적화", "리팩토링", "code polish", "cleanup code", "정리해", "optimize code", "코드 다듬어", "polish".
---

# Code Polish — 코드 정리 · 최적화 · 리팩토링

코딩 완료 후 이번 세션에서 변경된 파일만 자동으로 검토하고 정리한다.
동작을 바꾸지 않는 범위에서만 개선하며, 검증 실패 시 즉시 롤백한다.

---

## Step 1: 변경 파일 식별

```bash
# 이번 세션 커밋 + 미커밋 변경 파일 수집
SESSION_FILES=$(git log --oneline --since="4 hours ago" --name-only --pretty=format: 2>/dev/null | sort -u)
UNSTAGED=$(git diff --name-only 2>/dev/null)
STAGED=$(git diff --cached --name-only 2>/dev/null)

# 합산 후 중복 제거
echo "$SESSION_FILES" "$UNSTAGED" "$STAGED" | tr ' ' '\n' | sort -u | grep -v '^$'
```

### 대상 필터링

**포함**: 소스 코드 파일만
- `.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.rs`, `.java`, `.c`, `.cpp`, `.svelte`, `.vue`, `.rb`, `.php`, `.swift`, `.kt`

**제외** (절대 건드리지 않음):
- 설정 파일: `*.json`, `*.yaml`, `*.yml`, `*.toml`, `*.config.*`
- 문서: `*.md`, `*.txt`, `*.rst`
- 에셋: 이미지, 폰트, 바이너리
- 자동생성: `*.lock`, `*.min.*`, `dist/`, `build/`, `node_modules/`
- 테스트 픽스처/스냅샷

**변경 파일이 0개이면** → "정리할 파일 없음" 리포트 후 즉시 종료.

---

## Step 2: 변경 전 스냅샷 저장

정리 작업 전에 현재 상태를 보존한다:

```bash
# 미커밋 변경이 있으면 임시 스태시 (롤백용)
git stash push -m "code-polish-backup-$(date +%s)" --include-untracked 2>/dev/null
git stash pop 2>/dev/null  # 즉시 복원 (스태시 스택에 백업만 남김)
```

---

## Step 3: 파일별 정리 (순서대로)

각 대상 파일을 읽고 아래 체크리스트를 순서대로 적용.
**한 번에 모든 파일을 수정하지 말고, 파일 단위로 검토 → 수정 → 다음 파일.**

### 3.1 불필요 코드 제거 (가장 안전 — 먼저)

- 사용하지 않는 import/require/include 제거
- 사용하지 않는 변수, 함수, 타입, 클래스 제거
- 주석 처리된 코드 블록 제거 (TODO/FIXME/HACK 주석은 유지)
- `console.log`, `print()`, `debugger`, `dd()` 등 디버깅 코드 제거
- 연속 빈 줄 3줄 이상 → 1줄로 정리

### 3.2 코드 품질 개선 (가독성 중심)

- 의미 없는 변수명 개선: `a`, `tmp`, `data`, `res`, `val` → 맥락에 맞는 이름
  - 단, 관용적으로 허용되는 이름은 유지 (`i`, `j`, `k` 루프 변수, `e` 이벤트, `err` 에러 등)
- 매직 넘버/스트링 → 이름 있는 상수로 추출
- 중복 코드 3회 이상 반복 → 공통 함수/메서드로 추출
- 깊은 중첩 3단계 이상 → early return / guard clause로 평탄화
- 긴 함수 50줄 이상 → 논리 단위로 분리 **고려** (강제 아님)
- 불필요한 else → early return 후 else 제거

### 3.3 성능 최적화 (동작 불변 보장)

- 루프 안 불변 연산 → 루프 밖으로 이동
- 불필요한 재연산 → 변수 캐싱
- N+1 쿼리 패턴 → 배치 쿼리로 통합 (DB 코드)
- 불필요한 spread/deep copy → 직접 참조 (immutability 불필요 시)
- 연속 await → `Promise.all` / `asyncio.gather` 변환 (독립 작업 시)
- 불필요한 `.map().filter()` 체인 → 단일 `.reduce()` 또는 루프

### 3.4 타입/안전성 강화 (타입 시스템 있는 언어만)

- `any` 타입 → 구체적 타입 또는 제네릭으로 교체
- null/undefined 가능성 → optional chaining (`?.`) 또는 guard 추가
- 빈 catch 블록 → 최소한 에러 로깅 추가
- 타입 단언(`as`) 남용 → 타입 가드로 교체

---

## Step 4: 검증

변경 적용 후 프로젝트의 기존 검증 도구를 실행:

```bash
# 프로젝트 타입에 따라 자동 감지하여 실행

# Node.js (package.json 존재 시)
if [ -f package.json ]; then
  npm run lint --if-present 2>/dev/null
  npm run typecheck --if-present 2>/dev/null
  npm test -- --passWithNoTests 2>/dev/null
fi

# Python (pyproject.toml 또는 setup.py 존재 시)
if [ -f pyproject.toml ] || [ -f setup.py ]; then
  ruff check . 2>/dev/null || flake8 . 2>/dev/null
  mypy . 2>/dev/null
  pytest --tb=short 2>/dev/null
fi

# Go (go.mod 존재 시)
if [ -f go.mod ]; then
  go vet ./... 2>/dev/null
  go test ./... 2>/dev/null
fi

# Rust (Cargo.toml 존재 시)
if [ -f Cargo.toml ]; then
  cargo clippy 2>/dev/null
  cargo test 2>/dev/null
fi
```

### 검증 실패 처리

검증이 실패하면:
1. 실패 원인 분석
2. **해당 변경만 롤백** (`git checkout -- <file>`)
3. 나머지 성공한 변경은 유지
4. 리포트에 롤백 사유 기록

전체 실패 시 → 모든 변경 롤백:
```bash
git checkout -- . 2>/dev/null  # 미커밋 변경 전체 복원
```

---

## Step 5: 정리 리포트

```
## 코드 정리 완료

### 정리된 파일 (N개)
- `src/foo.ts` — 미사용 import 3개 제거, guard clause 적용
- `src/bar.py` — 중복 로직 extract_data()로 추출, 매직넘버 상수화

### 카테고리별 요약
- 불필요 코드 제거: N건
- 코드 품질 개선: N건
- 성능 최적화: N건
- 타입/안전성: N건

### 검증 결과
- Lint: PASS / FAIL / N/A
- Type check: PASS / FAIL / N/A
- Tests: PASS (N passed) / FAIL (N failed) / N/A

### 롤백된 변경 (있는 경우)
- `src/baz.ts` — guard clause 적용 시 테스트 실패 → 원복
```

개선할 것이 없으면:
```
## 코드 정리 — 정리할 항목 없음
변경된 파일 N개를 검토했으나 추가 정리가 필요한 항목이 없습니다.
```

---

## Step 6: 패턴 학습 (Self-Improving)

정리 작업 결과를 학습 데이터로 축적하여 다음 실행의 정확도를 높인다.

### 6.1 패턴 기록

정리 리포트의 각 변경 항목을 `.claude/polish-patterns.json`에 기록:

```json
{
  "patterns": [
    {
      "category": "dead-code",
      "pattern": "unused-import",
      "files": ["src/foo.ts", "src/bar.ts"],
      "count": 5,
      "firstSeen": "2026-03-01",
      "lastSeen": "2026-03-16"
    }
  ],
  "totalRuns": 12,
  "lastRun": "2026-03-16"
}
```

- 이미 기록된 패턴이면 `count` 증가, `lastSeen` 갱신, `files` 배열에 추가
- 새 패턴이면 항목 추가
- `totalRuns` 매 실행마다 +1

### 6.2 고빈도 패턴 우선 검사

다음 실행 시 Step 3 진입 전에 `.claude/polish-patterns.json`을 읽고:
- `count >= 3`인 패턴을 **우선 검사 대상**으로 설정
- 해당 카테고리의 체크리스트를 먼저 적용

### 6.3 반복 패턴 경고

`count >= 3`인 패턴이 있으면 리포트 끝에 추가:

```
### 반복 패턴 경고
- `unused-import` — 5회 반복 (dead-code). ESLint `no-unused-imports` 규칙 추가 권장
- `magic-number` — 3회 반복 (quality). 프로젝트 상수 파일 도입 권장

💡 반복 패턴은 lint 규칙이나 팀 컨벤션으로 근본 해결하는 것이 효과적입니다.
```

---

## Rules

1. **변경 파일만 대상** — 이번 세션에서 수정한 파일만 검토. 다른 파일은 절대 건드리지 않음
2. **동작 보존 최우선** — 리팩토링은 외부 동작을 변경하지 않는 범위에서만. 불확실하면 안 함
3. **검증 필수** — lint/test가 있으면 반드시 실행. 실패하면 해당 변경만 롤백
4. **과도한 최적화 금지** — 가독성을 해치는 최적화는 하지 않음. 가독성 > 성능
5. **컨벤션 존중** — 프로젝트의 기존 코드 스타일, 네이밍 규칙, 패턴을 따름
6. **변경 없으면 패스** — 개선할 것이 없으면 보고만 하고 끝. 억지로 변경하지 않음
7. **커밋 분리** — 코드 정리는 기능 커밋과 별도: `refactor: code polish — [요약]`
8. **사용자 확인 후 커밋** — 자동 커밋 금지. 변경 diff 보여주고 승인 받기
9. **안전 순서** — 제거 → 품질 → 최적화 → 타입 순서 (안전한 것부터)
10. **테스트 코드도 정리** — 테스트 파일이 변경 대상이면 동일 기준 적용 (단, 테스트 의도는 변경 금지)
11. **패턴 학습** — `.claude/polish-patterns.json`에 발견 패턴을 기록한다. 3회 이상 반복되면 리포트에 경고하고 lint 규칙 추가를 권장한다. 학습 데이터는 git에 커밋하지 않음
