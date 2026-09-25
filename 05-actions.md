# 05 — Actions

Actions are operations performed on locators (click, type, select...). **Every action auto-waits** for the element to pass **actionability checks** before acting.

## 1. Actionability Checks (Auto-wait)

| Check | Meaning | Applies to |
| ----- | ------- | ---------- |
| **Attached** | Element is in the DOM | all |
| **Visible** | Non-empty bounding box, not `visibility:hidden` | click, fill, hover, check... |
| **Stable** | Not animating (same box for 2 frames) | click, hover, check, tap |
| **Receives events** | Not covered by another element | click, hover, check, tap |
| **Enabled** | Not `disabled` | click, fill, check, select |
| **Editable** | Enabled and not `readonly` | fill, clear |

> Before an action, Playwright waits until the element is attached, visible, stable, enabled and not obscured. That's why we don't need explicit waits or sleeps.

`force: true` skips non-essential checks (use rarely, e.g. custom hidden checkbox).

---

## 2. Action Reference

| Action | Code |
| ------ | ---- |
| Navigate | `await page.goto('/login')` |
| Click | `await loc.click()` |
| Double click | `await loc.dblclick()` |
| Right click | `await loc.click({ button: 'right' })` |
| Click with modifier | `await loc.click({ modifiers: ['Shift'] })` (`Control`, `Alt`, `Meta`, `ControlOrMeta`) |
| Click at position | `await loc.click({ position: { x: 10, y: 5 } })` |
| Hover | `await loc.hover()` |
| Fill input (clears first) | `await loc.fill('text')` |
| Type char-by-char | `await loc.pressSequentially('text', { delay: 100 })` |
| Clear | `await loc.clear()` |
| Press key | `await loc.press('Enter')` |
| Check / uncheck | `await loc.check()` / `await loc.uncheck()` |
| Set checked state | `await loc.setChecked(true)` |
| Select dropdown | `await loc.selectOption('blue')` |
| Focus / blur | `await loc.focus()` / `await loc.blur()` |
| Upload file | `await loc.setInputFiles('file.pdf')` |
| Drag & drop | `await src.dragTo(target)` |
| Scroll into view | `await loc.scrollIntoViewIfNeeded()` |
| Tap (mobile) | `await loc.tap()` |
| Dispatch event | `await loc.dispatchEvent('click')` |
| Highlight (debug) | `await loc.highlight()` |

---

## 3. Text Input — `fill` vs `pressSequentially` vs `type`

| `fill()` | `pressSequentially()` | `type()` |
| -------- | --------------------- | -------- |
| Sets value instantly, clears first | Types key by key, fires keydown/keyup | **Deprecated** — use `pressSequentially` |
| Fast, recommended for most inputs | Needed for autocomplete, key handlers, masks | – |

```typescript
await page.getByLabel('Email').fill('user@test.com');
await page.getByLabel('City').pressSequentially('Bang', { delay: 150 });
await page.getByRole('option', { name: 'Bangalore' }).click();
```

---

## 4. Dropdowns

```typescript
// Native <select>
await page.getByLabel('Country').selectOption('IN');                 // by value
await page.getByLabel('Country').selectOption({ label: 'India' });   // by label
await page.getByLabel('Country').selectOption({ index: 2 });         // by index
await page.getByLabel('Colors').selectOption(['red', 'green']);      // multi-select

await expect(page.getByLabel('Country')).toHaveValue('IN');

// Custom (div-based) dropdown
await page.getByRole('combobox', { name: 'Country' }).click();
await page.getByRole('option', { name: 'India' }).click();
```

---

## 5. Checkboxes & Radio Buttons

```typescript
const terms = page.getByRole('checkbox', { name: 'I agree' });
await terms.check();
await expect(terms).toBeChecked();
await terms.uncheck();
await expect(terms).not.toBeChecked();

await page.getByRole('radio', { name: 'Female' }).check();
console.log(await terms.isChecked());   // boolean (no waiting)
```

---

## 6. Keyboard

```typescript
await page.keyboard.press('Tab');
await page.keyboard.press('Control+A');
await page.keyboard.press('ControlOrMeta+C');   // Ctrl on Win/Linux, Cmd on Mac
await page.keyboard.type('Hello');
await page.keyboard.down('Shift');
await page.keyboard.press('ArrowRight');
await page.keyboard.up('Shift');
await page.keyboard.insertText('😀');

await page.getByLabel('Search').press('Enter');
```

