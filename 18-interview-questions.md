# 18 — Interview Questions & Answers

> Answers are written the way you'd **say** them. Links point to the detailed topic file.

---

## 🟢 Basic

**1. What is Playwright?**
An open-source end-to-end automation framework by Microsoft. One API drives Chromium, Firefox and WebKit; it supports JS/TS, Python, Java and .NET; and `@playwright/test` adds a runner with auto-waiting, web-first assertions, fixtures, parallelism, tracing and reporting. → [01](01-introduction.md)

**2. Why Playwright over Selenium?**
- Persistent WebSocket connection instead of HTTP per command → faster
- Built-in auto-waiting → fewer flaky tests
- Built-in runner, assertions, reports, parallelism, trace viewer
- Network mocking, API testing, multi-tab, iframes and shadow DOM out of the box
- Browser contexts give cheap, isolated sessions

Selenium still wins on real legacy browsers, a larger ecosystem, and real-device testing via Appium.

**3. Which browsers does Playwright support?**
Chromium (Chrome, Edge), Firefox and WebKit (the Safari engine). Branded Chrome and Edge are available via `channel`.

**4. What is auto-waiting?**
Before an action, Playwright waits until the element is attached, visible, stable, enabled, editable where needed, and receiving events. Assertions retry until they pass or time out. → [05](05-actions.md#1-actionability-checks-auto-wait)

**5. What is a Locator? How is it different from ElementHandle?**
A locator is lazy, strict and auto-waiting, and it re-queries the DOM on every use, so it never goes stale. An ElementHandle points to one fixed node and can go stale. → [04](04-locators.md)

**6. What locator strategy do you prefer?**
`getByRole` first, then `getByLabel` / `getByPlaceholder` / `getByText`, then `getByTestId`. CSS or XPath is a last resort.

**7. What is strict mode?**
If an action's locator matches more than one element, Playwright throws a "strict mode violation" error instead of clicking the first match.

**8. Difference between Browser, BrowserContext and Page?**
Browser = the process. Context = an isolated incognito-like session. Page = a tab. → [07](07-browser-context-page.md)

**9. What are the default timeouts?**
Test: 30s. Expect: 5s. Action and navigation: no limit of their own (bounded by the test timeout). → [06](06-assertions-and-waits.md#10-timeouts--complete-picture)

**10. How do you run tests in headed mode or a single browser?**
`npx playwright test --headed --project=chromium`

**11. `fill()` vs `pressSequentially()`?**
`fill` sets the value instantly after clearing the field. `pressSequentially` types one key at a time and fires key events, which autocomplete and masked inputs need. `type()` is deprecated.

**12. What is codegen?**
`npx playwright codegen <url>` records your interactions and generates test code with good locators.

**13. What is `baseURL`?**
A config option that lets you write `page.goto('/login')`. It keeps tests independent of the environment.

**14. How do you handle dropdowns?**
Native `<select>`: `selectOption()` by value, label or index. Custom dropdowns: click the combobox, then click the option by role.

**15. How do you take a screenshot?**
`page.screenshot({ path, fullPage: true })`, `locator.screenshot()`, or automatically with `screenshot: 'only-on-failure'` in config.

---

## 🟡 Intermediate

**16. How do you handle alerts / confirm / prompt?**
With `page.on('dialog')`, `page.once('dialog')` or `page.waitForEvent('dialog')`, then call `dialog.accept(text?)` or `dialog.dismiss()`. Without a listener, Playwright auto-dismisses dialogs. Once a listener exists, you must handle the dialog or the action hangs. → [07](07-browser-context-page.md#5-javascript-dialogs-alert--confirm--prompt)

**17. How do you handle iframes?**
`page.frameLocator('#id').getByRole(...)` or `locator.contentFrame()`. No switching back and forth like Selenium's `switchTo()`.

**18. How do you handle new tabs / popups?**
Start `context.waitForEvent('page')` or `page.waitForEvent('popup')` before the click, then await it.

**19. How do you upload / download files?**
Upload: `setInputFiles()`, or the `filechooser` event if there's no input. Download: `waitForEvent('download')`, then `download.saveAs()`.

**20. What are web-first assertions?**
Async `expect` on locators and pages that auto-retries until the expect timeout. Example: `await expect(loc).toBeVisible()`, not `expect(await loc.isVisible()).toBe(true)`.

**21. What are soft assertions?**
`expect.soft()` records a failure without stopping the test. All failures are reported at the end.

**22. Hooks available?**
`beforeAll`, `beforeEach`, `afterEach`, `afterAll`. `beforeAll` runs once per worker, not once per run.

**23. What are fixtures? Why use them over hooks?**
Fixtures are dependency injection: reusable setup and teardown that runs only when a test requests it. They are composable and can be test-scoped or worker-scoped, auto, or options. → [08](08-test-structure-and-fixtures.md#9-fixtures)

**24. How do you share login across tests?**
A setup project logs in and saves `storageState`. Other projects load it via `use.storageState` and `dependencies`. → [09](09-authentication.md)

**25. How do you run tests in parallel?**
Files run in parallel across workers by default. `fullyParallel: true` also parallelizes tests within a file. Control the count with `workers`, and use `--shard` across machines.

**26. How do you tag and filter tests?**
`test('x', { tag: '@smoke' }, ...)`, then run `--grep @smoke` or `--grep-invert @slow`.

**27. How do you do data-driven testing?**
Loop over an array, JSON or CSV and create a `test()` for each item with a unique title. Or use option fixtures per project.

**28. What is `test.step`?**
It groups actions into named steps that appear in reports and traces, which improves readability.

**29. Explain `test.skip`, `test.fixme`, `test.fail`, `test.slow`.**
- skip: don't run
- fixme: skip and mark as needing a fix
- fail: expected to fail (the test fails if it unexpectedly passes)
- slow: triple the timeout

**30. What reporters have you used?**
list, dot, html, junit (for CI), json, blob (for shards), and Allure for stakeholders. → [13](13-reports-and-debugging.md)

**31. What is the Trace Viewer?**
A time-travel debugger. A trace zip holds actions, DOM snapshots, network, console and source. I set `trace: 'on-first-retry'` on CI and open traces with `show-trace`.

**32. How do you debug a failing test?**
Read the error. Check the screenshot and video. Open the trace. Reproduce with `--ui`, `--debug` or `page.pause()`, and use `--repeat-each` for flaky ones.

**33. What is POM? How do you implement it in Playwright?**
Page classes hold locators and actions, and fixtures inject them into tests. → [14](14-pom-framework-design.md)

---

## 🔴 Advanced

**34. How do you mock an API?**
`page.route(url, route => route.fulfill({ status, json }))`. To patch a real response: `route.fetch()`, modify it, then `fulfill({ response, json })`. Use `route.abort()` to block requests and `routeFromHAR` for record/replay. → [10](10-network-mocking.md)

**35. How do you verify the payload the UI sends to the backend?**
Start `page.waitForRequest(predicate)` before the action, then assert on `request.postDataJSON()`.

**36. How do you do API testing?**
With the `request` fixture (`get`/`post`/`put`/`patch`/`delete`), asserting status, headers, body and schema (Ajv/Zod). Use `page.request` to share browser cookies. → [11](11-api-testing.md)

**37. How do you combine API and UI in one test?**
Create data via API, verify it in the UI, and clean up via API. Or perform a UI action and verify via `page.request`.

**38. How does visual testing work? How do you avoid flaky visuals?**
`toHaveScreenshot()` compares against a baseline stored per browser and OS. To reduce flakiness: mask dynamic elements, disable animations, freeze time with `page.clock`, mock data, use the same Docker image as CI, and set tolerances like `maxDiffPixelRatio`. → [12](12-visual-testing.md)

**39. `expect.poll` vs `toPass`?**
`poll` retries a function that returns a value, with one matcher. `toPass` retries a whole block containing several assertions (its default timeout is 0, so set one).

**40. How do you handle flaky tests?**
Reproduce with `--repeat-each`, analyse the trace, and fix the root cause: missing awaits, non-retrying checks, races with network calls, shared data, animations. Retries plus trace on retry are only a safety net. → [15](15-parallel-retries-sharding.md#4-flaky-tests--causes--fixes)

**41. What's the difference between workers and shards?**
Workers are parallel processes on one machine. Shards split the suite across machines (`--shard=1/4`), and `merge-reports` combines the blob reports.

**42. How do you test multiple users interacting (chat, admin/user)?**
Create two contexts from the `browser` fixture, each with its own `storageState`, and drive both pages in one test.

**43. How do you handle random popups or cookie banners?**
`page.addLocatorHandler(locator, handler)` dismisses the overlay automatically before actions.

**44. How do you test session timeout / date-dependent features?**
`page.clock.install()`, then `fastForward('30:00')`, or `setFixedTime()`.

**45. How do you test on mobile?**
`devices['iPhone 15']` / `devices['Pixel 7']` in projects (emulation). For real devices, use Appium or a cloud grid.

**46. How do you manage multiple environments?**
`.env.qa` / `.env.staging` loaded with dotenv based on an `ENV` variable, which drives `baseURL`, credentials and API URL. Secrets come from CI.

**47. What is `globalSetup` vs a setup project?**
`globalSetup` is a plain function: no fixtures, no trace, not shown in the report. A setup project is a real test with fixtures and traces, visible in the report, and wired with `dependencies`/`teardown`. Setup projects are recommended.

**48. Worker-scoped vs test-scoped fixtures?**
Test-scoped fixtures are set up for each test. Worker-scoped ones are set up once per worker process and shared, which suits expensive resources like DB connections or per-worker accounts.

**49. How do you write a custom matcher?**
`expect.extend({ async toHaveAmount(locator, expected) { ...return { pass, message, name } } })`.

**50. How do you run the same tests as admin and as user?**
An option fixture `role: ['user', { option: true }]` plus two projects that set `use: { role }`. Or two storageState files with `test.use`.

**51. How does Playwright handle Shadow DOM?**
CSS and `getBy*` locators pierce open shadow roots automatically. XPath doesn't, and closed shadow roots are inaccessible.

**52. How do you test in CI?**
Official Docker image or `install --with-deps`, headless, retries, `forbidOnly`, sharding, JUnit plus HTML artifacts, traces on retry, secrets as env vars. → [17](17-ci-cd.md)

**53. What is `page.request` vs the `request` fixture?**
`page.request` shares cookies and storage with the browser context. The `request` fixture is a separate context that still uses config options like `baseURL` and headers.

**54. Can Playwright do performance or accessibility testing?**
Accessibility: yes, via `@axe-core/playwright` and `toMatchAriaSnapshot`. Performance: it can collect metrics (Navigation Timing, CDP, Lighthouse via plugins), but load testing needs k6 or JMeter.

**55. What happens when you forget `await`?**
The promise floats. The test may finish before the action, and errors surface randomly or not at all. ESLint's `no-floating-promises` and `playwright/missing-playwright-await` catch it.

---

## 🧩 Scenario-Based

**S1. The Login button sometimes isn't clicked on CI.**
Check the trace. It's usually an overlay intercepting the click ("element intercepts pointer events"), an animation, or a check done before the page is ready. Fix it with an assertion that waits for the right state, `addLocatorHandler` for overlays, or a better locator. Avoid `force: true` unless it's justified.

**S2. A table has 100 rows. Click "Edit" for the row where Email = x@y.com.**
```typescript
await page.getByRole('row').filter({ hasText: 'x@y.com' }).getByRole('button', { name: 'Edit' }).click();
```

**S3. Verify all product prices are sorted ascending.**
```typescript
await expect(page.locator('.price').first()).toBeVisible();
const prices = (await page.locator('.price').allTextContents()).map(p => Number(p.replace(/[^0-9.]/g, '')));
expect(prices).toEqual([...prices].sort((a, b) => a - b));
```

**S4. Verify every link on the page is not broken.**
```typescript
const hrefs = await page.locator('a[href^="http"]').evaluateAll(as => as.map(a => (a as HTMLAnchorElement).href));
for (const url of new Set(hrefs)) {
  const res = await page.request.get(url);
  expect.soft(res.status(), url).toBeLessThan(400);
}
```

**S5. The backend isn't ready, but the UI must be tested.**
Mock the endpoints with `page.route().fulfill()` from JSON fixtures, or `routeFromHAR`.

**S6. Tests pass locally and fail on CI.**
Differences in speed, screen size, fonts or OS, env vars and data, headless mode, or parallel data collisions. Use the trace, the same Docker image, an explicit viewport, and unique data per worker.

**S7. Verify the "Save" button calls the API with the correct body and the UI shows success.**
```typescript
const reqPromise = page.waitForRequest(r => r.url().endsWith('/api/profile') && r.method() === 'PUT');
await page.getByRole('button', { name: 'Save' }).click();
expect((await reqPromise).postDataJSON()).toMatchObject({ city: 'Pune' });
await expect(page.getByRole('alert')).toHaveText('Profile updated');
```

**S8. Test a download and verify its contents.**
```typescript
const dl = page.waitForEvent('download');
await page.getByRole('button', { name: 'Export CSV' }).click();
const file = await (await dl).path();
const content = fs.readFileSync(file!, 'utf-8');
expect(content).toContain('Order ID');
```

**S9. Test a feature that shows a banner only on Christmas.**
```typescript
await page.clock.setFixedTime(new Date('2025-12-25T10:00:00'));
await page.goto('/');
await expect(page.getByText('Merry Christmas')).toBeVisible();
```

**S10. Speed up a 2-hour suite.**
- Reuse `storageState`
- Set up data via API
- `fullyParallel` and more workers
- Shard in CI
- Block images and analytics
- Trace only on retry
- Smoke on PRs, full suite nightly
- Remove hard waits
- `--only-changed` locally

**S11. Handle an element that appears only sometimes (an optional popup).**
```typescript
await page.addLocatorHandler(page.getByRole('dialog', { name: 'Newsletter' }), async d => {
  await d.getByRole('button', { name: 'Close' }).click();
});
```

**S12. Scroll until an element is loaded (infinite scroll).**
Scroll with `mouse.wheel` in a loop until `count()` reaches the target, or wait for the paging API with `waitForResponse`.

---

## 🎤 "Tell me about your framework" — Sample Answer

> "I built a Playwright + TypeScript framework using the **Page Object Model** with component objects, injected through **custom fixtures**. Config is environment-driven with dotenv files, and **projects** cover Chromium, Firefox, WebKit and mobile emulation. Authentication happens once in a **setup project** that saves `storageState`, with separate state files for admin and user roles. Test data is created through **API clients** using Playwright's request context and Faker, and cleaned up afterwards. Tests are tagged `@smoke` / `@regression` and filtered with `--grep`. I use **network mocking** for edge cases like 500 errors or empty states, and `toHaveScreenshot` for key visual components. Reporting is HTML plus Allure, with JUnit for CI. On CI (GitHub Actions) we run headless in the official Docker image, sharded across 4 machines with merged blob reports, retries of 2, and **trace on first retry** for debugging. ESLint with the Playwright plugin enforces awaits and web-first assertions."
