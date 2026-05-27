# Section 9: Performance & NFR Testing

## 28. NFR Testing Integration
**Question:** The JD mentions K6 and Sitespeed.io. How would you integrate K6 with a Playwright framework for API performance testing?

**Answer & Explanation:**
Playwright is for functional E2E testing, not load testing. However, they can work together beautifully.

**Practical Snippet & Approach:**
```javascript
// Step 1: Generate a HAR file (HTTP Archive) using Playwright
const context = await browser.newContext({ recordHar: { path: 'flow.har' }});
const page = await context.newPage();
await page.goto('/checkout');
await page.getByRole('button', { name: 'Pay' }).click();
await context.close(); // Saves the HAR file
```

**How to explain in interview:**
"Playwright is too heavy to simulate 1,000 concurrent users. Instead, I use Playwright to record a `.har` file of the user journey. I then use a tool called `har-to-k6` to automatically convert that network traffic into a K6 performance script. I run the K6 script to simulate the 1,000 users hitting the API endpoints simultaneously."

*Cross Question Answers:* Defining NFR benchmarks in CI/CD means adding a threshold block to your K6 script (e.g., `p(95) < 500` meaning 95% of requests must complete under 500ms). If the API is slower than 500ms, the GitLab CI pipeline fails, preventing degraded performance from reaching production. Sitespeed.io is used similarly but focuses on frontend rendering metrics like LCP (Largest Contentful Paint).
