# Playwright Interview Handbook

# 1. Playwright Fundamentals

## What is Playwright?

Playwright is an open-source automation framework developed by Microsoft for end-to-end testing of modern web applications. Its Supported Languages are JavaScript, TypeScript, Python, Java, C# / .NET  and Supported Browser are Chromium, Firefox, WebKit

## Key features

- Web UI automation
- End-to-end testing
- API testing
- Cross-browser testing
- Mobile/device emulation
- Network interception and mocking
- Authentication/session reuse
- Parallel execution
- Trace, screenshot, and video capture
- Built-in assertions
- Test fixtures
- CI/CD execution
- Visual regression testing


### Playwright vs Cypress vs Selenium


| Topic                         | Playwright                                      | Cypress                                                  | Selenium                                      |
| ----------------------------- | ----------------------------------------------- | -------------------------------------------------------- | --------------------------------------------- |
| **What is it?**               | Modern browser automation and testing framework | Modern web application testing framework                 | Browser automation framework                  |
| **Developed by**              | Microsoft                                       | Cypress.io                                               | Selenium Project                              |
| **Architecture**              | Direct modern browser automation architecture   | Cypress-specific architecture                            | WebDriver architecture                        |
| **Main languages**            | TypeScript/JavaScript, Java, Python, .NET       | JavaScript/TypeScript                                    | Java, Python, C#, JavaScript, Ruby            |
| **Browser support**           | Chromium, Firefox, WebKit                       | Chrome-family, Firefox, Electron, etc.                   | Chrome, Firefox, Edge, Safari, etc.           |
| **Cross-browser testing**     | Yes                                             | Yes                                                      | Yes                                           |
| **Auto-waiting**              | Yes                                             | Yes                                                      | Requires proper wait strategy                 |
| **Actionability checks**      | Yes                                             | Yes                                                      | Not comparable to Playwright's built-in model |
| **Multiple tabs**             | Excellent                                       | More limited                                             | Supported                                     |
| **Multiple browser contexts** | Yes                                             | No direct equivalent                                     | No direct equivalent                          |
| **Popup handling**            | Excellent                                       | More constrained                                         | Supported                                     |
| **Iframe handling**           | Supported                                       | Supported                                                | Supported                                     |
| **Alerts/dialogs**            | Supported                                       | Supported                                                | Supported                                     |
| **File upload/download**      | Built-in support                                | Supported                                                | Supported                                     |
| **API testing**               | Built-in `APIRequestContext`                    | `cy.request()`                                           | Usually requires another API library          |
| **Network interception**      | `page.route()`                                  | `cy.intercept()`                                         | Requires additional approach/tooling          |
| **Parallel execution**        | Built into Playwright Test                      | Supported                                                | Commonly Selenium Grid + test framework       |
| **Test runner**               | Playwright Test                                 | Cypress Test Runner                                      | TestNG/JUnit/PyTest, etc.                     |
| **Assertions**                | Built-in web-first assertions                   | Built-in assertions                                      | Usually TestNG/JUnit/PyTest                   |
| **Debugging**                 | Inspector + Trace Viewer                        | Interactive Test Runner                                  | Framework/tool dependent                      |
| **Screenshots**               | Built-in                                        | Built-in                                                 | Supported                                     |
| **Video**                     | Supported                                       | Supported                                                | Usually additional configuration              |
| **POM**                       | Supported                                       | Supported                                                | Supported                                     |
| **Fixtures**                  | Powerful built-in fixtures                      | Fixtures supported                                       | Depends on test framework                     |
| **CI/CD**                     | Excellent                                       | Excellent                                                | Excellent                                     |
| **Setup**                     | Relatively easy                                 | Easy                                                     | More setup/configuration usually required     |
| **Best use case**             | UI + API + E2E + cross-browser                  | Frontend/UI + E2E testing                                | Enterprise web automation                     |
| **Major strength**            | Modern all-in-one automation capabilities       | Developer-friendly testing experience                    | Mature ecosystem and broad compatibility      |
| **Major limitation**          | Requires learning Playwright-specific concepts  | Multi-tab/multi-window scenarios can be more constrained | More synchronization/framework setup          |



Discuss architecture, browser automation model, synchronization, browser contexts, APIs, tracing/debugging, and test-runner features. Avoid reducing the comparison to only “Playwright is faster.”


