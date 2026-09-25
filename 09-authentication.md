# 09 — Authentication & Session Reuse

## 1. The Problem

Logging in via UI before **every** test is slow and flaky. Playwright solves this by **logging in once**, saving the browser state (**cookies + localStorage + IndexedDB**) to a JSON file — **`storageState`** — and loading it into every test's context.

```text
Setup (login once) ──▶ playwright/.auth/user.json ──▶ every test starts already logged in
```

> I log in once in a setup project, save `storageState` to a file, and configure projects to reuse it. That removes repetitive UI logins and makes the suite much faster.

⚠️ Add `playwright/.auth` to `.gitignore` — it contains session tokens.

---

## 2. Approaches Compared

| Approach | How | When |
| -------- | --- | ---- |
| **Setup project + storageState** ✅ | `auth.setup.ts` saves state; projects depend on it | Most apps — recommended |
| API login | `request.post('/login')` → save state | Fastest; when login API exists |
| Global setup | `globalSetup` saves state | Legacy |
| Per-worker account | Worker fixture logs in unique user per worker | Tests modify server-side user state |
| Multiple roles | One file per role | Admin/user flows |
| UI login in `beforeEach` | Log in every test | ❌ Only for testing login itself |

---

## 3. Setup Project (Recommended)

```typescript
// tests/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

const authFile = 'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Username').fill(process.env.APP_USER!);
  await page.getByLabel('Password').fill(process.env.APP_PASS!);
  await page.getByRole('button', { name: 'Sign in' }).click();

  await page.waitForURL('/dashboard');               // make sure cookies are set
  await expect(page.getByRole('button', { name: 'Profile' })).toBeVisible();

  await page.context().storageState({ path: authFile });
});
```

```typescript
// playwright.config.ts
projects: [
  { name: 'setup', testMatch: /.*\.setup\.ts/ },
  {
    name: 'chromium',
    use: { ...devices['Desktop Chrome'], storageState: 'playwright/.auth/user.json' },
    dependencies: ['setup'],
  },
],
```

Tests now start logged in:

```typescript
test('dashboard', async ({ page }) => {
  await page.goto('/dashboard');   // already authenticated
});
```

---

## 4. API Login (Fastest)

```typescript
setup('authenticate via API', async ({ request }) => {
  await request.post('/api/login', {
    form: { username: process.env.APP_USER!, password: process.env.APP_PASS! },
  });
  await request.storageState({ path: 'playwright/.auth/user.json' });
});
```

---

## 5. Multiple Roles

```typescript
// auth.setup.ts
const adminFile = 'playwright/.auth/admin.json';
const userFile  = 'playwright/.auth/user.json';

setup('admin login', async ({ page }) => {
  // ...login as admin...
  await page.context().storageState({ path: adminFile });
});

setup('user login', async ({ page }) => {
  // ...login as user...
  await page.context().storageState({ path: userFile });
});
```

Use per file/describe:

```typescript
test.describe('admin area', () => {
  test.use({ storageState: 'playwright/.auth/admin.json' });
  test('manage users', async ({ page }) => { /* ... */ });
});
```

Both roles in one test → see "Multi-Context" in [07](07-browser-context-page.md).

---

## 6. Logged-out Tests in an Authenticated Project

```typescript
test.use({ storageState: { cookies: [], origins: [] } });

test('login page shows error for wrong password', async ({ page }) => { /* ... */ });
```

---

## 7. Per-Worker Accounts

When tests change user state (e.g. settings), share one account per worker:

```typescript
export const test = base.extend<{}, { workerStorageState: string }>({
  storageState: ({ workerStorageState }, use) => use(workerStorageState),

  workerStorageState: [async ({ browser }, use) => {
    const id = test.info().parallelIndex;
    const fileName = path.resolve(test.info().project.outputDir, `.auth/${id}.json`);
    if (fs.existsSync(fileName)) { await use(fileName); return; }

    const page = await browser.newPage({ storageState: undefined });
    const account = await acquireAccount(id);          // your helper
    await page.goto('/login');
    await page.getByLabel('Username').fill(account.username);
    await page.getByLabel('Password').fill(account.password);
    await page.getByRole('button', { name: 'Sign in' }).click();
    await page.waitForURL('/dashboard');
    await page.context().storageState({ path: fileName });
    await page.close();
    await use(fileName);
  }, { scope: 'worker' }],
});
```

---

## 8. Other Auth Types

```typescript
// HTTP Basic auth
use: { httpCredentials: { username: 'user', password: 'pass' } }

// Bearer token on every request (UI + API)
use: { extraHTTPHeaders: { Authorization: `Bearer ${process.env.TOKEN}` } }

// Inject token into localStorage before load
await page.addInitScript(token => localStorage.setItem('authToken', token), process.env.TOKEN!);

// Set cookie directly
await context.addCookies([{ name: 'session', value: 'abc', domain: 'app.com', path: '/' }]);

// Session storage isn't saved by storageState — handle manually
const session = await page.evaluate(() => JSON.stringify(sessionStorage));
await context.addInitScript(s => {
  const data = JSON.parse(s);
  for (const [k, v] of Object.entries(data)) window.sessionStorage.setItem(k, v as string);
}, session);
```

### OTP / MFA
- Use a **test account with MFA disabled** in test env, or
- Generate TOTP with `otplib` from a shared secret, or
- Read OTP from email/SMS test inbox API (Mailosaur, MailSlurp).

```typescript
import { authenticator } from 'otplib';
const code = authenticator.generate(process.env.TOTP_SECRET!);
await page.getByLabel('OTP').fill(code);
```

### Save session with codegen

```bash
npx playwright codegen --save-storage=auth.json https://app.com
npx playwright codegen --load-storage=auth.json https://app.com
```

> **Interview one-liner:** I authenticate once in a setup project (via UI or, faster, via API), save cookies/localStorage with `storageState`, and point projects at that file using `dependencies`. For multiple roles I keep one state file per role, override with `test.use({ storageState })`, and for state-mutating tests I use a worker-scoped account.
