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

## 8. Abstract Class vs Interface
**Question:** Explain `abstract class LoginPage`.

**Practical Snippet & Answer:**
```typescript
// Line 1: Abstract classes cannot be instantiated directly.
abstract class BasePage {
    // Line 2: Concrete method with implementation. Interfaces CANNOT do this.
    async openUrl(url: string) {
        console.log(`Navigating to ${url}`);
    }
    
    // Line 3: Abstract method with NO implementation. Child classes MUST implement it.
    abstract verifyPageLoaded(): Promise<boolean>;
}

// Line 4: Child class extends the abstract class.
class LoginPage extends BasePage {
    async verifyPageLoaded() {
        return true; // Implementation forced here
    }
}
```

**Why & How it aligns with the question:**
Abstract classes are a powerful OOP concept used in Page Object Models. They allow you to share common code (like an `openUrl` method) across all pages, while enforcing child classes to define their own specific logic (like `verifyPageLoaded`).
*Cross Question Answers:* An `interface` is just a contract with zero implementation. An `abstract class` can provide default methods. Also, a class can implement *multiple* interfaces, but can only inherit from *one* abstract class (no multiple inheritance).

---

## 8.1. Architectural Alignment: TS Design in your AI Tool
**Question:** How did you use advanced TypeScript features (Interfaces and Abstract Classes) to architect your MCP CLI tool?

**Answer & Explanation:**
You want to show that your TypeScript knowledge scales beyond basic page objects.
"I used Abstract Classes and Interfaces to make the AI logic in my CLI model-agnostic (applying SOLID principles). I created an `interface ILLMProvider` and an `abstract class AIPromptBuilder` which contains the default logic for stripping PII from network payloads. 
Because of this strict architecture, if the company decides to switch from GitHub Copilot to OpenAI or Anthropic, I don't have to rewrite the core engine. I simply implement a new child class that `extends AIPromptBuilder`. This proves my TS code is highly scalable and maintainable."

---

## 8.2. Async / Await & Promise Combinators
**Question:** Playwright relies entirely on asynchronous execution. Explain the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`. When would you use them in automation?

**Practical Snippet & Answer:**
```javascript
// 1. Promise.all (Fails fast if ANY promise rejects)
// Use Case: Waiting for a popup/file chooser AND clicking a button simultaneously.
const [fileChooser] = await Promise.all([
    page.waitForEvent('filechooser'),
    page.locator('#upload-btn').click() // If this fails, the whole block throws immediately.
]);

// 2. Promise.allSettled (Waits for ALL promises, regardless of pass/fail)
// Use Case: Hitting multiple independent APIs. If one fails, you still want the data from the others.
const results = await Promise.allSettled([
    request.get('/api/users'),
    request.get('/api/orders')
]);
// results[0].status will be either 'fulfilled' or 'rejected'

// 3. Promise.race (Returns the FIRST promise to resolve OR reject)
// Use Case: Implementing a custom timeout wrapper.
const response = await Promise.race([
    page.waitForResponse('**/api/data'),
    new Promise((_, reject) => setTimeout(() => reject(new Error('Timeout!')), 5000))
]);

// 4. Promise.any (Returns the FIRST promise to RESOLVE successfully, ignoring rejections)
// Use Case: Querying multiple redundant backend servers and proceeding with the fastest success.
const fastResponse = await Promise.any([
    request.get('https://primary.com/ping'),
    request.get('https://backup.com/ping')
]);
```

**Why & How it aligns with the question:**
Playwright is fundamentally built on `async/await`. A senior engineer must know that `await` pauses the execution of that specific block until the Promise resolves. 
*Cross Question Answers:* If an interviewer asks "Why use `Promise.all` for a file upload or popup?", the answer is: "Because `page.waitForEvent` must start listening *before* the click happens. If you `await page.click()` first, the popup event might fire and finish before `waitForEvent` even starts listening, causing a deadlock. `Promise.all` executes them concurrently."

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
