---
name: qa-tester
description: E2E QA tester — detects app type (web/Electron/Tauri/Flutter/desktop), runs the app, tests user flows, captures screenshots, reports bugs. Default test approach is E2E. Triggers on "QA", "E2E 테스트", "qa test", "실사용 테스트", "e2e test", "QA 테스트", "통합 테스트".
---

# QA Tester — 실사용 E2E 테스트 · 플랫폼 자동 감지 · 버그 리포트

실제 앱을 실행하고 사용자 관점에서 테스트합니다.
TDD의 단위 테스트가 아닌 **E2E 테스트를 기본**으로 작성하며, 플랫폼에 맞는 도구를 자동 선택합니다.

---

## Step 1: 플랫폼 감지

```bash
# App type detection
if [ -f "electron-builder.yml" ] || [ -f "electron-builder.json5" ] || grep -q '"electron"' package.json 2>/dev/null; then
  echo "ELECTRON"
elif [ -f "src-tauri/tauri.conf.json" ] || [ -f "tauri.conf.json" ]; then
  echo "TAURI"
elif [ -f "pubspec.yaml" ] && grep -q "flutter" pubspec.yaml 2>/dev/null; then
  echo "FLUTTER"
elif [ -f "package.json" ] && (grep -q '"next\|react\|vue\|svelte\|angular\|vite"' package.json 2>/dev/null); then
  echo "WEB"
elif [ -f "package.json" ] && (grep -q '"express\|fastify\|nestjs\|koa\|hono"' package.json 2>/dev/null); then
  echo "API"
elif [ -f "pyproject.toml" ] || [ -f "requirements.txt" ]; then
  echo "PYTHON_API"
elif [ -f "go.mod" ]; then
  echo "GO_API"
else
  echo "UNKNOWN"
fi
```

### 플랫폼별 E2E 도구 선택

| Platform | E2E Tool | 설치 명령 | 특징 |
|----------|----------|----------|------|
| **WEB** | Playwright MCP / Playwright Test | `npm init playwright@latest` | 브라우저 자동화, 스크린샷, 네트워크 모킹 |
| **ELECTRON** | Playwright Electron API | `npm i -D playwright @playwright/test` | `_electron.launch()` 로 앱 직접 제어 |
| **TAURI** | WebDriver (WebdriverIO) | `npm i -D webdriverio` | WebDriver 프로토콜, msedgedriver 필요 |
| **FLUTTER** | integration_test + flutter-skill | 내장 | `flutter test integration_test/` |
| **API** | Playwright API Testing / supertest | `npm i -D supertest` 또는 Playwright | request/response 검증 |
| **PYTHON_API** | httpx + pytest | `pip install httpx pytest` | async API 테스트 |
| **GO_API** | net/http/httptest | 내장 | 표준 라이브러리 |

---

## Step 2: E2E 테스트 환경 구성

### 2.1 의존성 확인

감지된 플랫폼에 맞는 E2E 도구가 설치되어 있는지 확인:

```bash
# WEB / ELECTRON
npx playwright --version 2>/dev/null || echo "NEED_INSTALL: npm init playwright@latest"

# TAURI
npx wdio --version 2>/dev/null || echo "NEED_INSTALL: npm i -D webdriverio"

# FLUTTER
flutter --version 2>/dev/null || echo "NEED_INSTALL: flutter SDK"
```

설치가 필요하면 사용자에게 안내하고 확인 후 설치.

### 2.2 테스트 디렉토리 구조

```
e2e/                          # E2E 테스트 루트
├── fixtures/                 # 테스트 데이터, 시드
├── pages/                    # Page Object Model (POM)
│   ├── login.page.ts
│   └── dashboard.page.ts
├── flows/                    # 사용자 흐름별 테스트
│   ├── auth.spec.ts          # 인증 흐름
│   ├── main-feature.spec.ts  # 핵심 기능 흐름
│   └── edge-cases.spec.ts    # 엣지 케이스
├── helpers/                  # 유틸리티
│   └── test-utils.ts
└── playwright.config.ts      # (또는 wdio.conf.ts)
```

### 2.3 테스트 설정 파일 생성

플랫폼별 설정 파일을 생성합니다. 이미 있으면 스킵.

**Playwright (WEB/ELECTRON):**
```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e/flows',
  timeout: 30000,
  retries: 1,
  use: {
    screenshot: 'on',
    video: 'on-first-retry',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
  ],
});
```

**Playwright Electron:**
```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e/flows',
  timeout: 60000,
  use: {
    screenshot: 'on',
  },
});
```

---

