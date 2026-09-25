# 11 — API Testing

## 1. What is API Testing in Playwright?

Playwright can send HTTP requests **directly to a server without a browser** using **`APIRequestContext`**. It's used to:

- Test REST APIs (status, body, headers, schema)
- **Set up / tear down test data** fast before UI tests
- **Verify backend state** after a UI action
- Log in via API and reuse the session in UI (shared cookies)

| Ways to get an `APIRequestContext` | Shares cookies with browser? |
| ---------------------------------- | ---------------------------- |
| `request` fixture | ❌ Separate (but uses `baseURL`, `extraHTTPHeaders`, `httpCredentials` from config) |
| `page.request` / `context.request` | ✅ Shares cookies with the browser context |
| `await request.newContext({...})` (from `playwright` import) | ❌ Fully custom |

> Playwright's `APIRequestContext` lets me test REST APIs without a browser and combine API + UI in the same test — for example create data via API, verify in UI, and clean up via API.

---

## 2. Config for API Tests

```typescript
// playwright.config.ts
projects: [
  {
    name: 'api',
    testDir: './tests/api',
    use: {
      baseURL: 'https://reqres.in',
      extraHTTPHeaders: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${process.env.API_TOKEN}`,
      },
    },
  },
],
```

---

## 3. CRUD Examples

```typescript
import { test, expect } from '@playwright/test';

test.describe('Users API', () => {
  let userId: number;

  test('GET list of users', async ({ request }) => {
    const res = await request.get('/api/users', { params: { page: 2 } });

    expect(res.status()).toBe(200);
    await expect(res).toBeOK();
    expect(res.headers()['content-type']).toContain('application/json');

    const body = await res.json();
    expect(body.data.length).toBeGreaterThan(0);
    expect(body.data[0]).toHaveProperty('email');
  });

  test('POST create user', async ({ request }) => {
    const res = await request.post('/api/users', {
      data: { name: 'Swagatika', job: 'QA Engineer' },   // JSON body (auto content-type)
    });
    expect(res.status()).toBe(201);
    const body = await res.json();
    expect(body).toMatchObject({ name: 'Swagatika', job: 'QA Engineer' });
    expect(body.id).toBeDefined();
    userId = body.id;
  });

  test('PUT update user', async ({ request }) => {
    const res = await request.put('/api/users/2', { data: { name: 'Updated', job: 'Lead' } });
    expect(res.status()).toBe(200);
    expect((await res.json()).name).toBe('Updated');
  });

  test('PATCH partial update', async ({ request }) => {
    const res = await request.patch('/api/users/2', { data: { job: 'Manager' } });
    expect(res.ok()).toBeTruthy();
  });

  test('DELETE user', async ({ request }) => {
    const res = await request.delete('/api/users/2');
    expect(res.status()).toBe(204);
  });

  test('GET unknown user returns 404', async ({ request }) => {
    const res = await request.get('/api/users/23');
    expect(res.status()).toBe(404);
  });
});
```

---

## 4. Request Options

| Option | Purpose | Example |
| ------ | ------- | ------- |
| `data` | JSON / string / Buffer body | `data: { a: 1 }` |
| `form` | `application/x-www-form-urlencoded` | `form: { user: 'x' }` |
| `multipart` | File upload / multipart form | `multipart: { file: fs.createReadStream('a.png') }` |
| `params` | Query string | `params: { page: 2 }` |
| `headers` | Request headers | `headers: { 'x-api-key': 'k' }` |
| `timeout` | Request timeout | `timeout: 10_000` |
| `failOnStatusCode` | Throw on non-2xx/3xx | `failOnStatusCode: true` |
| `ignoreHTTPSErrors` | Skip TLS errors | `true` |
| `maxRedirects` | Redirect limit | `0` to not follow |
| `maxRetries` | Retry on network errors | `3` |

```typescript
// File upload
await request.post('/api/upload', {
  multipart: {
    file: { name: 'report.pdf', mimeType: 'application/pdf', buffer: fs.readFileSync('report.pdf') },
    description: 'Monthly report',
  },
});
```

## 5. Response Methods

| Method | Returns |
| ------ | ------- |
| `res.status()` | Status code |
| `res.statusText()` | Status text |
| `res.ok()` | `true` for 200–299 |
| `res.json()` | Parsed JSON |
| `res.text()` | Body as string |
| `res.body()` | Buffer |
| `res.headers()` | Headers object |
| `res.headersArray()` | Headers incl. duplicates |
| `res.url()` | Final URL |

---

## 6. Authentication for APIs

```typescript
// Token from login endpoint, reused in a custom context
import { test, expect, request as pwRequest } from '@playwright/test';

