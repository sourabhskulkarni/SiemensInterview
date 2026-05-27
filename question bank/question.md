Complete Siemens Playwright SDET Interview Preparation Guide
This document combines realistic Siemens Playwright SDET interview questions, coding programs, API automation scenarios, Playwright deep concepts, Git/GitLab CI/CD, Docker, framework architecture, and cross questions based on Siemens interview patterns and the provided JD.
1. JavaScript / TypeScript Coding Questions
1.	Reverse words while preserving special characters
Input:
"I.am@Sourabh"

Output:
"b.hru@oamruS"
Cross Questions:
•	Time complexity?
•	Can you solve without extra array?
•	How to optimize for huge strings?
2.	Find last repeating character
Input:
"automation"

Output:
"t"
Cross Questions:
•	Map vs Object?
•	Case-insensitive handling?
3.	Flatten nested array
Input:
[1,[2,[3,4]],5]

Output:
[1,2,3,4,5]
Cross Questions:
•	Recursive vs iterative solution?
•	Difference from flat()?
4.	Promise Output Prediction
console.log(1);

setTimeout(() => console.log(2));

Promise.resolve().then(() => console.log(3));

console.log(4);

// Output:
// 1
// 4
// 3
// 2
Cross Questions:
•	Explain Event Loop
•	Microtask queue vs callback queue?
5.	Implement debounce function
function debounce(fn, delay) {
  let timer;

  return function(...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      fn(...args);
    }, delay);
  };
}
Cross Questions:
•	Debounce vs throttle?
•	Automation use cases?

2. Advanced JavaScript / TypeScript
6.	Difference between var / let / const
var => function scoped
let => block scoped
const => block scoped and immutable reference
Cross Questions:
•	Hoisting?
•	Temporal Dead Zone?
7.	Interface vs Type in TypeScript
interface User {
  name: string;
}

type Employee = {
  id: number;
}
Cross Questions:
•	Declaration merging?
•	Extends vs intersection?
8.	Abstract Class vs Interface
abstract class LoginPage {
  abstract login();
}
Cross Questions:
•	Can interface have implementation?
•	Multiple inheritance?

3. Playwright Deep Questions
9.	Why Playwright over Selenium?
Advantages:
- Auto waiting
- Faster execution
- BrowserContext isolation
- Better parallel execution
Cross Questions:
•	When Selenium still useful?
•	Migration challenges?
10.	BrowserContext Example
const browser = await chromium.launch();

const context = await browser.newContext();

const page = await context.newPage();
Cross Questions:
•	Difference Browser vs Context vs Page?
•	Multi-user testing?
11.	Network Interception
await page.route('**/api/users', route => {
  route.fulfill({
    status: 200,
    body: JSON.stringify({name:'MockUser'})
  });
});
Cross Questions:
•	route.fulfill vs continue?
•	Offline simulation?
12.	Dynamic Table Handling
const rows = page.locator('table tr');

for(let i=0;i<await rows.count();i++) {
  const row = rows.nth(i);

  if(await row.textContent().includes('Failed')) {
    console.log(await row.textContent());
  }
}
Cross Questions:
•	Pagination handling?
•	Virtual scrolling tables?
13.	Calendar Handling
await page.locator('#date').click();

await page.getByText('15').click();
Cross Questions:
•	Timezone issues?
•	React calendar handling?

4. API Automation through Code
14.	POST API Request
const response = await request.post('/users', {
  data: {
    name: 'Sourabh'
  }
});
Cross Questions:
•	Difference body vs data?
•	How validate response?
15.	Token Authentication
const login = await request.post('/login', {
  data: {
    username: 'admin',
    password: 'admin123'
  }
});

const token = (await login.json()).token;
Cross Questions:
•	JWT vs OAuth?
•	Refresh token handling?
16.	Chained API Calls
const user = await request.post('/users');

const userId = (await user.json()).id;

