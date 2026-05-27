# Playwright Core Cheat Sheet

A quick reference for UI automation concepts in Playwright.

---

## 1. Architecture & Setup
**Hierarchy:** `Browser` (heavy process) > `Context` (isolated incognito session) > `Page` (browser tab).

```javascript
// Setup
const context = await browser.newContext();
const page = await context.newPage();
await page.goto('https://example.com');
```

**`playwright.config.ts` basics:**
```typescript
export default defineConfig({
  retries: 2,          // Retry failing tests on CI
  workers: 4,          // Concurrent workers
  use: {
    trace: 'retain-on-failure', 
    screenshot: 'only-on-failure',
  }
});
```

---

## 2. Locators & Actions
Prefer user-facing locators (`getByRole`, `getByText`) over CSS/XPath. Playwright auto-waits before acting.

```javascript
// Interactions
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByPlaceholder('Enter text').fill('hello');
await page.locator('#item').dblclick();
await page.locator('.menu').hover();
await page.locator('#source').dragTo(page.locator('#target'));

// Keyboard & Mouse
await page.keyboard.press('Enter');
await page.mouse.wheel(0, 100);
```

---

## 3. Assertions (Auto-Retrying)
Web-first assertions automatically wait and retry until the condition passes or times out.

```javascript
await expect(page.locator('.success')).toBeVisible();
await expect(page.locator('#email')).toHaveValue('test@test.com');
await expect(page.locator('button')).toBeDisabled();
await expect(page.locator('.error')).toContainText('Failed');
```

---

## 4. Alerts, Dialogs & iFrames
Alerts are auto-dismissed by default. You must attach a listener to accept them.

```javascript
// Alerts
page.on('dialog', async dialog => {
  await dialog.accept(); // Accept the JS alert
});
await page.getByText('Delete').click(); // Triggers alert

// iFrames
const frame = page.frameLocator('#payment-iframe');
await frame.getByRole('button', { name: 'Pay' }).click();
```

---

## 5. Custom Fixtures (Dependency Injection)
Provide initialized Page Objects directly into tests.

```javascript
import { test as base } from '@playwright/test';
import { LoginPage } from './LoginPage';

export const test = base.extend({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page)); // Inject into test
  }
});

// Usage
test('Login', async ({ loginPage }) => {
  await loginPage.login('user', 'pass');
});
```

---

## 6. Multi-Tab Handling
Wait for a new tab to open before interacting.

```javascript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open PDF').click()
]);
await newPage.waitForLoadState();
```

---

## 7. CLI & Debugging Commands
```bash
npx playwright test --ui               # Interactive UI mode
npx playwright test --debug            # Pauses execution for inspector
npx playwright show-trace trace.zip    # Time-travel debugging
npx playwright test --shard=1/4        # Parallel sharding for CI
```
