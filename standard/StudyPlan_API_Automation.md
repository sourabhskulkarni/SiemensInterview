# Study Plan: API Automation using Playwright

The JD requires API test automation through code for deeper integration, not just Postman.

---

## 1. Network Interception & Mocking
**Concept Used:** Service Virtualization, API Mocking, Shift-Left Testing.

**Context Story:**
Imagine testing a frontend application that relies on a third-party payment gateway or a slow backend service. Waiting for the real service makes tests slow and flaky. Furthermore, how do you test the UI's "Server Down" error page if the server is never down? Playwright allows us to intercept outgoing browser requests and mock the response, letting us test the UI in isolation.

**Snippet:**
```javascript
test('Mock API response for Server Error', async ({ page }) => {
    // Intercept the API call and fulfill with a mocked 500 error
    await page.route('**/api/payments', route => {
        route.fulfill({
            status: 500,
            contentType: 'application/json',
            body: JSON.stringify({ error: 'Internal Server Error' })
        });
    });

    await page.goto('/checkout');
    await page.locator('#pay-button').click();
    
    // Assert that the UI handles the 500 error gracefully
    await expect(page.locator('.error-banner')).toHaveText('Payment gateway is currently down.');
});
```

**How to Explain Code in Interview:**
"In this code, I use `page.route` to intercept any network call matching `**/api/payments`. Instead of letting the request hit the actual server, I immediately `fulfill` it with a 500 status code. This allows me to force the frontend into an error state reliably and instantaneously, proving that the UI's error handling works without actually bringing down a server."

**Questions that might be asked:**
- "What is the difference between `route.fulfill` and `route.continue`?"
- "Can you modify the request body before it goes to the server using Playwright?" *(Answer: Yes, using `route.continue({ postData: ... })`)*
- "Why not just test this at the API layer instead of the UI layer?" *(Answer: We test the API layer to ensure the backend works, but we mock the UI layer to ensure the frontend code reacts correctly to backend responses).*

---

## 2. API Authentication (Reusing State)
**Concept Used:** Session Management, Optimization, Setup Automation.

**Context Story:**
If you have 100 UI tests, and each one takes 10 seconds to log in through the UI, you waste 16 minutes just logging in. A senior SDET optimizes this by logging in once via an API call, capturing the authentication cookies/tokens, and injecting that state into every subsequent test.

**Snippet:**
```javascript
// global-setup.js
module.exports = async config => {
    const { request } = require('@playwright/test');
    const apiContext = await request.newContext();
    
    // Fast API login
    await apiContext.post('https://api.example.com/login', {
        data: { user: 'admin', pass: 'secret' }
    });
    
    // Save storage state (cookies/local storage) to a JSON file
    await apiContext.storageState({ path: 'state.json' });
};
```

**How to Explain Code in Interview:**
"To speed up execution, I use a Global Setup file. I use Playwright's `request` context to make a direct API POST call to the login endpoint. Because it's an API call, it takes milliseconds instead of seconds. The server responds with session cookies, which Playwright stores. I then use `storageState` to write these cookies to a `state.json` file. The test configuration is set to read this file, so every browser context starts pre-authenticated."

**Questions that might be asked:**
- "How do you handle token expiration in the middle of a test suite?"
- "How do you deal with tests that require different user roles (e.g., Admin vs User)?" *(Answer: Generate multiple state files, e.g., `adminState.json` and `userState.json`, and inject them using `test.use()`)*
- "Why use API login instead of UI login for test setup?"
