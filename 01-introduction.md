# 01 — Introduction to Playwright

## 1. What is Playwright?

**Playwright** is an open-source automation framework developed by **Microsoft** (first released in 2020) for reliable **end-to-end testing of modern web applications**. It drives real browsers using a single API and ships with its own test runner, **`@playwright/test`**.

| Supported Languages | Supported Browsers (engines) | Supported OS |
| ------------------- | ---------------------------- | ------------ |
| JavaScript / TypeScript | **Chromium** (Chrome, Edge) | Windows |
| Python | **Firefox** | macOS |
| Java | **WebKit** (Safari engine) | Linux |
| C# / .NET | Branded: `chrome`, `msedge` channels | Docker / CI |

> Playwright is an open-source, cross-browser automation framework from Microsoft. With one API it automates Chromium, Firefox and WebKit, supports JS/TS, Python, Java and .NET, and comes with a built-in test runner that has auto-waiting, web-first assertions, parallelism, tracing and reporting out of the box.

## 2. Key Features

- Web UI automation
- End-to-end testing
- API testing (`APIRequestContext`)
- Cross-browser testing (Chromium, Firefox, WebKit)
- Mobile / device emulation
- Network interception and mocking (`page.route`)
- Authentication / session reuse (`storageState`)
- Parallel execution (workers, sharding)
- Trace, screenshot and video capture
- Built-in web-first assertions with auto-retry
- **Auto-waiting** — no manual sleeps
- Test fixtures (dependency injection)
- Codegen (record & generate tests)
- UI Mode & Trace Viewer (time-travel debugging)
- CI/CD execution (headless, Docker images)
- Visual regression testing (`toHaveScreenshot`)
- Multiple tabs, iframes, popups, shadow DOM (pierced by default)
- Browser contexts — isolated, incognito-like sessions in one browser

---

## 3. Playwright Architecture

Playwright talks to browsers over a **single persistent WebSocket connection** using the browser's native debugging protocol (CDP for Chromium, patched protocols for Firefox/WebKit). Commands are sent over that open connection instead of one HTTP request per command.

```text
┌───────────────────┐   WebSocket (one persistent connection)   ┌───────────────┐
│  Test script (TS) │  ───────────────────────────────────────▶ │  Playwright   │
│  @playwright/test │  ◀─────────────────────────────────────── │  Server/Driver│
└───────────────────┘                                           └──────┬────────┘
                                                                        │ CDP / Juggler / WebKit protocol
                                        ┌───────────────┬───────────────┼───────────────┐
                                        ▼               ▼               ▼
                                    Chromium         Firefox          WebKit
```

**Object hierarchy:**

```text
Browser  ──▶  BrowserContext (isolated session: cookies, storage, cache)
                    └──▶  Page (a tab)
                              └──▶  Frame(s)
                                        └──▶  Locator ──▶ Element
```

### Playwright vs Selenium vs Cypress

| Feature | **Playwright** | **Selenium** | **Cypress** |
| ------- | -------------- | ------------ | ----------- |
| Protocol | WebSocket + CDP/native | W3C WebDriver (HTTP) | Runs inside the browser |
| Speed | Very fast | Slower | Fast |
| Auto-wait | ✅ Built-in | ❌ Explicit/implicit waits | ✅ Built-in |
| Browsers | Chromium, Firefox, WebKit | All major, incl. real Safari & IE | Chromium, Firefox, WebKit (experimental) |
| Languages | JS/TS, Python, Java, .NET | Java, Python, C#, JS, Ruby, Kotlin | JS/TS only |
| Multi-tab / multi-origin | ✅ | ✅ | ⚠️ Limited |
| iframes | ✅ Easy (`frameLocator`) | ✅ switchTo | ⚠️ Plugin needed |
| Network mocking | ✅ Built-in | ❌ (needs proxy / BiDi) | ✅ Built-in |
| API testing | ✅ Built-in | ❌ | ✅ `cy.request` |
| Test runner | ✅ Built-in | ❌ (TestNG/JUnit/pytest) | ✅ Built-in |
| Parallel | ✅ Free, built-in | Selenium Grid | Paid dashboard / plugins |
| Trace / time-travel | ✅ Trace Viewer | ❌ | ✅ Time-travel in runner |
| Mobile | Emulation only | Appium for real devices | Viewport only |

> Compared to Selenium, Playwright is faster because it uses a persistent WebSocket connection instead of HTTP per command, and it auto-waits so tests are less flaky. Compared to Cypress, it supports multiple tabs, multiple origins, multiple languages and free parallelism.

---

## 4. Installation

### Prerequisites
- **Node.js** LTS (18+ / 20+ / 22+)
- VS Code + **Playwright Test for VS Code** extension (recommended)

### New project

```bash
npm init playwright@latest
```

The wizard asks:
1. TypeScript or JavaScript → **TypeScript**
2. Tests folder → `tests`
3. Add GitHub Actions workflow → `y`
4. Install Playwright browsers → `y`

### Add to an existing project

```bash
npm i -D @playwright/test
npx playwright install              # download all browsers
npx playwright install chromium     # only chromium
npx playwright install --with-deps  # browsers + OS dependencies (Linux CI)
```

### Update

```bash
npm i -D @playwright/test@latest
npx playwright install
npx playwright --version
```

---

## 5. Project Structure

```text
my-project/
├── tests/                      # test specs  (*.spec.ts)
│   └── example.spec.ts
├── tests-examples/             # sample tests from the wizard
├── pages/                      # Page Objects
├── fixtures/                   # custom fixtures
├── utils/                      # helpers, data generators
├── test-data/                  # JSON / CSV data
├── playwright/.auth/           # saved storageState (gitignored)
├── playwright-report/          # HTML report (generated)
├── test-results/               # traces, screenshots, videos (generated)
├── playwright.config.ts        # configuration
├── package.json
└── .github/workflows/playwright.yml
```

---

## 6. First Test

```typescript
import { test, expect } from '@playwright/test';

test('has title', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await expect(page).toHaveTitle(/Playwright/);
});

test('get started link', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await page.getByRole('link', { name: 'Get started' }).click();
  await expect(page.getByRole('heading', { name: 'Installation' })).toBeVisible();
});
```

Run it:

```bash
npx playwright test
npx playwright show-report
```

### Key points
- `test` and `expect` come from `@playwright/test`.
- `{ page }` is a **built-in fixture** — a fresh page in a fresh context for every test (full isolation).
- Every Playwright call is **async** → always `await`.

⚠️ **Most common beginner bug:** forgetting `await` — the test finishes before the action and fails randomly.

---

## 7. Playwright Library vs Playwright Test

| `playwright` (library) | `@playwright/test` (test runner) |
| ---------------------- | -------------------------------- |
| Browser automation API only | Library + runner + assertions + fixtures |
| You manage browser launch/close | Runner manages browser/context/page |
| No `expect`, reports, retries | Built-in `expect`, reporters, retries, parallelism |
| Use for scraping, scripts, bots | Use for **testing** |

```typescript
// Library style
import { chromium } from 'playwright';

(async () => {
  const browser = await chromium.launch({ headless: false });
  const context = await browser.newContext();
  const page = await context.newPage();
  await page.goto('https://example.com');
  console.log(await page.title());
  await browser.close();
})();
```

> `playwright` is the automation library; `@playwright/test` is the full test framework built on top of it. For testing, always use `@playwright/test`.
