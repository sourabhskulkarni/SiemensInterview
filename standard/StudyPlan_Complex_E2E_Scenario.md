# Study Plan: Complex E2E Scenario (Master Class)

Interviewers for SDET roles often ask you to architect a complex, real-world end-to-end (E2E) scenario. This section covers a complete flow: **Login (Auth Saving) > Search > Add to Cart (Network validation) > Payment > Update > Cancel**. It demonstrates UI testing mixed with direct API calls and network monitoring.

---

## E2E Scenario: E-Commerce Complete Lifecycle
**Concept Used:** Hybrid Test Automation (UI + API), Network Event Listeners, State Rehydration.

**Context Story:**
In a real enterprise application, testing an entire checkout flow via the UI can take minutes. Furthermore, UI actions (like clicking "Add to Cart") trigger background API calls. If the UI proceeds before the API finishes, the test will be flaky. To solve this, a Senior SDET will:
1. Bypass UI login by injecting an auth token.
2. Use Playwright's network listening (`waitForResponse`) to wait for the backend to acknowledge actions before moving to the next UI step.
3. Use direct API calls to perform teardown actions (like cancelling the order) to save execution time.

**Snippet:**
```javascript
const { test, expect, request } = require('@playwright/test');

// Pre-condition: We assume storageState (auth token) is already injected via global setup.
test.use({ storageState: 'authState.json' });

test('Complete Order Lifecycle: Search, Cart, Pay, Cancel', async ({ page }) => {
    
    // 1. SEARCH PRODUCT (UI)
    await page.goto('/products');
    await page.getByPlaceholder('Search').fill('Laptop');
    await page.getByRole('button', { name: 'Search' }).click();
    
    // 2. ADD TO CART & VALIDATE NETWORK TAB (UI + Network Listening)
    // We click "Add to Cart", but we MUST wait for the backend API to return a 200 OK
    // before we try to go to the checkout page. This prevents flakiness.
    const [cartResponse] = await Promise.all([
        page.waitForResponse(response => response.url().includes('/api/cart') && response.status() === 200),
        page.getByRole('button', { name: 'Add to Cart' }).first().click()
    ]);
    
    // 3. PAYMENT & ORDER PLACED (UI)
    await page.goto('/checkout');
    await page.getByRole('button', { name: 'Pay Now' }).click();
    
    // Listen for the order confirmation network request to extract the Order ID
    const orderResponse = await page.waitForResponse('**/api/orders');
    const orderData = await orderResponse.json();
    const orderId = orderData.orderId;
    
    await expect(page.locator('.success-message')).toContainText('Order Placed Successfully');

    // 4. UPDATE ORDER (API)
    // Instead of clicking through UI to update, we use the API context directly for speed
    const apiContext = page.request;
    const updateRes = await apiContext.put(`/api/orders/${orderId}`, {
        data: { shippingSpeed: 'Express' }
    });
    expect(updateRes.ok()).toBeTruthy();

    // Validate UI reflects the API update
    await page.reload();
    await expect(page.locator('#shipping-speed')).toHaveText('Express');

    // 5. CANCEL ORDER (API - Teardown)
    // Clean up test data directly via API
    const cancelRes = await apiContext.delete(`/api/orders/${orderId}`);
    expect(cancelRes.status()).toBe(204); // 204 No Content
});
```

**How to Explain Code in Interview:**
"If asked to script an end-to-end checkout flow, I wouldn't just write a sequence of UI clicks. That leads to flaky tests. 
First, I bypass the UI login by using a pre-saved `storageState`, saving valuable execution time. 
During the 'Add to Cart' step, I don't use arbitrary `waitForTimeout` sleeps. Instead, I use `Promise.all` with `page.waitForResponse`. This tells Playwright to click the button and wait exactly until the `/api/cart` network call responds with a 200 status. It explicitly monitors the network tab, ensuring the backend state matches the UI state.
Finally, for updating and cancelling the order, I use Playwright's `page.request` API context. Rather than navigating to the 'Order History' page and clicking 'Cancel'—which adds unnecessary UI layers and execution time—I hit the API endpoint directly. I then reload the page to ensure the UI reflects the cancelled state."

**Questions that might be asked:**
- "Why did you use `Promise.all` when clicking the Add to Cart button?" *(Answer: Because the network request is triggered instantly when the button is clicked. If you await the click, and then await the response, the response might have already happened in those milliseconds between the two commands, causing the test to hang indefinitely. `Promise.all` starts listening and clicking simultaneously.)*
- "How do you extract a Bearer token from the network response and pass it to subsequent API calls?" *(Answer: In my API tests, I can await `response.json()`, extract the `token` variable, and then pass it in the `headers: { 'Authorization': 'Bearer ' + token }` of my `apiContext` calls.)*
- "What if the third-party payment gateway takes 30 seconds to respond? How do you handle that in the test?" *(Answer: I would use `page.route` to mock the payment gateway response completely, so my UI test isn't dependent on an external service's uptime or speed.)*
