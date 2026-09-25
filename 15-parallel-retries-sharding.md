# 15 — Parallelism, Retries, Sharding & Flaky Tests

## 1. How Parallelism Works

Playwright runs tests in **worker processes** (separate OS processes). Each worker has its own browser; each test gets a fresh context.

```text
Test Runner
 ├── Worker 0 ── Browser ── test A (ctx), test B (ctx)
 ├── Worker 1 ── Browser ── test C (ctx), test D (ctx)
 └── Worker 2 ── Browser ── test E (ctx)
```

| Setting | Behaviour |
| ------- | --------- |
| Default | **Files** run in parallel; tests **inside a file** run in order in one worker |
| `fullyParallel: true` | Every test can run in parallel |
| `test.describe.configure({ mode: 'parallel' })` | Parallel within a file/describe |
| `test.describe.configure({ mode: 'serial' })` | Sequential; stop on failure |
| `workers: 4` / `--workers=4` / `--workers=50%` | Number of workers |
| `workers: 1` | Fully sequential |

- `testInfo.workerIndex` — unique id of worker process (changes when worker restarts)
- `testInfo.parallelIndex` — 0..workers-1 (stable slot — use it for per-worker test data)

⚠️ A worker is **restarted after a test failure** (clean state), so `beforeAll` may run again.

> By default Playwright runs files in parallel across workers; `fullyParallel` also parallelizes tests within a file. Every test gets its own browser context, so parallel tests don't share cookies or storage.

---

## 2. Retries

```typescript
retries: process.env.CI ? 2 : 0,
```

```bash
npx playwright test --retries=3
```

```typescript
test.describe.configure({ retries: 2 });   // per describe
```

### Test outcomes

| Status | Meaning |
| ------ | ------- |
| **passed** | Passed first time |
| **flaky** | Failed first, passed on retry ⚠️ |
| **failed** | Failed on all attempts |
| **skipped** | Skipped |

Detect retry in code:

```typescript
test('x', async ({ page }, testInfo) => {
  if (testInfo.retry > 0) await clearCache();
});
```

---

## 3. Sharding (split across machines)

```bash
npx playwright test --shard=1/4
npx playwright test --shard=2/4
npx playwright test --shard=3/4
npx playwright test --shard=4/4
```

Merge reports with the **blob** reporter:

```typescript
reporter: process.env.CI ? 'blob' : 'html',
```

```bash
npx playwright merge-reports --reporter html ./all-blob-reports
```

| Workers | Shards |
| ------- | ------ |
| Parallel processes on **one machine** | Split suite across **many machines** |
| `--workers=4` | `--shard=1/4` |
| Limited by CPU/RAM | Scales horizontally in CI |

💡 Use `fullyParallel: true` for balanced shards (otherwise shards split by file).

---

## 4. Flaky Tests — Causes & Fixes

A **flaky test** passes and fails intermittently without code changes.

| Cause | Fix |
| ----- | --- |
| Hard waits (`waitForTimeout`) | Web-first assertions / `waitForResponse` |
| Non-awaited promises | `await` everything; ESLint `no-floating-promises` |
| Reading state without waiting (`isVisible()`, `count()`) | `expect(loc).toBeVisible()`, `toHaveCount` |
| Tests depend on each other / shared data | Isolated data per test (API setup, unique names) |
| Unstable test data / backend | Mock APIs, seed DB |
| Animations | `animations: 'disabled'` / wait for stable state |
| Brittle locators | `getByRole`, `getByTestId` |
| Race: action before listener | Register `waitForEvent`/`waitForResponse` **before** action |
| Random popups | `page.addLocatorHandler` |
| Timezone / date | `timezoneId`, `page.clock` |
| Parallel collisions (same user) | Per-worker accounts via `parallelIndex` |
| Env slowness | Tune timeouts, `test.slow()` |

### Detect & reproduce

```bash
npx playwright test --repeat-each=20 --workers=4 login.spec.ts
npx playwright test --fail-on-flaky-tests
```

> I first reproduce with `--repeat-each`, analyse the trace from the flaky attempt, then fix the root cause — usually a missing await, a non-retrying check, a race with a network call, or shared test data. Retries are a safety net, not a fix.

---

## 5. Controlling Order & Isolation

```typescript
// Test must run alone (e.g., modifies global settings)
test.describe.configure({ mode: 'serial' });

// Or a dedicated project with workers: 1 (per-project workers, v1.52+)
{ name: 'global-settings', testMatch: /settings\.spec\.ts/, workers: 1 }
```

Unique data per parallel test:

```typescript
test('create project', async ({ page }, testInfo) => {
  const name = `Project-${testInfo.parallelIndex}-${Date.now()}`;
});
```

---

## 6. Performance Tips

- 💡 Reuse auth with `storageState` (skip UI login).
- 💡 Create data via API, not UI.
- 💡 Block images / analytics with `route.abort()`.
- 💡 `fullyParallel: true` + appropriate workers.
- 💡 Shard in CI.
- 💡 Only record trace/video on retry/failure.
- 💡 Run smoke tags on PR, full regression nightly.
- 💡 `--only-changed` for fast feedback locally.

> **Interview one-liner:** Tests run in parallel worker processes, each test in its own context. I enable `fullyParallel`, tune workers, shard across CI machines with `--shard` and merge blob reports. I use retries with `trace: on-first-retry` to diagnose flakiness, and fix flaky tests at the root rather than relying on retries.