# 2. Playwright Architecture

## Definition

Playwright's architecture separates browser automation from test execution. The test code interacts with a browser engine through Playwright APIs, while Playwright Test provides the test runner, fixtures, configuration, assertions, retries, workers, projects, and reporters.

## Important layers to understand

1. Test code
2. Playwright Test runner
3. Browser / BrowserContext / Page
4. Locator and action APIs
5. Browser engine


# 3. Browser, BrowserContext, and Page

## Browser
- A Browser represents a launched browser instance such as Chromium, Firefox, or WebKit.

## BrowserContext
- A BrowserContext is an isolated browser session inside a Browser. It has its own cookies, local storage, permissions, cache-related state, and pages.

## Page
- A Page represents a single browser tab or webpage within a BrowserContext.

## Relationship

```text
Browser
  ├── BrowserContext 1
  │     ├── Page 1
  │     └── Page 2
  │
  └── BrowserContext 2
        └── Page 1
```

> A Browser is the browser process, a BrowserContext is an isolated session, and a Page is a single tab inside that context. The isolation provided by BrowserContext is especially useful for test independence and parallel execution.

# 4. Locators
- A Locator is Playwright's mechanism for identifying elements on the page. Locators are central to Playwright's auto-waiting and retry behavior and are evaluated against the current DOM when actions or assertions are performed.

## Recommended built-in locators

Priority should generally be:

1. `getByRole()`
2. `getByLabel()`
3. `getByPlaceholder()`
4. `getByText()` where appropriate
5. `getByTestId()` when the application exposes a stable test contract
6. CSS selectors when needed
7. XPath when there is a real reason to use it

## Strict mode

Playwright generally expects an action to target a single intended element. If a locator unexpectedly matches multiple elements, strictness helps expose the ambiguity instead of silently acting on the wrong element.

## `first()`, `last()`, `nth()`

These are useful when the target genuinely depends on position, but they should not be the first choice when a semantic or unique locator is available.

## Filtering

Useful methods include:

- `filter({ hasText })`
- `filter({ has })`
- `locator()` chaining
- `getByRole()` within a scoped locator

## locator vs ElementHandle

### Locator
- A Locator is a high-level, retryable way to identify an element and is the preferred abstraction for most test actions and assertions.

### ElementHandle
- An ElementHandle is a handle to a particular DOM element instance. It is a lower-level concept and is usually not preferred for routine test interactions.

### Interview answer

> I prefer Locators because they integrate with Playwright's auto-waiting and can resolve the current element when the action occurs. ElementHandle is lower level and can become less convenient when the DOM is re-rendered.

# 6. Actions
- Actions are operations that simulate user interaction or browser behavior.

## Common actions

- Click
- Double click
- Fill
- Type/keyboard input
- Check/uncheck
- Select option
- Hover
- Focus
- Press keyboard keys
- Mouse actions
- Drag and drop

## `fill()` vs `type()`

### `fill()`
- Sets the value of an input and is normally the preferred method for entering a complete value.

### `type()`
- Simulates typing characters one by one and is useful when the test specifically needs typing behavior.

> Use `fill()` for normal input population. Use keyboard typing behavior when the application reacts differently to actual key input.

## Example

```typescript
await page.getByLabel('Username').fill('srinivas');
await page.getByLabel('Password').fill('secret');
await page.getByRole('button', { name: 'Login' }).click();
```

# 7. Auto-Waiting and Actionability
- Auto-waiting means Playwright waits for an element to become suitable for an action before performing the action, instead of requiring manual sleep statements for normal synchronization.

## Why it matters

Modern web applications are asynchronous. Elements can be attached late, become visible later, change state, or re-render.

## Actionability checks

Before many actions, Playwright verifies conditions such as whether the element is visible, enabled, stable, able to receive events, or editable depending on the action.


## Why hard waits are discouraged

A hard wait:

- slows down fast tests
- can still be too short for slow environments
- hides the real synchronization problem
- can increase flakiness

## Better alternatives

- Locator auto-waiting
- Web-first assertions
- Wait for a specific page state
- Wait for a response when network synchronization is required
- Wait for a particular UI condition

