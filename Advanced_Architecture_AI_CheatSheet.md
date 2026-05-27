# Advanced Architecture & AI Cheat Sheet

A quick reference for the senior-level, architectural concepts from Sections 8, 9, and 10 (Agile, NFR, DB Testing, and AI).

---

## 1. Agile Strategy & Shift-Left Testing
**Concept:** Catch bugs before they merge, not after they deploy.
* **Pre-commit Hooks:** Use Husky to run unit tests and linters locally before a dev can `git commit`.
* **PR Quality Gates:** Azure DevOps / GitLab pipelines block code from merging into `main` if Playwright smoke tests fail.
* **JIRA Dashboards:** Use APIs to automatically post Playwright HTML reports to JIRA Xray/Zephyr for real-time Go/No-Go decisions.

---

## 2. Root Cause Analysis (GCP Logs)
**Concept:** Don't just report "API Failed." Tell the developer *why* it failed.
* **Traceability:** Inject an `X-Correlation-ID` into your Playwright API headers.
* **Execution:** If a test fails, search that ID in GCP Logs Explorer.
* **Value:** Immediately proves if an issue is Frontend (UI parsing) or Backend (Microservice crash), drastically reducing debugging time and helping meet 48-hour SLAs.

---

## 3. NFR & Performance Testing (K6)
**Concept:** Playwright is for functional testing; K6 is for load testing. Combine them for power.
```javascript
// Generate a HAR file of the UI journey in Playwright
const context = await browser.newContext({ recordHar: { path: 'flow.har' }});
const page = await context.newPage();
await page.goto('/checkout');
await context.close();
```
* **Process:** Use the `har-to-k6` tool to convert the HAR file into a K6 load testing script automatically. Run K6 to hit endpoints with 1,000 concurrent users. Fail the CI pipeline if `p(95) > 500ms`.

---

## 4. Database Validations (SQL)
**Concept:** Validate that UI/API actions actually wrote the correct data to the database.
```javascript
const { Pool } = require('pg'); // PostgreSQL client

test('Validate DB Insertion', async ({ page }) => {
    const pool = new Pool({ connectionString: process.env.DB_URL });
    const result = await pool.query("SELECT * FROM users WHERE email='test@test.com'");
    
    expect(result.rows.length).toBe(1); // Ensure record exists
    await pool.end(); // Always close connection!
});
```

---

## 5. Agentic AI (MCP CLI & LLM)
**Concept:** Using AI to generate Playwright scripts and reduce manual coding effort by 40%.
* **Security & Privacy:** The MCP CLI strips out PII and sensitive logic *before* sending the HTML DOM / API Swagger doc to the LLM.
* **Hallucination Mitigation:** Pass the AI-generated code through an automated dry-run. If it fails, feed the error back to the LLM to self-correct.

---

## 6. Runtime AI Script Synchronization (The Master Feature)
**Concept:** Keeping UI, API, and K6 scripts perfectly in sync automatically.
* **The Mechanism:** During Playwright UI execution, the framework sniffs network API traffic. 
* **The Smart Comparison:** The MCP CLI compares runtime payloads against your existing API/K6 script repository (using AST parsing).
* **The Action:** Instead of creating duplicate scripts, if it detects a modified endpoint or new payload field, it flags the difference and proposes an update diff.
* **The Impact:** You just hit "Review and Approve." The existing scripts are patched automatically. Zero test rot, zero duplicates, massive maintenance reduction.
