---
name: security-guard
description: Automated security scan for code changes — detects OWASP Top 10 vulnerabilities, blocks dangerous commands via PreToolUse hook, suggests fixes. Triggers on "보안 검사", "security check", "security scan", "보안 스캔", "security guard", "취약점 검사".
---

# Security Guard — 보안 자동 검사 · 취약점 탐지 · 위험 명령 차단

변경된 코드를 OWASP Top 10 기준으로 자동 스캔하고, 위험 패턴 발견 시 즉시 수정을 제안한다.
PreToolUse hook과 연동하여 위험 명령을 사전 차단할 수 있다.

---

## Step 1: 변경 파일 식별

code-polish와 동일한 방식으로 이번 세션 변경 파일 수집:

```bash
SESSION_FILES=$(git log --oneline --since="4 hours ago" --name-only --pretty=format: 2>/dev/null)
UNSTAGED=$(git diff --name-only 2>/dev/null)
STAGED=$(git diff --cached --name-only 2>/dev/null)
echo "$SESSION_FILES" "$UNSTAGED" "$STAGED" | tr ' ' '\n' | sort -u | grep -v '^$'
```

### 대상 필터링

**포함**: 소스 코드 + 설정 파일 (보안 설정 포함)
- `.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.rs`, `.java`, `.c`, `.cpp`, `.rb`, `.php`
- `.env`, `.yaml`, `.yml`, `.json` (secrets 검사용)

**제외**:
- 테스트 코드 (`*.test.*`, `*.spec.*`, `__tests__/`)
- 자동생성/빌드 산출물
- 문서 (`.md`, `.txt`)

**변경 파일 0개** → "검사할 파일 없음" 리포트 후 종료.

---

## Step 2: 보안 패턴 스캔

각 파일을 읽고 아래 카테고리별 패턴을 검사한다.
**CRITICAL은 즉시 수정 제안, WARNING은 문서화, INFO는 참고 표시.**

### 2.1 인젝션 (CRITICAL)

#### SQL Injection
- 문자열 연결로 SQL 쿼리 구성: `query("SELECT * FROM " + userInput)`
- f-string/template literal in SQL: `` `SELECT * FROM ${table}` ``
- **안전 패턴**: parameterized query, ORM 메서드 → PASS

#### Command Injection
- 사용자 입력이 shell 명령에 포함: `exec(cmd + userInput)`
- `child_process.exec()` with 변수: `exec(\`ls ${dir}\`)`
- **안전 패턴**: `execFile()` with argument array, allowlist → PASS

#### XSS (Cross-Site Scripting)
- `innerHTML`, `dangerouslySetInnerHTML` with 변수
- 템플릿에서 escape 없이 변수 출력
- **안전 패턴**: textContent, sanitize 라이브러리, React JSX → PASS

### 2.2 인증/인가 (CRITICAL)

- 하드코딩된 비밀번호, API 키, 토큰
  - 패턴: `password = "..."`, `api_key = "..."`, `token = "..."`
  - `.env` 파일에 실제 값이 포함된 경우
- JWT secret 하드코딩
- 인증 없는 API 엔드포인트 (auth middleware 누락)

### 2.3 데이터 노출 (WARNING)

- 에러 메시지에 스택 트레이스/내부 경로 노출
- API 응답에 민감 필드 (password, ssn, credit_card) 포함
- 로그에 민감 데이터 출력: `console.log(user)`, `logger.info(request.body)`

### 2.4 설정 취약점 (WARNING)

- CORS `*` (모든 origin 허용)
- HTTPS 미사용 (HTTP 엔드포인트)
- 디버그 모드 프로덕션 노출: `DEBUG=true`, `app.debug = True`
- 느슨한 파일 권한: `chmod 777`, `0o777`

### 2.5 의존성 (INFO)

- `package.json` / `requirements.txt`의 알려진 취약 버전 패턴
- `eval()`, `pickle.loads()`, `yaml.load()` (unsafe loader)
- `subprocess.call(shell=True)`

### 2.6 경로 조작 (WARNING)

- 사용자 입력으로 파일 경로 구성: `open(user_path)`
- `../` traversal 가능성
- **안전 패턴**: `path.resolve()` + base directory 검증 → PASS

### 2.7 SSRF (WARNING)

- 사용자 입력 URL로 HTTP 요청: `fetch(userUrl)`
- 내부 네트워크 접근 가능성
- **안전 패턴**: URL allowlist, scheme 검증 → PASS

---

## Step 3: 수정 제안

CRITICAL 발견 시 즉시 수정 코드를 제안:

```
### 🔴 CRITICAL — SQL Injection
**File**: src/db/users.ts:42
**현재 코드**:
const result = await db.query(`SELECT * FROM users WHERE id = '${userId}'`);

**수정 제안**:
const result = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

**이유**: 사용자 입력이 직접 쿼리에 삽입되어 SQL 인젝션 가능.
Parameterized query로 변경하면 DB 드라이버가 자동으로 이스케이프.
```

수정 적용 여부는 사용자 확인 후 결정.

---

## Step 4: 보안 리포트

```
## 보안 검사 완료

### 검사 파일: N개

### 발견 사항
| # | 심각도 | 카테고리 | 파일 | 내용 |
|---|--------|---------|------|------|
| 1 | 🔴 CRITICAL | SQL Injection | src/db/users.ts:42 | 문자열 연결 쿼리 |
| 2 | 🟡 WARNING | Data Exposure | src/api/handler.ts:15 | 로그에 request.body 출력 |
| 3 | 🔵 INFO | Unsafe Function | src/utils/parse.ts:8 | eval() 사용 |

### 요약
- 🔴 CRITICAL: 1건 (즉시 수정 필요)
- 🟡 WARNING: 1건 (수정 권장)
- 🔵 INFO: 1건 (참고)

### 수정 적용 여부
CRITICAL 항목에 대한 수정을 적용할까요? (Y/N)
```

발견 없으면:
```
## 보안 검사 완료 — 취약점 없음
변경된 N개 파일을 검사했으나 보안 문제가 발견되지 않았습니다.
```

---

## Step 5: Hook 연동 안내

보안 게이트를 자동화하려면 project-setup 스킬로 PreToolUse hook 설정을 권장:

```
위험 명령 자동 차단을 설정하시겠습니까?
→ project-setup 스킬의 B10 (PreToolUse Security Gate) 템플릿을 적용합니다.
```

---

## Rules

1. **변경 파일만 검사** — 이번 세션에서 수정한 파일만. 전체 프로젝트 스캔은 별도 요청 시에만
2. **False Positive 최소화** — 안전 패턴(ORM, parameterized query, sanitize)은 PASS 처리
3. **수정 강제 금지** — CRITICAL이라도 사용자 확인 없이 코드 수정하지 않음
4. **테스트 코드 제외** — 테스트에서의 하드코딩, eval 등은 의도적 사용이므로 제외
5. **컨텍스트 인식** — 프레임워크별 보안 패턴 인지 (React의 자동 XSS 방지, Django의 ORM 등)
6. **비밀 로그 금지** — 검사 리포트에 실제 비밀 값을 노출하지 않음. 패턴과 위치만 표시
7. **환경 파일 주의** — `.env` 파일은 값 자체가 아닌 구조만 검사. 실제 키 값은 절대 리포트에 포함 금지
