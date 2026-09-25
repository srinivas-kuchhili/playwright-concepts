# 06 — Assertions, Auto-Waiting & Timeouts

## 1. What are Web-First Assertions?

Playwright's `expect` for locators/pages are **async, auto-retrying assertions**. They keep re-checking the condition until it passes or the **expect timeout** (default **5s**) expires.

```typescript
await expect(page.getByText('Saved')).toBeVisible();  // retries up to 5s
```

| Auto-retrying (web-first) ✅ | Non-retrying (generic) |
| --------------------------- | ---------------------- |
| `await expect(locator).toBeVisible()` | `expect(value).toBe(5)` |
| Works on `Locator`, `Page`, `APIResponse` | Works on plain values |
| **Must be awaited** | Synchronous |
| Retries until timeout | Checks once |

⚠️ Anti-pattern:
```typescript
expect(await loc.isVisible()).toBe(true);    // ❌ checks once — flaky
await expect(loc).toBeVisible();             // ✅ retries
```

---

## 2. Locator Assertions

| Assertion | Checks |
| --------- | ------ |
| `toBeVisible()` / `toBeHidden()` | Visibility |
| `toBeAttached()` | In DOM |
| `toBeEnabled()` / `toBeDisabled()` | Enabled state |
| `toBeEditable()` | Editable |
| `toBeChecked()` | Checkbox/radio |
| `toBeFocused()` | Has focus |
| `toBeEmpty()` | No text / empty input |
| `toBeInViewport()` | In viewport |
| `toHaveText('x')` / `toHaveText(/x/)` | Exact / regex text (arrays for lists) |
| `toContainText('x')` | Substring |
| `toHaveValue('x')` | Input value |
| `toHaveValues(['a','b'])` | Multi-select values |
| `toHaveAttribute('href', '/x')` | Attribute |
| `toHaveClass(/active/)` | class attribute |
| `toContainClass('active')` | Contains class token |
| `toHaveCSS('color', 'rgb(255, 0, 0)')` | Computed style |
| `toHaveId('submit')` | id |
| `toHaveCount(3)` | Number of elements |
| `toHaveJSProperty('checked', true)` | JS property |
| `toHaveAccessibleName('Submit')` | a11y name |
| `toHaveAccessibleDescription('...')` | a11y description |
| `toHaveRole('button')` | ARIA role |
| `toHaveScreenshot()` | Visual comparison |
| `toMatchAriaSnapshot()` | Accessibility tree structure |

## 3. Page & API Assertions

```typescript
await expect(page).toHaveTitle(/Dashboard/);
await expect(page).toHaveURL('/dashboard');
await expect(page).toHaveScreenshot('home.png');

const res = await request.get('/api/users');
await expect(res).toBeOK();                    // status 200–299
```

## 4. Generic (Value) Assertions

```typescript
expect(5).toBe(5);
expect({ a: 1 }).toEqual({ a: 1 });            // deep equality
expect(obj).toStrictEqual({ a: 1 });           // + undefined props & types
expect(arr).toContain('apple');
expect(arr).toContainEqual({ id: 1 });
expect(arr).toHaveLength(3);
expect(str).toMatch(/hello/i);
expect(num).toBeGreaterThan(10);
expect(num).toBeCloseTo(0.3, 5);
expect(val).toBeTruthy();  expect(val).toBeFalsy();
expect(val).toBeNull();    expect(val).toBeUndefined();  expect(val).toBeDefined();
expect(obj).toHaveProperty('user.name', 'Sam');
expect(obj).toMatchObject({ status: 'active' });   // partial match
expect(() => fn()).toThrow('error');
expect(body).toEqual(expect.objectContaining({ id: expect.any(Number) }));
```

---

## 5. Negation, Custom Message, Custom Timeout

```typescript
await expect(loc).not.toBeVisible();
await expect(loc, 'Cart badge should show 2 items').toHaveText('2');
await expect(loc).toBeVisible({ timeout: 15_000 });
```

---

## 6. Soft Assertions

A **soft assertion** records the failure but **does not stop** the test — all failures are reported at the end.

```typescript
await expect.soft(page.getByTestId('status')).toHaveText('Success');
await expect.soft(page.getByTestId('total')).toHaveText('$100');
await page.getByRole('link', { name: 'Next' }).click();   // still runs

// Stop if any soft assertion failed so far
expect(test.info().errors).toHaveLength(0);
```