## Step 3: E2E 테스트 작성

### 3.1 핵심 원칙

- **사용자 관점**: 기술적 구현이 아닌 사용자 행동 기반으로 작성
- **Page Object Model**: 페이지 로직과 테스트 로직 분리
- **독립 실행**: 각 테스트는 독립적으로 실행 가능 (순서 의존 없음)
- **데이터 격리**: 테스트별 고유 데이터 사용 (다른 테스트에 영향 없음)

### 3.2 테스트 작성 패턴

#### WEB (Playwright)
```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/login.page';

test.describe('로그인 흐름', () => {
  test('유효한 자격으로 로그인 성공', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('user@test.com', 'password123');

    // 대시보드로 이동 확인
    await expect(page).toHaveURL(/dashboard/);
    await expect(page.getByRole('heading', { name: '대시보드' })).toBeVisible();
  });

  test('잘못된 비밀번호로 에러 표시', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('user@test.com', 'wrong');

    await expect(page.getByText('비밀번호가 일치하지 않습니다')).toBeVisible();
    await expect(page).toHaveURL(/login/);
  });
});
```

#### ELECTRON (Playwright Electron API)
```typescript
import { test, expect, _electron as electron } from '@playwright/test';

test('앱이 정상적으로 시작되고 메인 윈도우가 표시됨', async () => {
  const app = await electron.launch({ args: ['.'] });
  const window = await app.firstWindow();

  // 윈도우 타이틀 확인
  const title = await window.title();
  expect(title).toContain('My App');

  // 메인 UI 요소 확인
  await expect(window.getByRole('navigation')).toBeVisible();

  // 스크린샷 캡처
  await window.screenshot({ path: 'e2e/screenshots/main-window.png' });

  await app.close();
});
```

#### TAURI (WebdriverIO)
```typescript
// e2e/flows/main.spec.ts
import { browser, $ } from '@wdio/globals';

describe('메인 앱 흐름', () => {
  it('앱 시작 후 메인 화면 표시', async () => {
    const title = await browser.getTitle();
    expect(title).toContain('My App');

    const mainContent = await $('#main-content');
    expect(await mainContent.isDisplayed()).toBe(true);
  });
});
```

#### FLUTTER (integration_test)
```dart
// integration_test/app_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('로그인 후 홈 화면 표시', (tester) async {
    app.main();
    await tester.pumpAndSettle();

    await tester.enterText(find.byKey(Key('email')), 'user@test.com');
    await tester.enterText(find.byKey(Key('password')), 'password123');
    await tester.tap(find.byKey(Key('login-button')));
    await tester.pumpAndSettle();

    expect(find.text('홈'), findsOneWidget);
  });
}
```

#### API (supertest / Playwright API)
```typescript
import { test, expect } from '@playwright/test';

test.describe('Users API', () => {
  test('POST /api/users — 사용자 생성', async ({ request }) => {
    const response = await request.post('/api/users', {
      data: { email: 'new@test.com', name: 'Test User' }
    });

    expect(response.status()).toBe(201);
    const body = await response.json();
    expect(body.email).toBe('new@test.com');
  });

  test('GET /api/users/:id — 존재하지 않는 사용자', async ({ request }) => {
    const response = await request.get('/api/users/999999');
    expect(response.status()).toBe(404);
  });
});
```

---

## Step 4: 앱 실행 & 테스트 수행

### 4.1 개발 서버 실행

```bash
# WEB
npm run dev &
DEV_PID=$!
sleep 5  # 서버 기동 대기

# ELECTRON
# Playwright Electron API가 자동으로 앱 실행

# TAURI
cargo tauri dev &
DEV_PID=$!
sleep 10

# FLUTTER
flutter run -d linux &  # 또는 windows, macos
DEV_PID=$!
sleep 15
```

### 4.2 E2E 테스트 실행

```bash
# Playwright (WEB/ELECTRON)
npx playwright test --reporter=html

# WebdriverIO (TAURI)
npx wdio run ./wdio.conf.ts

# Flutter
flutter test integration_test/

# API
npx playwright test --project=api
```

### 4.3 스크린샷 분석

테스트 중 캡처한 스크린샷을 시각적으로 분석:

```bash
# Playwright가 자동 저장한 스크린샷 확인
ls e2e/screenshots/ test-results/
```

스크린샷 파일을 Read 도구로 읽어 시각적 이상 확인 (Claude는 멀티모달).

---

## Step 5: 버그 리포트

### 5.1 실패 테스트 분석

