# Playwright Interview Questions and Answers
---

## Playwright Fundamentals

### 1. What is Playwright?
Playwright is a modern browser automation framework developed by Microsoft for end-to-end testing of web applications. It supports Chromium, Firefox, and WebKit and allows automation for UI workflows, API validation, network interception, and browser-level testing.

In real projects, Playwright is used for login flows, form validation, checkout flows, regression testing, and cross-browser testing. It is widely preferred because of built-in auto-waiting, reliable locators, and strong debugging tools.

### 2. What are the key features of Playwright?
Key features include:
- Cross-browser automation using Chromium, Firefox, and WebKit
- Auto-waiting for elements and actions
- Reliable locator strategies such as getByRole, getByLabel, getByText
- Network interception and API mocking using route()
- Handling popups, dialogs, tabs, files, uploads, and downloads
- Built-in tracing, screenshots, video recording, and debugging
- Support for both UI and API testing
- Parallel execution and CI/CD readiness

### 3. Which programming languages does Playwright support?
Playwright supports JavaScript/TypeScript, Python, Java, and .NET. In most enterprise setups, TypeScript is the most common because it gives better maintainability and IDE support.

For 4–6 years experience, it is important to mention that the framework is language-agnostic at the core, and the engineering choice depends on the tech stack and team preference.

### 4. How does Playwright differ from Selenium?
Playwright is a newer and more modern automation framework compared to Selenium.

Main differences:
- Playwright has built-in auto-waiting, Selenium usually requires explicit waits
- Playwright supports multiple browsers consistently with a single API
- Playwright offers better handling for frames, dialogs, popups, network mocking, and browser context isolation
- Playwright has excellent trace and debugging capabilities
- Selenium is still useful and widely used, but Playwright is generally more reliable for modern web apps

A good interview answer is: “Playwright gives better out-of-the-box stability for modern web apps, while Selenium is more traditional and often requires more custom logic for waits and browser interactions.”

### 5. What are the main components of Playwright?
The main components are:
- Browser: actual browser instance
- BrowserContext: isolated session with separate cookies, storage, and permissions
- Page: tab or document
- Locator: object to find elements on a page
- APIRequestContext: for API testing
- expect: assertion library for web-first assertions

These components combine to make Playwright a robust automation framework for UI and API testing.

### 6. What is a BrowserContext?
A BrowserContext is an isolated browser session. It behaves like a fresh profile and has its own cookies, local storage, session data, and permissions.

This is important because tests should not share state. Each test should create a new context for isolation.

```ts
const browser = await chromium.launch();
const context = await browser.newContext();
const page = await context.newPage();
```

This is better than reusing a single browser session because it avoids data leakage between tests.

### 7. What is a Locator?
A Locator is Playwright’s way of identifying and interacting with elements in the DOM. It is more modern and stable than raw CSS selectors or XPath in most cases.

Examples:

```ts
await page.getByRole('textbox', { name: 'Username' }).fill('admin');
await page.getByLabel('Password').fill('secret');
await page.getByText('Login').click();
```

A good locator strategy is to prefer user-facing selectors like role, label, and text before CSS/XPath.

### 8. How does auto-waiting work in Playwright?
Playwright automatically waits for elements to become actionable before performing an action like click, fill, or select.

This reduces flaky tests because the framework waits for a condition such as visibility, enabled state, non-stale state, or network idle before proceeding.

Example:

```ts
await page.getByRole('button', { name: 'Submit' }).click();
```

Playwright will wait for the button to be visible and enabled before clicking. This is one of the biggest advantages over older automation frameworks.

### 9. What is the purpose of expect?
The expect API in Playwright is used for assertions. It is built to work with web-first assertions and automatically retries until the condition is satisfied.

Example:

```ts
await expect(page.getByText('Welcome')).toBeVisible();
await expect(page).toHaveURL(/dashboard/);
await expect(page.locator('#error')).toContainText('Invalid credentials');
```

This makes tests more reliable, especially when the UI loads asynchronously.

### 10. What is the difference between Page and BrowserContext?
A Page represents a single tab or document. A BrowserContext represents an isolated browser session that can contain multiple pages.

Example:

```ts
const context = await browser.newContext();
const page1 = await context.newPage();
const page2 = await context.newPage();
```

A context can host several pages, but each page shares the same browser session boundaries. The browser context isolates cookies, storage, and permissions, which is critical for test independence.

---

## Browser and Page Handling

### 1. How do you launch and manage browsers in Playwright?
You launch a browser using the Playwright library and then create a context and page.

