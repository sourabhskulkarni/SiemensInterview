# Study Plan: Playwright for End-to-End Testing

Playwright is a modern framework for robust E2E testing. Here is how to explain its core concepts.

---

## 1. Page Object Model (POM)
**Concept Used:** Design Patterns, Object-Oriented Programming (Encapsulation), DRY (Don't Repeat Yourself).

**Context Story:**
As a framework scales to hundreds of tests, UI locators change frequently. If you hardcode `page.locator('#username')` in 50 tests, a single UI change will break all 50. By creating a `LoginPage` class, you store locators and actions in one place. If the UI changes, you update the POM, and all 50 tests are automatically fixed.

**Snippet:**
```javascript
class LoginPage {
    constructor(page) {
        this.page = page;
        this.usernameInput = page.locator('#username');
        this.passwordInput = page.locator('#password');
        this.loginBtn = page.locator('button[type="submit"]');
    }

    async login(username, password) {
        await this.usernameInput.fill(username);
        await this.passwordInput.fill(password);
        await this.loginBtn.click();
    }
}
```

**How to Explain Code in Interview:**
"This is a standard Page Object class. In the `constructor`, I define all the locators specific to the login page using Playwright's `locator()` method. Then, I expose an action method called `login()` which performs the sequential actions. This abstracts the UI implementation details away from the actual test file, making the test read more like business logic."

**Questions that might be asked:**
- "How do you handle dynamic locators in a Page Object?"
- "Is it a good idea to put assertions (like `expect`) inside the Page Object class?" *(Answer: No, assertions belong in the test file so the POM remains reusable).*
- "How do you pass the `page` instance to your Page Objects across multiple tests?"

---

## 2. Custom Fixtures
**Concept Used:** Dependency Injection, Test Setup/Teardown management.

**Context Story:**
In traditional frameworks (like Selenium/Mocha), we use `beforeEach` and `afterEach` hooks to initialize things like Page Objects. But this gets messy when you have dozens of Page Objects. Playwright Fixtures solve this by injecting the required Page Object directly into the test only when the test asks for it.

**Snippet:**
```javascript
const { test: base } = require('@playwright/test');
const { LoginPage } = require('./LoginPage');

// Defining a custom fixture
const test = base.extend({
    loginPage: async ({ page }, use) => {
        // Setup phase
        const loginPage = new LoginPage(page);
        await loginPage.goto();
        
        // Pass to the test
        await use(loginPage);
        
        // Teardown phase (if needed)
    }
});

// Using the fixture
test('Should login successfully', async ({ loginPage }) => {
    await loginPage.login('user', 'pass'); // loginPage is automatically instantiated!
});
```

**How to Explain Code in Interview:**
"Here, I'm extending Playwright's base `test` object to create a custom `loginPage` fixture. The magic happens with the `use` function. Everything before `use()` is setup code, and anything after is teardown. In the test itself, I just destructure `{ loginPage }` in the arguments, and Playwright automatically injects the fully instantiated Page Object. It's much cleaner than using `beforeEach`."

**Questions that might be asked:**
- "Why are Playwright Fixtures better than `beforeAll` or `beforeEach` hooks?"
- "Can fixtures depend on other fixtures?"
- "How do fixtures help with parallel test execution?" *(Answer: Fixtures provide isolated environments for each test).*
