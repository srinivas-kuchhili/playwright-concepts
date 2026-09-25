# 10 — Network Interception & Mocking

## 1. What is Network Interception?

**Network interception** lets a test **observe, modify, block, or fake** HTTP(S) requests made by the page. It's done with **`page.route()`** / **`context.route()`**.

### Why mock?

- Test UI **independently of backend** (backend not ready / unstable)
- Simulate **edge cases**: empty list, 500 error, timeout, slow network
- **Speed up** tests by blocking images, ads, analytics
- Make tests **deterministic** (fixed data)

| Term | Meaning |
| ---- | ------- |
| **Intercept** | Catch a request before it goes out |
| **Mock / Stub** | Return a fake response without hitting the server |
| **Modify** | Change request (headers/body) or response (patch JSON) |
| **Abort** | Block the request |
| **Spy** | Just observe (`waitForRequest` / `page.on('request')`) |

---

## 2. `route` Methods

| Method | Purpose |
| ------ | ------- |
| `route.fulfill({...})` | Return a mock response |
| `route.continue({...})` | Send request to server (optionally modified) |
| `route.abort()` | Block request (`'failed'`, `'timedout'`, `'accessdenied'`, …) |
| `route.fetch()` | Perform request & get real response (to patch it) |
| `route.fallback()` | Pass to next registered handler |
| `route.request()` | Access the intercepted request |

| `page.route` | `context.route` |
| ------------ | --------------- |
| Only this page | All pages in context (incl. popups) |

URL patterns: glob `'**/api/users'`, `'**/*.{png,jpg}'`, regex `/\/api\/users\/\d+/`, or predicate `url => url.pathname.startsWith('/api')`.

⚠️ Register routes **before** `page.goto()` / the action that triggers the request.

---

## 3. Mock a Response

```typescript
test('shows mocked users', async ({ page }) => {
  await page.route('**/api/users', async route => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([{ id: 1, name: 'Mock User' }]),
      // or: json: [{ id: 1, name: 'Mock User' }]
    });
  });

  await page.goto('/users');
  await expect(page.getByText('Mock User')).toBeVisible();
});
```

From a file:

```typescript
await page.route('**/api/products', route => route.fulfill({ path: 'test-data/products.json' }));
```

---

## 4. Mock Error / Empty / Slow States

```typescript
// 500 error
await page.route('**/api/orders', route =>
  route.fulfill({ status: 500, json: { message: 'Internal Server Error' } }));
await page.goto('/orders');
await expect(page.getByText('Something went wrong')).toBeVisible();

// Empty list
await page.route('**/api/orders', route => route.fulfill({ json: [] }));

// Network failure
await page.route('**/api/orders', route => route.abort('failed'));

// Slow response (loading spinner test)
await page.route('**/api/orders', async route => {
  await new Promise(r => setTimeout(r, 3000));
  await route.continue();
});
await page.goto('/orders');
await expect(page.getByTestId('spinner')).toBeVisible();
```

---

## 5. Modify a Real Response (Patch)

```typescript
await page.route('**/api/profile', async route => {
  const response = await route.fetch();          // real call
  const json = await response.json();
  json.name = 'Patched Name';
  json.isPremium = true;
  await route.fulfill({ response, json });       // keep status/headers, replace body
});
```

---

## 6. Modify a Request

```typescript
await page.route('**/api/**', async route => {
  const headers = { ...route.request().headers(), 'x-test-header': 'playwright' };
  delete headers['cookie'];
  await route.continue({ headers });
});

// Change method / body / URL
await page.route('**/api/search', route =>
  route.continue({ method: 'POST', postData: JSON.stringify({ q: 'test' }) }));
```

---

## 7. Block Resources (speed-up)

