# 13 — Reports, Trace Viewer, Screenshots, Video & Debugging

## 1. Built-in Reporters

| Reporter | Output | Best for |
| -------- | ------ | -------- |
| `list` | One line per test (default locally) | Local runs |
| `line` | Single updating line | Large suites locally |
| `dot` | `·` per test (default on CI) | CI logs |
| `html` | Interactive web report | Humans — the main report |
| `json` | JSON file | Custom dashboards |
| `junit` | XML | Jenkins, Azure DevOps, GitLab test tabs |
| `blob` | Binary zip | **Merging sharded reports** |
| `github` | Annotations on PR | GitHub Actions |
| `null` | Nothing | – |

```typescript
reporter: [
  ['list'],
  ['html', { outputFolder: 'playwright-report', open: 'on-failure' }], // 'always' | 'never' | 'on-failure'
  ['junit', { outputFile: 'results/junit.xml' }],
  ['json', { outputFile: 'results/results.json' }],
  process.env.CI ? ['github'] : ['null'],
],
```

```bash
npx playwright test --reporter=html
npx playwright show-report
npx playwright show-report playwright-report
```

### HTML report features
- Filter by passed / failed / flaky / skipped, by project, by tag
- Steps (`test.step`) with timings
- Errors with code snippet
- Attached screenshots, videos, **traces**, visual diffs
- Retries shown as separate attempts

---

## 2. Third-Party Reporters

### Allure

```bash
npm i -D allure-playwright allure-commandline
```

```typescript
reporter: [['line'], ['allure-playwright', { resultsDir: 'allure-results' }]],
```

```bash
npx playwright test
npx allure generate allure-results --clean -o allure-report
npx allure open allure-report
# or
npx allure serve allure-results
```

```typescript
import { allure } from 'allure-playwright';

test('checkout', async ({ page }) => {
  await allure.epic('Shop');
  await allure.feature('Checkout');
  await allure.severity('critical');
  await allure.owner('Swagatika');
  await allure.link('https://jira/PROJ-1', 'JIRA');
});
```

Others: **Monocart**, **ReportPortal**, **Currents**, **Testmo**, **Ortoni**.

### Custom reporter

```typescript
// my-reporter.ts
import type { Reporter, TestCase, TestResult, FullResult } from '@playwright/test/reporter';

class MyReporter implements Reporter {
  onBegin(config, suite) { console.log(`Running ${suite.allTests().length} tests`); }
  onTestEnd(test: TestCase, result: TestResult) { console.log(`${test.title}: ${result.status}`); }
  onEnd(result: FullResult) { console.log(`Finished: ${result.status}`); }
}
export default MyReporter;
```

```typescript
reporter: [['./my-reporter.ts']],
```

---

## 3. Screenshots

| Config value | Behaviour |
| ------------ | --------- |
| `'off'` | Never |
| `'on'` | After every test |
| `'only-on-failure'` | ✅ Only failed tests |
| `{ mode: 'only-on-failure', fullPage: true }` | With options |

```typescript
// Manual
await page.screenshot({ path: 'screens/home.png', fullPage: true });
await page.getByTestId('header').screenshot({ path: 'screens/header.png' });
const buffer = await page.screenshot();                       // in memory
await test.info().attach('home', { body: buffer, contentType: 'image/png' });  // into report
```

---

## 4. Video

| Value | Behaviour |
| ----- | --------- |
| `'off'` | No video |
| `'on'` | Every test |
| `'retain-on-failure'` | ✅ Record all, keep only failures |
| `'on-first-retry'` | Only when retried |

```typescript
use: { video: { mode: 'retain-on-failure', size: { width: 1280, height: 720 } } }

// Manual (library)
const context = await browser.newContext({ recordVideo: { dir: 'videos/' } });
// video saved when context closes
const path = await page.video()?.path();
```

---

## 5. Trace Viewer ⭐