await request.get(`/users/${userId}`);
Cross Questions:
•	Dependency failures?
•	Dynamic payload generation?
17.	API Mocking
await page.route('**/api/orders', route => {
  route.fulfill({
    status: 200,
    body: JSON.stringify({status:'success'})
  });
});
Cross Questions:
•	Why API mocking?
•	CI stability benefits?
18.	API + UI Hybrid Validation
Create order via API

Validate order via UI
Cross Questions:
•	Why hybrid approach?
•	Execution speed advantage?

5. Framework Design
19.	Enterprise Framework Structure
src/
 pages/
 api/
 fixtures/
 utils/
 data/
 hooks/
 reports/
Cross Questions:
•	Why fixtures?
•	Environment management?
20.	Reduce Flaky Tests
Strategies:
- Stable locators
- Auto waiting
- API mocking
- Retry mechanism
Cross Questions:
•	Retry anti-pattern?
•	CI instability handling?

6. Git / GitLab CI/CD / Docker
21.	Git Commands
git pull
git fetch
git rebase
git stash
git cherry-pick
Cross Questions:
•	merge vs rebase?
•	Conflict resolution?
22.	GitLab CI Pipeline
stages:
  - test

playwright-test:
  stage: test
  script:
    - npm install
    - npx playwright test
Cross Questions:
•	Artifacts?
•	Parallel jobs?
23.	Docker for Playwright
FROM mcr.microsoft.com/playwright:v1.52.0-jammy

WORKDIR /app

COPY . .

RUN npm install

CMD ["npx","playwright","test"]
Cross Questions:
•	Why Docker?
•	Browser dependency management?

7. Scenario-Based Questions
24.	Lift Synchronization Testing
Test Areas:
- Concurrent requests
- Emergency handling
- Floor synchronization
- Overload handling
Cross Questions:
•	How automate?
•	Race condition validation?
25.	CAPTCHA Automation
Expected Approach:
- Avoid real captcha automation
- Disable in lower environment
- Token injection
Cross Questions:
•	OCR limitations?
•	Security concerns?

8. Agile, Test Management & Strategy
26.	Agile Methodologies & Shift-Left
Question: How do you embed shift-left quality gates into Agile sprints?
Cross Questions:
•	JIRA/ALM integration with ADO pipelines?
•	Risk-based go/no-go decisions?
27.	Defect Lifecycle & Root Cause
Question: Explain how you leveraged GCP log monitoring for defect root cause analysis.
Cross Questions:
•	Differentiating frontend vs microservices backend issues?
•	Handling 48-hour SLA resolution targets?

9. Performance & NFR Testing
28.	NFR Testing Integration
Question: The JD mentions K6 and Sitespeed.io. How would you integrate K6 with a Playwright framework for API performance testing?
Cross Questions:
•	Generating K6 scripts from Playwright HAR files?
•	Defining NFR benchmarks in CI/CD?

10. Agentic AI & Advanced QE Architecture
29.	AI-Assisted Automation (MCP CLI / LLM)
Question: You built an AI-assisted test generation framework using MCP CLI. How do you handle security protocols and data privacy when passing context to an LLM?
Cross Questions:
•	How does it reduce scripting dependency by 40%?
•	Hallucination mitigation in test generation?
30.	Azure DevOps (ADO) & DB Layers
Question: Compare GitLab CI (JD requirement) with Azure DevOps CI/CD (CV experience). How do you validate DB layers (SQL) in Playwright tests?
Cross Questions:
•	Managing DB connection pools in Node.js?
•	Seeding/teardown data securely in ADO pipelines?

11. Final Preparation Guidance
If you thoroughly prepare all questions in this document, understand the coding logic, practice explaining your approach, and can answer the cross questions confidently, you will be strongly prepared for the Siemens Playwright SDET technical round.
Most important focus areas: JavaScript async concepts, Playwright architecture, API automation through code, framework design, CI/CD (GitLab & ADO), Docker basics, AI automation claims, and real-time automation scenarios.
For your profile level, expect deep cross-questioning on framework scalability, CI/CD integration, API automation design, flaky test reduction, and your Agentic AI integration.
