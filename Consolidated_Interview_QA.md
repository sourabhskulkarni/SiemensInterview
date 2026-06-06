# Maxxton / SDET Lead Interview — Consolidated Q&A (Simple & Real-Time Examples)

> **Purpose:** This revision guide covers the Maxxton Lead Quality Engineer / Playwright Consultant interview (1 hour, panelists: QA Manager actively working in AI + QA Automation Specialist in Playwright). All answers use **simple language and real-time examples** — exactly how a senior engineer explains on a whiteboard.

---

## 🎯 60-Minute Interview Strategy (Maxxton Panel)

| Phase | Time | Focus | Panelist |
|---|---|---|---|
| **1: Intro** | 5 min | "Tell me about yourself" + leadership pitch | Both |
| **2: Playwright Architecture** | 10 min | Deep architecture, execution flow diagram, fixtures | Playwright Specialist |
| **3: JS/TS Coding** | 15 min | Arrays, Promises, Async/Await, live coding | Playwright Specialist |
| **4: APIs & Reporting** | 10 min | API testing, report attachment, hybrid testing | Both |
| **5: CI/CD & Performance** | 10 min | Parallelisation, storage state, Docker, K6 | QA Manager |
| **6: Strategy & Domain** | 5 min | Test pyramid, hospitality domain, AI in QA, mentoring | QA Manager |
| **7: Closing** | 5 min | Your questions for them | Both |

---

## PHASE 1: INTRODUCTION

### Q1. "Tell me about yourself"
> "I have 9.6 years of experience in QA. Right now, I lead a team of 6 at Infosys.
> 
> My main job is building testing frameworks from scratch using **Playwright** and **JavaScript**. I don't just do basic UI testing—I heavily focus on API testing. I write code to test APIs instead of just using Postman because it's much faster and integrates perfectly into our CI/CD pipelines in GitLab.
> 
> Recently, I built a cool internal tool: an AI agent that automatically reads our failed test logs and fixes broken UI locators for us. It saved my team 40% of the time we used to spend fixing broken tests."
> 
> > 🔄 **Cross-Question:** "Why did you choose Playwright over older tools like Selenium?"
> >
> > **Answer:** "Selenium is like a puppet master pulling strings from far away—it has to send a network request for every single click, which is slow. Playwright uses a WebSocket, which is like being on a live phone call directly with the browser. If I need to test a scenario where an Admin approves a User's request, Playwright lets me open two completely separate, incognito-like tabs in the exact same test, which Selenium struggles with."

---

## PHASE 2: PLAYWRIGHT UI ARCHITECTURE

### Q2. "Explain the Playwright architecture and its execution flow in detail."

> **IMPORTANT (Siemens asked this verbatim — Maxxton will too. Explain with the diagram below.)**

**Architecture Diagram:**

```
┌─────────────────────────────────────────────────────────────────┐
│                     YOUR TEST SCRIPT (Node.js)                  │
│   test('Book a room', async ({ page }) => { ... })              │
└───────────────────────────┬─────────────────────────────────────┘
                            │  1. Playwright Client API call
                            │     (e.g., page.click('#book-btn'))
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│               PLAYWRIGHT NODE.JS LIBRARY (Client)              │
│  • Serialises the command into a JSON-RPC message               │
│  • Sends it over a persistent WebSocket connection              │
└───────────────────────────┬─────────────────────────────────────┘
                            │  2. WebSocket (JSON-RPC over WS)
                            │     ws://127.0.0.1:<port>/<browserId>
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│           PLAYWRIGHT BROWSER SERVER (per-browser process)       │
│  • Chromium / Firefox / WebKit spawned as a child process       │
│  • Speaks CDP (Chrome DevTools Protocol) or Firefox Remote      │
│  • Acts as a bridge: WS message ➜ CDP command ➜ browser action │
└───────────────────────────┬─────────────────────────────────────┘
                            │  3. CDP / Firefox Remote Protocol
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      BROWSER (Chromium)                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Browser     │  │  Browser     │  │  Browser             │  │
│  │  Context 1   │  │  Context 2   │  │  Context 3           │  │
│  │  (User A)    │  │  (User B)    │  │  (Admin)             │  │
│  │  ┌────────┐  │  │  ┌────────┐  │  │  ┌────────────────┐  │  │
│  │  │ Page 1 │  │  │  │ Page 1 │  │  │  │ Page 1         │  │  │
│  │  │(tab)   │  │  │  │(tab)   │  │  │  │(tab)           │  │  │
│  │  └────────┘  │  │  └────────┘  │  │  └────────────────┘  │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                            │  4. Browser fires back DOM events,
                            │     network responses, console logs
                            ▼  (same WS — full-duplex)
                     YOUR TEST RECEIVES RESPONSE
```

**Execution Flow — Step by Step:**

> **Real-World Example:** Testing a property reservation workflow on Maxxton's hospitality platform:
>
> **Step 1 — Playwright launches a Browser process** (e.g., Chromium) as a child process of Node.js. One browser process can hold multiple isolated Browser Contexts, each acting like a completely fresh incognito window with its own cookies, local storage, and service workers.
>
> **Step 2 — Browser Context is created.** Think of it as a tenant in a building. A context for 'Front Desk Agent', another for 'Revenue Manager', another for 'Admin'. They share the same browser process but are 100% isolated from each other.
>
> **Step 3 — Page (Tab) is opened inside a Context.** `const page = await context.newPage()`. Page represents one browser tab. Multiple pages in the same context share cookies (like you browsing two tabs while still logged in).
>
> **Step 4 — Test calls `page.click('#reserve-btn')`**. Playwright serialises this into a JSON-RPC command `{ method: 'DOM.querySelector', params: {...} }` and sends it over the already-open WebSocket. **No new HTTP connection is opened** — this is why Playwright is fast; the WebSocket is always live.
>
> **Step 5 — CDP command hits the browser.** The browser engine executes the action in the renderer process — it finds the DOM node, fires a trusted click event. The browser's response travels back on the same WebSocket.
>
> **Step 6 — Auto-waiting.** Before any action, Playwright runs a series of internal **actionability checks**: Is the element attached to the DOM? Is it visible? Is it stable (not animating)? Is it enabled? Is it not covered by another element? Only when all checks pass does the actual click fire. This is why Playwright rarely needs explicit waits.
>
> **Step 7 — Events come back.** After the click, the browser fires network requests, DOM mutations, navigation events — all streamed back over the same WebSocket. Playwright resolves the awaited promise only when the expected state is reached (e.g., navigation complete, new URL matches pattern)."

> > 🔄 **Cross-Question (Maxxton Specialist will ask):** "What is the difference between BrowserContext and a separate Browser instance?"
> >
> > **Answer:** "A separate browser instance means launching a second Chromium process — expensive, slow, high memory. A BrowserContext is lightweight, isolated, and shares the same browser process. In Playwright, 1 browser process + multiple contexts is the standard pattern for multi-user testing. For example, testing a hotel Front Desk login and a Revenue Manager login simultaneously: I create 2 contexts in the same test, navigate both to the app, and interact with them in parallel using `Promise.all`." 

### Q3. "How do you avoid writing duplicate code in your framework, and what is a fixture?"
> "I keep it simple using Page Object Models (POM) and Playwright **Fixtures**. 
> 
> **Real-World Example:** In our project, login is handled via a complex SSO. To handle this, I created a custom async fixture called `loginUsingExistingEdgeProfile()`. Think of a fixture as a setup butler. Now, instead of writing login code in every test, my test simply calls this fixture. The fixture automatically launches the browser using a pre-authenticated local Edge profile, inherits the cookies, and drops the test directly onto the Home Page. It keeps the actual test code incredibly short and clean."

