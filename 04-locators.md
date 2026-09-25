# 04 — Locators & Selectors

## 1. What is a Locator?

A **Locator** is Playwright's way to **find element(s) on the page at any moment**. It is **lazy** — it does not search the DOM when created; it resolves the element **every time an action or assertion is performed**, and it **auto-waits** for the element to be actionable.

```typescript
const loginBtn = page.getByRole('button', { name: 'Login' }); // nothing happens yet
await loginBtn.click();                                         // resolved + auto-waited here
```

### Locator vs ElementHandle

| `Locator` ✅ | `ElementHandle` ❌ (discouraged) |
| ----------- | -------------------------------- |
| Lazy — re-queries DOM on every use | Points to one specific DOM node |
| Never stale — survives re-renders | Becomes stale if DOM re-renders |
| Auto-waits & retries | No auto-retry |
| Strict — errors if multiple match | Takes whatever matched |
| `page.locator()`, `page.getByRole()` | `page.$()`, `page.$$()` |

> A locator is a lazy, auto-waiting, strict reference to an element. Because it re-queries the DOM each time, it never goes stale — unlike Selenium's `WebElement` or Playwright's old `ElementHandle`.

---

## 2. Built-in (Recommended) Locators — Priority Order

| Priority | Locator | Finds by | Example |
| -------- | ------- | -------- | ------- |
| 1 | `getByRole()` | ARIA role + accessible name | `page.getByRole('button', { name: 'Submit' })` |
| 2 | `getByLabel()` | `<label>` / `aria-label` text | `page.getByLabel('Email')` |
| 3 | `getByPlaceholder()` | placeholder attribute | `page.getByPlaceholder('Search')` |
| 4 | `getByText()` | visible text | `page.getByText('Welcome back')` |
| 5 | `getByAltText()` | `alt` of images | `page.getByAltText('logo')` |
| 6 | `getByTitle()` | `title` attribute | `page.getByTitle('Close')` |
| 7 | `getByTestId()` | `data-testid` (configurable) | `page.getByTestId('cart-count')` |
| 8 | `locator()` | CSS / XPath | `page.locator('#id')`, `page.locator('//div')` |

💡 **Why `getByRole` first?** It mirrors how users & assistive technologies perceive the page, it's resilient to CSS/DOM changes, and it doubles as an accessibility check.

### Examples

```typescript
// Roles: button, link, textbox, checkbox, radio, combobox, heading, listitem, row, cell, tab, dialog, img, navigation...
await page.getByRole('button', { name: 'Sign in' }).click();
await page.getByRole('link', { name: /forgot password/i }).click();
await page.getByRole('heading', { name: 'Dashboard', level: 1 });
await page.getByRole('checkbox', { name: 'Remember me' }).check();
await page.getByRole('textbox', { name: 'Username' }).fill('admin');
await page.getByRole('row', { name: 'John Doe' });
await page.getByRole('button', { name: 'Save', exact: true });   // exact match
await page.getByRole('tab', { selected: true });
await page.getByRole('checkbox', { checked: true });
await page.getByRole('button', { disabled: true });

await page.getByLabel('Password').fill('secret');
await page.getByPlaceholder('name@example.com').fill('a@b.com');
await page.getByText('Order placed', { exact: true });
await page.getByText(/total: \$\d+/i);
await page.getByAltText('Company logo').click();
await page.getByTitle('Issues count');
await page.getByTestId('submit-btn').click();
```

Change test id attribute:

```typescript
// playwright.config.ts
use: { testIdAttribute: 'data-qa' }
```

---

## 3. CSS & XPath Locators

```typescript
// CSS
page.locator('#username');
page.locator('.btn.primary');
page.locator('input[name="email"]');
page.locator('form >> button');                 // legacy chaining syntax
page.locator('ul > li:nth-child(2)');
page.locator('button:has-text("Save")');        // Playwright pseudo-class
page.locator('div:has(> img)');
page.locator('button:visible');

// XPath (auto-detected when starting with // or ..)
page.locator('//button[text()="Login"]');
page.locator('xpath=//input[@id="email"]');
page.locator('//td[text()="John"]/following-sibling::td[2]');
```

⚠️ XPath and long CSS chains are brittle — use only when no user-facing attribute exists.

---

## 4. Filtering Locators

