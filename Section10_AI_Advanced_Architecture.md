# Section 10: Agentic AI & Advanced QE Architecture

## 29. AI-Assisted Automation (MCP CLI / LLM)
**Question:** You built an AI-assisted test generation framework using MCP CLI. How do you handle security protocols and data privacy when passing context to an LLM?

**Answer & Explanation:**
This is your flagship achievement on your CV. You must own this confidently.
**Practical Approach:**
"I built a CLI tool using the Model Context Protocol (MCP) to bridge our internal codebase with LLMs like GitHub Copilot. To ensure data privacy, the CLI strips out all PII (Personally Identifiable Information), API keys, and sensitive business logic before creating the LLM prompt. It only passes the DOM structure or Swagger API contracts to the LLM. The LLM then generates the Playwright boilerplate. This reduced our team's scripting dependency by 40%, as they no longer had to manually write tedious locators and basic assertions."

*Cross Question Answers:* Hallucination mitigation: LLMs often invent locators that don't exist. We mitigate this by having the AI generate code that is immediately passed through an automated dry-run. If Playwright throws a "Locator not found" error, the CLI feeds the error back to the LLM for self-correction before presenting the final code to the SDET for review.

---

## 30. Azure DevOps (ADO) & DB Layers
**Question:** Compare GitLab CI (JD requirement) with Azure DevOps CI/CD (CV experience). How do you validate DB layers (SQL) in Playwright tests?

**Answer & Explanation:**
**GitLab CI vs ADO:** "Conceptually, they are identical. Both use YAML to define stages and jobs. Where ADO uses `azure-pipelines.yml`, GitLab uses `.gitlab-ci.yml`. My experience scaling pipelines, configuring Docker images, and setting up parallel sharding in ADO translates 1:1 to GitLab."

**Database Testing Snippet:**
```javascript
const { test, expect } = require('@playwright/test');
const { Pool } = require('pg'); // PostgreSQL client for Node.js

test('Validate DB Layer', async ({ page }) => {
    // 1. Create user via UI
    await page.goto('/register');
    await page.fill('#email', 'test@example.com');
    await page.click('#submit');

    // 2. Validate exact record insertion in the Database
    const pool = new Pool({ connectionString: process.env.DB_URL });
    const result = await pool.query("SELECT * FROM users WHERE email = 'test@example.com'");
    
    expect(result.rows.length).toBe(1);
    expect(result.rows[0].status).toBe('active');
    
    await pool.end();
});
```

*Cross Question Answers:* Managing connection pools: Opening a DB connection takes time. In a real framework, you initialize the `Pool` once in a `globalSetup` file and reuse it across tests, rather than opening/closing it in every single test. For ADO/GitLab CI, you pass the `DB_URL` as a masked CI/CD Secret Variable so it is never hardcoded.

---

## 31. Runtime AI Generation for API & K6 (The Master Feature)
**Question:** Explain how your MCP CLI framework dynamically manages and updates API and Performance (K6) scripts during UI execution without creating duplicate code.

**Concept Used:** Network Traffic Sniffing, AST (Abstract Syntax Tree) Comparison, Agentic AI Code Generation.

**Story of the Problem:**
"In modern microservices, APIs change constantly. Maintaining separate UI, API, and Performance scripts is a massive overhead. Often, a developer adds a new payload field; the UI test still passes, but the separate API and K6 load tests fail because they are out of sync. I wanted to eliminate this maintenance nightmare."

**Solution & Practical Snippet:**
"To solve this, I engineered a framework that listens to all network traffic during Playwright UI execution. Instead of running API/K6 scripts separately, my MCP + CLI tool captures the real-time API payloads triggered by the UI. 

Instead of blindly generating *new* API/K6 scripts (which creates massive duplication), the tool smartly compares the runtime network traffic against our existing script repository. If it detects that an API module was merged, a field was added, or an endpoint changed, it flags the exact difference. The AI then automatically proposes a code diff to update those specific K6 and API test features. It presents this diff to me. Once I review and approve it, the framework automatically patches and corrects the existing scripts."

```javascript
// High-level conceptual snippet of what your framework does under the hood
test('Listen to UI and validate/update API scripts', async ({ page }) => {
    // 1. Framework listens to all responses during UI execution
    page.on('response', async response => {
        if (response.url().includes('/api/')) {
            const requestPayload = await response.request().postDataJSON();
            
            // 2. Pass payload to MCP CLI to compare with existing K6/API scripts
            const diff = await mcpCli.compareAndProposeUpdates(response.url(), requestPayload);
            
            // 3. If diff found, flag for manual review
            if (diff.hasChanges) {
                console.log(`[ACTION REQUIRED] API changed. Proposed updates for K6/API scripts generated.`);
                mcpCli.saveProposedDiffForReview(diff);
            }
        }
    });

    // Proceed with normal UI flow...
    await page.goto('/checkout');
    await page.click('button#pay');
});
```

**Impact & Why it matters:**
"This completely changed our workflow. 
1. **Zero Duplicate Code:** We never generate redundant scripts.
2. **100% Synchronization:** Our UI, API, and Performance layers are never out of sync.
3. **Massive ROI:** It shifted script maintenance from a manual, tedious coding task to a simple 'Review and Approve' click, drastically reducing QE effort."

*Cross Question Answers:* If they ask "How does the AI know *where* to update the script?", you answer: "My CLI parses our existing API/K6 codebase into an Abstract Syntax Tree (AST) to understand the code structure, allowing the AI to safely pinpoint the exact variable or payload object to update without breaking the surrounding logic."
