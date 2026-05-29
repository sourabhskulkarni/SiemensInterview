# Section 2: Advanced JavaScript / TypeScript

## 6. Difference between var / let / const
**Question:** Explain the difference between var, let, and const.

**Practical Snippet & Answer:**
```javascript
function scopeTest() {
    // Line 1: 'var' is hoisted and accessible anywhere inside this function
    var functionScoped = "I am visible everywhere in scopeTest";
    
    if (true) {
        // Line 2: 'let' is block-scoped. Only visible inside this if-block.
        let blockScoped = "I am only visible in this IF block";
        
        // Line 3: 'const' is block-scoped AND cannot be reassigned.
        const constantValue = 10;
        // constantValue = 20; // ERROR: Assignment to constant variable.
    }
    // console.log(blockScoped); // ERROR: blockScoped is not defined here
}
```

**Why & How it aligns with the question:**
SDETs must know how variables live in memory. Using `var` in modern JS causes bugs due to scope leakage.
*Cross Question Answers:* 
- **Hoisting:** `var` is hoisted and initialized as `undefined`. `let` and `const` are hoisted but NOT initialized, placing them in the **Temporal Dead Zone (TDZ)** until the code execution reaches their declaration line.

---

## 7. Interface vs Type in TypeScript
**Question:** Compare `interface User` vs `type Employee`.

**Practical Snippet & Answer:**
```typescript
// Line 1: Interface is ideal for defining object shapes (like Page Objects or API Responses)
interface User {
    name: string;
}

// Line 2: Interfaces can be merged (Declaration Merging)
interface User {
    age: number; // User now has name AND age automatically
}

// Line 3: Type aliases are better for primitives, unions, and tuples.
type Status = "Success" | "Failed"; // Cannot do this with interface

type Employee = {
    id: number;
}
// type Employee = { role: string } // ERROR: Duplicate identifier
```

**Why & How it aligns with the question:**
TypeScript strictly types automation frameworks. Knowing when to use which is an architectural decision.
*Cross Question Answers:* `interface` supports declaration merging (adding fields later), `type` does not. `interface` uses `extends` to inherit, while `type` uses intersection (`&`). Generally, prefer `interface` for objects and `type` for everything else.

---

## 8. Type vs Interface vs Abstract Class (The Practical Framework Approach)
**Question:** Explain the difference between `type`, `interface`, and `abstract class`. How do you use them practically in your framework?

**The Simple Definitions First:**
> "To put it simply, here is how I differentiate them:
> 1. **`type`**: This is just a custom alias for specific, exact values. It’s like a strict drop-down menu of choices.
> 2. **`interface`**: This is just a blueprint or a strict checklist. It tells you *what* names or properties must exist, but has zero actual logic or code inside it.
> 3. **`abstract class`**: This is a base template that *does* have actual, reusable code inside it, but you can't use it directly on its own—you have to extend it."

**Practical Snippet & Answer (The Framework Context):**
> "In my framework, I don't just use these randomly. I have a very specific architectural rule for when to use each:"

```typescript
// 1. TYPE: Used for strict literal selections (e.g., Browser Configuration)
// I use types to restrict engineers to specific browser values so they can't misspell it in the config.
type BrowserChoice = 'chromium' | 'firefox' | 'webkit';

// 2. INTERFACE: Used as a contract for defining Locators/Selectors
// I use interfaces to enforce that every Page Object must define its locators, but without any actual logic.
interface ILoginPageLocators {
    readonly usernameInput: string;
    readonly passwordInput: string;
    readonly loginBtn: string;
}

// 3. ABSTRACT CLASS: Used for the BasePage to hold runtime generic logic
// I use an abstract class for the BasePage. It provides actual reusable implementation (like clickElement), 
// but forces child pages to implement their own specific logic (like verifyPageLoaded).
abstract class BasePage {
    // Concrete method with logic (Interfaces can't do this)
    async clickElement(selector: string) {
        console.log(`Waiting and clicking element: ${selector}`);
    }
    
    // Abstract method (Forces the child class to implement it)
    abstract verifyPageLoaded(): Promise<boolean>;
}

// 4. BRINGING IT TOGETHER in a Concrete Page Object
class LoginPage extends BasePage implements ILoginPageLocators {
    // Satisfying the Interface contract
    usernameInput = '#user';
    passwordInput = '#pass';
    loginBtn = '#login';

    // Satisfying the Abstract Class contract
    async verifyPageLoaded() {
        return true; 
    }
}
```