> I avoid hard-coded waits. I rely on Playwright's auto-waiting and web-first assertions. When explicit synchronization is needed, I wait for a meaningful event or state, such as an expected response, URL change, or visible application state.

# 8. Assertions
- An assertion verifies that the application is in the expected state.

## Web-first assertions

Playwright assertions are designed to retry until the expected condition is satisfied or the assertion timeout is reached.

## Common assertions

- `toBeVisible()`
- `toBeHidden()`
- `toBeEnabled()`
- `toBeDisabled()`
- `toBeChecked()`
- `toHaveText()`
- `toContainText()`
- `toHaveValue()`
- `toHaveAttribute()`
- `toHaveURL()`
- `toHaveTitle()`
- `toHaveCount()`
- `toBeEmpty()`

## Soft assertions
- A soft assertion records a failure but can allow the test to continue so multiple validations can be reported together.

Use soft assertions intentionally. Do not use them to hide important failures.

---

# 9. Text Fields, Checkboxes, Radio Buttons, and Basic Controls

## Text field
- A text field is an input control used to collect text from the user.

```typescript
await page.getByLabel('Email').fill('test@example.com');
```

## Checkbox
- A checkbox represents an independent on/off selection and can often have both checked and unchecked states.

```typescript
await page.getByLabel('Terms and Conditions').check();
await expect(page.getByLabel('Terms and Conditions')).toBeChecked();
```

## Radio button
- A radio button represents one choice from a related mutually exclusive group.

```typescript
await page.getByLabel('Male').check();
```

> Checkbox → multiple independent selections can be possible.
> Radio group → normally one option from the group.

---

# 10. Dropdowns
- A dropdown is a UI control that allows the user to select one or more values from a predefined list.

## Types of dropdowns

1. Native HTML `<select>` dropdown
2. Custom dropdown
3. Searchable/autocomplete dropdown
4. Multi-select dropdown
5. Framework-based custom dropdown such as React/Angular/Material UI
6. Keyboard-driven dropdown

## Native `<select>`

Use `selectOption()`.

### By value

```typescript
await page.locator('#country').selectOption('india');
```

### By label

```typescript
await page.locator('#country').selectOption({ label: 'India' });
```

### By index

```typescript
await page.locator('#country').selectOption({ index: 1 });
```

## Custom dropdown

A custom dropdown is normally implemented with elements such as `button`, `div`, `ul`, `li`, or `input`, not a native `select`.

Handle it like normal UI:

```typescript
await page.getByRole('button', { name: 'Select Country' }).click();
await page.getByRole('option', { name: 'India' }).click();
```

## Searchable dropdown
- a dropdown in which the user types text to narrow the available options.

Typical approach:

1. locate the input
2. fill search text
3. wait for/select matching option

```typescript
await page.getByPlaceholder('Search country').fill('Ind');
await page.getByRole('option', { name: 'India' }).click();
```

## Multi-select dropdown
- a dropdown that allows more than one value.

The exact handling depends on the implementation. It may involve checkboxes, options, tags, or a combination.


> I first identify whether the dropdown is a native select or a custom component. For a native select I use `selectOption()` and can select by value, label, or index. For a custom dropdown I use stable locators, click to open it, and select the required option. Searchable dropdowns require filling the input before choosing a matching option. Multi-select handling depends on whether the options are represented as checkboxes, listbox options, or another custom component.

---
# 11. Alerts / Dialogs

- A JavaScript browser dialog is a browser-level dialog generated by functions such as `alert()`, `confirm()`, and `prompt()`.

## Types
1. **Alert** – Displays a message and requires acknowledgment.
2. **Confirm** – Provides OK/Cancel options.
3. **Prompt** – Accepts user input.
4. **beforeunload** – Appears when a page is about to be unloaded.

---

## Three Ways to Handle Dialogs