A **trace** is a zip recording **everything** about a test: every action, DOM snapshots before/after, screenshots filmstrip, console logs, network calls, source code, errors. It's a **time-travel debugger** for CI failures.

| Trace value | Behaviour |
| ----------- | --------- |
| `'off'` | Never |
| `'on'` | Every test (heavy) |
| `'retain-on-failure'` | Record all, keep failures |
| `'on-first-retry'` | ✅ Recommended with retries on CI |
| `'on-all-retries'` | Every retry |
| `'retain-on-first-failure'` | Keep only first failed run |

```bash
npx playwright test --trace on
npx playwright show-trace test-results/<test>/trace.zip
# or drag into https://trace.playwright.dev (runs locally in browser, nothing uploaded)
```

### Trace Viewer panels

| Panel | Shows |
| ----- | ----- |
| **Actions** | Every step with locator + duration |
| **Timeline / Filmstrip** | Screenshots over time |
| **Before / After / Action** snapshots | Interactive DOM at that moment (inspect with devtools!) |
| **Source** | Line of code for the action |
| **Call** | Params, timings, auto-wait logs |
| **Log** | Actionability logs ("waiting for element to be visible…") |
| **Console** | Browser console |
| **Network** | Requests / responses |
| **Errors** | Stack trace |
| **Attachments** | Screenshots, diffs |

### Manual tracing (library)

```typescript
await context.tracing.start({ screenshots: true, snapshots: true, sources: true });
// ... actions ...
await context.tracing.stop({ path: 'trace.zip' });
```

> When a test fails on CI, I download the trace from the HTML report and open it in Trace Viewer — I can see the DOM at each step, network calls, console errors and exactly why the locator didn't match.

---

## 6. Debugging Techniques

| Technique | How | Use |
| --------- | --- | --- |
| **UI Mode** | `npx playwright test --ui` | Best dev experience: watch mode, time-travel, pick locator |
| **Inspector** | `npx playwright test --debug` | Step over actions, edit locators live |
| **`page.pause()`** | In code | Stop at a point (headed) |
| **VS Code extension** | Breakpoints, ▶ run/debug, "Show browser" | Everyday debugging |
| **Headed + slowMo** | `--headed`, `launchOptions: { slowMo: 500 }` | Watch it happen |
| **Trace Viewer** | `show-trace` | Post-mortem (CI) |
| **Verbose logs** | `DEBUG=pw:api` | Deep internals |
| **Console logs** | `page.on('console')` | App errors |
| **`locator.highlight()`** | In code | See what matched |

### UI Mode features
- Watch mode (re-run on save) 👁
- Filter by tag, project, status
- Timeline, DOM snapshot, network, console for each step
- **Pick locator** tool
- Run single test / describe

### Debug a single test

```bash
npx playwright test tests/login.spec.ts:15 --debug
npx playwright test -g "valid login" --headed --workers=1
```

### VS Code Playwright extension
- Run / debug tests from the gutter
- Set breakpoints
- **Pick locator**, **Record new**, **Record at cursor**
- Toggle projects, "Show browser", "Show trace viewer"

---

## 7. Common Failure Analysis Checklist

1. Read the **error message** — timeout? strict mode violation? assertion mismatch?
2. Open **screenshot** / **video** in HTML report.
3. Open **trace** → check the action's DOM snapshot & "Log" tab.
4. Check **network** tab — did the API fail / return unexpected data?
5. Check **console** — JS errors?
6. Run locally with `--ui` / `--debug` / `--repeat-each=10` to reproduce.

> **Interview one-liner:** I use the HTML reporter for humans, JUnit for CI dashboards and Allure for stakeholder reports. On CI I set `trace: 'on-first-retry'`, `screenshot: 'only-on-failure'`, `video: 'retain-on-failure'`. For debugging I use UI Mode locally, the Inspector with `--debug`, and Trace Viewer for CI failures.
