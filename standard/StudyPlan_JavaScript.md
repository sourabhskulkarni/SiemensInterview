# Study Plan: JavaScript / TypeScript for SDET

As an SDET, you need strong logical and coding skills in JavaScript (Node.js). You will likely be asked to write code on the spot.

---

## 1. Promises and Async/Await
**Concept Used:** Asynchronous Programming, Non-blocking execution.

**Context Story:** 
In UI/API automation, everything takes time—clicking a button, waiting for a network response, querying a database. If JavaScript didn't wait for these actions to complete, your test would fail before the page even loaded. `async/await` is the modern, readable way to handle this, ensuring the code pauses until a promise is resolved.

**Snippet:**
```javascript
// Using async/await for clean asynchronous code
async function fetchUserData() {
    try {
        const response = await fetch('https://api.example.com/user');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error("Failed to fetch:", error);
    }
}
```

**How to Explain Code in Interview:**
"In this snippet, the `async` keyword tells the function it will handle asynchronous operations. The `await` keyword pauses the execution of `fetchUserData` until the `fetch` API call completes. I also wrap it in a `try...catch` block to gracefully handle any network failures or 500 errors from the server. This is exactly how we handle API validations in test scripts to prevent unhandled promise rejections."

**Questions that might be asked:**
- "What happens if you forget the `await` keyword before `fetch`?"
- "How is `async/await` different from using `.then().catch()`?"
- "How would you run three independent API calls at the same time using Promises?" *(Answer: `Promise.all()`)*

---

## 2. Array Manipulation (Map, Filter, Reduce)
**Concept Used:** Functional Programming, Array Iteration, Data Transformation.

**Context Story:**
When you hit an API endpoint that returns a list of 100 users, and you only need to verify the permissions of users who are "active", you don't want to use clunky `for` loops. Array methods like `filter` and `map` allow you to chain operations elegantly, making your test assertions clean and robust.

**Snippet:**
```javascript
const users = [
    { name: 'Alice', active: true },
    { name: 'Bob', active: false },
    { name: 'Charlie', active: true }
];

// Extract names of active users
const activeNames = users.filter(u => u.active).map(u => u.name);
console.log(activeNames); // ['Alice', 'Charlie']
```

**How to Explain Code in Interview:**
"Here, I'm chaining `.filter()` and `.map()`. First, `.filter(u => u.active)` iterates through the array and returns a new array containing only the objects where `active` is true. Then, `.map(u => u.name)` iterates over that new array and extracts just the `name` property. This approach is highly readable and doesn't mutate the original array, which is a best practice."

**Questions that might be asked:**
- "What is the difference between `map` and `forEach`?"
- "How would you calculate the total age of all users using `.reduce()`?"
- "Does `.filter()` modify the original array?"

---

## 3. Closures
**Concept Used:** Lexical Scoping, Data Privacy.

**Context Story:**
Sometimes in a test framework, you want to maintain a piece of state—like a counter for unique test data, or a cached authentication token—without polluting the global scope. A closure allows a function to "remember" the variables of its outer function even after that outer function has finished executing.

**Snippet:**
```javascript
function createCounter() {
    let count = 0; // This variable is protected
    return function() {
        count++;
        return count;
    }
}
const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

**How to Explain Code in Interview:**
"In this code, `createCounter` defines a local variable `count` and returns an anonymous function. Even though `createCounter` finishes running, the returned function maintains access to the `count` variable. This is a closure. It's a great pattern in frameworks to create private variables that cannot be accidentally modified from the outside."

**Questions that might be asked:**
- "Can you explain what a closure is in your own words?"
- "Where have you practically used closures in your automation framework?"
- "What are the potential drawbacks of closures? (Hint: Memory leaks if not handled properly)"