| `page.on('dialog')`                                      | `page.once('dialog')`                                           | `page.waitForEvent('dialog')`                                                   |
| -------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Registers a **persistent event listener**                | Registers a **one-time event listener**                         | Explicitly **waits for a specific dialog event**                                |
| Handler remains active for future dialogs                | Handler is automatically removed after the first matching event | Returns a Promise for the dialog                                                |
| Useful when multiple dialogs may occur throughout a test | Useful when only one dialog needs to be handled                 | Useful when you want to synchronize the dialog with the action that triggers it |
| Can handle multiple dialogs using the same handler       | Handles only the next dialog                                    | Usually used with `Promise.all()`                                               |
| Does not directly wait for the dialog                    | Does not directly wait for the dialog                           | Explicitly waits for the dialog                                                 |
| Good for global/repeated dialog handling                 | Good for one-time handling                                      | Good for action + dialog synchronization                                        |
| Example: `page.on('dialog', handler)`                    | Example: `page.once('dialog', handler)`                         | Example: `page.waitForEvent('dialog')`                                          |

---

## 1. `page.on('dialog')`

Use this when you want a **persistent listener** that can handle multiple dialogs.

```typescript
page.on('dialog', async dialog => {
  console.log(dialog.message());
  await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();
await page.getByRole('button', { name: 'Delete Again' }).click();
```

### Key point

The listener remains registered and can handle multiple dialogs.

```text
Dialog 1 → Handler
Dialog 2 → Handler
Dialog 3 → Handler
```

> `page.on('dialog')` is used when I want a persistent dialog listener that can handle multiple dialogs during the test.

---

# 2. `page.once('dialog')`

Use this when you expect **only one dialog**.

```typescript
page.once('dialog', async dialog => {
  expect(dialog.message()).toBe('Login successful');

  await dialog.accept();
});

await page.getByRole('button', { name: 'Login' }).click();
```

The listener automatically handles the first dialog and is then removed.

```text
First Dialog → Handler → Listener Removed
Second Dialog → No handler
```
> `page.once('dialog')` registers a one-time listener. It is useful when I expect only one dialog and don't want the same handler to remain active for subsequent dialogs.

---

# 3. `page.waitForEvent('dialog')`

Use this when you want to **explicitly wait for the dialog triggered by a particular action**.

The recommended pattern is:

```typescript
const dialogPromise = page.waitForEvent('dialog');

await page.getByRole('button', { name: 'Delete' }).click();

const dialog = await dialogPromise;

console.log(dialog.message());

await dialog.accept();
```

Or, commonly, use `Promise.all()`:

```typescript
const [dialog] = await Promise.all([
  page.waitForEvent('dialog'),
  page.getByRole('button', { name: 'Delete' }).click()
]);

expect(dialog.message()).toBe('Are you sure?');

await dialog.accept();
```

### Why `Promise.all()`?

The dialog can appear immediately after the click.

So we register the wait **before** triggering the action.

```text
Register wait
      ↓
Click button
      ↓
Dialog appears
      ↓
waitForEvent resolves
      ↓
Handle dialog
```

## Useful dialog methods

- `dialog.type()`
- `dialog.message()`
- `dialog.defaultValue()`
- `dialog.accept()`
- `dialog.dismiss()`

> `page.waitForEvent('dialog')` is useful when I need to explicitly synchronize with a dialog triggered by a specific action. I normally register the event wait before the action and use `Promise.all()` when appropriate.

---

# 12. HTML Modals and Toast Messages

## HTML Modal — Definition

An HTML modal is an application-level UI component rendered inside the page DOM. It is not the same as a browser JavaScript dialog.

## Handling

1. Locate the modal container
2. Validate heading/content
3. Locate action buttons
4. Perform action
5. Assert the expected result

## Toast Message — Definition

A toast is a temporary notification displayed by the application, often after an action succeeds or fails.

## Important points

- Use a stable locator
- Assert text and visibility
- Avoid arbitrary sleep for disappearing messages
- Prefer web-first assertions

---

# 13. Frames and iFrames
- An iframe is a separate browsing context embedded inside a page. Elements inside the iframe belong to that frame's document and need frame-aware location.

## Why it matters

A normal page locator does not automatically search inside another frame's document.

## Recommended approach

Use `frameLocator()` when possible.

```typescript
const frame = page.frameLocator('#payment-frame');
await frame.getByLabel('Card Number').fill('4111111111111111');
```

## Alternative approach

Use frame objects when you need lower-level frame access or frame events.

## Nested frames

For nested frames, chain `frameLocator()` as needed.

> An iframe is a separate browsing context. I cannot normally locate elements inside it using a top-level page locator. I use `frameLocator()` to interact with elements inside the frame, or obtain the relevant Frame object when I need frame-level operations.

