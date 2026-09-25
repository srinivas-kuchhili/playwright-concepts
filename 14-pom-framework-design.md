# 14 — Page Object Model & Framework Design

## 1. What is the Page Object Model (POM)?

**POM** is a design pattern where each page (or component) of the app is represented by a **class** that holds its **locators** and **actions**. Tests call these methods instead of using raw locators.

| Without POM | With POM |
| ----------- | -------- |
| Locators duplicated across tests | Locators defined once |
| UI change → fix many tests | UI change → fix one class |
| Tests are long & low-level | Tests read like business steps |
| Hard to maintain | Reusable, maintainable |

> POM separates *what* a test does from *how* the page is manipulated — locators and actions live in page classes, tests only express business flow and assertions.

---

## 2. Base Page

```typescript
// pages/BasePage.ts
import { Page, Locator, expect } from '@playwright/test';

export abstract class BasePage {
  constructor(protected readonly page: Page) {}

  async navigate(path: string) {
    await this.page.goto(path);
  }

  async getTitle() {
    return this.page.title();
  }

  async expectToast(message: string) {
    await expect(this.page.getByRole('alert')).toHaveText(message);
  }
}
```

## 3. Page Class

```typescript
// pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  readonly username: Locator;
  readonly password: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    super(page);
    this.username = page.getByLabel('Username');
    this.password = page.getByLabel('Password');
    this.loginButton = page.getByRole('button', { name: 'Login' });
    this.errorMessage = page.getByTestId('login-error');
  }

  async goto() {
    await this.navigate('/login');
  }

  async login(user: string, pass: string) {
    await this.username.fill(user);
    await this.password.fill(pass);
    await this.loginButton.click();
  }

  async expectError(text: string) {
    await expect(this.errorMessage).toHaveText(text);
  }
}
```

## 4. Component Object (reusable UI parts)

```typescript
// components/Header.ts
export class Header {
  readonly root: Locator;
  readonly cartBadge: Locator;

  constructor(private readonly page: Page) {
    this.root = page.getByRole('banner');
    this.cartBadge = this.root.getByTestId('cart-count');
  }

  async search(term: string) {
    await this.root.getByRole('searchbox').fill(term);
    await this.root.getByRole('searchbox').press('Enter');
  }
}

// pages/HomePage.ts
export class HomePage extends BasePage {
  readonly header: Header;

  constructor(page: Page) {
    super(page);
    this.header = new Header(page);
  }
}
```

## 5. Fixtures to Inject Pages

```typescript
// fixtures/pages.fixture.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { HomePage } from '../pages/HomePage';

type Pages = { loginPage: LoginPage; homePage: HomePage };

export const test = base.extend<Pages>({
  loginPage: async ({ page }, use) => use(new LoginPage(page)),
  homePage: async ({ page }, use) => use(new HomePage(page)),
});
export { expect } from '@playwright/test';
```

## 6. Test

```typescript
import { test, expect } from '../fixtures/pages.fixture';

test.describe('Login', { tag: '@smoke' }, () => {
  test('valid credentials', async ({ loginPage, homePage }) => {
    await loginPage.goto();
    await loginPage.login(process.env.APP_USER!, process.env.APP_PASS!);
    await expect(homePage.header.root).toBeVisible();
  });

  test('invalid credentials', async ({ loginPage }) => {
    await loginPage.goto();
    await loginPage.login('wrong', 'wrong');
    await loginPage.expectError('Invalid username or password');
  });
});
```

### POM Best Practices
- 💡 Locators as `readonly` properties, defined in constructor.
- 💡 Methods = **user intentions** (`login`, `addToCart`), not `clickButton1`.
- 💡 Keep **assertions mostly in tests**; small reusable `expectX()` helpers are fine.
- 💡 Return the next page object for fluent flows if helpful (`return new DashboardPage(this.page)`).
- 💡 Don't put `waitForTimeout` in page objects.
- 💡 Use component objects for header, footer, modals, tables.

---

## 7. Recommended Framework Structure

```text
playwright-framework/
├── .github/workflows/playwright.yml
├── config/
│   ├── .env.qa
│   └── .env.staging
├── src/
│   ├── pages/            # Page objects
│   ├── components/       # Header, Footer, Modal
│   ├── api/              # API clients (UserApi, OrderApi)
│   ├── fixtures/         # test.extend (pages, api, data, auth)
│   ├── utils/            # helpers (date, random, logger)
│   └── data/             # static test data (JSON/CSV)
├── tests/
│   ├── auth.setup.ts
│   ├── ui/
│   ├── api/
│   └── visual/
├── playwright.config.ts
├── tsconfig.json
├── package.json
├── .eslintrc / eslint.config.mjs
├── .prettierrc
└── README.md
```

### Framework layers

```text
Tests (specs)  →  Fixtures  →  Page Objects / API clients  →  Playwright API
                      ↑
               Test data, utils, config (env)
```

### Framework features checklist (talk about these in interviews)

| Feature | Implementation |
| ------- | -------------- |
| Multi-env | `.env.*` + `dotenv`, `baseURL` from env |
| Cross-browser | Projects |
| Auth reuse | Setup project + `storageState` |
| Test data | JSON/CSV + Faker, API-based data creation |
| Reporting | HTML + Allure + JUnit |
| Logging | Custom logger / `test.step` |
| Retries & flake detection | `retries`, trace on retry |
| Parallelism | `fullyParallel`, workers, sharding |
| Tagging | `@smoke`, `@regression` with `--grep` |
| CI/CD | GitHub Actions / Jenkins with artifacts |
| Code quality | ESLint (`eslint-plugin-playwright`), Prettier, Husky |
| Secrets | CI secrets, never commit credentials |

---

## 8. Test Data with Faker

```bash
npm i -D @faker-js/faker
```

```typescript
import { faker } from '@faker-js/faker';

export const createUser = () => ({
  firstName: faker.person.firstName(),
  lastName: faker.person.lastName(),
  email: faker.internet.email(),
  phone: faker.phone.number(),
  address: faker.location.streetAddress(),
});
```

---

## 9. ESLint for Playwright

```bash
npm i -D eslint eslint-plugin-playwright typescript-eslint
```

Useful rules: `playwright/no-wait-for-timeout`, `playwright/missing-playwright-await`, `playwright/no-focused-test`, `playwright/prefer-web-first-assertions`, `@typescript-eslint/no-floating-promises` (catches missing `await`).

---

## 10. Writing Good Tests — Best Practices

1. **Test user-visible behaviour**, not implementation details.
2. **Isolate tests** — each test has own data & context; no dependency on order.
3. **Use user-facing locators** (`getByRole`), avoid XPath/CSS chains.
4. **Use web-first assertions**; never `waitForTimeout`.
5. **Don't test third-party sites** — mock them.
6. **Use API for setup/teardown** — UI only for what you're testing.
7. **Keep tests small & focused** — one behaviour per test.
8. **Use `test.step`** for readable reports.
9. **Run on CI** on every PR, with traces on failure.
10. **Lint** to catch missing `await`.

> **Interview one-liner:** My framework uses POM with component objects, injected via custom fixtures; API clients for data setup; setup projects with `storageState` for auth; env-driven config; tags for suites; HTML/Allure reporting; and GitHub Actions with sharding and trace artifacts. ESLint with the Playwright plugin enforces awaits and web-first assertions.