### Q4. "How do you handle dynamic UIs where IDs constantly change?"
> "Never trust IDs in modern apps like React. A 'Save' button might have an ID of `btn-123` today and `btn-999` tomorrow.
> 
> **Real-World Example:** Instead of looking at the messy HTML code, I use locators based on what the user actually sees:
> 1. **Roles:** `page.getByRole('button', { name: 'Save' })`
> 2. **Test IDs:** `page.getByTestId('save-button')`
> 
> If a button is hidden behind a loading spinner, I don't add a hardcoded 5-second wait. Playwright automatically checks if the button is visible, stable, and not blocked by another element before it clicks."

### Q5. "How does your self-healing framework work?"
> "Let's say a developer changes a button's class, and my test breaks overnight.
> 
> **Real-World Example:** My AI script reads the failure report. It looks at the HTML of the page, finds a button that still says 'Submit' or looks functionally identical, and figures out the new locator. It's basically like having a junior tester figure out why the test broke and handing me the exact line of code to fix it. I just review it and hit approve."

### Q5-B. "How do you apply OOPs concepts in your Playwright Framework?"
> **This is a very common senior-level question. Don't give textbook definitions (like "dogs and animals"); give framework examples.**
> 
> "I use the four pillars of OOP every day in my framework architecture:
> 
> 1. **Encapsulation (Hiding data):** We use the Page Object Model (POM). In my `LoginPage` class, the locators (like `readonly usernameInput`) are kept private. The test script can't access them directly. The test must call a public method like `login(user, pass)`. The test doesn't know *how* the button is clicked, it only cares about the result. This protects locators from being messed up by the test.
> 
> 2. **Inheritance (Reusability):** I have a `BasePage` class that contains standard utility methods like `waitForPageLoad()`, `verifyUrl()`, or a robust `clickWithRetry()`. Every other page (e.g., `class DashboardPage extends BasePage`) inherits these methods so I never duplicate code.
> 
> 3. **Polymorphism (Many forms / Overriding):** I have a `navigateTo()` method in `BasePage`. However, the `AdminDashboardPage` overrides this method. Why? Because going to the admin page requires injecting a special Admin Token in the local storage before navigating. It's the same method name, but a different, specialized behavior.
> 
> 4. **Abstraction (Hiding complexity):** I use Playwright **Fixtures** to achieve this. In the test script, I just ask for `({ authContext })`. All the complex, messy logic of making an API call, getting a JWT token, writing it to `storageState.json`, and injecting it into the browser is completely hidden from the test writer. The test just gets a ready-to-use logged-in page."

---

## PHASE 3: CORE JAVASCRIPT PROGRAMMING

### Q6. "Explain `var`, `let`, and `const`. What is a closure?"
> "- **`var`** leaks everywhere. Never use it.
> - **`let`** is for variables that change (like a loop counter).
> - **`const`** is for values that shouldn't change.
> 
> **Closure Example:** In automation, if I write a custom test step logger, I wrap it in a closure. The closure encapsulates a private `stepCount` variable inside its lexical scope. Every time I call the logger in my test, it remembers the previous `stepCount` (Step 1, Step 2...) without me having to expose `stepCount` as a global variable. This is critical because global variables would get overwritten and corrupted when running tests in parallel workers."

### Q7. "What is the difference between `type`, `interface`, and `abstract class` in TypeScript?"
> "**Real-World Example:**
> - **`type`** is for strict constraints. E.g., The environment configuration parameter can ONLY be 'QA', 'STAGING', or 'PROD'. You can't pass anything else.
> - **`interface`** defines the shape of a JSON payload. If the API returns a User Profile, the interface guarantees it has a `username` string and `id` number, but it contains no actual execution logic.
> - **`abstract class`** is a base blueprint. A `BasePage` class has real working methods (like a robust `clickElement()` with built-in retry logic) that child pages (like `LoginPage`) inherit, but you cannot instantiate the `BasePage` directly on its own."

### Q8. "How does the JavaScript Event Loop work? (Micro vs Macro tasks)"
> "JavaScript is single-threaded and non-blocking. It achieves concurrency using the Event Loop.
> 
> **Real-World Example:** Imagine testing a complex web form with a checkbox, file upload, dropdown, textbox, and a calendar. The 'Submit' button is disabled until all mandatory fields are filled.
> 
> 1. **Synchronous (Main Thread):** You type in the textbox. The script instantly runs a synchronous check to see if all fields are filled, instantly enabling the Submit button.
> 2. **Macrotask (`setTimeout`):** The form has a 500ms debounce timer running in the background to auto-save a draft of your inputs.
> 3. **Microtask (Promises):** You upload a file. The script sends an async API request to the backend to validate the file. When the API responds, the callback is a Microtask.
> 
> If the API file validation finishes at the exact same millisecond the 500ms auto-save timer fires, the Event Loop will always process the **Microtask** (the API Promise) first. It will completely finish updating the UI for the file upload before it even looks at the Macrotask (auto-save timer)."

### Q9. "Coding Challenge: Write an async retry function in JavaScript"
> "If a database takes a few seconds to update, I need a function that checks the UI 3 times, waiting 1 second between each try. I use a recursive function with a delay."

```javascript
// Simple async retry
async function retry(action, retries = 3) {
    try {
        await action(); // Try the action
    } catch (error) {
        if (retries === 0) throw error; // If out of retries, fail
        console.log(`Failed. Retrying... (${retries} left)`);
        await new Promise(resolve => setTimeout(resolve, 1000)); // Wait 1 second
        await retry(action, retries - 1); // Try again
    }
}
```
> > 🔄 **Cross-Question:** "Why not just put this retry around everything?"
> > **Answer:** "Because Playwright already has built-in retries for UI actions! Wrapping Playwright clicks in a custom retry loop just adds unnecessary sleep delays. I only use this custom retry for flaky backend APIs or database checks."

### Q10. "Coding Challenge: Parse a JSON response using map, filter, reduce"
> "**Real-World Example:** Imagine you get a JSON list of users, and you need to find out how much money the 'Engineering' department spent in total.
> 1. **`filter()`**: Keep only the users in Engineering.
> 2. **`map()`**: Extract their order totals.
> 3. **`reduce()`**: Add all those totals together like a calculator."

```javascript
const users = [
    { name: "Alice", dept: "Engineering", spend: 150 },
    { name: "Bob", dept: "HR", spend: 120 },
    { name: "Charlie", dept: "Engineering", spend: 500 }
];

const totalEngineeringSpend = users
    .filter(user => user.dept === "Engineering") // Gets Alice & Charlie
    .map(user => user.spend) // Gets [150, 500]
    .reduce((total, amount) => total + amount, 0); // Adds them: 650

console.log(totalEngineeringSpend); // 650
```

### Q11. "Explain `async/await`, `Promise.all`, and `Promise.allSettled`"
> "`async` and `await` are just syntactic sugar over Promises, allowing us to write asynchronous code that reads like synchronous code. 
> 
> **Real-World Example:**
> - **`Promise.all`**: Imagine a registration form. I use `await Promise.all()` to simultaneously type into the username, email, and password fields. It waits until ALL fields are successfully filled concurrently before it enables the 'Submit' button. If even one field fails to fill, the entire block throws an error instantly (all-or-nothing).
> - **`Promise.allSettled`**: Imagine a test teardown phase (`afterAll`). I use `await Promise.allSettled()` to simultaneously clear the browser cache, wipe local storage, and delete cookies. It waits for all three teardown tasks to finish, regardless of whether they passed or failed. This ensures the environment is always reset for the next test without the teardown script crashing halfway through."

