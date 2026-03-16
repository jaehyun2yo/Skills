# Tech Stack Profiles

When generating setup files, replace `{{PLACEHOLDER}}` values based on the detected tech stack.
Only read the section matching the detected stack — do not load entire file.

---

## Next.js / React

### Detection
- `package.json` contains `next` dependency

### Template Values
```
{{TEST_FRAMEWORK}}: Jest or Vitest
{{TEST_COMMAND}}: npm run test / pnpm test
{{TYPECHECK_COMMAND}}: npx tsc --noEmit
{{LINT_COMMAND}}: npm run lint / pnpm lint
{{FORMAT_COMMAND}}: npx prettier --write "$CLAUDE_FILE_PATH" 2>/dev/null || true
{{CONVENTIONS_CHECKLIST}}:
  - No `any` types
  - Server Components by default, 'use client' only when interactive
  - React Query queryKeys factory (no raw string arrays)
  - No window.location.reload() (use query invalidation)
  - @/ absolute imports only
  - No console.log (use project logger if exists)

{{FRAMEWORK_SPECIFIC_CHECKS}}:
  - Server/Client component separation
  - React Query cache invalidation correctness
  - Next.js API route error handling
  - Metadata/SEO configuration

{{COMMON_BUG_PATTERNS}}:
  1. React Query invalidation missing
  2. Server Component using client-only hook
  3. API route missing error response
  4. Middleware path matching incorrect
  5. Hydration mismatch (server/client content differs)
```

---

## Next.js + NestJS (Monorepo)

### Detection
- `package.json` contains `next` AND `webhard-api/` or `api/` directory with NestJS

### Additional Template Values
```
{{CONVENTIONS_CHECKLIST}} (append):
  - Supabase tables → Next.js only. Prisma tables → NestJS only.
  - NestJS DTOs with class-validator
  - Global ValidationPipe (whitelist + forbidNonWhitelisted)

{{COMMON_BUG_PATTERNS}} (append):
  6. ORM boundary violation (Supabase ↔ Prisma mix)
  7. NestJS DTO validation fail
  8. CORS configuration mismatch
  9. Prisma migration not applied

{{PROJECT_DATA_FLOW}}:
  UI → API Route/Server Action → Supabase (direct) or NestJS API → Prisma → PostgreSQL
```

---

## Electron

### Detection
- `package.json` contains `electron`

### Template Values
```
{{TEST_FRAMEWORK}}: Vitest or Jest
{{TEST_COMMAND}}: npm run test
{{TYPECHECK_COMMAND}}: npm run typecheck / npx tsc --noEmit
{{LINT_COMMAND}}: npm run lint
{{FORMAT_COMMAND}}: npx prettier --write "$CLAUDE_FILE_PATH" 2>/dev/null || true
{{CONVENTIONS_CHECKLIST}}:
  - Interface prefix: I (ILogger, ISyncEngine)
  - IPC types defined in shared types file
  - No direct Node.js API in renderer (use preload bridge)
  - contextBridge for all main↔renderer communication
  - Zustand stores for renderer state

{{FRAMEWORK_SPECIFIC_CHECKS}}:
  - IPC channel type safety
  - Main/Renderer process boundary
  - contextBridge exposure correctness
  - Native module rebuild (better-sqlite3, etc.)

{{COMMON_BUG_PATTERNS}}:
  1. IPC channel name mismatch
  2. Native module not rebuilt for Electron version
  3. Main process blocking (sync I/O)
  4. Renderer accessing Node.js APIs directly
  5. Event listener leak (missing cleanup)
  6. SQLite concurrent access locking

{{PROJECT_DATA_FLOW}}:
  Renderer (React) → IPC invoke → Main Process → Core Services → External APIs/DB
```

---

## Python (FastAPI / Django / Flask)

### Detection
- `pyproject.toml` or `requirements.txt` with fastapi/django/flask

