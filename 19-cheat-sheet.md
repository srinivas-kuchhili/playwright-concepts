# 19 — One-Page Cheat Sheet

## Setup
```bash
npm init playwright@latest
npx playwright install --with-deps
```

## Run
```bash
npx playwright test                         # all
npx playwright test file.spec.ts:12         # one test by line
npx playwright test -g "title" | --grep @smoke | --grep-invert @slow
npx playwright test --project=chromium --headed --workers=1
npx playwright test --ui | --debug
npx playwright test --last-failed | --only-changed
npx playwright test --repeat-each=10 --retries=2
npx playwright test --shard=1/4
npx playwright test -u                      # update snapshots
npx playwright show-report | show-trace trace.zip
npx playwright codegen <url>
```

## Locators
```typescript
page.getByRole('button', { name: 'Save', exact: true })
page.getByLabel('Email')          page.getByPlaceholder('Search')
page.getByText('Welcome')         page.getByAltText('logo')
page.getByTitle('Close')          page.getByTestId('cart')
page.locator('#id')               page.locator('//xpath')
loc.filter({ hasText, hasNotText, has, hasNot, visible })
loc.first() .last() .nth(i) .count() .all()
loc1.and(loc2)   loc1.or(loc2)
page.frameLocator('#frame').getByRole('button')
```

## Actions
```typescript
await page.goto('/path')
await loc.click() .dblclick() .click({ button: 'right' }) .hover()
await loc.fill('x') .pressSequentially('x') .clear() .press('Enter')
await loc.check() .uncheck() .selectOption('v')
await loc.setInputFiles('f.pdf')      await src.dragTo(dst)
await page.keyboard.press('ControlOrMeta+A')
await page.mouse.wheel(0, 500)
```

## Assertions (auto-retry)
```typescript
await expect(page).toHaveTitle(/x/)    await expect(page).toHaveURL(/x/)
await expect(loc).toBeVisible() .toBeHidden() .toBeEnabled() .toBeChecked()
await expect(loc).toHaveText('x') .toContainText('x') .toHaveValue('x')
await expect(loc).toHaveAttribute('k','v') .toHaveClass(/c/) .toHaveCount(3)
await expect(loc).toHaveScreenshot()   await expect(res).toBeOK()
await expect.soft(loc).toHaveText('x')
await expect.poll(fn).toBe(v)          await expect(async () => {...}).toPass()
```

## Events (register BEFORE the action)
```typescript
const p = page.waitForEvent('popup' | 'download' | 'filechooser' | 'dialog')
const t = context.waitForEvent('page')
const r = page.waitForResponse('**/api/x')
const q = page.waitForRequest('**/api/x')
page.once('dialog', d => d.accept())
```

## Network
```typescript
await page.route('**/api/x', r => r.fulfill({ status: 200, json: {} }))
await page.route('**/api/x', r => r.abort())
await page.route('**/api/x', async r => { const res = await r.fetch(); await r.fulfill({ response: res, json: {...} }) })
await page.route('**/*', r => r.continue({ headers: {...} }))
await page.routeFromHAR('app.har', { update: false })
```

## API
```typescript
const res = await request.get('/api/users', { params: { page: 2 } })
await request.post('/api/users', { data: {...} })     // form / multipart / headers
res.status()  res.ok()  await res.json()  res.headers()
```

## Test structure
```typescript
test.describe('x', { tag: '@smoke' }, () => {
  test.beforeEach(async ({ page }) => {})
  test('y', async ({ page }, testInfo) => {
    await test.step('step', async () => {})
  })
})
test.only / skip / fixme / fail / slow
test.use({ storageState: 'auth.json', viewport: {...}, locale: 'fr-FR' })
test.describe.configure({ mode: 'serial' | 'parallel', retries: 2 })
```

## Fixtures
```typescript
export const test = base.extend<{ loginPage: LoginPage }>({
  loginPage: async ({ page }, use) => { /* setup */ await use(new LoginPage(page)); /* teardown */ },
});
// worker: [fn, { scope: 'worker' }]   auto: [fn, { auto: true }]   option: [default, { option: true }]
```

## Config highlights
```typescript
defineConfig({
  testDir, timeout: 30_000, expect: { timeout: 5_000 }, retries, workers, fullyParallel,
  reporter: [['html'], ['junit', { outputFile }]],
  use: { baseURL, headless, trace: 'on-first-retry', screenshot: 'only-on-failure',
         video: 'retain-on-failure', storageState, testIdAttribute },
  projects: [{ name: 'setup', testMatch: /.*\.setup\.ts/ },
             { name: 'chromium', use: devices['Desktop Chrome'], dependencies: ['setup'] }],
  webServer: { command, url, reuseExistingServer },
})
```

## Golden Rules ⭐
1. Always `await`.
2. User-facing locators (`getByRole`) > test ids > CSS > XPath.
3. Web-first assertions; **never** `waitForTimeout`.
4. Start waiting for events **before** the triggering action.
5. Isolate tests: own context, own data.
6. Log in once → `storageState`.
7. Set up data via API, test through UI.
8. Mock third parties & edge cases with `page.route`.
9. Trace on retry in CI.
10. Fix flakiness at the root; retries are a safety net.