---

# 14. Multiple Tabs, Windows, and Popups

## Page / tab
- A Page represents a browser tab or page inside a BrowserContext.

## Popup
- A popup is a new Page opened as a result of an action in another page, such as clicking a link that opens a new tab/window.

## `page.waitForEvent('popup')`
- Use when the new page is directly associated with the current page.

```typescript
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.getByRole('link', { name: 'Open Report' }).click()
]);

await popup.waitForLoadState();
```

## `context.waitForEvent('page')`
- Use when you want to observe a new page at the BrowserContext level.

```typescript
const newPagePromise = context.waitForEvent('page');
await page.getByText('Open').click();
const newPage = await newPagePromise;
```

> `page.waitForEvent('popup')` is useful for a popup initiated from a particular page. `context.waitForEvent('page')` listens at the BrowserContext level for a newly created page.

---

# 15. File Upload and Download

## File Upload
- File upload automation means providing a local file to an application's file input or upload control.

## Common approach

```typescript
await page.getByLabel('Upload file').setInputFiles('test-data/sample.pdf');
```

## Multiple files

```typescript
await page.getByLabel('Upload files').setInputFiles([
  'test-data/a.pdf',
  'test-data/b.pdf'
]);
```

## File Download
- A download occurs when the browser receives a downloadable file from the application.

```typescript
const downloadPromise = page.waitForEvent('download');
await page.getByRole('button', { name: 'Download' }).click();
const download = await downloadPromise;

await download.saveAs('downloads/report.pdf');
```
---

# 16. Tables and Dynamic Tables
- A web table displays structured data in rows and columns. A dynamic table can change based on search, sorting, pagination, filters, API data, or user interaction.

## Automation considerations

- Identify header cells
- Find the correct row using business data
- Locate the action within that row
- Avoid hard-coded row numbers
- Handle pagination
- Validate row counts when required

## Example strategy

Find the row containing a unique value, then operate inside that row:

```typescript
const row = page.getByRole('row').filter({ hasText: 'Automation' });
await row.getByRole('button', { name: 'Edit' }).click();
```
---

# 17. Date Pickers, Tooltips, Sliders, and Rich UI Controls

## Date picker 
- A date picker is a UI control that lets users select dates, often using a calendar widget or an input field.

## Automation approaches

- Fill date directly when allowed
- Select calendar date by role/text
- Navigate month/year controls
- Handle disabled dates
- Validate final value

## Tooltip 
- A tooltip is contextual information displayed when a user hovers, focuses, or otherwise interacts with an element.

## Slider
- A slider allows selection of a value along a range.

## Rich text editor
- A rich text editor allows formatted text input using an editable container, often a contenteditable element or embedded frame.
---

# 18. Authentication and Session Handling
- Authentication automation verifies that users can establish and reuse valid sessions. In Playwright, authentication state can be saved and reused to avoid repeating login steps in every test.

## Common authentication strategies

- UI login for dedicated authentication tests
- API login to obtain a session/token
- Reusing `storageState`
- Context-level cookies/session setup
- Role-specific authenticated projects

## Why authentication reuse matters

Running a full login flow before every test increases execution time and creates unnecessary dependencies.

> I separate authentication coverage from ordinary business-flow tests. I can establish authentication once, save the browser storage state, and reuse it for tests that do not need to validate the login flow itself. This reduces execution time while keeping the login tests independently covered.

---

# 19. Cookies, Local Storage, and Storage State

## Cookie
- A cookie is browser-managed key/value data associated with a web origin and used for things such as sessions, preferences, and tracking.

## Local Storage
- Local Storage stores key/value data in the browser for a web origin and persists across page reloads and browser sessions until cleared, subject to the application's behavior.

## Storage State
- Playwright storage state represents reusable browser state, commonly including cookies and origin storage data, that can be loaded into a new context.

## Why storage state matters

It supports authenticated test setup and avoids repeated login flows.

## Example

```typescript
await page.context().storageState({ path: 'playwright/.auth/user.json' });
```

And then configure a project/use setting to reuse that state.

---

# 20. API Testing with Playwright
- Playwright provides an API testing capability through an API request context. This allows tests to send HTTP requests and validate responses without opening a browser page for every API test.

## Why API testing matters