Key names: `Enter`, `Tab`, `Escape`, `Backspace`, `Delete`, `ArrowUp/Down/Left/Right`, `Home`, `End`, `PageUp`, `PageDown`, `F1–F12`, `Shift`, `Control`, `Alt`, `Meta`.

---

## 7. Mouse

```typescript
await page.mouse.move(100, 200);
await page.mouse.down();
await page.mouse.move(300, 200, { steps: 10 });
await page.mouse.up();
await page.mouse.click(50, 50);
await page.mouse.dblclick(50, 50);
await page.mouse.wheel(0, 500);          // scroll down
```

---

## 8. Drag and Drop

```typescript
// Simple
await page.locator('#source').dragTo(page.locator('#target'));

// Manual (for libraries needing intermediate moves)
await page.locator('#source').hover();
await page.mouse.down();
await page.locator('#target').hover();
await page.locator('#target').hover();   // second move triggers dragover in some libs
await page.mouse.up();
```

---

## 9. File Upload

```typescript
// Single / multiple
await page.getByLabel('Upload').setInputFiles('test-data/resume.pdf');
await page.getByLabel('Upload').setInputFiles(['a.png', 'b.png']);

// From memory buffer
await page.getByLabel('Upload').setInputFiles({
  name: 'data.txt',
  mimeType: 'text/plain',
  buffer: Buffer.from('hello'),
});

// Clear selection
await page.getByLabel('Upload').setInputFiles([]);

// When there is no <input type=file> visible (button opens OS chooser)
const fileChooserPromise = page.waitForEvent('filechooser');
await page.getByRole('button', { name: 'Upload file' }).click();
const fileChooser = await fileChooserPromise;
await fileChooser.setFiles('test-data/resume.pdf');
```

---

## 10. File Download

```typescript
const downloadPromise = page.waitForEvent('download');
await page.getByRole('link', { name: 'Download report' }).click();
const download = await downloadPromise;

console.log(download.suggestedFilename());
await download.saveAs(`downloads/${download.suggestedFilename()}`);
const path = await download.path();      // temp path
```

⚠️ Start waiting **before** the click so the event isn't missed.

---

## 11. Reading Values (no auto-wait — prefer assertions)

| Method | Returns |
| ------ | ------- |
| `textContent()` | All text incl. hidden |
| `innerText()` | Rendered visible text |
| `inputValue()` | Value of input/textarea/select |
| `getAttribute('href')` | Attribute value |
| `isVisible()` / `isHidden()` | boolean, immediate |
| `isEnabled()` / `isDisabled()` / `isEditable()` / `isChecked()` | boolean |
| `boundingBox()` | `{x, y, width, height}` |
| `page.title()` / `page.url()` | strings |

⚠️ `isVisible()` does **not** wait. For verification, use `await expect(loc).toBeVisible()`.

---

## 12. JavaScript Execution

```typescript
const title = await page.evaluate(() => document.title);
const width = await page.evaluate(() => window.innerWidth);
await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));

// pass args
const sum = await page.evaluate(([a, b]) => a + b, [2, 3]);

// on an element
const color = await page.getByRole('button').evaluate(el => getComputedStyle(el).color);

// on all matches
const hrefs = await page.locator('a').evaluateAll(els => els.map(e => e.getAttribute('href')));

// inject script before page loads
await page.addInitScript(() => { (window as any).__TEST__ = true; });

// expose Node function to the browser
await page.exposeFunction('sha256', (text: string) => require('crypto').createHash('sha256').update(text).digest('hex'));
```

---

## 13. Navigation

```typescript
await page.goto('https://site.com', { waitUntil: 'domcontentloaded' }); // 'load' (default) | 'domcontentloaded' | 'networkidle' | 'commit'
await page.goBack();
await page.goForward();
await page.reload();
await page.waitForURL('**/dashboard');
await expect(page).toHaveURL(/dashboard/);
```

⚠️ `networkidle` is discouraged for testing — assert on UI state instead.

---

## 14. Infinite Scroll / Lazy Load

```typescript
const items = page.locator('.item');
while ((await items.count()) < 50) {
  await page.mouse.wheel(0, 2000);
  await page.waitForTimeout(300);   // ok here only as a throttle; prefer waiting on a response
}
```

---

> **Interview one-liner:** Every Playwright action auto-waits for actionability — attached, visible, stable, enabled and receiving events. I use `fill` for inputs, `pressSequentially` for key-driven widgets, `selectOption` for native selects, `setInputFiles` / `filechooser` for uploads, and `waitForEvent('download')` for downloads — always registering the wait before the triggering click.
