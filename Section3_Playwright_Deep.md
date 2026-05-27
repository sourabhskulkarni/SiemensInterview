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

---

## 13.1. Standard Select Dropdown vs Custom UI Dropdown
**Question:** How do you handle `<select>` dropdowns versus modern custom `<div>` dropdowns?

**Practical Snippet & Answer:**
```javascript
// 1. Standard HTML <select> Dropdown (Simple)
await page.locator('#country').selectOption({ label: 'India' });
// or by value: await page.locator('#country').selectOption('IND');

// 2. Custom UI Dropdown (React/Angular Custom Divs)
await page.locator('#custom-dropdown-trigger').click(); // Open the menu
await page.getByRole('option', { name: 'Pune' }).click(); // Select the item
```

**Why & How it aligns with the question:**
Interviews often trick you by asking how to select a dropdown, expecting you to know the difference between native `<select>` tags (which use `.selectOption()`) and modern web frameworks that use hidden divs (which require a `.click()` to open and another `.click()` to select).

---

## 13.2. Multiselect Checkbox Handling
**Question:** How do you select multiple checkboxes dynamically from a list?

**Practical Snippet & Answer:**
```javascript
const rolesToSelect = ['Admin', 'Editor', 'Viewer'];

for (const role of rolesToSelect) {
    // Locate the checkbox by its associated label text and check it
    await page.getByLabel(role).check();
}

// To verify they are checked:
for (const role of rolesToSelect) {
    await expect(page.getByLabel(role)).toBeChecked();
}
```

**Why & How it aligns with the question:**
Shows you know how to iterate over an array of data to dynamically interact with UI elements. `.check()` is preferred over `.click()` for checkboxes because `.check()` ensures the box is checked regardless of its current state, whereas `.click()` might uncheck it if it was already checked!

---

## 13.3. Multiselect Textbox (Location/Tags Filter)
**Question:** How do you automate a multiselect textbox where you type to search, select an option, and it appears as a tag (e.g., Location filters)?

**Practical Snippet & Answer:**
```javascript
const locations = ['Pune', 'Mumbai', 'Bangalore'];

for (const loc of locations) {
    // 1. Type the location into the search box
    await page.getByPlaceholder('Search locations...').fill(loc);
    
    // 2. Wait for the auto-suggest dropdown to appear and click the exact match
    await page.getByRole('listbox').getByText(loc, { exact: true }).click();
}

// 3. Verify all selected tags appear in the filter box
for (const loc of locations) {
    await expect(page.locator('.selected-tags-container')).toContainText(loc);
}
```

**Why & How it aligns with the question:**
This is a very common scenario in complex dashboards. It demonstrates understanding of typing, waiting for asynchronous dropdowns (auto-suggest), and verifying state changes in a DOM container.

---

## 13.4. File Uploads (Standard vs Hidden)
**Question:** How do you handle file uploads, especially if the `<input type="file">` is hidden by a custom UI button?

**Practical Snippet & Answer:**
```javascript
// 1. Standard approach (if input is visible)
await page.locator('input[type="file"]').setInputFiles('path/to/test.pdf');

// 2. Advanced approach (if input is hidden or triggered dynamically)
const [fileChooser] = await Promise.all([
    page.waitForEvent('filechooser'),
    page.getByRole('button', { name: 'Upload Document' }).click() // Triggers the file chooser
]);
await fileChooser.setFiles('path/to/test.pdf');
```
**Why & How it aligns with the question:**
Interviewers know that custom UI frameworks often hide the actual file input. Attempting to click a hidden input fails. The `Promise.all` approach using `waitForEvent('filechooser')` is the enterprise standard to handle this elegantly.

---

## 13.5. Handling JS Alerts & Dialogs
**Question:** Your test clicks a 'Delete' button and a browser confirmation alert appears. How does Playwright handle this?

**Practical Snippet & Answer:**
```javascript
// Playwright automatically dismisses alerts! You MUST attach a listener to accept it.
page.on('dialog', async dialog => {
    console.log(`Alert message was: ${dialog.message()}`);
    await dialog.accept(); // Or dialog.dismiss()
});

await page.getByRole('button', { name: 'Delete User' }).click(); // This triggers the alert
```
**Why & How it aligns with the question:**
This is a classic trap question. If a candidate says "I wait for the alert to appear and click OK," they are thinking in Selenium. Playwright auto-dismisses dialogs by default unless explicitly caught by the `page.on('dialog')` event listener.

---

## 13.6. Browser Pop-ups (New Tabs / Windows)
**Question:** Clicking a link opens a PDF report in a completely new browser tab. How do you validate the contents of that new tab?

**Practical Snippet & Answer:**
```javascript
// Start waiting for the new page BEFORE clicking the link
const [newPage] = await Promise.all([
    context.waitForEvent('page'),
    page.getByRole('link', { name: 'Download PDF Report' }).click()
]);

// Wait for the new tab to fully load
await newPage.waitForLoadState();

// Validate the new tab
await expect(newPage).toHaveTitle('Monthly Report');
await newPage.getByRole('button', { name: 'Close' }).click();

// Switch back to original page
await page.bringToFront();
```
**Why & How it aligns with the question:**
This tests your understanding of concurrency (`Promise.all`) and Playwright's `BrowserContext`. You must capture the new page instance as it's spawned to interact with it, rather than trying to use `page.locator()` which only looks at the original tab.

---

## 13.7. Shadow DOM Locators
**Question:** The element you are trying to click is inside a Web Component with a closed/open Shadow Root. How do you locate it?

**Practical Snippet & Answer:**
```javascript
// Playwright pierces open Shadow DOMs automatically!
// You don't need any special syntax for open shadow roots.
await page.getByRole('button', { name: 'Submit' }).click();

// If you MUST target something explicitly inside a shadow root using CSS:
await page.locator('.custom-web-component >> css=.internal-button').click();
```
**Why & How it aligns with the question:**
Another trap! Selenium requires complex JavaScript executors (`executeScript`) to pierce Shadow DOMs. Playwright's engine natively pierces open Shadow DOMs. You just write your locator as if the shadow root doesn't exist. This is a huge selling point for why Playwright is superior for modern web apps like Salesforce (LWC).