API tests are generally useful for validating service contracts, business logic, integration behavior, status codes, schemas, authentication, and test-data setup.

## HTTP methods to know

- GET — retrieve data
- POST — create/send data
- PUT — replace or update a resource
- PATCH — partially update a resource
- DELETE — remove a resource

## Important API concepts

- URL
- Headers
- Query parameters
- Path parameters
- Request body
- Authentication
- Status code
- Response headers
- Response body
- JSON validation
- Schema validation

## Example

```typescript
const response = await request.get('/users/123');
expect(response.ok()).toBeTruthy();
expect(response.status()).toBe(200);

const body = await response.json();
expect(body.id).toBe(123);
```

> I use Playwright API testing to validate backend behavior independently of the UI, prepare test data efficiently, and combine API and UI checks in end-to-end workflows. I validate status codes, headers, response payloads, schema/business fields, and authentication behavior.

---


# 22. Network Interception and Mocking
- Network interception means observing, modifying, continuing, aborting, or fulfilling requests made by the page or browser context.

## Why it matters

It is useful when:

- backend services are unstable
- a third-party dependency is unavailable
- specific edge-case responses are difficult to produce
- you need deterministic test data
- you need to validate frontend behavior for error states

## Important APIs/concepts

- `page.route()`
- context routing
- `route.continue()`
- `route.fulfill()`
- `route.abort()`
- Request/response events

## Example

```typescript
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({ users: [] })
  });
});
```


### Mocking
- Return a controlled response without calling the real backend.

### Interception
- Observe or modify the request/response path.
- These concepts overlap in practice, but the purpose is different.

---

# 23. Test Data and Data-Driven Testing
- Test data is the input required to validate a scenario. Data-driven testing separates test logic from the data variations used by that logic.

## Common sources

- JSON
- CSV
- Environment variables
- API-generated data
- Database/service data through supported test utilities
- Factory functions
- Parameterized test arrays

## Example

```typescript
const users = [
  { username: 'user1', role: 'Admin' },
  { username: 'user2', role: 'Viewer' }
];
```
---

# 24. Fixtures
- A fixture provides test setup and resources to tests in a reusable way.

## Built-in fixtures commonly used

- `page`
- `context`
- `browser`
- `browserName`
- `request`

## Sample code

```typescript
// fixtures.ts
import { test as base } from '@playwright/test';
import { HomePage } from './pages/HomePage'; 

type MyFixtures = {
  homePage: HomePage;
};

export const test = base.extend<MyFixtures>({
  homePage: async ({ page }, use) => {
    const homePage = new HomePage(page);
    await use(homePage);
  },
});

```

## Custom fixtures

Custom fixtures allow teams to build domain-specific setup such as:

- authenticated user
- application client
- page objects
- seeded test data
- API clients
- tenant configuration

## Why fixtures are important

They improve reuse and keep test files focused on business behavior instead of repeating setup.

> Fixtures are reusable setup and dependency injection mechanisms in Playwright Test. I use them to provide common resources such as page objects, authenticated sessions, API clients, or test data while keeping test cases clean.

---

# 25. Hooks and Test Lifecycle
- Hooks are lifecycle functions that execute before or after tests or test groups.

## Main hooks

- `beforeEach`
- `afterEach`
- `beforeAll`
- `afterAll`

## Use cases

### beforeEach
- Common per-test setup.

### afterEach
- Per-test cleanup or diagnostic logic.

### beforeAll
- Setup shared at the suite/file level when appropriate.

### afterAll
- Cleanup after the relevant suite/file.

---

# 26. Playwright Test Runner
- Playwright Test is the built-in testing framework that provides test discovery, fixtures, assertions, hooks, retries, timeouts, workers, projects, reporting, and command-line execution.

## Basic structure

```typescript
import { test, expect } from '@playwright/test';

test('login works', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Username').fill('user');
  await page.getByLabel('Password').fill('password');
  await page.getByRole('button', { name: 'Login' }).click();

  await expect(page).toHaveURL(/dashboard/);
});
```

## Core concepts

- Test files
- Test cases
- Test suites
- Fixtures
- Hooks
- Assertions
- Projects
- Workers
- Retries
- Reporters

---

# 27. Test Organization, Tags, Annotations, Skip/Only/Fail/Slow
- Test organization controls how tests are grouped, selected, skipped, annotated, and executed.