**Why this is a 10/10 Answer:**
You aren't just reciting definitions from MDN. You are showing them exactly how an Architect uses TypeScript features to build guardrails into a complex framework (Types for config strictness, Interfaces for locator shape, Abstract Classes for reusable runtime actions).

---

## 8.1. Architectural Alignment: TS Design in your AI Tool
**Question:** How did you use advanced TypeScript features (Interfaces and Abstract Classes) to architect your MCP CLI tool?

**Answer & Explanation:**
You want to show that your TypeScript knowledge scales beyond basic page objects.
"I used Abstract Classes and Interfaces to make the AI logic in my CLI model-agnostic (applying SOLID principles). I created an `interface ILLMProvider` and an `abstract class AIPromptBuilder` which contains the default logic for stripping PII from network payloads. 
Because of this strict architecture, if the company decides to switch from GitHub Copilot to OpenAI or Anthropic, I don't have to rewrite the core engine. I simply implement a new child class that `extends AIPromptBuilder`. This proves my TS code is highly scalable and maintainable."

---

## 8.2. Async / Await & Promise Combinators (The Practical Framework Approach)
**Question:** Playwright relies entirely on asynchronous execution. Explain what async/await/Promises are, and the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`.

**The Simple Definitions First:**
> "To explain async/await simply, I tie it directly to how Playwright executes:
> 1. **`Promise`**: This is just a background task that hasn't finished yet. For example, when an auto-save API fires after uploading a file, Playwright creates a Promise. It means 'I am waiting for the backend to respond, and I will let you know if it passes or fails.'
> 2. **`async`**: This is just a label on a function telling JavaScript, 'This function contains background tasks (Promises) that take time, like clicking or API calls.'
> 3. **`await`**: This tells Playwright to pause the test exactly at that line and wait for the background task (like an API response or file upload) to fully finish before moving to the next step."

**Practical Snippet & Answer (The Promise Combinators in Automation):**
> "In my framework, handling multiple Promises concurrently is where the real power is. Here is how and why I use the different combinators:"

```javascript
// 1. Promise.all -> "All or Nothing" (Fails fast if ANY promise rejects)
// Use Case: Clicking a button and waiting for the API response.
// In Playwright, you MUST start listening for the response at the exact same time you click the button.
// If you click first, the API might finish before you start listening, causing a timeout.
const [response] = await Promise.all([
    page.waitForResponse('**/api/submit'), // Start listening
    page.locator('#submit-btn').click()    // Perform the action
]);
// If the click fails, or the API returns an error, the block fails immediately.

// 2. Promise.allSettled -> "Finish Everything" (Waits for all, pass or fail)
// Use Case: Playwright Fixture / Hook Teardown.
// After a test, we want to completely wipe the browser state. If clearing cookies fails 
// for some reason, we STILL want it to attempt clearing local storage so the next test doesn't fail.
const teardownStatus = await Promise.allSettled([
    context.clearCookies(),
    page.evaluate(() => window.localStorage.clear()),
    page.evaluate(() => window.sessionStorage.clear())
]);

// 3. Promise.race -> "First to Finish Wins (Pass or Fail)"
// Use Case: Racing UI States (Success vs Error).
// After filling mandatory fields, the Submit button should enable. But if there's a validation issue, 
// an error toast appears. We race them to see which UI state happens first so we don't wait blindly.
const outcome = await Promise.race([
    page.waitForSelector('#submit-btn:not([disabled])'), // State 1: Button enables
    page.waitForSelector('.error-toast') // State 2: Error appears
]);