```ts
import { chromium, test } from '@playwright/test';

test('launch browser', async () => {
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext();
  const page = await context.newPage();

  await page.goto('https://example.com');
  await browser.close();
});
```

For professional automation, we usually keep browser launch logic inside fixtures or test setup so that there is a reusable and consistent framework pattern.

### 2. What is headless mode?
Headless mode runs the browser without a visible UI. It is commonly used in CI/CD and fast test execution.

```ts
const browser = await chromium.launch({ headless: true });
```

Headed mode is useful for debugging and local troubleshooting. A common interview answer is: “Headless is faster and suitable for automation runs, while headed mode is preferred when debugging visual or interaction issues.”

### 3. How do you capture screenshots?
Use page.screenshot() or browserContext.screenshot().

```ts
await page.screenshot({ path: 'screenshot.png', fullPage: true });
```

In real projects, screenshots are helpful for debugging failures, capturing visual regressions, and attaching evidence to CI reports.

### 4. How do you record test execution videos?
Video recording is configured in the Playwright config or test use block.

```ts
use: {
  video: 'on-first-retry'
}
```

This is useful to review failed tests and understand the exact sequence of UI actions in a failing scenario.

### 5. How do you handle multiple tabs or pages?
You can use context.waitForEvent('page') and then interact with the new page.

```ts
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('link', { name: 'Open' }).click()
]);

await newPage.waitForLoadState('domcontentloaded');
```

This is often used for OAuth flows, new user registrations, or external payment pages.

### 6. How do you handle browser popups?
Browser popups are often handled by waiting for a new page or by using browserContext to accept or control permissions.

Example:

```ts
const [popup] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('button', { name: 'Open Popup' }).click()
]);
```

For authentication or permission popups, we usually handle them using browser permissions or popup page interactions depending on the app behavior.

### 7. How do you work with frames and iframes?
Use frameLocator or page.frame().

```ts
const paymentFrame = page.frameLocator('#payment-frame');
await paymentFrame.getByRole('button', { name: 'Pay Now' }).click();
```

This is important when an application embeds payment or widget components inside an iframe.

### 8. How do you execute tests in parallel?
Playwright supports parallel execution at the project or test level. In config, you can set workers.

```ts
export default defineConfig({
  workers: 4,
});
```

For enterprise test suites, we usually balance workers with execution time, browser resources, and test stability. Running too many tests in parallel can create flakiness if tests share state or resources.

### 9. How do you configure test retries?
Retries are configured in Playwright config.

```ts
export default defineConfig({
  retries: 2,
});
```

This is useful for flaky UI behaviors, but retries should not be used as a blanket fix for unstable tests. The better approach is to fix root cause and use retries only as a safeguard.

### 10. How do you generate test reports?
Playwright supports HTML report generation via config.

```ts
reporter: [['html', { open: 'never' }]]
```

This gives rich test results, screenshots, traces, and summary data. In CI/CD, this is useful for quick debugging of failed builds and test summary reporting.

---

## API, Network, and Authentication

### 1. How do you capture API responses?
Use the request fixture or page.waitForResponse() for UI-triggered API calls.

```ts
const [response] = await Promise.all([
  page.waitForResponse('**/api/users'),
  page.getByRole('button', { name: 'Load Data' }).click()
]);

const status = response.status();
console.log(status);
```

We also use request for direct API validation.

### 2. How do you mock API requests and responses?
Use page.route() or context.route() to intercept network calls.

```ts
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([{ id: 1, name: 'Test User' }])
  });
});
```

This is very useful for testing edge conditions without depending on the backend service.

### 3. How do you handle authentication?
There are multiple approaches:
- Use login flow in setup before tests
- Reuse saved storage state
- Use API login for token generation and then continue UI flows

Example for storage state:

```bash
npx playwright test --storage-state=storage.json
```

A senior automation engineer usually prefers a stable, reusable auth strategy that avoids repetition and reduces test runtime.

### 4. How do you manage cookies?
Use browserContext to manage cookies and session state.

```ts
await context.addCookies([
  { name: 'session', value: '123', domain: 'example.com', path: '/' }
]);
```

This is commonly used for login-state reuse and session validation.

### 5. How do you handle sessions and browser storage?
Session state is managed through BrowserContext, which has isolated cookies, localStorage, and sessionStorage. For reuse, we can save storage state and load it in subsequent tests.

```ts
await context.storageState({ path: 'storage.json' });
```

This is useful when login is a common setup step across many tests.

### 6. How do you test WebSockets?
Playwright has support for WebSocket inspection via page.waitForEvent('websocket') and ws events.

