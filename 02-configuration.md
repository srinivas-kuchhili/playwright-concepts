# 02 — Configuration (`playwright.config.ts`)

`playwright.config.ts` is the central file that controls **where tests live, how they run, which browsers are used, timeouts, retries, reporters and default browser options**.

---

## 1. Complete Annotated Config

```typescript
import { defineConfig, devices } from '@playwright/test';
import dotenv from 'dotenv';
import path from 'path';

// Load env file: ENV=qa npx playwright test
dotenv.config({ path: path.resolve(__dirname, `.env.${process.env.ENV ?? 'qa'}`) });

export default defineConfig({
  // ---------- Test discovery ----------
  testDir: './tests',
  testMatch: '**/*.spec.ts',          // default: **/*.@(spec|test).?(c|m)[jt]s?(x)
  testIgnore: '**/legacy/**',
  outputDir: 'test-results',          // traces, screenshots, videos

  // ---------- Execution ----------
  fullyParallel: true,                // run tests inside a file in parallel too
  workers: process.env.CI ? 2 : undefined, // undefined = 50% of CPU cores
  retries: process.env.CI ? 2 : 0,
  forbidOnly: !!process.env.CI,       // fail CI if test.only is committed
  maxFailures: process.env.CI ? 10 : undefined, // stop after N failures
  repeatEach: 0,                      // repeat each test N times (flakiness check)

  // ---------- Timeouts ----------
  timeout: 30_000,                    // per test (default 30s)
  globalTimeout: 60 * 60 * 1000,      // whole run
  expect: {
    timeout: 5_000,                   // per assertion (default 5s)
    toHaveScreenshot: { maxDiffPixelRatio: 0.01 },
  },

  // ---------- Reporters ----------
  reporter: [
    ['list'],
    ['html', { outputFolder: 'playwright-report', open: 'never' }],
    ['junit', { outputFile: 'results/junit.xml' }],
    ['json', { outputFile: 'results/results.json' }],
  ],

  // ---------- Global setup / teardown ----------
  globalSetup: require.resolve('./global-setup'),
  globalTeardown: require.resolve('./global-teardown'),

  // ---------- Shared options for all projects ----------
  use: {
    baseURL: process.env.BASE_URL ?? 'http://localhost:3000',
    headless: true,
    viewport: { width: 1280, height: 720 },
    ignoreHTTPSErrors: true,
    actionTimeout: 10_000,            // click/fill etc. (default 0 = no limit)
    navigationTimeout: 30_000,        // goto/waitForURL
    testIdAttribute: 'data-testid',   // used by getByTestId
    locale: 'en-US',
    timezoneId: 'Asia/Kolkata',

    // Artifacts
    trace: 'on-first-retry',          // 'off' | 'on' | 'retain-on-failure' | 'on-first-retry' | 'on-all-retries'
    screenshot: 'only-on-failure',    // 'off' | 'on' | 'only-on-failure'
    video: 'retain-on-failure',       // 'off' | 'on' | 'retain-on-failure' | 'on-first-retry'

    extraHTTPHeaders: { 'x-test-run': 'playwright' },
    launchOptions: { slowMo: 0 },
  },

  // ---------- Projects (browsers / environments / setups) ----------
  projects: [
    { name: 'setup', testMatch: /.*\.setup\.ts/ },

    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'], storageState: 'playwright/.auth/user.json' },
      dependencies: ['setup'],
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'], storageState: 'playwright/.auth/user.json' },
      dependencies: ['setup'],
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'], storageState: 'playwright/.auth/user.json' },
      dependencies: ['setup'],
    },
    { name: 'Mobile Chrome', use: { ...devices['Pixel 7'] } },
    { name: 'Mobile Safari', use: { ...devices['iPhone 15'] } },
    { name: 'Google Chrome', use: { ...devices['Desktop Chrome'], channel: 'chrome' } },
    { name: 'Microsoft Edge', use: { ...devices['Desktop Edge'], channel: 'msedge' } },
    { name: 'api', testDir: './tests/api', use: { baseURL: process.env.API_URL } },
  ],

  // ---------- Start app before tests ----------
  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
});
```

---

## 2. Most Important Options — Quick Table

