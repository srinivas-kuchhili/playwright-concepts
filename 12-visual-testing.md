# 12 — Visual Regression Testing

## 1. What is Visual Testing?

**Visual regression testing** compares a **screenshot of the current UI** against a stored **baseline (golden) image** pixel by pixel. If differences exceed a threshold, the test fails and shows **expected / actual / diff** images.

Catches issues functional tests miss: broken CSS, overlapping elements, wrong colors, fonts, layout shifts.

```text
First run  → no baseline → screenshot saved as baseline → test FAILS ("snapshot doesn't exist, writing actual")
Next runs  → compare with baseline → pass / fail with diff
UI changed intentionally → npx playwright test --update-snapshots
```

> Playwright has built-in visual comparison via `toHaveScreenshot()`. The first run creates baselines; later runs compare pixel-by-pixel and produce a diff image in the HTML report.

---

## 2. Basic Usage

```typescript
test('homepage visual', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot();                  // auto name
  await expect(page).toHaveScreenshot('home.png');        // explicit name
  await expect(page).toHaveScreenshot({ fullPage: true });
});

test('component visual', async ({ page }) => {
  await page.goto('/pricing');
  await expect(page.getByTestId('pricing-card')).toHaveScreenshot('pricing-card.png');
});
```

Baseline location:

```text
tests/
  home.spec.ts
  home.spec.ts-snapshots/
     home-chromium-win32.png
     home-firefox-linux.png
```

⚠️ Baselines are **per browser + OS** (fonts & rendering differ). Generate baselines in the **same environment as CI** (use the Playwright Docker image).

---

## 3. Options

| Option | Purpose |
| ------ | ------- |
| `maxDiffPixels: 100` | Allow up to N different pixels |
| `maxDiffPixelRatio: 0.02` | Allow 2% different pixels |
| `threshold: 0.2` | Per-pixel color tolerance (0–1, default 0.2) |
| `fullPage: true` | Whole scrollable page |
| `mask: [locator]` | Cover dynamic elements with a box |
| `maskColor: '#FF00FF'` | Mask color |
| `animations: 'disabled'` | Stop CSS animations (default for toHaveScreenshot) |
| `caret: 'hide'` | Hide text cursor (default) |
| `clip: {x,y,width,height}` | Area of page |
| `omitBackground: true` | Transparent background |
| `stylePath: 'hide.css'` | Inject CSS before screenshot |
| `scale: 'css'` | CSS pixels vs device pixels |
| `timeout` | Retry time for stable screenshot |

```typescript
await expect(page).toHaveScreenshot('dashboard.png', {
  fullPage: true,
  maxDiffPixelRatio: 0.01,
  mask: [page.getByTestId('clock'), page.locator('.ad-banner'), page.getByTestId('avatar')],
  animations: 'disabled',
});
```

Global config:

```typescript
expect: {
  toHaveScreenshot: { maxDiffPixels: 50, animations: 'disabled' },
  toMatchSnapshot: { maxDiffPixelRatio: 0.1 },
},
snapshotPathTemplate: '{testDir}/__screenshots__/{testFilePath}/{arg}{ext}',  // OS-agnostic path
```

---

## 4. Handling Dynamic Content (Flaky Visuals)

| Problem | Solution |
| ------- | -------- |
| Dates / clocks | `mask`, or `page.clock.setFixedTime()` |
| Ads, avatars, random images | `mask` or block via `page.route` |
| Animations / carousels | `animations: 'disabled'`, CSS via `stylePath` |
| Dynamic API data | Mock API with `page.route` |
| Fonts loading late | `await page.evaluate(() => document.fonts.ready)` |
| Lazy images | Scroll / wait for images before snapshot |
| OS/font differences | Run in Docker (same image locally & CI) |

```css
/* hide.css — passed via stylePath */
.carousel, .live-chat, [data-testid="timestamp"] { visibility: hidden !important; }
```

```typescript
await page.clock.setFixedTime(new Date('2025-01-01T10:00:00'));
await page.goto('/');
await expect(page).toHaveScreenshot({ stylePath: 'tests/hide.css' });
```

---

## 5. `toHaveScreenshot` vs `toMatchSnapshot`

| `toHaveScreenshot()` | `toMatchSnapshot()` |
| -------------------- | ------------------- |
| Takes screenshot itself | Compares any value you pass (Buffer/string) |
| **Waits until 2 consecutive screenshots match** (stable) | No retry |
| Auto disables animations, hides caret | No automatic handling |
| For pages/elements | For non-page data (text, images, PDFs) |

```typescript
expect(await page.getByRole('main').textContent()).toMatchSnapshot('main-text.txt');
```

---

## 6. Update Baselines

```bash
npx playwright test --update-snapshots          # or -u : update all
npx playwright test --update-snapshots=changed  # only failing ones
npx playwright test --update-snapshots=missing  # only create missing
npx playwright test visual.spec.ts -u --project=chromium
```

Generate Linux baselines on Windows/Mac using Docker:

```bash
docker run --rm --network host -v "$(pwd)":/work -w /work -it mcr.microsoft.com/playwright:v1.55.0-noble \
  /bin/bash -c "npm ci && npx playwright test --update-snapshots"
```

(Match the image tag to your `@playwright/test` version.)

---

## 7. Responsive Visual Tests

```typescript
const viewports = [
  { name: 'mobile', width: 375, height: 812 },
  { name: 'tablet', width: 768, height: 1024 },
  { name: 'desktop', width: 1440, height: 900 },
];

for (const vp of viewports) {
  test(`home visual - ${vp.name}`, async ({ page }) => {
    await page.setViewportSize({ width: vp.width, height: vp.height });
    await page.goto('/');
    await expect(page).toHaveScreenshot(`home-${vp.name}.png`, { fullPage: true });
  });
}
```

---

## 8. Reviewing Failures

The HTML report shows **Expected**, **Actual**, **Diff**, and a **slider / side-by-side** comparison.

```text
test-results/<test>/home-expected.png
test-results/<test>/home-actual.png
test-results/<test>/home-diff.png
```

---

## 9. Visual Testing Best Practices

- 💡 Keep visual tests in a **separate project** (`--project=visual`, often Chromium only).
- 💡 Snapshot **components** more than full pages — smaller, less flaky.
- 💡 Stabilize data (mock APIs, fixed clock, masks) before capturing.
- 💡 Use the **same Docker image** locally and in CI.
- 💡 Commit baselines to git; review diffs in PRs.
- 💡 For large-scale / cross-browser cloud visual AI: Applitools Eyes, Percy, Argos, Chromatic.

## 10. Accessibility Testing (bonus)

```bash
npm i -D @axe-core/playwright
```

```typescript
import AxeBuilder from '@axe-core/playwright';

test('home has no a11y violations', async ({ page }) => {
  await page.goto('/');
  const results = await new AxeBuilder({ page }).withTags(['wcag2a', 'wcag2aa']).exclude('#ads').analyze();
  expect(results.violations).toEqual([]);
});
```

> **Interview one-liner:** I use `toHaveScreenshot()` for visual regression — first run creates a per-browser/OS baseline, later runs diff against it with tolerances like `maxDiffPixelRatio`. I stabilize screenshots by masking dynamic elements, freezing time with `page.clock`, disabling animations, mocking data, and generating baselines in the same Docker image as CI; `--update-snapshots` refreshes baselines for intended changes.
