# 16 — Emulation: Devices, Viewport, Geolocation, Locale, Timezone, Clock

## 1. What is Emulation?

**Emulation** makes a desktop browser **behave like another device or environment** — screen size, user agent, touch, geolocation, language, timezone, color scheme, network — without real hardware.

⚠️ It's **emulation, not real devices**. For real mobile devices/native apps use Appium or cloud grids (BrowserStack, Sauce Labs, LambdaTest).

---

## 2. Devices

```typescript
import { devices } from '@playwright/test';

projects: [
  { name: 'Mobile Chrome', use: { ...devices['Pixel 7'] } },
  { name: 'Mobile Safari', use: { ...devices['iPhone 15 Pro'] } },
  { name: 'Tablet',        use: { ...devices['iPad Pro 11'] } },
  { name: 'Landscape',     use: { ...devices['iPhone 13 landscape'] } },
]
```

A device descriptor sets: `userAgent`, `viewport`, `deviceScaleFactor`, `isMobile`, `hasTouch`, `defaultBrowserType`.

```typescript
test.use({ ...devices['iPhone 15'] });   // per file
```

## 3. Viewport

```typescript
use: { viewport: { width: 1920, height: 1080 } }

test('responsive menu', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 667 });
  await page.goto('/');
  await expect(page.getByRole('button', { name: 'Menu' })).toBeVisible();
});

use: { viewport: null }   // use real window size (with --start-maximized launch arg, Chromium)
```

## 4. Options Reference

| Option | Example |
| ------ | ------- |
| `viewport` | `{ width: 1280, height: 720 }` |
| `userAgent` | `'MyBot/1.0'` |
| `isMobile` / `hasTouch` | `true` |
| `deviceScaleFactor` | `2` |
| `locale` | `'fr-FR'` |
| `timezoneId` | `'America/New_York'` |
| `geolocation` | `{ latitude: 20.29, longitude: 85.82 }` |
| `permissions` | `['geolocation', 'notifications']` |
| `colorScheme` | `'dark' \| 'light' \| 'no-preference'` |
| `reducedMotion` | `'reduce'` |
| `forcedColors` | `'active'` |
| `offline` | `true` |
| `javaScriptEnabled` | `false` |
| `acceptDownloads` | `true` (default) |
| `bypassCSP` | `true` |
| `httpCredentials` | `{ username, password }` |
| `proxy` | `{ server: 'http://proxy:3128' }` |

## 5. Geolocation

```typescript
test.use({
  geolocation: { latitude: 20.2961, longitude: 85.8245 },   // Bhubaneswar
  permissions: ['geolocation'],
});

test('shows nearest store', async ({ page, context }) => {
  await page.goto('/stores');
  await expect(page.getByText('Bhubaneswar')).toBeVisible();

  await context.setGeolocation({ latitude: 19.076, longitude: 72.8777 });   // Mumbai
  await page.reload();
});
```

## 6. Locale & Timezone

```typescript
test.use({ locale: 'de-DE', timezoneId: 'Europe/Berlin' });

test('german formatting', async ({ page }) => {
  await page.goto('/checkout');
  await expect(page.getByTestId('price')).toHaveText('1.234,50 €');
});
```

## 7. Color Scheme / Media

```typescript
test.use({ colorScheme: 'dark' });

await page.emulateMedia({ colorScheme: 'dark' });
await page.emulateMedia({ media: 'print' });
await page.emulateMedia({ reducedMotion: 'reduce' });
```

## 8. Clock API — Control Time ⏰

```typescript
// Freeze Date.now() but timers still run
await page.clock.setFixedTime(new Date('2025-12-25T10:00:00'));
await page.goto('/');
await expect(page.getByTestId('greeting')).toHaveText('Merry Christmas!');

// Full control of timers
await page.clock.install({ time: new Date('2025-01-01T08:00:00') });
await page.goto('/session');
await page.clock.fastForward('30:00');         // jump 30 minutes
await expect(page.getByText('Session expired')).toBeVisible();

await page.clock.runFor(5000);                 // run timers for 5s
await page.clock.pauseAt(new Date('2025-01-01T09:00:00'));
await page.clock.resume();
```

Use cases: session timeouts, countdowns, date-based banners, OTP expiry, stable screenshots.

## 9. Browser Channels & Launch Options

```typescript
use: {
  channel: 'chrome',            // 'chrome', 'chrome-beta', 'msedge', 'msedge-dev'
  launchOptions: {
    slowMo: 200,
    args: ['--start-maximized', '--disable-extensions'],
    downloadsPath: 'downloads',
  },
}
```

## 10. Persistent Context (real profile / extensions)

```typescript
import { chromium } from '@playwright/test';
import path from 'path';

const pathToExtension = path.join(__dirname, 'my-extension');
const context = await chromium.launchPersistentContext('user-data-dir', {
  channel: 'chromium',
  args: [`--disable-extensions-except=${pathToExtension}`, `--load-extension=${pathToExtension}`],
});
```

## 11. Connect to Existing Browser / Remote

```typescript
const browser = await chromium.connectOverCDP('http://localhost:9222');   // existing Chrome
const remote  = await chromium.connect('wss://grid.example.com/playwright'); // remote Playwright server
```

```bash
npx playwright run-server --port 3000
```

> **Interview one-liner:** I use `devices[...]` presets in projects for mobile/tablet emulation, `geolocation` + `permissions` for location features, `locale`/`timezoneId` for i18n, `colorScheme` for dark mode, and `page.clock` to freeze or fast-forward time for session-expiry and date-based tests.