## Common features

- `test.describe()` — group related tests
- `test.skip()` — skip a test
- `test.only()` — run only selected test(s)
- `test.fail()` — mark an expected failure
- `test.slow()` — increase timeout for a slow test
- annotations/tags — attach metadata or enable filtering

- Do not leave `test.only()` in committed code. CI can be configured to fail when accidental focused tests remain.

---

# 28. Configuration
- `playwright.config.ts` is the central configuration file for Playwright Test. It controls how tests are discovered and executed and contains shared and project-specific settings.

## Important configuration areas

- `testDir`
- `baseURL`
- `projects`
- `timeout`
- `expect`
- `retries`
- `workers`
- `fullyParallel`
- `reporter`
- `use`
- `globalSetup`
- `globalTeardown`
- screenshots
- videos
- traces

## Example

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30_000,
  retries: process.env.CI ? 2 : 0,
  use: {
    baseURL: 'https://example.com',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure'
  }
});
```

---

# 29. Projects and Multi-Environment / Multi-Browser Execution
- A Playwright project is a logical group of tests using the same configuration. Projects can represent browsers, environments, test suites, authentication states, or other execution variations.

## Examples

- Chromium smoke
- Firefox regression
- WebKit regression
- Staging environment
- Production-safe checks
- Logged-in tests
- Logged-out tests
- Mobile emulation

> I use projects when the same test suite must run under different browsers, devices, environments, or configuration profiles. This keeps variations explicit and manageable from the configuration file.

---

# 30. Parallel Execution, Workers, and Sharding

## Parallel execution
- Parallel execution runs independent tests concurrently to reduce overall suite execution time.

## Workers
- Workers are Playwright Test worker processes used to execute tests concurrently.

## Sharding
- Sharding distributes a test suite across multiple independent CI jobs or machines.

---

# 31. Retries, Timeouts, and Flaky Tests

## Retry
- A retry reruns a failed test according to configured retry rules.

## Timeout
- A timeout is the maximum allowed time for an operation, assertion, test, fixture, or overall run before Playwright considers it timed out.

## Types of timeout to understand

- Test timeout
- Expect/assertion timeout
- Action/navigation-related timeouts
- Global timeout

## Flaky test
- A flaky test is a test that produces inconsistent results without a meaningful application change, often due to synchronization issues, shared state, unstable data, timing assumptions, or environment dependencies.

## Common causes of flakiness

- Hard waits
- Weak locators
- Race conditions
- Shared test data
- Order dependency
- Unstable external systems
- Improper cleanup
- Over-reliance on network timing
- Incorrect parallelization

## Debugging approach

1. Reproduce
2. Inspect trace/logs
3. Identify actual synchronization problem
4. Replace brittle waits
5. Improve locator or data isolation
6. Re-run repeatedly
7. Validate in CI

---

# 32. Page Object Model and Framework Design

## Page Object Model
- Page Object Model (POM) is a design approach where page-specific locators and interactions are grouped into classes or components, separating UI details from test intent.

## Why POM is useful

- Reusability
- Maintainability
- Cleaner tests
- Centralized locator changes
- Reduced duplication

## What belongs in a page object

- Locators
- Reusable page interactions
- Page-specific workflows
- Helper methods related to that page/component

## What should generally stay in tests

- Business intent
- Scenario-specific assertions
- High-level workflow orchestration

## Example

```typescript
export class LoginPage {
  constructor(private page: Page) {}

  username = this.page.getByLabel('Username');
  password = this.page.getByLabel('Password');
  loginButton = this.page.getByRole('button', { name: 'Login' });