| Method | Purpose |
| ------ | ------- |
| `.filter({ hasText })` | Contains text |
| `.filter({ hasNotText })` | Does not contain text |
| `.filter({ has })` | Contains a child locator |
| `.filter({ hasNot })` | Does not contain a child locator |
| `.filter({ visible: true })` | Only visible elements |

```typescript
// Click "Add to cart" for the product called "Backpack"
await page
  .getByRole('listitem')
  .filter({ hasText: 'Backpack' })
  .getByRole('button', { name: 'Add to cart' })
  .click();

// Row that contains a "Delete" button
page.getByRole('row').filter({ has: page.getByRole('button', { name: 'Delete' }) });

// Items NOT out of stock
page.getByRole('listitem').filter({ hasNotText: 'Out of stock' });
```

---

## 5. Chaining, `and`, `or`

```typescript
// Chaining = search inside
const form = page.locator('form#login');
await form.getByLabel('Email').fill('x@y.com');

// AND — element must match both
page.getByRole('button').and(page.getByTitle('Subscribe'));

// OR — whichever appears (e.g. dialog OR button)
const newEmail = page.getByRole('button', { name: 'New' });
const dialog = page.getByText('Confirm security settings');
await expect(newEmail.or(dialog).first()).toBeVisible();
```

---

## 6. Handling Multiple Elements

| Method | Returns |
| ------ | ------- |
| `.first()` | First match |
| `.last()` | Last match |
| `.nth(i)` | i-th (0-based) |
| `.count()` | Number of matches |
| `.all()` | `Locator[]` (no waiting!) |
| `.allTextContents()` | `string[]` |
| `.allInnerTexts()` | `string[]` visible text |

```typescript
const items = page.getByRole('listitem');
await expect(items).toHaveCount(5);          // ✅ waits
console.log(await items.count());            // ⚠️ no waiting, snapshot value

for (const item of await items.all()) {
  console.log(await item.textContent());
}

const names = await page.locator('.product-name').allTextContents();
```

⚠️ `.all()` and `.count()` don't wait — assert the count first (`toHaveCount`) to avoid empty arrays.

---

## 7. Strictness

Locators are **strict**: an action on a locator matching **more than one** element throws:

```text
Error: strict mode violation: getByRole('button') resolved to 3 elements
```

Fixes: make the locator more specific, use `filter()`, `exact: true`, or `first()/nth()` (last resort).

> Strict mode prevents clicking the wrong element silently — Playwright forces you to write unambiguous locators.

---

## 8. Special Cases

### Shadow DOM
Playwright locators **pierce open shadow DOM automatically** (CSS & getBy*). XPath does **not** pierce shadow roots.

```typescript
await page.locator('my-component button').click();  // works through shadow root
```

### iframes

```typescript
const frame = page.frameLocator('#payment-iframe');
await frame.getByLabel('Card number').fill('4242 4242 4242 4242');

// or from a locator
await page.locator('#payment-iframe').contentFrame().getByRole('button', { name: 'Pay' }).click();
```

### Dynamic elements / tables

```typescript
// Get the "Status" cell of the row whose name is "Alice"
const row = page.getByRole('row', { name: /Alice/ });
await expect(row.getByRole('cell').nth(3)).toHaveText('Active');
```

### Parent / sibling via XPath when unavoidable

```typescript
page.getByText('Alice').locator('xpath=..');                         // parent
page.getByText('Alice').locator('xpath=following-sibling::td[1]');
```

---

## 9. Picking Locators Quickly

- `npx playwright codegen <url>` → hover to see the best locator
- UI Mode (`--ui`) → **Pick locator** button
- VS Code extension → **Pick locator** / **Record at cursor**
- `await page.pause()` → Inspector → **Pick locator**

---

## 10. ARIA Snapshot (structure assertion)

```typescript
await expect(page.getByRole('navigation')).toMatchAriaSnapshot(`
  - navigation:
    - link "Home"
    - link "Products"
    - link "Contact"
`);
```

---

> **Interview one-liner:** I prefer user-facing locators — `getByRole`, `getByLabel`, `getByText`, `getByTestId` — over CSS/XPath because they're resilient and reflect accessibility. Locators are lazy, strict and auto-waiting; I narrow them with `filter()`, chaining, `and/or`, and use `frameLocator` for iframes. Shadow DOM is pierced by default.
