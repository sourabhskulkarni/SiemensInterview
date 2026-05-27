# Section 3: Playwright Deep Questions

## 9. Why Playwright over Selenium?
**Question:** List advantages of Playwright.

**Practical Explanation:**
In an interview, you'd explain: "I migrated to Playwright because of its native auto-waiting. In Selenium, I had to write `WebDriverWait` for every element. Playwright waits for elements to be actionable (visible, enabled, stable) automatically before clicking. It also runs completely isolated Browser Contexts in milliseconds, allowing lightning-fast parallel test execution without spinning up heavy JVM instances."
*Cross Question Answers:* Selenium is still useful for legacy browsers like IE11 or native mobile apps (via Appium). Migration challenges include rewriting XPath to CSS/Text locators and shifting from synchronous Java/Python logic to asynchronous `async/await` JavaScript logic.

---

## 10. BrowserContext Example
**Question:** Explain `Browser` vs `Context` vs `Page`.

**Practical Snippet & Answer:**
```javascript
// Line 1: Launches the heavy browser process (Chrome/Edge). You only do this once.
const browser = await chromium.launch();

// Line 2: Creates an isolated incognito session (Context) instantly. No shared cookies.
const user1Context = await browser.newContext();
const user2Context = await browser.newContext(); 

// Line 3: Creates a tab inside that context.
const pageForUser1 = await user1Context.newPage();
const pageForUser2 = await user2Context.newPage();
```

**Why & How it aligns with the question:**
This proves you understand Playwright's architecture. Instead of launching 10 heavy browsers for 10 tests, Playwright launches 1 browser and 10 fast, lightweight contexts.
*Cross Question Answers:* Multi-user testing is incredibly easy. `user1Context` logs in as Admin, `user2Context` logs in as User. They do not share cookies, so you can test real-time chat or multi-role workflows in the exact same test file.

---

## 11. Network Interception
**Question:** Intercept and mock `/api/users`.

**Practical Snippet & Answer:**
```javascript
// Line 1: Use page.route to intercept any outgoing network request matching the URL
await page.route('**/api/users', async route => {
    // Line 2: Fulfill (mock) the request with a fake 200 OK response and custom JSON body
    await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({ name: 'MockUser' })
    });
});
```

**Why & How it aligns with the question:**
This demonstrates "Shift-Left" testing. We don't need a real backend to test if the UI renders user data correctly.
*Cross Question Answers:* `route.fulfill` completely blocks the request from hitting the server and returns fake data. `route.continue` allows the request to actually go to the real server (useful if you just want to log it or add an auth header). This is heavily used to simulate offline modes or 500 server errors.

---

## 12. Dynamic Table Handling
**Question:** Parse a table and find a "Failed" row.

**Practical Snippet & Answer:**
```javascript
// Line 1: Get all rows in the table
const rows = page.locator('table tr');

// Line 2: Count the rows and loop through them
const rowCount = await rows.count();
for(let i = 0; i < rowCount; i++) {
    const row = rows.nth(i); // Line 3: Target the specific row at index i
    
    // Line 4: Check if the text inside this row includes 'Failed'
    if(await row.textContent() === 'Failed') {
        console.log(`Found failed item at row ${i}`);
        // Can also do: await row.locator('button').click() to click action on that row
    }
}
```

**Why & How it aligns with the question:**
Handling dynamic data grids is a daily SDET task. By using `.nth(i)` we iterate through elements dynamically.
*Cross Question Answers:* If a table has pagination, you would wrap this in a `while` loop that clicks the "Next Page" button until the element is found or the Next button is disabled.

---

## 13. Calendar Handling
**Practical Snippet & Answer:**
```javascript
// Line 1: Open the calendar widget
await page.locator('#date-picker-icon').click();

// Line 2: Look for the exact text '15' inside the calendar popup and click it
await page.getByRole('button', { name: '15', exact: true }).click();
```

**Why & How it aligns with the question:**
This is the simplest way to handle standard Date Pickers. 
*Cross Question Answers:* Timezone issues occur because Playwright runs in UTC on CI/CD but locally on IST. Fix this by passing `timezoneId: 'Asia/Kolkata'` in the Playwright config. For complex React calendars, do not click dates; instead, try injecting the date string directly via `page.fill('#date', '2026-08-03')` or `evaluate()` for robustness.