  async login(username: string, password: string) {
    await this.username.fill(username);
    await this.password.fill(password);
    await this.loginButton.click();
  }
}
```

Test:

```typescript
const loginPage = new LoginPage(page);
await loginPage.login('user', 'password');
await expect(page).toHaveURL(/dashboard/);
```

---

# 33. Reusable Components, Utilities, and Base Classes

## Reusable component
- A component object represents a reusable UI fragment such as a header, date picker, table, navigation menu, or modal.

## Utility
- A utility is a reusable helper that performs a generic technical function, such as date generation, file validation, parsing, or data transformation.

## Base class
- A base class contains common behavior shared by related page objects or framework objects.

## Framework design principle
- Reuse behavior, but do not create huge “god” utility classes that contain unrelated functionality.

---

# 34. Debugging
- Debugging is the process of determining why an automation test failed and whether the root cause is the application, test code, data, environment, or synchronization.

## Useful Playwright tools

- Inspector
- Debug mode
- Trace Viewer
- Screenshots
- Video
- Console/network information
- Test output
- HTML report

## Debugging approach

1. Identify exact failing action
2. Check locator resolution
3. Check application state before action
4. Inspect trace
5. Check network/API failures
6. Check test data
7. Re-run locally and in CI

---

# 35. Trace Viewer
- Trace Viewer is a Playwright diagnostic tool that lets you inspect the timeline of a test, including actions, snapshots, screenshots, network information, and other execution details depending on the trace configuration.
---

# 36. Screenshots and Videos

## Screenshot
- A screenshot captures the visual state of a page or element at a specific point in time.

## Video
- Video records the browser session during execution when video recording is enabled.

---

# 37. Reporting
- A test report communicates test execution results, including passed/failed/skipped tests, timing, errors, attachments, and diagnostic artifacts.

## Common Playwright reporting options

- HTML
- JSON
- JUnit
- Line/list/dot style console reporting
- Custom reporters
- Allure through an external integration

---

# 38. Visual Testing
- Visual testing compares the current rendered UI against a trusted baseline image to detect unintended visual changes.

- Playwright supports screenshot assertions such as `toHaveScreenshot()`.

## Important concepts

- Baseline image
- Actual image
- Pixel/visual comparison
- Threshold/tolerance
- Dynamic content
- Masking
- Full-page vs element screenshots

## Example

```typescript
await expect(page).toHaveScreenshot('dashboard.png');
```

---

# 39. Cross-Browser and Device Testing
- Cross-browser testing validates the application across supported browser engines and configurations.

## Device emulation
- Playwright can emulate selected device characteristics and browser context settings.

## What can be varied

- viewport
- user agent/device profile
- locale
- timezone
- geolocation
- permissions
- color scheme

---

# 40. CI/CD
- Continuous Integration and Continuous Delivery/Deployment automate the build, test, and delivery process whenever code changes or when scheduled triggers occur.

## Playwright in CI

Typical flow:

```text
Developer Push / PR
        ↓
Install dependencies
        ↓
Install Playwright browsers
        ↓
Run smoke/regression/API tests
        ↓
Collect report + artifacts
        ↓
Publish result
```

## Common platforms

- GitHub Actions
- Jenkins
- Azure DevOps
- GitLab CI

## CI concerns

- Browser installation
- Environment variables
- Secrets
- Test data
- Parallelism
- Retries
- Artifacts
- Report publishing
- Failure notifications

> integrate Playwright into CI so every relevant change can trigger smoke or regression tests. The pipeline installs dependencies and browser binaries, runs the selected project or suite, and stores reports, screenshots, videos, or traces as build artifacts.

---

# Official References

The concepts in this handbook were checked against the official Playwright documentation:

- Playwright documentation: https://playwright.dev/docs/intro
- Locators: https://playwright.dev/docs/locators
- Locator API: https://playwright.dev/docs/api/class-locator
- Browser Contexts: https://playwright.dev/docs/browser-contexts
- Actions: https://playwright.dev/docs/input
- Assertions: https://playwright.dev/docs/test-assertions
- Dialogs: https://playwright.dev/docs/dialogs
- Frames: https://playwright.dev/docs/frames
- Pages / popup handling: https://playwright.dev/docs/pages
- Authentication: https://playwright.dev/docs/auth
- API testing: https://playwright.dev/docs/api-testing
- Network: https://playwright.dev/docs/network
- Fixtures: https://playwright.dev/docs/test-fixtures
- Test configuration: https://playwright.dev/docs/test-configuration
- Projects: https://playwright.dev/docs/test-projects
- Parallelism: https://playwright.dev/docs/test-parallel
- CI: https://playwright.dev/docs/ci
- Trace Viewer: https://playwright.dev/docs/trace-viewer
- Screenshots: https://playwright.dev/docs/screenshots
- Visual comparisons: https://playwright.dev/docs/test-snapshots
