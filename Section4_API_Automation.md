# Section 4: API Automation through Code

## 14. POST API Request
**Question:** Make a POST request.

**Practical Snippet & Answer:**
```javascript
// Line 1: Use Playwright's built-in request context to make a POST call
const response = await request.post('https://api.example.com/users', {
    // Line 2: Pass the JSON payload inside the 'data' property
    data: { name: 'Sourabh' },
    // Line 3: Add necessary headers
    headers: { 'Accept': 'application/json' } 
});

// Line 4: Assert that the status is 201 Created
expect(response.status()).toBe(201);
```

**Why & How it aligns with the question:**
Shows direct use of Playwright's API abilities.
*Cross Question Answers:* Playwright uses `data` for JSON payloads (and auto-serializes it) but uses `multipart` for file uploads or form data. You validate the response using standard `expect(response.ok()).toBeTruthy()` and by parsing the body via `await response.json()`.

---

## 15. Token Authentication
**Practical Snippet & Answer:**
```javascript
// Line 1: Hit the login API with credentials
const loginResponse = await request.post('/login', {
    data: { username: 'admin', password: 'admin123' }
});

// Line 2: Parse the response into a JS object
const responseBody = await loginResponse.json();

// Line 3: Extract the JWT token
const token = responseBody.token;

// Line 4: Use the token in a subsequent API call in the headers
await request.get('/secure-data', {
    headers: { 'Authorization': `Bearer ${token}` }
});
```

**Why & How it aligns with the question:**
Auth is mandatory for backend testing. Extracting tokens dynamically proves real API automation capability.
*Cross Question Answers:* JWT is stateless (server doesn't store the session, it just verifies the token signature), while OAuth is a full delegation protocol. If a token expires mid-test, you write a helper function to catch the 401 Unauthorized error, hit the `/refresh` token endpoint, and retry the request.

---

## 16. Chained API Calls
**Question:** Use data from one API as input to another.

**Practical Snippet & Answer:**
```javascript
// Line 1: Create a user
const createUserRes = await request.post('/users', { data: { name: 'Bob' } });
const newUser = await createUserRes.json();
const userId = newUser.id; // Extract dynamic ID

// Line 2: Use the newly created ID to GET the specific user
const fetchUserRes = await request.get(`/users/${userId}`);
const fetchedUser = await fetchUserRes.json();

// Line 3: Assert the data matches
expect(fetchedUser.name).toBe('Bob');
```

**Why & How it aligns with the question:**
Tests dependency management.
*Cross Question Answers:* If the first API fails, the test aborts immediately, which is good (fail fast). For dynamic payload generation, you use libraries like `faker.js` to ensure the first API generates unique emails/names every time the test runs.

---

## 17. API Mocking
**Practical Snippet & Answer:**
*(Covered in Section 3, Question 11)*
*Cross Question Answers:* API Mocking guarantees CI stability. If a third-party payment gateway is down, your UI tests shouldn't fail. Mocking isolates the frontend.

---

## 18. API + UI Hybrid Validation
**Question:** Combine API and UI in one test.

**Practical Snippet & Answer:**
```javascript
test('Hybrid Create and Verify', async ({ page, request }) => {
    // Line 1: Pre-condition - Create data directly via API (Takes 0.1 seconds)
    const res = await request.post('/api/orders', { data: { item: 'Laptop' } });
    const orderId = (await res.json()).id;
    
    // Line 2: Navigate to UI and verify the data rendered properly (Takes 2 seconds)
    await page.goto(`/orders/${orderId}`);
    await expect(page.locator('.order-title')).toHaveText('Laptop');
});
```

**Why & How it aligns with the question:**
Hybrid testing is the mark of a Senior SDET. Creating an order via UI might require 15 clicks (takes 10 seconds). Creating it via API takes 0.1 seconds. You use the API to instantly set up state, and the UI just to verify it rendered, drastically cutting regression time.