### Q12. "What is the difference between a Deep Copy and a Shallow Copy?"
> "**Real-World Example:** If I have a JSON template for a 'Test User', and I make a shallow copy for Test A, and another shallow copy for Test B. If Test A changes the user's City, Test B's user also gets changed! Why? Because they share the same memory reference.
> 
> To fix this, I use `structuredClone()`. This creates a completely independent Deep Copy, so Test A and Test B don't interfere with each other."

---

## PHASE 4: MODERN API AUTOMATION

### Q13. "Why do you write API tests in code instead of using Postman?"
> "**Real-World Example:** Postman is an isolated app. If I use Postman, I can't easily share a login token with my UI tests.
> 
> By writing API tests in Playwright code:
> 1. Everything lives in the same Git repo.
> 2. I can use an API to instantly create a test user, grab the auth token, and inject it directly into the browser. Now my UI test skips the login screen entirely and runs 10x faster."

### Q14. "How do you handle API mocking in Playwright?"
> "**Real-World Example:** Say the payment gateway API (like Stripe) is down in the QA environment. My UI tests will fail even though the UI is fine. 
> 
> Instead, I use `page.route()`. I tell Playwright: 'If the browser tries to call the payment API, block the real network request, and just return a fake { status: 200 } JSON'. This lets me test my UI without depending on third-party servers."

```javascript
// Mocking the payment API
await page.route('**/api/payment', async route => {
    await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({ status: 'success', id: 'MOCK_123' })
    });
});
```

### Q15. "How do you chain API requests together?"
> "**Real-World Example:** I want to update a user profile. First, I send a POST request to login and get a token. Then, I pass that exact token into the Headers of a PUT request to update the profile. It's just standard async/await code."

```typescript
// 1. Get Token
const loginRes = await request.post('/api/login', { data: credentials });
const token = (await loginRes.json()).access_token;

// 2. Use Token to Update Profile
const updateRes = await request.put('/api/profile', {
    headers: { 'Authorization': `Bearer ${token}` },
    data: { name: "New Name" }
});
```

### Q16. "How do you handle Authentication State so you don't log in 100 times?"
> "**Real-World Example:** Logging into a website via the UI takes 5 seconds. If I have 100 tests, that's 500 seconds wasted just typing passwords.
> 
> In Playwright, I log in exactly once. I save the browser cookies to a local file (`auth_state.json`). Then, for the other 99 tests, I tell Playwright to use that file (`storageState`). Every test opens the browser already logged in."

```typescript
// playwright.config.ts — inject saved auth state globally
export default defineConfig({
    use: {
        storageState: 'playwright/.auth/user.json' // All tests start logged in
    }
});

// global.setup.ts — runs once to save cookies
await page.goto('/login');
await page.fill('#username', 'testuser');
await page.fill('#password', 'secret');
await page.click('#submit');
await page.context().storageState({ path: 'playwright/.auth/user.json' });
```

### Q17. "How do you validate massive JSON payloads? (JSON Schema Validation)"
> "**Real-World Example:** Imagine an API returns a user profile with 50 fields. Writing 50 `expect` statements is a nightmare.
> 
> Instead, I use a tool called `Ajv`. I define a 'Schema' (a blueprint that says 'name must be a string, age must be a number'). Then I compare the API response to the blueprint in one single line of code. If a developer accidentally changes a field type, my test catches it instantly."

```javascript
import Ajv from 'ajv';
const ajv = new Ajv();

const schema = {
    type: 'object',
    properties: {
        id:   { type: 'number' },
        name: { type: 'string' },
        active: { type: 'boolean' }
    },
    required: ['id', 'name', 'active']
};

const response = await request.get('/api/user/1');
const body = await response.json();

const validate = ajv.compile(schema);
expect(validate(body)).toBe(true); // One line catches all type mismatches
```

### Q18. "How do you handle complex SSO (Single Sign-On) authentication in automation?"
> "**Real-World Example:** Our application uses a strict SSO that is extremely difficult to automate dynamically through the UI. 
> 
> Instead of fighting the UI, we use a local browser profile strategy. We manually log into our Virtual Machine once via SSO. This stores the cookies, local storage, and security keys directly into the local Edge browser profile. 
> 
> Then, in our Playwright framework, we use a custom async fixture called `loginUsingExistingEdgeProfile()`. When Playwright launches, it doesn't open a blank incognito window; it points directly to that saved Edge profile folder. Playwright instantly inherits all the active security keys and cookies, completely bypassing the SSO screen and landing us directly on the Home Page."

```typescript
// custom fixture: loginUsingExistingEdgeProfile()
async function loginUsingExistingEdgeProfile(browser) {
    const context = await browser.launchPersistentContext(
        'C:/Users/TestUser/EdgeProfile', // Path to the saved Edge profile
        { channel: 'msedge', headless: false }
    );
    const page = await context.newPage();
    await page.goto(process.env.APP_URL); // Goes straight to Home Page — no SSO prompt
    return page;
}
```