```ts
const ws = await page.waitForEvent('websocket');
```

This is valuable when testing live dashboards, chat apps, or real-time notifications.

### 7. How do you simulate network failures?
Use route() to abort or fulfill requests, or use browser context to emulate offline status.

```ts
await page.context().setOffline(true);
```

This helps validate failure handling, error states, retry experiences, and offline behavior.

### 8. How do you test under slow network conditions?
Playwright supports route-based throttling or browser context emulation. We commonly set a custom network condition using browserContext.route or chromium launch arguments depending on the use case.

Example:

```ts
await page.route('**/*', async route => {
  await new Promise(r => setTimeout(r, 200));
  await route.continue();
});
```

This is useful for testing loading states, spinner behavior, and resilience under slow connectivity.

---

## Common Automation Scenarios

### 1. What is Page Object Model (POM) in Playwright?
POM is a design pattern where page-specific selectors and actions are encapsulated in dedicated classes. This keeps tests cleaner and reduces duplication.

Example:

```ts
class LoginPage {
  constructor(page) {
    this.page = page;
    this.username = page.getByLabel('Username');
    this.password = page.getByLabel('Password');
    this.loginButton = page.getByRole('button', { name: 'Login' });
  }

  async login(username, password) {
    await this.username.fill(username);
    await this.password.fill(password);
    await this.loginButton.click();
  }
}
```

This pattern improves maintainability when the UI changes or when a project grows.

### 2. What are the different waiting mechanisms?
Playwright provides multiple waiting strategies:
- Explicit waits: waitForTimeout, waitForURL, waitForLoadState
- Implicit/automatic waits: built-in action waits
- Assertions with expect: automatically retry until pass

The best practice is to avoid arbitrary sleep and rely on web-first waits and assertions.

### 3. How do you upload files?
Use setInputFiles().

```ts
await page.locator('#upload').setInputFiles('test-data/file.pdf');
```

This is used for document upload, profile picture upload, attachments, and other file-based forms.

### 4. How do you test file downloads?
Use waitForEvent('download').

```ts
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.getByRole('button', { name: 'Download' }).click()
]);

const path = await download.path();
```

This is useful for validating exported files such as PDF, CSV, and Excel downloads.

### 5. How do you handle scrolling?
Scrolling can be done with page.evaluate or locator.scrollIntoViewIfNeeded().

```ts
await page.locator('#footer').scrollIntoViewIfNeeded();
```

This is helpful for lazy-loaded elements, infinite scroll pages, or elements outside the current viewport.

### 6. How do you handle alerts and dialogs?
Use page.on('dialog') or page.waitForEvent('dialog').

```ts
page.on('dialog', async dialog => {
  console.log(dialog.message());
  await dialog.accept();
});
```

This is common for confirmation boxes and browser prompts.

### 7. How do you perform drag-and-drop?
Playwright supports drag-to-drop operations using dragTo().

```ts
await page.locator('#source').dragTo(page.locator('#target'));
```

This is used for reorderable lists, dashboard widgets, and custom drag interactions.

### 8. How do you handle dynamic elements and content?
Use resilient locators and retries instead of brittle timing logic.

Examples:
- Prefer getByRole() and getByText()
- Wait on expected text or visibility
- Avoid hard-coded delays

Dynamic content often appears after API calls, so using expect with a condition is the best practice.

### 9. How do you automate login functionality?
In a mature automation project, login is usually handled in one of these ways:
- Shared login helper method
- Auth fixture or setup step
- API login followed by UI validation
- Storage-state reuse across tests

Example:

```ts
await page.getByLabel('Username').fill('admin');
await page.getByLabel('Password').fill('admin123');
await page.getByRole('button', { name: 'Login' }).click();
```

For better stability, we usually verify post-login state rather than checking only that a button was clicked.

### 10. How do you test mobile or responsive applications?
Use Playwright emulation for mobile devices and browser viewport sizes.

```ts
const context = await browser.newContext({
  viewport: { width: 390, height: 844 },
  userAgent: 'Mozilla/5.0 ...',
  isMobile: true,
  hasTouch: true
});
```

This is useful for validating responsive layouts, touch interactions, and mobile-specific behaviors.

---

## Advanced Testing and Framework Concepts

### 1. What is Trace Viewer?
Trace Viewer is a Playwright feature that records test execution steps, network activity, DOM snapshots, and timeline events. It helps diagnose failing tests visually and step-by-step.

It is usually configured as:

```ts
use: {
  trace: 'on-first-retry'
}
```

This is extremely useful in CI and debugging sessions.

