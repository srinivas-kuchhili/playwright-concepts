# 08 — Test Structure, Hooks, Annotations, Tags & Fixtures

## 1. Test Structure

```typescript
import { test, expect } from '@playwright/test';

test.describe('Login feature', () => {
  test.beforeAll(async () => { /* once per worker, before all tests in this describe */ });
  test.beforeEach(async ({ page }) => { await page.goto('/login'); });
  test.afterEach(async ({ page }, testInfo) => { /* cleanup */ });
  test.afterAll(async () => { /* once per worker, after all */ });

  test('valid login', async ({ page }) => {
    await test.step('Enter credentials', async () => {
      await page.getByLabel('Username').fill('admin');
      await page.getByLabel('Password').fill('admin123');
    });
    await test.step('Submit', async () => {
      await page.getByRole('button', { name: 'Login' }).click();
    });
    await expect(page).toHaveURL(/dashboard/);
  });
});
```

## 2. Hooks

| Hook | Runs | Fixtures available |
| ---- | ---- | ------------------ |
| `beforeAll` | Once per **worker** before tests in scope | Worker fixtures only (`browser`) — no `page` |
| `beforeEach` | Before **every** test | All (`page`, `context`...) |
| `afterEach` | After every test (even on failure) | All + `testInfo` |
| `afterAll` | Once per worker after tests | Worker fixtures |

⚠️ `beforeAll` runs **once per worker**, not once per run. With 4 workers it may run 4 times. For once-per-run logic use a **setup project** or `globalSetup`.

💡 Prefer **fixtures** over hooks for reusable setup/teardown.

---

## 3. `test.step`

Groups actions into named steps — shown in HTML report and trace.

```typescript
const orderId = await test.step('Place order', async () => {
  await page.getByRole('button', { name: 'Checkout' }).click();
  return await page.getByTestId('order-id').textContent();
});

await test.step('Optional step', async () => { /* ... */ }, { box: true }); // errors point to step call site
```

---

## 4. Annotations

| Annotation | Behaviour |
| ---------- | --------- |
| `test.only()` | Run only this test (focus) |
| `test.skip()` | Skip test |
| `test.skip(condition, 'reason')` | Conditional skip |
| `test.fixme()` | Skip; marks "needs fixing" |
| `test.fail()` | Expect test to fail (passes if it fails) |
| `test.slow()` | Triple the timeout |
| `test.describe.only / skip / fixme` | Apply to group |

```typescript
test.skip('not ready', async ({ page }) => {});

test('not on webkit', async ({ page, browserName }) => {
  test.skip(browserName === 'webkit', 'Feature not supported on Safari');
});

test('known bug', async ({ page }) => {
  test.fail();   // passes as long as it fails; alerts you once bug is fixed
});

test.fixme('broken after redesign', async ({ page }) => {});

// Custom annotation (shows in report)
test('checkout', {
  annotation: { type: 'issue', description: 'https://jira.company.com/PROJ-123' },
}, async ({ page }) => {});
```

---

## 5. Tags

```typescript
test('login works', { tag: '@smoke' }, async ({ page }) => {});
test('full checkout', { tag: ['@regression', '@payments'] }, async ({ page }) => {});

test.describe('Cart', { tag: '@cart' }, () => {
  test('add item', async ({ page }) => {});
});

// Legacy: tag in title
test('search @sanity', async ({ page }) => {});
```

```bash
npx playwright test --grep @smoke
npx playwright test --grep-invert @slow
npx playwright test --grep "@smoke|@sanity"
```

Or in config per project: `grep: /@smoke/`, `grepInvert: /@slow/`.

---

## 6. Serial vs Parallel inside a File

```typescript
test.describe.configure({ mode: 'serial' });    // run in order; if one fails, rest are skipped
test.describe.configure({ mode: 'parallel' });  // run tests in the file in parallel
test.describe.configure({ mode: 'default' });   // in order, but failures don't skip others
test.describe.configure({ retries: 2, timeout: 60_000 });
```

⚠️ Serial mode = tests depend on each other → anti-pattern; use only for truly sequential flows.

---

## 7. `testInfo`

```typescript
test('info', async ({ page }, testInfo) => {
  console.log(testInfo.title, testInfo.project.name, testInfo.retry, testInfo.workerIndex);

  await testInfo.attach('screenshot', { body: await page.screenshot(), contentType: 'image/png' });
  await testInfo.attach('data', { body: JSON.stringify({ a: 1 }), contentType: 'application/json' });

  const file = testInfo.outputPath('log.txt');   // per-test output folder
});

test.afterEach(async ({ page }, testInfo) => {
  if (testInfo.status !== testInfo.expectedStatus) {
    await page.screenshot({ path: `failures/${testInfo.title}.png` });
  }
});
```

Useful properties: `title`, `titlePath`, `status`, `expectedStatus`, `duration`, `retry`, `project`, `workerIndex`, `parallelIndex`, `errors`, `annotations`, `outputDir`.

