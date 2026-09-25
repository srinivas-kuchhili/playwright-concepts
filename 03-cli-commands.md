# 03 — CLI Commands (Complete Reference)

All commands are run with `npx playwright <command>`.

---

## 1. Installation & Setup

| Command | Purpose |
| ------- | ------- |
| `npm init playwright@latest` | Scaffold a new project |
| `npm i -D @playwright/test` | Add to existing project |
| `npx playwright install` | Download all browsers |
| `npx playwright install chromium firefox` | Specific browsers |
| `npx playwright install --with-deps` | Browsers + OS libraries (Linux/CI) |
| `npx playwright install-deps` | Only OS dependencies |
| `npx playwright install chrome` / `msedge` | Branded browsers |
| `npx playwright uninstall` | Remove browsers of this version |
| `npx playwright uninstall --all` | Remove all browsers |
| `npx playwright --version` | Show version |
| `npx playwright --help` | Help |

---

## 2. Running Tests

| Command | Purpose |
| ------- | ------- |
| `npx playwright test` | Run all tests (headless) |
| `npx playwright test login.spec.ts` | Run one file |
| `npx playwright test tests/auth/` | Run a folder |
| `npx playwright test login signup` | Files whose name contains "login" or "signup" |
| `npx playwright test login.spec.ts:25` | Run test at line 25 |
| `npx playwright test -g "valid login"` | Run by title (`--grep`) |
| `npx playwright test --grep @smoke` | Run by tag |
| `npx playwright test --grep-invert @slow` | Exclude tag |
| `npx playwright test --grep "@smoke\|@sanity"` | OR of tags (regex) |
| `npx playwright test --grep "(?=.*@smoke)(?=.*@login)"` | AND of tags |
| `npx playwright test --project=chromium` | One project/browser |
| `npx playwright test --headed` | Show browser |
| `npx playwright test --workers=4` | 4 parallel workers |
| `npx playwright test --workers=1` | Serial execution |
| `npx playwright test --fully-parallel` | Parallelize inside files |
| `npx playwright test --retries=2` | Retry failed tests |
| `npx playwright test --repeat-each=5` | Run each test 5 times (flaky check) |
| `npx playwright test --max-failures=3` / `-x` | Stop after N / first failure |
| `npx playwright test --timeout=60000` | Per-test timeout |
| `npx playwright test --global-timeout=600000` | Whole run timeout |
| `npx playwright test --last-failed` | Re-run only the failed tests of last run |
| `npx playwright test --only-changed` | Tests affected by git changes |
| `npx playwright test --only-changed=main` | Changed vs a branch |
| `npx playwright test --forbid-only` | Fail if `test.only` exists |
| `npx playwright test --list` | List tests without running |
| `npx playwright test --pass-with-no-tests` | Don't fail when nothing matches |
| `npx playwright test --shard=1/4` | Run shard 1 of 4 |
| `npx playwright test --config=playwright.qa.config.ts` | Custom config |
| `npx playwright test --output=out` | Artifacts folder |
| `npx playwright test --quiet` | Hide stdout from tests |
| `npx playwright test --tsconfig=tsconfig.test.json` | Custom tsconfig |
| `npx playwright test --fail-on-flaky-tests` | Treat flaky tests as failures |
| `npx playwright test --no-deps` | Skip project dependencies |
| `npx playwright test --ignore-snapshots` | Skip screenshot assertions |

---

## 3. Debugging

| Command | Purpose |
| ------- | ------- |
| `npx playwright test --ui` | **UI Mode** — watch, time-travel, pick locators |
| `npx playwright test --debug` | Opens **Playwright Inspector**, step through |
| `npx playwright test login.spec.ts:10 --debug` | Debug a single test |
| `npx playwright test --trace on` | Force trace for all tests |
| `npx playwright test --trace retain-on-failure` | Keep trace for failures |
| `PWDEBUG=1 npx playwright test` | Inspector via env var |
| `DEBUG=pw:api npx playwright test` | Verbose API logs |
| `DEBUG=pw:browser npx playwright test` | Browser logs |

Windows PowerShell: `$env:PWDEBUG=1; npx playwright test`

In code:

```typescript
await page.pause();   // pause and open Inspector (headed)
```

---

## 4. Reports & Traces

| Command | Purpose |
| ------- | ------- |
| `npx playwright show-report` | Open last HTML report |
| `npx playwright show-report my-report` | Open report from a folder |
| `npx playwright test --reporter=list` | Reporter via CLI |
| `npx playwright test --reporter=dot,html` | Multiple reporters |
| `npx playwright test --reporter=line` | Compact reporter |
| `npx playwright show-trace trace.zip` | Open trace viewer |
| `npx playwright merge-reports --reporter html ./blob-report` | Merge shard reports |

Online viewer: drag trace.zip onto **https://trace.playwright.dev**

---

## 5. Codegen (Record Tests)

| Command | Purpose |
| ------- | ------- |
| `npx playwright codegen` | Start recorder |
| `npx playwright codegen https://demo.com` | Record on a URL |
| `npx playwright codegen --target=python` | Generate in another language (`javascript`, `python`, `java`, `csharp`) |
| `npx playwright codegen -o tests/rec.spec.ts` | Save output |
| `npx playwright codegen --device="iPhone 15"` | Emulate device |
| `npx playwright codegen --viewport-size=800,600` | Viewport |
| `npx playwright codegen --color-scheme=dark` | Dark mode |
| `npx playwright codegen --geolocation="41.89,12.49" --lang="it-IT" --timezone="Europe/Rome"` | Emulation |
| `npx playwright codegen --save-storage=auth.json` | Log in manually & save session |
| `npx playwright codegen --load-storage=auth.json` | Start logged in |
| `npx playwright codegen -b firefox` | Choose browser |

---

## 6. Visual / Snapshots

| Command | Purpose |
| ------- | ------- |
| `npx playwright test --update-snapshots` / `-u` | Update baseline screenshots |
| `npx playwright test -u --update-source-method=overwrite` | Update inline aria snapshots in source |
| `npx playwright test --update-snapshots=changed` | Only update changed snapshots |

---

## 7. Other Utilities

| Command | Purpose |
| ------- | ------- |
| `npx playwright open https://example.com` | Open a browser page |
| `npx playwright screenshot https://example.com shot.png` | Screenshot from CLI |
| `npx playwright screenshot --full-page URL out.png` | Full page screenshot |
| `npx playwright pdf https://example.com out.pdf` | PDF (Chromium) |
| `npx playwright clear-cache` | Clear Playwright cache |
| `npx playwright init-agents --loop=vscode` | (v1.56+) Scaffold AI test agents — planner, generator, healer (`--loop=claude` / `opencode` also supported) |

---

## 8. Most Used — Memorize These 10

```bash
npx playwright test
npx playwright test --headed
npx playwright test --ui
npx playwright test --debug
npx playwright test --project=chromium
npx playwright test -g "title"      # or --grep @tag
npx playwright test --last-failed
npx playwright test -u
npx playwright show-report
npx playwright codegen <url>
```

> **Interview one-liner:** I run tests with `npx playwright test`, filter using `--grep` tags and `--project`, debug with `--ui` or `--debug`, re-run failures with `--last-failed`, open reports with `show-report`, analyse CI failures via `show-trace`, and use `--shard` + `merge-reports` to split large suites across machines.