// 4. Promise.any -> "First SUCCESS Wins" (Ignores failures)
// Use Case: Optional Fields enabling a Next Button.
// The user only needs to fill ANY ONE of the 3 optional fields to enable the Next button.
// We use Promise.any to wait until the first successful input triggers the button state.
const nextButtonEnabled = await Promise.any([
    page.locator('#phone-input').fill('1234567890').then(() => page.waitForSelector('#next-btn:not([disabled])')),
    page.locator('#email-input').fill('test@test.com').then(() => page.waitForSelector('#next-btn:not([disabled])')),
    page.locator('#address-input').fill('123 Main St').then(() => page.waitForSelector('#next-btn:not([disabled])'))
]);
```

**Why this is a 10/10 Answer:**
You completely avoided dry textbook definitions like "the event loop" or "microtask queue." Instead, you immediately tied the core concepts to highly relatable, real-world framework scenarios (API network interception, racing UI button states, and resilient teardowns). This proves you don't just know the syntax—you know exactly how to use it to optimize execution time in a Senior SDET role.

---

## 8.3. Anonymous Functions & Callbacks
**Question:** What is a Callback function and an Anonymous function? How are they used in your Playwright framework?

**Practical Snippet & Answer:**
```javascript
// 1. Callback Function: A function passed as an argument to another function
function executeTest(testName, callback) {
    console.log(`Starting test: ${testName}`);
    callback(); // Executing the passed function
}

// 2. Anonymous Function: A function without a name (often used as callbacks)
// Here, the second argument () => { ... } is an anonymous arrow function acting as a callback.
executeTest('Login Scenario', () => {
    console.log('Test executed successfully!');
});

// Playwright Native Example:
// `route => { ... }` is an anonymous callback function passed into page.route()
await page.route('**/api/users', route => {
    route.fulfill({ status: 200, body: 'Mocked Data' });
});
```

**Why & How it aligns with the question:**
A Senior SDET must know that Playwright's core API relies heavily on callbacks. For example, `test('name', async ({ page }) => { ... })` uses an anonymous async callback function. 
*Cross Question Answers:* If an interviewer asks "What is Callback Hell?", you explain that before Promises and `async/await`, chaining multiple asynchronous operations required nesting callbacks inside callbacks, leading to unreadable "pyramid of doom" code. We avoid this in Playwright by using `async/await`.

---

## 8.4. Node.js Modules: CommonJS vs ES6 Imports
**Question:** Since Playwright runs on Node.js (as per the JD), explain the difference between CommonJS (`require`) and ES Modules (`import`). Which one does your framework use?

**Practical Snippet & Answer:**
```javascript
// 1. CommonJS (The legacy Node.js default)
const { test, expect } = require('@playwright/test');
module.exports = { myHelper };

// 2. ES6 Modules (The modern standard used in TypeScript)
import { test, expect } from '@playwright/test';
export const myHelper = () => {};
```
**Why & How it aligns with the question:**
An SDET Lead must understand Node.js architecture. Playwright supports both, but ES Modules (ESM) allow for static analysis, making framework imports cleaner. Since you write your framework in TypeScript, you naturally write in ES Modules, and the compiler transpiles it so Node.js can execute it efficiently.

---

## 8.5. Advanced Error Handling (Try / Catch / Finally)
**Question:** How do you handle unpredictable errors gracefully in your automation scripts without crashing the entire suite or causing memory leaks?

**Practical Snippet & Answer:**
```javascript
test('Database cleanup on failure', async ({ page }) => {
    let dbConnection;
    try {
        dbConnection = await connectToDB(); // Connect to backend DB
        await page.goto('/dashboard');
        await expect(page.locator('.header')).toBeVisible(); // If this fails, code jumps to catch
    } catch (error) {
        console.error(`Test failed abruptly: ${error.message}`);
        // Optionally trigger a custom Slack/Teams alert here via API
        throw error; // MUST rethrow so Playwright correctly marks the test as FAILED
    } finally {
        // The finally block ALWAYS executes, whether the test passed OR failed!
        if (dbConnection) {
            await dbConnection.close();
            console.log('Database connection securely closed.');
        }
    }
});
```
**Why & How it aligns with the question:**
The JD emphasizes building robust, scalable frameworks. Relying only on Playwright's default assertions isn't enough when dealing with DBs or third-party APIs. Using `finally` guarantees that expensive resources (like DB connections, API sessions, or open file streams) are safely closed even if the UI assertion fails, preventing memory leaks and locked databases.