let apiContext;

test.beforeAll(async () => {
  const loginCtx = await pwRequest.newContext({ baseURL: process.env.API_URL });
  const res = await loginCtx.post('/auth/login', { data: { username: 'admin', password: 'admin' } });
  const { token } = await res.json();

  apiContext = await pwRequest.newContext({
    baseURL: process.env.API_URL,
    extraHTTPHeaders: { Authorization: `Bearer ${token}` },
  });
});

test.afterAll(async () => { await apiContext.dispose(); });

test('get profile', async () => {
  const res = await apiContext.get('/me');
  await expect(res).toBeOK();
});
```

Other options: `httpCredentials: { username, password }` (Basic auth), API key header, `storageState` for cookie sessions.

---

## 7. Schema Validation (Ajv / Zod)

```typescript
import Ajv from 'ajv';

const schema = {
  type: 'object',
  required: ['id', 'email', 'first_name'],
  properties: {
    id: { type: 'number' },
    email: { type: 'string', format: 'email' },
    first_name: { type: 'string' },
  },
};

test('user schema', async ({ request }) => {
  const res = await request.get('/api/users/2');
  const { data } = await res.json();
  const ajv = new Ajv();
  const validate = ajv.compile(schema);
  expect(validate(data), JSON.stringify(validate.errors)).toBe(true);
});
```

```typescript
// Zod alternative
import { z } from 'zod';
const User = z.object({ id: z.number(), email: z.string().email() });
User.parse(data);   // throws if invalid
```

---

## 8. API + UI Combined (Hybrid Tests)

```typescript
test('user created via API appears in UI', async ({ page, request }) => {
  // Arrange — API
  const res = await request.post('/api/products', { data: { name: 'Test Phone', price: 999 } });
  const { id } = await res.json();

  // Act + Assert — UI
  await page.goto('/products');
  await expect(page.getByText('Test Phone')).toBeVisible();

  // Cleanup — API
  await request.delete(`/api/products/${id}`);
});

test('UI action verified via API', async ({ page }) => {
  await page.goto('/profile');
  await page.getByLabel('City').fill('Bhubaneswar');
  await page.getByRole('button', { name: 'Save' }).click();

  const res = await page.request.get('/api/profile');   // shares browser cookies
  expect((await res.json()).city).toBe('Bhubaneswar');
});
```

---

## 9. API Client Fixture (Framework Style)

```typescript
// api/UserApi.ts
import { APIRequestContext, expect } from '@playwright/test';

export class UserApi {
  constructor(private request: APIRequestContext) {}

  async create(user: { name: string; job: string }) {
    const res = await this.request.post('/api/users', { data: user });
    await expect(res).toBeOK();
    return res.json();
  }

  async get(id: number) {
    return this.request.get(`/api/users/${id}`);
  }
}

// fixtures
export const test = base.extend<{ userApi: UserApi }>({
  userApi: async ({ request }, use) => use(new UserApi(request)),
});
```

---

## 10. Response Time Check

```typescript
const start = Date.now();
const res = await request.get('/api/users');
const duration = Date.now() - start;
expect(duration).toBeLessThan(1000);
```

⚠️ Playwright is not a load-testing tool — use k6 / JMeter / Artillery for performance testing.

---

## 11. GraphQL

```typescript
const res = await request.post('/graphql', {
  data: {
    query: `query GetUser($id: ID!) { user(id: $id) { id name } }`,
    variables: { id: '1' },
  },
});
const { data, errors } = await res.json();
expect(errors).toBeUndefined();
expect(data.user.name).toBe('Leanne Graham');
```

> **Interview one-liner:** I use the `request` fixture (or `page.request` to share browser cookies) for REST testing — validating status, headers, body and JSON schema with Ajv/Zod. I wrap endpoints in API client classes injected via fixtures, and use APIs in UI tests to set up and clean test data quickly.
