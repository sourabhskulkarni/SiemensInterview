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
