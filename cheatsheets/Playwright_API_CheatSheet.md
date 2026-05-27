# Playwright API Automation Cheat Sheet

A quick reference for backend testing, API mocking, and network handling in Playwright.

---

## 1. Direct API Calls (`request` context)
Used to validate endpoints directly or bypass UI setup for faster execution.

```javascript
const { test, expect } = require('@playwright/test');

test('API Validation', async ({ request }) => {
  // GET Call
  const getRes = await request.get('/api/users/1');
  expect(getRes.ok()).toBeTruthy();
  
  // POST Call with Headers and Body
  const postRes = await request.post('/api/users', {
    headers: { 'Authorization': 'Bearer token123' },
    data: { name: 'Sourabh', role: 'SDET' }
  });
  
  // Validate Response Body
  const body = await postRes.json();
  expect(body.name).toBe('Sourabh');
  expect(postRes.status()).toBe(201);
});
```

---

## 2. Network Interception & Mocking (`page.route`)
Block or modify browser network requests (Shift-Left testing).

```javascript
// 1. Mocking (Fulfill) - Return a fake response
await page.route('**/api/users', route => {
  route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({ name: 'Mock User' })
  });
});

// 2. Modifying (Continue) - Add a header to an outgoing request
await page.route('**/api/secure', route => {
  const headers = { ...route.request().headers(), 'X-Custom-Auth': '123' };
  route.continue({ headers });
});
```

---

## 3. Waiting for Network (Prevent UI Flakiness)
Explicitly wait for the backend to respond before moving to the next UI action.

```javascript
// Promise.all ensures we start listening BEFORE the click happens
const [response] = await Promise.all([
  page.waitForResponse(res => res.url().includes('/api/checkout') && res.status() === 200),
  page.getByRole('button', { name: 'Pay Now' }).click() 
]);

// Extract data from the network call
const responseBody = await response.json();
console.log('Order ID:', responseBody.orderId);
```

---

## 4. Authentication State Rehydration
Log in once via API, save cookies/tokens, and inject them into every test context automatically.

```javascript
// global-setup.js
const { request } = require('@playwright/test');

module.exports = async () => {
  const context = await request.newContext();
  await context.post('https://api.example.com/login', {
    data: { user: 'admin', pass: 'secret' }
  });
  
  // Save cookies to a file
  await context.storageState({ path: 'state.json' });
};

// playwright.config.ts
export default defineConfig({
  use: { storageState: 'state.json' } // All tests start logged in
});
```