### Template Values
```
{{TEST_FRAMEWORK}}: pytest
{{TEST_COMMAND}}: pytest / python -m pytest
{{TYPECHECK_COMMAND}}: mypy . / pyright
{{LINT_COMMAND}}: ruff check . / flake8
{{FORMAT_COMMAND}}: ruff format "$CLAUDE_FILE_PATH" 2>/dev/null || black "$CLAUDE_FILE_PATH" 2>/dev/null || true
{{CONVENTIONS_CHECKLIST}}:
  - Type hints on all function signatures
  - Pydantic models for request/response validation
  - Async functions where applicable
  - No print() statements (use logging module)
  - Virtual environment isolation

{{FRAMEWORK_SPECIFIC_CHECKS}}:
  - Pydantic model validation
  - Dependency injection pattern
  - Async/sync boundary correctness
  - Database session management

{{COMMON_BUG_PATTERNS}}:
  1. Missing Pydantic validation
  2. Async/sync mixing causing deadlocks
  3. Database session not properly closed
  4. Import circular dependency
  5. Environment variable missing
```

---

## Rust

### Detection
- `Cargo.toml` exists

### Template Values
```
{{TEST_FRAMEWORK}}: cargo test (built-in)
{{TEST_COMMAND}}: cargo test
{{TYPECHECK_COMMAND}}: cargo check
{{LINT_COMMAND}}: cargo clippy
{{FORMAT_COMMAND}}: rustfmt "$CLAUDE_FILE_PATH" 2>/dev/null || true
{{CONVENTIONS_CHECKLIST}}:
  - Proper error handling (Result<T, E>, no unwrap in production)
  - Derive macros for common traits
  - Module organization follows convention
  - Documentation comments on public items

{{COMMON_BUG_PATTERNS}}:
  1. Ownership/borrowing issues
  2. Deadlock from mutex ordering
  3. Async runtime misuse (tokio)
  4. Serialization/deserialization failures
```

---

## Flutter / Dart

### Detection
- `pubspec.yaml` exists

### Template Values
```
{{TEST_FRAMEWORK}}: flutter test (built-in)
{{TEST_COMMAND}}: flutter test
{{TYPECHECK_COMMAND}}: flutter analyze
{{LINT_COMMAND}}: dart format --set-exit-if-changed .
{{FORMAT_COMMAND}}: dart format "$CLAUDE_FILE_PATH" 2>/dev/null || true
{{CONVENTIONS_CHECKLIST}}:
  - BLoC/Cubit for state management
  - Freezed for immutable models
  - GoRouter for navigation
  - Repository pattern for data layer

{{COMMON_BUG_PATTERNS}}:
  1. State not properly disposed
  2. Widget rebuild excessive
  3. Platform-specific code missing
  4. Null safety violation
```

---

## Go

### Detection
- `go.mod` exists

### Template Values
```
{{TEST_FRAMEWORK}}: go test (built-in)
{{TEST_COMMAND}}: go test ./...
{{TYPECHECK_COMMAND}}: go vet ./...
{{LINT_COMMAND}}: golangci-lint run
{{FORMAT_COMMAND}}: gofmt -w "$CLAUDE_FILE_PATH" 2>/dev/null || true
{{CONVENTIONS_CHECKLIST}}:
  - Error handling (no ignored errors)
  - Interface-based design
  - Context propagation
  - Goroutine leak prevention (proper cancellation)

{{COMMON_BUG_PATTERNS}}:
  1. Goroutine leak (missing context cancel)
  2. Data race (missing mutex)
  3. nil pointer dereference
  4. Channel deadlock
```

---

## Generic / Unknown

### Fallback values when tech stack is not in profiles above
```
{{TEST_FRAMEWORK}}: (detect from config files)
{{TEST_COMMAND}}: (detect from package.json scripts or Makefile)
{{TYPECHECK_COMMAND}}: (if available)
{{LINT_COMMAND}}: (if available)
{{FORMAT_COMMAND}}: echo "No formatter configured" (detect from project config)
{{CONVENTIONS_CHECKLIST}}:
  - No hardcoded secrets
  - Consistent naming conventions
  - Error handling present
  - Functions < 50 lines

{{COMMON_BUG_PATTERNS}}:
  1. Missing error handling
  2. Hardcoded configuration
  3. Resource leak (unclosed connections/files)
  4. Race condition
```