### Q19. "How do you debug and resolve 404 (Not Found) API errors in automation?"
> "**Real-World Example:** A 404 error usually means the test asked for data that doesn't exist yet, or the URL changed.
> 
> When my test gets a 404, I don't just guess what went wrong. Playwright's API context automatically logs the exact Request URL and the Payload. 
> 1. First, I check if the test created the test data correctly in the previous step (maybe User #123 wasn't actually created, so asking for User #123 gives a 404).
> 2. Second, I check the environment variables. If my test is hitting `qa-server/api/v1` but the developers upgraded it to `v2`, I'll get a 404. I fix this by keeping all base URLs dynamic in my `.env` file."

```typescript
const response = await request.get(`${process.env.BASE_URL}/api/v2/user/123`);

if (response.status() === 404) {
    console.error('404 Debug Info:', {
        url:     response.url(),
        status:  response.status(),
        body:    await response.text()
    });
}
expect(response.ok()).toBeTruthy(); // Fails with clear debug output above
```

### Q20. "How do you validate the API and the UI simultaneously (Hybrid Testing)?"
> "**Real-World Example:** Imagine testing an 'Add to Cart' button. If you just click it and check if the cart icon says '1', the UI might look fine, but the backend database might be completely broken.
> 
> I do Hybrid Testing. In the exact same test:
> 1. I click 'Add to Cart' on the UI.
> 2. At the same time, I use `Promise.all` to intercept the background API network call.
> 3. I assert that the UI cart updated to '1' AND the API returned a `201 Created` with the correct item ID in the JSON response. This guarantees both the front-end and back-end are perfectly in sync."

```typescript
// Intercept API and click button simultaneously
const [apiResponse] = await Promise.all([
    page.waitForResponse(r => r.url().includes('/api/cart') && r.status() === 201),
    page.getByTestId('add-to-cart-btn').click()
]);

// Assert UI updated
await expect(page.getByTestId('cart-count')).toHaveText('1');

// Assert API response body
const body = await apiResponse.json();
expect(body.itemId).toBe('PROD_456');
```

### Q21. "How do you auto-generate API scripts at runtime while executing UI tests?"
> "**Real-World Example:** Writing hundreds of API tests manually takes forever. Instead, I let the UI do the heavy lifting.
> 
> I wrote a custom script generator using Playwright's network sniffing (`page.on('request')`). While my UI automation is running normally (like filling out a checkout form), my script silently listens to all the background API calls. It automatically extracts the endpoints, JSON payloads, and headers, and writes them into a new `api-test.spec.js` file on the fly. By the time the UI test finishes, it has literally coded the API test for me!"

```typescript
import fs from 'fs';

page.on('request', request => {
    if (request.resourceType() === 'fetch') {
        const snippet = `
// Auto-generated API test
test('${request.method()} ${request.url()}', async ({ request }) => {
    const res = await request.${request.method().toLowerCase()}('${request.url()}', {
        data: ${JSON.stringify(request.postDataJSON())}
    });
    expect(res.ok()).toBeTruthy();
});`;
        fs.appendFileSync('generated-api-tests.spec.js', snippet);
    }
});
```

### Q22. "How do you integrate API testing directly into your CI/CD pipeline?"
> "**Real-World Example:** In GitLab, I don't need to configure a separate server like Newman just for API tests. 
> 
> My pipeline just runs `npx playwright test`. Because my API tests and UI tests are written in the same language and use the same Playwright runner, GitLab executes them in parallel on the same machine. It generates one unified HTML report showing both the API health and the UI test results for the whole team."

---

## ⭐ NEW — QUESTIONS FROM SIEMENS INTERVIEW (Must Know for Maxxton)

> These were actual questions asked in the Siemens interview. Maxxton panelists — especially the Playwright Specialist — will likely ask the same.

---

### Q-S1. "How do you add the full API response (headers + body + JSON) into the Playwright report generically — without a util method or test-specific storage?"

> **IMPORTANT — This is a very specific technical question. The interviewer wants a generic, inline approach using Playwright's built-in `test.info().attach()` — not a separate utility function.**

**Answer:**
> "The cleanest way is to use `test.info().attach()` directly inside the test. Playwright's test runner exposes `testInfo` — this is the live test context object. I call `.attach()` on it to embed any data — body, headers, JSON — directly into the HTML report as downloadable attachments. No util class. No helper function. Just one line per piece of data."

```typescript
import { test, expect } from '@playwright/test';

test('Verify booking creation API response is complete', async ({ request }) => {
    const response = await request.post('/api/v1/bookings', {
        data: { propertyId: 'HOTEL_101', checkIn: '2025-08-01', checkOut: '2025-08-05' }
    });

    // ── Generic: attach EVERYTHING to the report inline ──────────────────

    // 1. Attach Status Line
    await test.info().attach('response-status', {
        body: Buffer.from(`${response.status()} ${response.statusText()}`),
        contentType: 'text/plain'
    });

    // 2. Attach ALL Response Headers (generic — no field names hardcoded)
    const headers = response.headers(); // returns a plain key-value object
    await test.info().attach('response-headers', {
        // JSON.stringify(..., null, 2) pretty-prints the object with 2 spaces indentation.
        // Buffer.from() converts the string to raw bytes since .attach() writes binary data to the report.
        body: Buffer.from(JSON.stringify(headers, null, 2)),
        contentType: 'application/json'
    });

    // 3. Attach raw body text
    const rawBody = await response.text();
    await test.info().attach('response-body-raw', {
        body: Buffer.from(rawBody),
        contentType: 'text/plain'
    });

    // 4. Attach parsed JSON (pretty-printed)
    const json = await response.json();
    await test.info().attach('response-body-json', {
        // null, 2 ensures the JSON is readable in the HTML report instead of one long line.
        body: Buffer.from(JSON.stringify(json, null, 2)),
        contentType: 'application/json'
    });
    // ─────────────────────────────────────────────────────────────────────

    expect(response.ok()).toBeTruthy();
    expect(json.bookingId).toBeDefined();
});
```

> **What to say in the interview (One-Liner):**
> *"I use `JSON.stringify` with `null, 2` to pretty-print the JSON object, then wrap it in `Buffer.from()` because Playwright's `.attach()` requires raw bytes to write the file. Using `contentType: 'application/json'` tells the HTML report to render it automatically as a collapsible, readable JSON viewer."*

> **Why this is "generic":** I'm not storing values in a custom `reportHelper.attachJson(response)` — I'm calling `test.info().attach()` directly. The `contentType: 'application/json'` flag makes Playwright HTML report render it as a collapsible JSON viewer. You can see headers, body, and status all as separate named attachments in the Allure or Playwright HTML report without any custom code.

> > 🔄 **Cross-Question:** "How do you do this for ALL tests globally without repeating it?"
> >
> > **Answer:** "I add a `fixture` that wraps the Playwright `APIRequestContext`. The fixture intercepts the response, calls `test.info().attach()` once, and returns the response. Every test that uses this fixture automatically gets the attachment — no copy-paste needed."

---

### Q-S2. "Workers = 1. How do you still speed up execution without running tests in parallel processes?"

> **This is a tricky question. The interviewer is checking if you understand Playwright's concurrency model at the Page level, not just Worker level.**

**Answer (exactly what you said in the interview — expand on this):**

> "When `workers = 1`, only one Worker process runs at a time, so you can't use multi-process parallelism. But you can still gain speed at two levels:
>
> **Level 1 — Separate Pages within the same BrowserContext:**
> Instead of running tests sequentially one-by-one (open tab → test → close tab → repeat), I keep one browser context alive across the test suite and open a **new page (tab)** for each test. Opening a new tab is near-instant — no browser restart, no login overhead.
>
> **Level 2 — `storageState` Fixture to skip login from Test 2 onwards:**
> The expensive part of any E2E test is login. I solve this with a scoped `storageState` fixture:
> - **First test** logs in via the UI, saves `page.context().storageState()` to a JSON file.
> - **From the 2nd test onward,** the fixture injects `storageState: 'auth.json'` when creating the context. The browser opens already authenticated. The login page is never visited again.
> - Combined: single worker + shared context + pre-saved auth = suite runs almost as fast as parallel."

```typescript
// fixtures/authFixture.ts
import { test as base, expect } from '@playwright/test';
import * as fs from 'fs';

const AUTH_FILE = 'playwright/.auth/user.json';

// This fixture runs ONCE — saves auth state so all subsequent tests skip login
export const test = base.extend<{ authenticatedPage: any }>({   
    authenticatedPage: async ({ browser }, use) => {
        let context;

        if (fs.existsSync(AUTH_FILE)) {
            // ✅ Auth state already saved → skip login, open new page instantly
            context = await browser.newContext({ storageState: AUTH_FILE });
        } else {
            // 🔑 First time: login via UI and save the cookies
            context = await browser.newContext();
            const page = await context.newPage();
            await page.goto(process.env.APP_URL!);
            await page.fill('#username', process.env.USER!);
            await page.fill('#password', process.env.PASS!);
            await page.click('#login-btn');
            await page.waitForURL('**/dashboard');
            // Save cookies + localStorage to file
            await context.storageState({ path: AUTH_FILE });
        }

        // Each test gets its own Page (tab) within the shared context
        const page = await context.newPage();
        await use(page);
        await page.close(); // Only close the tab, NOT the context
    }
});

// test-1.spec.ts — uses the fixture
test('Verify property listing loads', async ({ authenticatedPage }) => {
    await authenticatedPage.goto('/properties');
    await expect(authenticatedPage.getByRole('heading', { name: 'Properties' })).toBeVisible();
});

test('Verify booking form opens', async ({ authenticatedPage }) => {
    // Auth state already injected — no login happened
    await authenticatedPage.goto('/bookings/new');
    await expect(authenticatedPage.getByTestId('booking-form')).toBeVisible();
});
```

> **Key insight to say in the interview:** "This works because Playwright contexts are isolated, but within a context, pages share the same session. With `storageState`, Test 2, 3, 4... all launch directly on the dashboard. The bottleneck (login) happens exactly once regardless of how many tests you have."

---

### Q-S3. "How do you manually write an API test in Playwright — validating headers, body, and large JSON?"

> **This is the core question. The interviewer is NOT asking about auto-generation (AJS/ATS scripts). They want to know: when YOU write the test yourself, how do you validate everything properly?**

**Answer — 5 layers of validation:**

> "I validate API responses in 5 layers, from most basic to most advanced. Each layer catches a different class of bug."

```typescript
import { test, expect } from '@playwright/test';
import Ajv from 'ajv'; // npm install ajv

test('Booking API — full validation: status, headers, body, large JSON', async ({ request }) => {

    // ─── MAKE THE CALL ────────────────────────────────────────────────────────
    const response = await request.post('/api/v1/bookings', {
        headers: {
            'Authorization': `Bearer ${process.env.AUTH_TOKEN}`,
            'Content-Type':  'application/json',
            'Accept':        'application/json'
        },
        data: {
            propertyId: 'HOTEL_101',
            roomType:   'DELUXE',
            checkIn:    '2025-08-01',
            checkOut:   '2025-08-05',
            guests:     2
        }
    });

    // ─── LAYER 1: STATUS CODE ─────────────────────────────────────────────────
    // Always validate this first. Wrong status = everything below is meaningless.
    expect(response.status()).toBe(201);           // Exact status
    expect(response.ok()).toBeTruthy();            // 200-299 range check (good for flexible smoke tests)

    // ─── LAYER 2: RESPONSE HEADERS ───────────────────────────────────────────
    // Validate specific headers you care about
    const headers = response.headers(); // Returns plain { key: value } object

    // Specific header checks
    expect(headers['content-type']).toContain('application/json');  // NOT strict equals — server may append charset
    expect(headers['x-request-id']).toBeDefined();                  // Traceability header must exist
    expect(headers['cache-control']).toBe('no-store');              // POST should never be cached

    // Check a header does NOT exist (security check)
    expect(headers['x-powered-by']).toBeUndefined(); // Server should not expose tech stack

    // ─── LAYER 3: RESPONSE BODY — SPECIFIC FIELD ASSERTIONS ─────────────────
    // Parse once, assert many. Good for small-medium payloads.
    const body = await response.json();

    // Exact value assertions
    expect(body.status).toBe('CONFIRMED');
    expect(body.propertyId).toBe('HOTEL_101');
    expect(body.guests).toBe(2);

    // Type assertions (does the field exist AND is it the right type?)
    expect(typeof body.bookingId).toBe('string');
    expect(typeof body.totalPrice).toBe('number');
    expect(Array.isArray(body.roomDetails)).toBe(true);

    // Existence check without caring about value
    expect(body.confirmationCode).toBeDefined();
    expect(body.confirmationCode).not.toBeNull();

    // Date format check using regex
    expect(body.checkIn).toMatch(/^\d{4}-\d{2}-\d{2}$/); // Must be YYYY-MM-DD

    // ─── LAYER 4: LARGE JSON — PARTIAL / NESTED ASSERTIONS ──────────────────
    // For large payloads with 50+ fields, don't write 50 expects.
    // Use expect(body).toMatchObject() — checks ONLY the fields you specify.
    // Extra fields in the response are IGNORED. Perfect for large JSON.

    expect(body).toMatchObject({
        status:     'CONFIRMED',
        propertyId: 'HOTEL_101',
        guest: {
            // Nested object — only checks these fields, ignores other guest fields
            adults:   2,
            children: 0
        },
        payment: {
            currency: 'EUR',
            method:   'CARD'
            // 'transactionId', 'amount' etc are NOT listed here — that's fine
        }
    });

    // For arrays — check at least one element matches
    expect(body.roomDetails).toEqual(
        expect.arrayContaining([
            expect.objectContaining({
                roomType: 'DELUXE',
                floor:    expect.any(Number)  // any number, don't care which floor
            })
        ])
    );

    // ─── LAYER 5: SCHEMA VALIDATION — FULL JSON SHAPE ENFORCEMENT ───────────
    // When the JSON has 50+ fields, write a schema ONCE.
    // Ajv validates ALL fields, types, and required fields in one line.
    // A developer silently changing 'bookingRef' to 'booking_ref' will fail here.

    const ajv = new Ajv();
    const bookingSchema = {
        type: 'object',
        required: ['bookingId', 'status', 'propertyId', 'checkIn', 'checkOut',
                   'totalPrice', 'confirmationCode', 'guest', 'payment'],
        properties: {
            bookingId:        { type: 'string' },
            status:           { type: 'string', enum: ['CONFIRMED', 'PENDING', 'CANCELLED'] },
            propertyId:       { type: 'string' },
            checkIn:          { type: 'string', format: 'date' },
            checkOut:         { type: 'string', format: 'date' },
            totalPrice:       { type: 'number', minimum: 0 },
            confirmationCode: { type: 'string' },
            guest: {
                type: 'object',
                required: ['adults'],
                properties: {
                    adults:   { type: 'integer', minimum: 1 },
                    children: { type: 'integer', minimum: 0 }
                }
            },
            payment: {
                type: 'object',
                required: ['currency', 'method'],
                properties: {
                    currency: { type: 'string' },
                    method:   { type: 'string' }
                }
            },
            roomDetails: {
                type: 'array',
                items: {
                    type: 'object',
                    required: ['roomType'],
                    properties: {
                        roomType: { type: 'string' },
                        floor:    { type: 'integer' }
                    }
                }
            }
        },
        additionalProperties: true // Allow extra fields — set to false to be strict
    };

    const validate = ajv.compile(bookingSchema);
    const isValid  = validate(body);

    if (!isValid) {
        // Print exactly what field failed — interviewer loves this
        console.error('Schema validation errors:', ajv.errorsText(validate.errors));
    }
    expect(isValid).toBe(true); // One assertion covers ALL 50+ fields
});
```

**Summary of the 5 layers — say this out loud:**

> | Layer | What you check | Playwright API used |
> |---|---|---|
> | **1 — Status** | `201`, `ok()` | `response.status()`, `response.ok()` |
> | **2 — Headers** | `content-type`, `cache-control`, security headers | `response.headers()` |
> | **3 — Specific fields** | Exact values, types, format (date regex) | `response.json()` + `expect(body.field)` |
> | **4 — Large JSON partial** | Nested objects, arrays — only the fields you care about | `toMatchObject()`, `arrayContaining()`, `objectContaining()` |
> | **5 — Schema** | ALL fields, ALL types, required fields — one assertion | `Ajv` schema validation |

> > 🔄 **Cross-Question:** "What is the difference between `toEqual` and `toMatchObject`?"
> >
> > **Answer:**
> > - `toEqual` → **exact match** — every field in the expected object must match every field in the actual. Extra fields in actual = **FAIL**. Use when you know the complete shape.
> > - `toMatchObject` → **partial match** — actual can have MORE fields than expected and it still passes. Use for **large JSON** where you only care about key fields, not the entire 50-field payload.

> > 🔄 **Cross-Question:** "Why use Ajv over writing 50 `expect` statements?"
> >
> > **Answer:** "Speed and maintenance. Writing 50 assertions takes 30 minutes and breaks every time a field name changes. The Ajv schema takes 10 minutes to write once, and if a developer accidentally renames `bookingId` to `booking_id`, it fails instantly with a clear error message: `'bookingId' is required`. I also reuse schemas across tests — define once in a `schemas/` folder, import everywhere."

---

## PHASE 5: DEVOPS, DOCKER, & GITLAB CI/CD

### Q23. "Explain your GitLab CI YAML configuration."
> "YAML is just a list of instructions for the CI server.
> 
> **Real-World Example:** My CI pipeline has 3 stages: Install, Test, Publish.
> - **Install:** I use `npm ci` instead of `npm install` because `npm ci` installs the exact package versions from the lockfile. `npm install` might upgrade something and break my tests.
> - **Test (Sharding):** Instead of running 100 tests on 1 machine (which takes an hour), I split it across 3 machines at the same time using a Parallel Matrix. Each machine runs 33 tests, and the whole suite finishes in 20 minutes."

### Q24. "Why do you use Docker for UI testing?"
> "**Real-World Example:** Docker solves the 'It works on my machine' problem. My laptop might be a Mac, but the CI server is Linux. 
> 
> I use a Dockerfile to package the tests. I never run the tests as the 'root' admin user inside Docker for security reasons. I create a specific, restricted user called `pwuser` to run the tests."

### Q25. "Why is `--shm-size` important in Docker for Playwright?"
> "**Real-World Example:** Browsers need memory (RAM) to draw web pages. Docker, by default, only gives containers a tiny 64MB of shared memory. A modern web app will instantly crash Chrome if it only has 64MB. 
> 
> I set `--shm-size=2gb` in my Docker run command to give the browser enough RAM so my headless tests never crash."

### Q26. "How do you fix a merge conflict in `package-lock.json`?"
> "**Real-World Example:** You should NEVER open `package-lock.json` and edit it manually. You will corrupt it. 
> 
> If I get a conflict, I switch to my branch, pull the latest changes from `main`, and run `npm install`. This forces the npm tool to regenerate the lockfile correctly on its own."

### Q27. "What are Shift-Left quality gates?"
> "**Real-World Example:** Shift-left just means finding bugs early. Before a developer can even click 'commit', a tool called Husky runs my linter and basic tests locally on their laptop. 
> 
> For security, I never hardcode API keys in my script. I store them as secret variables in GitLab. That way, if someone looks at my code on GitHub, they can't steal the passwords."

---

## PHASE 6: SCENARIOS & CLOSING

### Q28. "How would you automate a CAPTCHA?"
> "**Real-World Example:** You literally cannot automate CAPTCHA—its entire purpose is to block bots like Playwright! 
> 
> If you try to bypass it with OCR or AI, it's going to be extremely flaky. Instead, I ask the backend developers to disable CAPTCHA in the QA environment, or they give us a special secret header (like `X-Bypass-Captcha: true`) that tells the server to let our automation through safely."

### Q29. "How do you do performance testing with K6?"
> "**Real-World Example:** I use K6 to simulate 100 users logging in at the same time. 
> 
> I set a strict threshold: `p(95) < 500ms`. This means 95% of users must experience a page load faster than half a second. If a cold-start makes 1 request slow, the test still passes. But if the whole system slows down, K6 fails the CI pipeline to stop the bad code from deploying."

### Q30. "Do you have any questions for us?"
> "Yes, how do developers and QA work together here? Do developers help write UI tests, or do they just throw the code over the wall and expect QA to automate everything?"

---

## ⭐ CODING CHALLENGES (Siemens asked — Maxxton will too)

### Q-C1. "Write a function to flatten a nested array."

**Answer:**
> "There are three ways. I'll show all of them because the interviewer often asks 'what other approaches do you know?'"

```javascript
// Input:  [1, [2, [3, [4, 5]]], 6]
// Output: [1, 2, 3, 4, 5, 6]

// ── Approach 1: Built-in flat() with Infinity ──────────────────────────────
const nested = [1, [2, [3, [4, 5]]], 6];
const result1 = nested.flat(Infinity);
console.log(result1); // [1, 2, 3, 4, 5, 6]

// ── Approach 2: Recursive (good for explaining logic step by step) ─────────
function flattenRecursive(arr) {
    return arr.reduce((acc, item) => {
        if (Array.isArray(item)) {
            return acc.concat(flattenRecursive(item)); // Go deeper if it's an array
        }
        return acc.concat(item); // It's a primitive, just add it
    }, []);
}
console.log(flattenRecursive(nested)); // [1, 2, 3, 4, 5, 6]

// ── Approach 3: Stack-based (no recursion — interviewer may ask this) ──────
function flattenStack(arr) {
    const stack = [...arr];
    const result = [];
    while (stack.length) {
        const item = stack.pop();
        if (Array.isArray(item)) {
            stack.push(...item); // Unpack array back into stack
        } else {
            result.unshift(item); // Add primitive to front (preserves order)
        }
    }
    return result;
}
console.log(flattenStack(nested)); // [1, 2, 3, 4, 5, 6]
```

> > 🔄 **Cross-Question:** "When would you use the recursive approach over `flat(Infinity)`?"
> >
> > **Answer:** "In production code I'd use `flat(Infinity)` — it's concise, readable, and native. The recursive approach is useful if I need custom logic during flattening, like filtering out null values while flattening, or transforming each element. In an interview, showing the recursive approach proves I understand the underlying algorithm."

---

### Q-C2. "Move all zeros to the end of an array, maintaining the order of non-zero elements."

**Problem:**
```
Input:  [2, 0, 0, 0, 3, 0, 0, 5, 0, 7, 6, 0]
Output: [2, 3, 5, 7, 6, 0, 0, 0, 0, 0, 0, 0]
```

**Answer:**
> "The interviewer wants to see if I understand in-place manipulation vs. creating new arrays. I'll show both."

```javascript
const arr = [2, 0, 0, 0, 3, 0, 0, 5, 0, 7, 6, 0];

// ── Approach 1: filter + fill (Clean, functional, 2 lines) ────────────────
function moveZerosClean(arr) {
    const nonZeros = arr.filter(x => x !== 0);         // [2, 3, 5, 7, 6]
    const zeros    = new Array(arr.length - nonZeros.length).fill(0); // [0,0,0,0,0,0,0]
    return [...nonZeros, ...zeros];
}
console.log(moveZerosClean(arr)); // [2, 3, 5, 7, 6, 0, 0, 0, 0, 0, 0, 0]

// ── Approach 2: Two-pointer (in-place, O(n) time, O(1) space) ─────────────
// This is the "impressive" answer — shows algorithm knowledge
function moveZerosInPlace(arr) {
    let writePointer = 0; // Points to where the next non-zero should go

    // Pass 1: Move all non-zeros to the front
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] !== 0) {
            arr[writePointer] = arr[i];
            writePointer++;
        }
    }
    // Pass 2: Fill the rest with zeros
    for (let i = writePointer; i < arr.length; i++) {
        arr[i] = 0;
    }
    return arr;
}
console.log(moveZerosInPlace([2, 0, 0, 0, 3, 0, 0, 5, 0, 7, 6, 0]));
// [2, 3, 5, 7, 6, 0, 0, 0, 0, 0, 0, 0] ✅
```

> **What to say:** "I prefer the two-pointer approach when memory matters (like processing large telemetry logs in automation). The `filter` approach creates two new arrays — fine for small data. In an interview, I always mention the trade-off: clean code vs. memory efficiency."

> > 🔄 **Cross-Question:** "What is the time and space complexity?"
> >
> > **Answer:**
> > - Filter approach: O(n) time, O(n) space (creates new array)
> > - Two-pointer: O(n) time, **O(1) space** (modifies in place) — this is the optimal answer.

---

## ⭐ MAXXTON-SPECIFIC TOPICS (QA Manager — AI + Strategy Panel)

### Q-M1. "What is a test pyramid and how do you apply it for a hospitality platform?"

> **Answer:**
> "The test pyramid has 3 layers. The idea is: write more tests at the bottom (fast, cheap) and fewer at the top (slow, expensive).
>
> ```
>           /\\
>          /UI\\ ← fewest: E2E tests (slow, brittle, expensive)
>         /────\\
>        / API  \\ ← middle: integration/API tests (fast, stable)
>       /────────\\
>      /  UNIT    \\ ← most: unit tests (instant, cheap, developer-owned)
>     /────────────\\
> ```
>
> **For Maxxton's hospitality platform specifically:**
> - **Unit tests:** Business logic — pricing engine calculations, discount rules, date math for check-in/check-out
> - **API tests:** Booking creation API, PMS sync endpoints, payment gateway integration, channel manager feed API
> - **UI E2E tests:** Critical guest journeys only — Search property → Select room → Fill guest details → Confirm booking → Receive confirmation email
>
> I follow an **80/15/5 split**: 80% unit, 15% API, 5% UI. This keeps the suite fast and reduces flakiness."

---

### Q-M2. "How do you handle contract testing for microservices?"

> **Answer:**
> "In a hospitality platform, services like Booking, PMS, Channel Manager, and Payment all talk to each other via APIs. If the Booking service changes the shape of its response (renames a field), the PMS service breaks — and you only find out in production.
>
> Contract testing solves this. I use **Pact**. Here's how it works:
> - The **Consumer** (PMS) defines a 'contract' — 'I expect the Booking API to return `{ bookingId: string, guestName: string }`'
> - The **Provider** (Booking service) runs against this contract in CI to verify it still honours it.
> - If the Booking team renames `guestName` to `guest_name`, the Pact contract test fails immediately — before any UI test even runs.
>
> This is 10x faster than finding the breakage through E2E tests."

---

### Q-M3. "How are you using AI in QA? What have you built?"

> **Answer (tailored for the QA Manager who works in AI):**
> "I've built two AI-assisted tools:
>
> **1. Self-healing Locator Agent:**
> When a UI test fails because a developer changed a CSS class or removed a `data-testid`, my agent reads the Playwright failure screenshot and the current DOM snapshot. It calls an LLM (GPT-4 or Gemini) with a prompt: 'Here is the broken locator. Here is the current HTML. Suggest a stable replacement locator.' The agent updates the spec file with the new locator and raises a PR for review. This cut our locator maintenance time by 40%.
>
> **2. Auto-generated API test scripts:**
> While UI tests run, I intercept all XHR/fetch network calls using `page.on('request')`. My script captures the endpoint, method, payload, and headers, and auto-generates Playwright API test stubs. A manual suite of 50 API tests was generated automatically in under an hour.
>
> For Maxxton, I'd extend this to generate booking workflow test cases from OpenAPI/Swagger specs — so the moment the API contract changes, new tests are auto-generated."

---

### Q-M4. "How do you mentor junior SDETs?"

> **Answer:**
> "I follow a 'See one, Do one, Teach one' model:
>
> **See one:** I pair-program with the junior on their first task. I write the code, they watch and ask questions.
>
> **Do one:** The second task, they write the code, I review it live and explain why I'd change certain things — not just 'what', but 'why'. For example: 'Why are you using `page.waitForTimeout(2000)` here? What happens if the server is slow and 2 seconds isn't enough?' This teaches judgment, not just syntax.
>
> **Teach one:** Once they're comfortable, I ask them to do a 15-minute knowledge share with the team on what they learned. Teaching solidifies understanding.
>
> For code reviews, I have a checklist: Are locators stable? Is test data hardcoded or dynamic? Is there proper error handling? Does this test have a single responsibility? I document common mistakes as a team 'Anti-patterns' guide so we don't repeat them."

---

### Q-M5. "How do you handle flaky tests? What is your quarantine strategy?"

> **Answer:**
> "Flaky tests are the #1 trust killer for automation. If the team ignores flaky failures, they'll start ignoring all failures.
>
> **My quarantine strategy:**
> 1. **Detect:** I track the pass/fail history of every test in CI. Any test that fails > 2 times in the last 10 runs without a code change is flagged as 'flaky'.
> 2. **Tag:** I tag it `@flaky` immediately. In GitLab CI, flaky tests run in a separate job that doesn't block the main pipeline. The main gate is not broken, but the team sees the flaky count.
> 3. **Investigate:** I look at Playwright traces (`.zip` trace files). 90% of flaky tests have one of three root causes: timing issue (fixed with better `waitFor`), test data collision (fixed with isolated data per test), or environment issue (fixed with retry or pre-condition check).
> 4. **Fix or Delete:** If a test is flaky because the feature itself is unstable, I escalate it as a product bug. If it can't be fixed in a sprint, I delete it — a flaky test is worse than no test."

---

### Q-M6. "How would you test the booking/reservation workflow end-to-end?"

> **Answer:**
> "For a hospitality platform like Maxxton, the booking flow is the most critical path. I'd design it as a layered test:
>
> **API Layer (smoke):**
> - POST `/api/bookings` → verify `201 Created`, check `bookingId` in response
> - GET `/api/bookings/{id}` → verify booking details match what was sent
> - GET `/api/availability` → verify room shows as unavailable after booking
>
> **UI Layer (critical path E2E):**
> - Search → Select property → Choose room → Fill guest info → Enter payment → Confirm → Assert confirmation email
>
> **Negative Paths:**
> - Book a room on dates that are already fully booked → expect `409 Conflict`
> - Submit payment with expired card → expect error message on UI
> - Attempt booking with check-out before check-in → expect validation error
>
> **Integration check (Hybrid):**
> - UI submits booking → simultaneously intercept the `/api/pms/sync` call → assert PMS received the correct guest record
>
> This gives me full confidence the front-end, back-end, and third-party integrations are all in sync."

---

### Q-M7. "Tell me about your quality KPIs, metrics, and Release Governance."

> **Answer:**
> "Release Governance means having a strict 'Quality Gate' before code goes to Production. My pipeline checks these 5 KPIs automatically:
>
> | KPI | What it means | My target |
> |---|---|---|
> | **Automation Coverage** | % of test cases that are automated | > 80% |
> | **Flaky Rate** | % of tests that are non-deterministic | < 2% |
> | **Escape Rate** | Bugs found in production that should've been caught in QA | < 5% |
> | **MTTD** (Mean Time to Detect) | How quickly automation catches a bug after it's introduced | < 30 min |
> | **Pipeline Duration** | How long the full test suite takes | < 20 min |
>
> I report these weekly in Confluence. If escape rate spikes, I do a root-cause analysis — usually it's a coverage gap in the API layer."

---

### Q-M8. "How do you do visual regression testing?"

> **Answer:**
> "I use Playwright's built-in screenshot comparison: `await expect(page).toHaveScreenshot('booking-form.png')`. The first run captures a baseline. Every subsequent run compares pixel-by-pixel against that baseline.
>
> For a hospitality platform, I apply visual tests to:
> - The booking confirmation page layout
> - The room calendar/availability grid
> - The guest checkout flow on mobile viewports
>
> When a screenshot diff is detected, Playwright generates a diff image showing exactly which pixels changed (red overlay). This catches CSS regressions that functional tests completely miss — like a button moving 10px off center due to a CSS change.
>
> For Maxxton's cross-browser requirement, I run the same visual tests on Chromium, Firefox, and WebKit in the CI matrix."

---

### Q-M9. "How do you perform Root Cause Analysis (RCA) on production incidents, and how does Playwright Trace Viewer help?"
> **(Directly targets JD: "Drive root-cause analysis on production incidents", "trace viewer")**
> 
> "When a production incident occurs (e.g., a guest couldn't book a room):
> 1. **Analyze:** I look at Datadog/Splunk logs to find the exact API failure or JS error.
> 2. **Reproduce:** I try to reproduce the state in our Staging environment.
> 3. **Trace Viewer:** If a CI/CD test fails intermittently while reproducing, I don't guess. I open the **Playwright Trace Viewer**. It gives me a full timeline: the DOM snapshot before and after every click, network logs, and console errors. It is basically time-travel debugging. I can see *exactly* what the browser looked like at the millisecond the test failed.
> 4. **Durable Coverage:** Once I identify the root cause, I write an automated test specifically for that edge case and add it to our regression suite to ensure that bug *never* escapes to production again."

### Q-M10. "How do you handle complex test data generation for hospitality workflows?"
> **(Directly targets JD: "data-setup utilities", "data builders", "PMS operations")**
> 
> "For complex hospitality workflows, you can't rely on hardcoded data because rooms get booked and states change. I use the **Data Builder Pattern (Factory Pattern)**.
> 
> Instead of creating a massive JSON object in every test, I have a `BookingDataBuilder` utility:
> ```typescript
> const guestBooking = new BookingDataBuilder()
>     .withRoomType('Suite')
>     .withCheckInDaysFromNow(5)
>     .withAdults(2)
>     .build(); // Generates random, unique names, emails, and dates
> ```
> This data is then pushed directly to the database or via a fast internal API to prep the system state *before* the UI test starts. This makes tests incredibly fast and deterministic."

### Q-M11. "You have a Java background. Why do you use TypeScript for Playwright, and when would you use Java?"
> **(Directly targets JD: "Strong programming skills in both TypeScript/JavaScript and Java")**
> 
> "While Playwright *does* support Java, its native language is TypeScript/JavaScript. I strongly prefer **TypeScript for UI automation** because:
> 1. Playwright's TS API gets new features months before the Java API.
> 2. It lives right next to the frontend React/Angular codebase, so frontend developers can easily review and contribute to my tests.
> 3. TypeScript provides the strict type-safety of Java, but with the flexibility of modern JavaScript asynchronous handling (Promises/Microtasks).
> 
> However, I use **Java** when building massive backend test utilities or dealing with legacy SOAP services and complex backend data generation, as the JVM ecosystem (like REST Assured) is extremely mature for API integration."

### Q-M12. "What is 'Smart Test Selection' in your CI/CD pipeline?"
> **(Directly targets JD: "smart test selection", "flaky-test quarantine")**
> 
> "If a developer changes one line of CSS in the 'Payments' module, we shouldn't run all 5,000 UI tests across the entire hospitality suite. That wastes CI compute and time.
> 
> **Smart Test Selection** means our pipeline analyzes the Git commit diff. If the changes are in the `/payments` directory, the GitLab pipeline dynamically selects only tests tagged with `@payments` or tests located in the `tests/payments/` folder. We run a full suite only on nightly builds or merge-to-main. This shift-left approach gives developers feedback in 5 minutes instead of 2 hours."

### Q-M13. "Can you explain the Screenplay Pattern, and how is it different from the Page Object Model (POM)?"
> **(Targets JD: "test architecture patterns — page object model, screenplay")**
> 
> "POM focuses on the **Page** (e.g., `LoginPage.js`). As the page grows, the class becomes a massive, unmaintainable 'God Object' with 100 methods.
> 
> **Screenplay Pattern** focuses on the **Actor** and their **Tasks**. Instead of `loginPage.login(user, pass)`, you write: `actor.attemptsTo(Login.with(user, pass))`.
> 1. **Actor:** Who is doing it (e.g., 'FrontDeskAgent').
> 2. **Task:** What they are doing (e.g., 'MakeReservation').
> 3. **Action:** Low-level clicks (e.g., 'Click the submit button').
> 
> I use POM for simple CRUD apps. I use Screenplay for highly complex workflow apps (like a Hospitality PMS) because it makes code modular, reusable, and reads exactly like a BDD business requirement."

### Q-M14. "My Playwright tests pass on my local machine but fail randomly in the CI/CD pipeline. How do you troubleshoot this?"
> **(Targets JD: "cross-browser stability patterns", "flaky-test quarantine")**
> 
> "This is the most common issue in automation. When a test fails only in CI, it's almost always a **Race Condition** or **Resource Starvation**.
> 
> 1. **Resource Starvation (Docker/CI):** My laptop has 32GB of RAM. The CI container might only have 2GB and a tiny shared memory size. This causes the browser to crash or run extremely slowly. I fix this by setting `--shm-size=2gb` in Docker and ensuring the CI worker isn't running 10 parallel processes when it only has 2 CPU cores.
> 2. **Race Conditions:** On my fast local machine, the API responds in 50ms. In CI, the API might take 2 seconds. If the test uses a hardcoded `page.waitForTimeout(1000)`, it fails in CI. I fix this by deleting all hardcoded sleeps and using Playwright's `page.waitForResponse('/api/bookings')` to dynamically wait for the exact network call to finish.
> 3. **Trace Viewer:** If it's still failing, I download the **Playwright Trace file** from the CI artifacts. It shows me exactly what the CI machine saw (DOM, Network, Console) at the exact millisecond it failed."

### Q-M15. "How do you see Agentic AI, RAG, and MCP shaping the future of Test Automation?"
> **(Crucial for the QA Manager actively building AI solutions)**
> 
> "Generative AI (like Copilot) just writes code blocks, but **Agentic AI** acts autonomously.
> 
> 1. **Agentic AI:** Instead of me writing step-by-step locator scripts, an Agentic framework takes a prompt: *"Book a standard room for tomorrow."* The agent dynamically explores the DOM, figures out where the calendar is, handles pop-ups autonomously, and completes the flow without strict hardcoded locators. 
> 2. **RAG (Retrieval-Augmented Generation):** AI struggles with hallucinations. By using RAG, we can feed our internal Confluence PRDs (Product Requirements Documents) and Swagger API specs directly into the LLM context. So when the AI writes a test or investigates a failure, it is looking at our *actual proprietary business rules*, not generic internet data.
> 3. **MCP (Model Context Protocol):** This is the game-changer for CI/CD. MCP allows our AI agents to securely 'talk' to our internal databases, GitHub repos, and Jira. An AI Agent using MCP could detect a test failure, query the backend DB to see if the test data was corrupted, check Jira for known bugs, and automatically raise a highly-detailed bug report with logs attached—completely autonomously."

---

## PHASE 7: CLOSING

### Q31. "Do you have any questions for us?"
> "Yes, I have two:
> 1. *(For QA Manager)*: You mentioned you are actively working on AI solutions. Are you using AI mainly for test generation (like GitHub Copilot), or are you building AI agents to do self-healing and intelligent log analysis?
> 2. *(For QA Specialist)*: What is the biggest bottleneck in your current Playwright execution pipeline today? Is it test flakiness, execution time, or test data management?"