---

## 8. Data-Driven (Parameterized) Tests

```typescript
const users = [
  { username: 'admin', password: 'admin123', expected: 'Dashboard' },
  { username: 'guest', password: 'guest123', expected: 'Home' },
];

for (const u of users) {
  test(`login as ${u.username}`, async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Username').fill(u.username);
    await page.getByLabel('Password').fill(u.password);
    await page.getByRole('button', { name: 'Login' }).click();
    await expect(page.getByRole('heading')).toHaveText(u.expected);
  });
}
```

From JSON / CSV:

```typescript
import data from '../test-data/users.json';

import fs from 'fs';
import { parse } from 'csv-parse/sync';
const records = parse(fs.readFileSync('test-data/users.csv'), { columns: true, skip_empty_lines: true });
```

⚠️ Test titles must be **unique** — include the parameter in the title.

---

## 9. Fixtures

### What is a Fixture?

A **fixture** is a reusable piece of **setup + teardown** that Playwright injects into a test **only when the test asks for it** (dependency injection). `page`, `context`, `browser`, `request` are built-in fixtures.

| Hooks | Fixtures |
| ----- | -------- |
| Run for every test in scope | Run **only** for tests that request them |
| Setup & teardown split across two hooks | Setup + teardown in one place (`use`) |
| Hard to reuse across files | Reusable across the whole project |
| Not composable | Composable (fixtures can depend on fixtures) |
| – | Can be test- or worker-scoped, auto, overridable |

### Built-in fixtures

| Fixture | Type | Scope |
| ------- | ---- | ----- |
| `page` | Page | test |
| `context` | BrowserContext | test |
| `browser` | Browser | worker |
| `browserName` | string | worker |
| `request` | APIRequestContext | test |

### Custom fixture — Page Objects

```typescript
// fixtures/base.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

type MyFixtures = {
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
};

export const test = base.extend<MyFixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));       // setup before, teardown after `use`
  },
  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  },
});

export { expect } from '@playwright/test';
```

```typescript
// tests/login.spec.ts
import { test, expect } from '../fixtures/base';

test('login', async ({ loginPage, dashboardPage }) => {
  await loginPage.goto();
  await loginPage.login('admin', 'admin123');
  await expect(dashboardPage.heading).toBeVisible();
});
```

### Fixture with setup & teardown

```typescript
export const test = base.extend<{ todoPage: TodoPage }>({
  todoPage: async ({ page }, use) => {
    const todoPage = new TodoPage(page);
    await todoPage.goto();
    await todoPage.addTodo('item 1');     // ---- setup

    await use(todoPage);                  // ---- test runs here

    await todoPage.removeAll();           // ---- teardown
  },
});
```

### Worker-scoped fixture (shared by all tests in a worker)

```typescript
type WorkerFixtures = { dbConnection: DB };

export const test = base.extend<{}, WorkerFixtures>({
  dbConnection: [async ({}, use) => {
    const db = await DB.connect();
    await use(db);
    await db.close();
  }, { scope: 'worker' }],
});
```

### Automatic fixture (runs for every test without being requested)

```typescript
export const test = base.extend<{ logConsole: void }>({
  logConsole: [async ({ page }, use, testInfo) => {
    const logs: string[] = [];
    page.on('console', m => logs.push(m.text()));
    await use();
    if (testInfo.status === 'failed') {
      await testInfo.attach('console', { body: logs.join('\n'), contentType: 'text/plain' });
    }
  }, { auto: true }],
});
```

### Option fixture (configurable per project)

```typescript
type Options = { role: 'admin' | 'user' };

export const test = base.extend<Options>({
  role: ['user', { option: true }],
});

// playwright.config.ts
projects: [
  { name: 'admin', use: { role: 'admin' } },
  { name: 'user',  use: { role: 'user' } },
]
```

### Override a built-in fixture

```typescript
export const test = base.extend({
  page: async ({ page }, use) => {
    await page.goto('/');                       // every test starts on home
    await use(page);
  },
});
```

### Merge fixtures from multiple modules

```typescript
import { mergeTests, mergeExpects } from '@playwright/test';
import { test as dbTest } from './db-fixtures';
import { test as a11yTest } from './a11y-fixtures';

export const test = mergeTests(dbTest, a11yTest);
```

### Fixture execution order

```text
worker fixtures setup (browser, dbConnection)
   └─ test fixtures setup (context → page → loginPage)
        └─ beforeEach → TEST → afterEach
   └─ test fixtures teardown (reverse order)
worker fixtures teardown (when worker shuts down)
```

> **Interview one-liner:** Fixtures are Playwright's dependency-injection mechanism: reusable setup/teardown that runs only when a test requests it. I use `test.extend` to inject Page Objects, API clients and test data; worker-scoped fixtures for expensive shared resources like DB connections; auto fixtures for logging; and option fixtures to parametrize projects.