### 2. How do you generate and analyze traces?
You enable trace recording in the config and then open the generated trace file.

```bash
npx playwright show-report
```

The trace gives detailed execution timeline, which helps debug timing issues, failed selectors, and unexpected page states.

### 3. How do you perform visual testing?
Visual testing compares screenshots against approved baselines. Playwright can capture screenshots and integrate with visual testing tools or screenshot assertions.

Examples:
- snapshot pages in critical flows
- compare component screenshots after UI changes
- validate layout consistency across browsers

### 4. How do you test accessibility?
Use Playwright together with accessibility testing tools such as axe-core. Playwright can also validate semantic roles and accessible names using getByRole and getByLabel.

Example:

```ts
await page.getByRole('button', { name: 'Submit' }).click();
```

This helps ensure keyboard support, accessible names, and semantic structure are maintained.

### 5. How do you run tests across different browsers?
Use Playwright projects in config.

```ts
projects: [
  { name: 'chromium', use: { browserName: 'chromium' } },
  { name: 'firefox', use: { browserName: 'firefox' } },
  { name: 'webkit', use: { browserName: 'webkit' } }
]
```

This is critical for cross-browser validation in modern web apps.

### 6. How do you manage different environments?
Use environment-specific config values such as baseURL, API URLs, credentials, and feature flags. In a professional setup, we often use `.env` files or config parameters to support QA, staging, and production execution.

Example:

```ts
use: {
  baseURL: process.env.BASE_URL,
}
```

This avoids hard-coded URLs and improves environment safety.

### 7. How do you integrate Playwright with CI/CD pipelines?
Typical CI/CD integration steps include:
- Install dependencies
- Run `npx playwright install --with-deps`
- Execute tests using Playwright runner
- Upload HTML report and trace files as artifacts
- Fail the build if tests fail

Example command:

```bash
npx playwright test
```

In enterprise projects, we often integrate with GitHub Actions, Azure DevOps, Jenkins, or GitLab CI.

### 8. How do you debug Playwright tests?
Common debugging methods include:
- Run in headed mode
- Use --debug flag
- Use traces and screenshots
- Inspect the DOM with page.locator()
- Log state before/after action
- Use Playwright Inspector

```bash
npx playwright test --debug
```

Debugging is a major strength of Playwright and often differentiates a strong automation engineer from a weaker one.

### 9. How do you structure a scalable Playwright project?
A scalable Playwright project usually has:
- tests/ folder for specs
- pages/ or src/pages/ for page objects
- fixtures/ for shared setup
- utils/ for helper functions
- data/ for test data
- config files for env-specific setup
- reporting and trace integration

This structure improves maintainability as the suite grows.

### 10. How do you identify and handle flaky tests?
Flaky tests are usually caused by timing issues, race conditions, state leakage, or over-reliance on brittle selectors. To handle them:
- Prefer stable locators
- Use expect-based assertions
- Avoid fixed sleeps
- Isolate tests with BrowserContext
- Review traces and logs after failures
- Split large tests into smaller scenarios

The key is to fix the root cause, not just add retries blindly.

### 11. How do you approach performance-related testing?
Performance testing in Playwright usually focuses on:
- page load timing
- API response time
- slow network simulation
- UI responsiveness
- user-level workflow duration

We can use browser performance APIs and custom metrics to evaluate responsiveness, but it is usually not a substitute for dedicated performance testing tools.

### 12. What are the best practices for maintaining stable Playwright tests?
Best practices include:
- Prefer role-based locators over CSS/XPath
- Use web-first assertions and auto-waiting
- Avoid arbitrary sleeps
- Keep tests independent and isolated
- Reuse fixtures and page objects
- Use trace and screenshots for debugging
- Run tests in CI with consistent environment setup
- Review flaky tests frequently and refactor them

This is the foundation of a healthy automation framework.

---

## Final Interview Summary

For a Playwright engineer with 4 to 6 years of experience, interviewers usually expect you to speak beyond basic syntax. They want to hear that you understand:
- Browser architecture and isolation
- Robust locator strategies
- Auto-waiting and assertions
- API + UI testing together
- CI/CD integration and debugging
- Scalable framework design
- Flakiness management and test stability

A strong answer is not just “I know Playwright syntax,” but “I know how to build a reliable, maintainable, and scalable automation framework using Playwright.”

---

## Short Interview Closing Statement

“I have used Playwright for end-to-end UI testing, API validation, network mocking, and CI-based execution. My focus has been on building stable, maintainable automation frameworks with proper locator strategies, reusable page objects, environment handling, and debugging workflows to reduce flakiness and improve delivery confidence.”
