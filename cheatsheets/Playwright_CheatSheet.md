# SDET Master Cheat Sheet: Playwright, API, CI/CD, Docker & Git

A condensed, quick-reference guide for your technical round.

---

## 1. Playwright Core
**Architecture:** `Browser` (heavy) > `Context` (isolated session/cookies) > `Page` (tab).
**Locators:** Prefer user-facing (`getByRole`, `getByText`). Avoid XPath.
**Auto-Waiting:** Playwright automatically waits for elements to be visible/stable before interacting.

```javascript
// Setup & Basic Actions
const context = await browser.newContext();
const page = await context.newPage();

await page.goto('https://app.com');
await page.getByRole('button', { name: 'Submit' }).click(); // Auto-waits
await page.locator('#item').dragTo(page.locator('#target'));

// Assertions (Auto-retrying)
await expect(page.locator('.success')).toBeVisible();
await expect(page.locator('#email')).toHaveValue('test@test.com');
```

---

## 2. API Automation (Deeper Integration)
**Concept:** Use API to bypass UI for faster test execution, mocking, and state setup.

```javascript
// 1. Direct API Calls (request context)
const response = await request.post('/api/users', { data: { name: 'Sourabh' } });
expect(response.status()).toBe(201);
const body = await response.json();

// 2. Network Interception & Mocking (Shift-Left)
// Block real request, return fake 500 error to test UI handling
await page.route('**/api/payments', route => route.fulfill({ status: 500 }));

// 3. Waiting for Network in UI Tests (Avoid Flakiness)
const [res] = await Promise.all([
  page.waitForResponse(r => r.url().includes('/cart') && r.status() === 200),
  page.getByText('Add to Cart').click()
]);

// 4. Auth State Rehydration (Global Setup)
await context.storageState({ path: 'auth.json' }); // Save login session
// Use in config: `use: { storageState: 'auth.json' }`
```

---

## 3. Git & Version Control
**Concept:** Manage source code, integrate feature branches cleanly.

```bash
git fetch            # Download changes from remote, DO NOT merge
git pull             # Download AND merge changes from remote
git stash            # Temporarily save uncommitted changes

# Rebase vs Merge
git merge main       # Combines branches (creates a merge commit, messy history)
git rebase main      # Applies your commits ON TOP of main (linear, clean history)

# Conflict Resolution
# Fix markers (<<<< HEAD) in file -> git add . -> git rebase --continue
```

---

## 4. CI/CD & YAML (GitLab)
**Concept:** Automate test execution on code commits using YAML configuration.

```yaml
# .gitlab-ci.yml
stages:
  - test                  # Define pipeline stages

e2e_tests:                # Job name
  stage: test
  image: mcr.microsoft.com/playwright:v1.40.0-jammy # Playwright Docker image
  script:
    - npm ci              # Clean install dependencies (Reads package-lock.json strictly)
    - npx playwright test # Execute tests
  artifacts:              # Save output (reports) even if tests fail
    when: always
    paths:
      - playwright-report/
```

**Sharding:** Splitting tests across multiple runners to reduce execution time.
`npx playwright test --shard=1/3` (Runs 1st third of tests)

---

## 5. Docker (Containerization)
**Concept:** Package tests and dependencies into a standard unit so they run identically everywhere ("Works on my machine" -> "Works everywhere").

```dockerfile
# Dockerfile
# 1. Use official image (pre-installed browsers + OS dependencies)
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

WORKDIR /app

# 2. Copy package.json and install (Leverages Docker layer caching)
COPY package*.json ./
RUN npm ci

# 3. Copy framework code
COPY . .

# 4. Default command to execute
CMD ["npx", "playwright", "test"]
```