| Hard assertion | Soft assertion |
| -------------- | -------------- |
| Stops test on failure | Continues test |
| `expect()` | `expect.soft()` |
| Critical checks | Multiple field validations on a page |

---

## 7. `expect.poll` and `expect().toPass()`

**`expect.poll`** — retries a **function returning a value** (e.g. an API) until it matches.

```typescript
await expect.poll(async () => {
  const res = await page.request.get('/api/job/123');
  return (await res.json()).status;
}, {
  message: 'job should complete',
  intervals: [1_000, 2_000, 5_000],
  timeout: 60_000,
}).toBe('COMPLETED');
```

**`toPass`** — retries a **whole block** of code until all assertions inside pass.

```typescript
await expect(async () => {
  const res = await page.request.get('/api/orders');
  expect(res.status()).toBe(200);
  expect((await res.json()).length).toBeGreaterThan(0);
}).toPass({ intervals: [1_000, 2_000], timeout: 30_000 });
```

| `expect.poll` | `toPass` |
| ------------- | -------- |
| Polls a value | Retries a code block |
| One matcher | Many assertions |
| Default timeout: expect timeout | Default timeout: **0** (set it!) |

---

## 8. Custom Expect Config & Matchers

```typescript
// Reusable slow expect
const slowExpect = expect.configure({ timeout: 20_000 });
await slowExpect(page.getByText('Report ready')).toBeVisible();

const softExpect = expect.configure({ soft: true });

// Custom matcher
import { expect as base, Locator } from '@playwright/test';
export const expect = base.extend({
  async toHaveAmount(locator: Locator, expected: number, options?: { timeout?: number }) {
    let pass = false; let actual = '';
    try {
      await base(locator).toHaveText(`$${expected}`, options);
      pass = true;
    } catch (e: any) { actual = e.message; }
    return {
      pass,
      message: () => `expected amount $${expected}. ${actual}`,
      name: 'toHaveAmount',
    };
  },
});
```

---

## 9. Waits — What & When

💡 **Rule:** With auto-waiting + web-first assertions, you rarely need explicit waits.

| Wait | Use for |
| ---- | ------- |
| `await expect(loc).toBeVisible()` | ✅ Waiting for UI state (preferred) |
| `await loc.waitFor({ state: 'visible' })` | Wait without asserting (`attached`, `detached`, `visible`, `hidden`) |
| `await page.waitForURL('**/home')` | Navigation |
| `await page.waitForLoadState('domcontentloaded')` | Load state (`load`, `domcontentloaded`, `networkidle`) |
| `await page.waitForResponse('**/api/users')` | Wait for API response |
| `await page.waitForRequest(/\/api\/login/)` | Wait for request |
| `await page.waitForEvent('popup')` | Events (popup, download, dialog, filechooser, console) |
| `await page.waitForFunction(() => window.appReady)` | Custom JS condition |
| `await page.waitForTimeout(2000)` | ❌ Hard sleep — debugging only |

### Waiting for an API response triggered by a click

```typescript
const responsePromise = page.waitForResponse(res =>
  res.url().includes('/api/save') && res.status() === 200
);
await page.getByRole('button', { name: 'Save' }).click();
const response = await responsePromise;
expect(await response.json()).toMatchObject({ success: true });
```

---

## 10. Timeouts — Complete Picture

| Timeout | Default | Config | Override |
| ------- | ------- | ------ | -------- |
| **Test timeout** | 30s | `timeout` | `test.setTimeout(60_000)`, `test.slow()` (×3) |
| **Expect timeout** | 5s | `expect.timeout` | `{ timeout }` on the assertion |
| **Action timeout** | none (bounded by test) | `use.actionTimeout` | `click({ timeout })` |
| **Navigation timeout** | none (bounded by test) | `use.navigationTimeout` | `goto(url, { timeout })` |
| **Global timeout** | none | `globalTimeout` | `--global-timeout` |
| **Hook timeout** | = test timeout | – | `testInfo.setTimeout()` in hook |
| **webServer timeout** | 60s | `webServer.timeout` | – |

```typescript
test('long flow', async ({ page }) => {
  test.slow();                  // triples the timeout
  test.setTimeout(120_000);     // or explicit
});
```

> **Interview one-liner:** Playwright assertions are web-first — they auto-retry until the condition is met or the expect timeout hits, so tests don't need sleeps. I use `expect.soft` for multiple non-blocking checks, `expect.poll` / `toPass` for async backend states, and `waitForResponse` when a UI action depends on an API. I never use `waitForTimeout` in real tests.
