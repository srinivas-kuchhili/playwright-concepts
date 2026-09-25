# Playwright — Complete Study & Interview Guide (TypeScript)

> A topic-wise handbook for learning Playwright and preparing for interviews.
> Each topic follows the same pattern: **What it is → Types / Comparison table → Code → Key points → Interview one-liner**.

---

## 📚 Table of Contents

| #  | Topic | File |
| -- | ----- | ---- |
| 01 | What is Playwright, Architecture, Installation, Project Structure | [01-introduction.md](01-introduction.md) |
| 02 | Configuration (`playwright.config.ts`) | [02-configuration.md](02-configuration.md) |
| 03 | CLI Commands — every command & flag | [03-cli-commands.md](03-cli-commands.md) |
| 04 | Locators & Selectors | [04-locators.md](04-locators.md) |
| 05 | Actions (click, fill, keyboard, mouse, upload, download, drag) | [05-actions.md](05-actions.md) |
| 06 | Assertions, Auto-Waiting & Timeouts | [06-assertions-and-waits.md](06-assertions-and-waits.md) |
| 07 | Browser, Context, Page, Tabs, Frames, Dialogs, Shadow DOM | [07-browser-context-page.md](07-browser-context-page.md) |
| 08 | Test Structure, Hooks, Annotations, Tags, Fixtures | [08-test-structure-and-fixtures.md](08-test-structure-and-fixtures.md) |
| 09 | Authentication & Session Reuse | [09-authentication.md](09-authentication.md) |
| 10 | Network Interception & Mocking | [10-network-mocking.md](10-network-mocking.md) |
| 11 | API Testing | [11-api-testing.md](11-api-testing.md) |
| 12 | Visual Regression Testing | [12-visual-testing.md](12-visual-testing.md) |
| 13 | Reports, Trace Viewer, Screenshots, Video, Debugging | [13-reports-and-debugging.md](13-reports-and-debugging.md) |
| 14 | Page Object Model & Framework Design | [14-pom-framework-design.md](14-pom-framework-design.md) |
| 15 | Parallelism, Retries, Sharding, Flaky Tests | [15-parallel-retries-sharding.md](15-parallel-retries-sharding.md) |
| 16 | Emulation — Devices, Geolocation, Timezone, Clock | [16-emulation.md](16-emulation.md) |
| 17 | CI/CD — GitHub Actions, Jenkins, Azure, Docker | [17-ci-cd.md](17-ci-cd.md) |
| 18 | Interview Questions (Basic → Advanced + Scenarios) | [18-interview-questions.md](18-interview-questions.md) |
| 19 | Cheat Sheet (one-page revision) | [19-cheat-sheet.md](19-cheat-sheet.md) |

---

## 🧭 Suggested Study Order

```text
Beginner      → 01 → 02 → 03 → 04 → 05 → 06
Intermediate  → 07 → 08 → 09 → 13 → 14
Advanced      → 10 → 11 → 12 → 15 → 16 → 17
Interview day → 18 → 19
```

## ✅ Conventions Used

- All examples use **`@playwright/test` with TypeScript**.
- `>` blockquotes = **what to say in an interview**.
- ⚠️ = common mistake / gotcha.
- 💡 = best practice.