```typescript
await page.route('**/*.{png,jpg,jpeg,gif,svg,webp}', route => route.abort());

await page.route('**/*', route => {
  const type = route.request().resourceType();   // document, script, image, font, stylesheet, xhr, fetch...
  return ['image', 'font', 'media'].includes(type) ? route.abort() : route.continue();
});

await context.route(/google-analytics|doubleclick|hotjar/, route => route.abort());
```

---

## 8. Conditional Mocking (method / body)

```typescript
await page.route('**/api/users', async route => {
  const req = route.request();
  if (req.method() === 'POST') {
    const body = req.postDataJSON();
    expect(body.email).toContain('@');
    return route.fulfill({ status: 201, json: { id: 99, ...body } });
  }
  return route.fallback();       // let other handlers / network handle GET
});
```

---

## 9. Remove Routes

```typescript
await page.unroute('**/api/users');
await page.unrouteAll({ behavior: 'ignoreErrors' });
```

---

## 10. Spying — Wait for / Verify Requests

```typescript
// Verify the request payload sent by UI
const requestPromise = page.waitForRequest(req =>
  req.url().includes('/api/login') && req.method() === 'POST');
await page.getByRole('button', { name: 'Login' }).click();
const request = await requestPromise;
expect(request.postDataJSON()).toEqual({ username: 'admin', password: 'admin123' });

// Verify the response
const responsePromise = page.waitForResponse('**/api/login');
await page.getByRole('button', { name: 'Login' }).click();
const response = await responsePromise;
expect(response.status()).toBe(200);

// Log all API calls
page.on('request', r => r.url().includes('/api/') && console.log(r.method(), r.url()));
```

---

## 11. HAR Recording & Replay

**HAR** (HTTP Archive) = a file of recorded requests/responses. Record once, replay offline.

```typescript
// Record (update: true writes the HAR)
await page.routeFromHAR('hars/app.har', { url: '**/api/**', update: true });
await page.goto('/');

// Replay (serves responses from HAR; unmatched → abort or fallback)
await page.routeFromHAR('hars/app.har', { url: '**/api/**', notFound: 'fallback' });
```

```bash
npx playwright open --save-har=hars/app.har --save-har-glob="**/api/**" https://app.com
```

Record HAR for a whole context:

```typescript
const context = await browser.newContext({ recordHar: { path: 'network.har', urlFilter: '**/api/**' } });
// ... must close context to save
await context.close();
```

---

## 12. Mock WebSockets (v1.48+)

```typescript
await page.routeWebSocket('wss://example.com/ws', ws => {
  ws.onMessage(message => {
    if (message === 'request') ws.send('mocked-response');
  });
});
```

---

## 13. Offline Mode & Network Throttling

```typescript
await context.setOffline(true);
await page.reload().catch(() => {});
await expect(page.getByText('You are offline')).toBeVisible();
await context.setOffline(false);

// Throttle (Chromium only, via CDP)
const cdp = await context.newCDPSession(page);
await cdp.send('Network.emulateNetworkConditions', {
  offline: false, latency: 400, downloadThroughput: 50 * 1024, uploadThroughput: 20 * 1024,
});
```

---

## 14. Reusable Mock Fixture

```typescript
export const test = base.extend<{ mockApi: (url: string, data: unknown, status?: number) => Promise<void> }>({
  mockApi: async ({ page }, use) => {
    await use(async (url, data, status = 200) => {
      await page.route(url, route => route.fulfill({ status, json: data }));
    });
  },
});

test('empty cart', async ({ page, mockApi }) => {
  await mockApi('**/api/cart', { items: [] });
  await page.goto('/cart');
  await expect(page.getByText('Your cart is empty')).toBeVisible();
});
```

> **Interview one-liner:** I use `page.route()` to intercept network calls — `fulfill` to mock responses (empty lists, 500s), `route.fetch()` + `fulfill` to patch real responses, `continue` to modify requests, and `abort` to block images/analytics. I verify payloads with `waitForRequest`, record/replay APIs with `routeFromHAR`, and always register routes before navigation.
