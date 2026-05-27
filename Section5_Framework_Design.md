# Section 5: Framework Design

## 19. Enterprise Framework Structure
**Question:** Explain your folder structure.

**Answer & Explanation:**
```text
src/
 ├── pages/      # Page Object Classes (Locators and Action Methods)
 ├── api/        # API helper classes (Login, Create Order methods)
 ├── fixtures/   # Playwright custom fixtures extending the base test
 ├── utils/      # Helpers (Date formatters, random string generators)
 ├── data/       # Test data (JSON files, environment config data)
 ├── tests/      # The actual .spec.ts files
playwright.config.ts # Global configuration (Timeouts, Browsers, Workers)
```

**Why & How it aligns with the question:**
This shows a scalable, modular architecture. 
*Cross Question Answers:* Fixtures are used for Dependency Injection (providing `loginPage` directly to tests without using `beforeEach`). Environment management is handled by creating a `.env` file and using the `dotenv` npm package to load URLs and credentials securely without hardcoding them in the repo.

---

## 20. Reduce Flaky Tests
**Question:** How do you handle flakiness?

**Answer & Practical Strategies:**
1. **Stable Locators:** Never use XPath like `/div[2]/span[1]`. Use Playwright's `getByRole` or `getByTestId`.
2. **Auto-waiting:** Do not use `page.waitForTimeout(5000)`. Rely on Playwright's native auto-wait or `waitForResponse()` for network calls.
3. **API Setup:** Setup test data via API instead of UI to reduce the points of failure.

**Why & How it aligns with the question:**
Flaky tests destroy trust in automation.
*Cross Question Answers:* Using retry mechanisms (e.g., `retries: 2` in config) is an anti-pattern if used as a permanent fix. It masks the root cause of the flakiness. Retry should be a safety net for network blips on CI, but the test itself should be fixed.