| Option | Level | Default | Purpose |
| ------ | ----- | ------- | ------- |
| `testDir` | top | `.` | Where tests are |
| `timeout` | top | 30000 | Max time per test |
| `expect.timeout` | top | 5000 | Max time per assertion retry |
| `retries` | top | 0 | Re-run failed tests |
| `workers` | top | 50% cores | Parallel processes |
| `fullyParallel` | top | false | Parallelize tests within a file |
| `reporter` | top | `list` (`dot` on CI) | Output format |
| `projects` | top | – | Browsers / env variations |
| `webServer` | top | – | Start local server first |
| `baseURL` | `use` | – | Allows `page.goto('/login')` |
| `headless` | `use` | true | Show/hide browser |
| `actionTimeout` | `use` | 0 (none) | Per action |
| `navigationTimeout` | `use` | 0 (none) | Per navigation |
| `trace` / `screenshot` / `video` | `use` | off | Artifacts |
| `storageState` | `use` | – | Pre-authenticated state |
| `viewport` | `use` | 1280×720 | Window size |

---

## 3. Config Precedence (highest → lowest)

```text
test.use() inside a spec / describe
        ▼
project.use
        ▼
top-level use
        ▼
Playwright defaults
```

CLI flags (`--workers`, `--retries`, `--headed`, `--project`) override config values.

```typescript
// Override for one file / describe block
test.use({ viewport: { width: 375, height: 667 }, locale: 'fr-FR' });
```

---

## 4. Projects — Why & How

A **project** is a logical group of tests that run with the **same configuration**. Use projects for:

| Use case | Example |
| -------- | ------- |
| Cross-browser | chromium / firefox / webkit |
| Devices | Desktop vs Mobile |
| Environments | `qa`, `staging` with different `baseURL` |
| Setup dependencies | `setup` project logs in before others |
| Split test types | `ui` vs `api` vs `visual` |

```bash
npx playwright test --project=chromium
npx playwright test --project=chromium --project=firefox
```

### Project dependencies & teardown

```typescript
projects: [
  { name: 'setup db', testMatch: /global\.setup\.ts/, teardown: 'cleanup db' },
  { name: 'cleanup db', testMatch: /global\.teardown\.ts/ },
  { name: 'chromium', use: devices['Desktop Chrome'], dependencies: ['setup db'] },
]
```

> 💡 Project dependencies are preferred over `globalSetup` because setup runs appear in the report, produce traces, and can use fixtures.

---

## 5. Global Setup vs Setup Project

| `globalSetup` | Setup project (`dependencies`) |
| ------------- | ------------------------------ |
| Plain function run once before all | A normal test file run as a project |
| No fixtures (`page`) — must launch browser manually | Full fixtures available |
| Not visible in HTML report, no trace | Visible in report, trace supported |
| Legacy style | ✅ Recommended |

```typescript
// global-setup.ts
import { chromium, FullConfig } from '@playwright/test';

export default async function globalSetup(config: FullConfig) {
  const { baseURL } = config.projects[0].use;
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto(`${baseURL}/login`);
  await page.getByLabel('Username').fill(process.env.USER!);
  await page.getByLabel('Password').fill(process.env.PASS!);
  await page.getByRole('button', { name: 'Sign in' }).click();
  await page.context().storageState({ path: 'playwright/.auth/user.json' });
  await browser.close();
}
```

---

## 6. Environment Variables & Multiple Environments

```text
.env.qa        BASE_URL=https://qa.myapp.com
.env.staging   BASE_URL=https://staging.myapp.com
```

```bash
# bash / mac / linux
ENV=staging npx playwright test

# Windows PowerShell
$env:ENV="staging"; npx playwright test

# cross-platform (npm i -D cross-env)
npx cross-env ENV=staging playwright test
```

`package.json` scripts:

```json
{
  "scripts": {
    "test": "playwright test",
    "test:headed": "playwright test --headed",
    "test:ui": "playwright test --ui",
    "test:chrome": "playwright test --project=chromium",
    "test:smoke": "playwright test --grep @smoke",
    "test:staging": "cross-env ENV=staging playwright test",
    "report": "playwright show-report",
    "codegen": "playwright codegen"
  }
}
```

---

## 7. `tsconfig.json` Path Aliases (clean imports)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "baseUrl": ".",
    "paths": {
      "@pages/*": ["pages/*"],
      "@fixtures/*": ["fixtures/*"],
      "@utils/*": ["utils/*"]
    }
  }
}
```

```typescript
import { LoginPage } from '@pages/LoginPage';
```

> Playwright reads `tsconfig.json` `paths` automatically — no extra bundler needed.

---

> **Interview one-liner:** `playwright.config.ts` controls test discovery, parallelism, retries, timeouts, reporters and default browser options via `use`. I use **projects** for cross-browser and environment runs, **setup projects with dependencies** for login, **webServer** to boot the app, and env-driven `baseURL` for multiple environments.