```
## QA 테스트 결과

### 환경
- Platform: {WEB|ELECTRON|TAURI|FLUTTER|API}
- Tool: {Playwright|WebdriverIO|integration_test}
- App URL/Path: {url_or_path}

### 결과 요약
| # | 테스트 | 결과 | 시간 |
|---|--------|------|------|
| 1 | 로그인 흐름 | ✅ PASS | 2.3s |
| 2 | 대시보드 로드 | ✅ PASS | 1.8s |
| 3 | 파일 업로드 | ❌ FAIL | 5.2s |

**Pass Rate: 66.7% (2/3)**
```

### 5.2 버그 상세 리포트

```
### 🐛 Bug #1 — 파일 업로드 실패

**심각도:** 🔴 CRITICAL
**재현 경로:**
1. 로그인 → 대시보드 이동
2. "파일 업로드" 버튼 클릭
3. 10MB 이상 파일 선택
4. "업로드" 버튼 클릭

**기대 결과:** 업로드 진행률 표시 후 완료 메시지
**실제 결과:** 5초 후 타임아웃 에러, 콘솔에 "413 Payload Too Large"

**스크린샷:** e2e/screenshots/file-upload-error.png
**관련 코드:** src/api/upload.ts:42 — maxFileSize 미설정

**수정 제안:**
- src/api/upload.ts에 maxFileSize 설정 추가
- 프론트엔드에 파일 크기 사전 검증 추가
```

### 5.3 수정 적용

CRITICAL 버그의 수정을 적용할지 사용자에게 확인.
수정 적용 시:
1. 버그 재현 E2E 테스트가 이미 존재 (Step 3에서 작성됨)
2. 코드 수정
3. E2E 재실행으로 수정 검증
4. 전체 E2E 스위트 리그레션 확인

---

## Step 6: Agent Team 연동

QA Tester가 팀원으로 참여할 때의 워크플로우:

```
Lead (opus) — 전체 조율
├── Implementer (sonnet) — 코드 구현
└── QA Tester (sonnet) — E2E 작성 + 실행
     │
     ├── 1. 구현 시작 전: 사용자 흐름 기반 E2E 테스트 먼저 작성 (RED)
     ├── 2. 구현 완료 대기: Implementer 작업 완료 후
     ├── 3. E2E 실행: 앱 실행 → 전체 E2E 스위트 실행
     ├── 4. 스크린샷 검증: 시각적 이상 확인
     ├── 5. 버그 리포트: 실패 시 Lead에게 상세 리포트
     └── 6. 리그레션: 수정 후 전체 E2E 재실행
```

### QA Tester Spawn 프롬프트 (팀에서 사용)

```
You are the QA Tester. Your role is E2E testing from the user's perspective.

CRITICAL RULES:
1. ALWAYS write E2E tests, not unit tests
2. ALWAYS start the actual app and test against it
3. Use the right tool for the platform:
   - Web → Playwright
   - Electron → Playwright Electron API (_electron.launch)
   - Tauri → WebdriverIO
   - Flutter → integration_test
   - API → Playwright API testing or supertest
4. Capture screenshots at every major step
5. Test user FLOWS, not individual functions
6. Report bugs with: reproduction steps + screenshot + suggested fix
7. After implementation changes, run FULL E2E suite (regression)

SCOPE: e2e/ directory ONLY (read-only access to src/ for understanding)

WORKFLOW:
1. Write E2E tests for the feature BEFORE implementation starts (RED)
2. Wait for implementer to finish
3. Run E2E suite against running app
4. Report results to lead via SendMessage
```

---

## Rules

1. **E2E가 기본** — 단위 테스트가 아닌 E2E 테스트를 기본으로 작성. 사용자 흐름 기반
2. **플랫폼 자동 감지** — 프로젝트 타입을 자동 감지하여 적절한 E2E 도구 선택
3. **실제 앱 실행** — 항상 앱을 실행하고 테스트. 모킹 최소화
4. **스크린샷 필수** — 주요 단계마다 스크린샷 캡처. 시각적 검증에 활용
5. **Page Object Model** — 테스트 코드와 페이지 로직 분리. 유지보수성 확보
6. **독립 실행** — 각 테스트는 순서 무관하게 독립 실행 가능
7. **버그 리포트 필수 항목** — 재현 경로, 기대/실제 결과, 스크린샷, 관련 코드, 수정 제안
8. **리그레션 필수** — 코드 수정 후 반드시 전체 E2E 스위트 재실행
9. **서버 정리** — 테스트 완료 후 개발 서버 프로세스 종료
10. **데이터 격리** — 테스트별 고유 데이터 사용. 다른 테스트에 영향 없음
