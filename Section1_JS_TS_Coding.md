# Section 1: JavaScript / TypeScript Coding Questions

## 1. Reverse words while preserving special characters
**Question:** Reverse the alphabetical characters in a string without moving the special characters.
**Input:** `"I.am@Sourabh"`
**Output:** `"b.hru@oamruS"`

**Practical Snippet & Answer:**
```javascript
function reversePreservingSpecials(str) {
    // Line 1: Convert string to array so we can manipulate it
    let arr = str.split(''); 
    
    // Line 2: Extract only the letters using regex and reverse them
    let letters = arr.filter(char => /[a-zA-Z]/.test(char)).reverse(); 
    
    // Line 3: Map over original array. If it's a letter, replace it with the next reversed letter
    let result = arr.map(char => /[a-zA-Z]/.test(char) ? letters.shift() : char); 
    
    // Line 4: Join the array back into a string
    return result.join(''); 
}

console.log(reversePreservingSpecials("I.am@Sourabh")); // "b.hru@oamruS"
```

**Why & How it aligns with the question:**
The question tests your ability to manipulate strings and arrays based on conditions. By separating the letters, reversing them, and re-injecting them into the original array's positions while leaving special characters untouched, we directly solve the problem. 
*Cross Question Answers:* Time complexity is O(N) because we iterate through the array a few times sequentially. To solve without an extra array, you can use the two-pointer approach (left and right pointers swapping letters until they meet).

---

## 2. Find last repeating character
**Question:** Find the last character in a string that repeats.
**Input:** `"automation"`
**Output:** `"t"`

**Practical Snippet & Answer:**
```javascript
function findLastRepeating(str) {
    // Line 1: Create an object to store character frequencies
    let charCount = {}; 
    
    // Line 2: Iterate through the string to count occurrences
    for (let char of str) { 
        // Line 3: Increment the count, handling undefined as 0
        charCount[char] = (charCount[char] || 0) + 1; 
    }
    
    // Line 4: Iterate backwards through the string
    for (let i = str.length - 1; i >= 0; i--) { 
        // Line 5: The first character we hit from the right with a count > 1 is the last repeating character
        if (charCount[str[i]] > 1) { 
            return str[i]; // Line 6: Return it immediately
        }
    }
    return null; // Line 7: Return null if no repeating characters
}

console.log(findLastRepeating("automation")); // "t"
```

**Why & How it aligns with the question:**
This demonstrates knowledge of Hash Maps (or JavaScript Objects) for O(N) time complexity frequency counting. It avoids nested O(N^2) loops. 
*Cross Question Answers:* A `Map` is slightly better than an `Object` if you have complex keys, but for characters, an Object is fine. For case-insensitive handling, you would convert the string to `str.toLowerCase()` before iterating.

---

## 3. Flatten nested array
**Question:** Flatten `[1,[2,[3,4]],5]` to `[1,2,3,4,5]`

**Practical Snippet & Answer:**
```javascript
function flattenArray(arr) {
    // Line 1: Use reduce to accumulate values into a new array
    return arr.reduce((acc, val) => {
        // Line 2: Check if current value is an array
        if (Array.isArray(val)) {
            // Line 3: If it is, recursively call flattenArray and concat the result
            return acc.concat(flattenArray(val));
        } else {
            // Line 4: If it's not an array, just push/concat it to the accumulator
            return acc.concat(val);
        }
    }, []); // Line 5: Initialize the accumulator as an empty array
}

console.log(flattenArray([1,[2,[3,4]],5])); // [1,2,3,4,5]
```

**Why & How it aligns with the question:**
This tests understanding of recursion and array methods like `reduce()`. It perfectly handles deeply nested structures.
*Cross Question Answers:* The built-in `Array.prototype.flat(Infinity)` does exactly this natively. A recursive approach is great, but an iterative approach using a Stack can avoid maximum call stack size exceeded errors on massive arrays.

---

## 4. Promise Output Prediction
**Code Given:**
```javascript
console.log(1);
setTimeout(() => console.log(2));
Promise.resolve().then(() => console.log(3));
console.log(4);
```

**Answer & Explanation:**
**Output:**
```text
1
4
3
2
```

**Why & How it aligns with the question:**
This tests the JavaScript Event Loop. 
- Line 1: `console.log(1)` is synchronous. Executed immediately. (Outputs 1)
- Line 2: `setTimeout` pushes the callback `() => console.log(2)` to the **Macrotask Queue** (Callback Queue).
- Line 3: `Promise.resolve().then` pushes its callback to the **Microtask Queue**.
- Line 4: `console.log(4)` is synchronous. Executed immediately. (Outputs 4)
- *Event loop rule:* Microtasks are executed completely before Macrotasks. So the Promise runs (Outputs 3), and finally the Timeout runs (Outputs 2).
*Cross Question Answers:* The Microtask queue handles Promises and MutationObserver. The Macrotask queue handles setTimeout, setInterval, and I/O. 

---

## 5. Implement debounce function
**Practical Snippet & Answer:**
```javascript
function debounce(fn, delay) {
    let timer; // Line 1: Closure variable to hold the timeout ID
    
    return function(...args) { // Line 2: Return a function that wraps the original function
        clearTimeout(timer); // Line 3: Cancel any existing timer
        
        timer = setTimeout(() => { // Line 4: Set a new timer
            fn.apply(this, args); // Line 5: Execute the function with correct context and arguments after delay
        }, delay);
    };
}
```

**Why & How it aligns with the question:**
Debounce limits the rate at which a function fires. It's heavily used in UI for search bars. 
*Cross Question Answers:* Throttle fires a function exactly once every X milliseconds (like a steady heartbeat), while Debounce waits for the user to *stop* triggering it for X milliseconds before firing. In automation, it's used to avoid double-clicking or handling rapid events.

---

## 5.1. Architectural Alignment: How JS Concepts Power Your AI Tool
**Question:** How did you apply these core JavaScript concepts (like Recursion and the Event Loop) when building your MCP CLI AI Tool?

**Answer & Explanation:**
When interviewers ask basic coding questions, a Senior SDET strategically ties them back to enterprise features.
1. **Recursion (Array Flattening):** "The AST (Abstract Syntax Tree) that my MCP CLI parses is essentially a deeply nested JSON object. To find the exact API payload node to patch, my script must recursively traverse this tree structure, applying the exact same recursive logic as the array flattening problem."
2. **Event Loop (Promises):** "Sniffing Playwright network traffic in real-time (`page.on`) while simultaneously sending asynchronous API calls to an LLM requires deep mastery of the Node.js Microtask queue. If I didn't structure my Promises correctly, the heavy LLM network calls would block the single JavaScript thread, crashing the UI automation."
