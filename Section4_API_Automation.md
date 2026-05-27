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

---

## 18.1. GET Request (Query Params & Deep Validation)
**Question:** How do you send a GET request with query parameters and deeply validate the headers, status, and nested JSON response?

**Practical Snippet & Answer:**
```javascript
const response = await request.get('/api/users', {
    params: { status: 'active', role: 'admin' }, // Appends ?status=active&role=admin
    headers: { 'Accept': 'application/json' }
});

// 1. Status Validation
expect(response.status()).toBe(200);
expect(response.ok()).toBeTruthy();

// 2. Header Validation
const headers = response.headers();
expect(headers['content-type']).toContain('application/json');

// 3. Body Validation
const body = await response.json();
expect(body.users.length).toBeGreaterThan(0);
expect(body.users[0]).toHaveProperty('id');
```
**Why & How it aligns:** Validating headers and complex nested objects proves hands-on proficiency, separating junior testers (who just check status 200) from senior engineers.

---

## 18.2. PUT Request (Full Update)
**Question:** How do you execute a PUT request to update an entire resource?

**Practical Snippet & Answer:**
```javascript
const response = await request.put('/api/users/123', {
    data: {
        name: 'Sourabh Kulkarni',
        email: 'sourabh@example.com',
        role: 'SDET Lead' // PUT replaces the entire object
    }
});

expect(response.status()).toBe(200); // Or 204 No Content
const body = await response.json();
expect(body.name).toBe('Sourabh Kulkarni');
```
**Why & How it aligns:** Interviewers look for you knowing the difference between PUT and PATCH. PUT replaces the whole entity.

---

## 18.3. PATCH Request (Partial Update)
**Question:** How do you execute a PATCH request to update just a single field of a resource?

**Practical Snippet & Answer:**
```javascript
const response = await request.patch('/api/users/123', {
    data: {
        role: 'Principal SDET' // PATCH only updates the provided fields
    }
});

expect(response.status()).toBe(200);
const body = await response.json();
expect(body.role).toBe('Principal SDET');
// We expect the original name/email to remain intact
expect(body).toHaveProperty('name'); 
```

---

## 18.4. DELETE Request & Teardown Verification
**Question:** How do you perform a DELETE request, and how do you verify it was actually deleted?

**Practical Snippet & Answer:**
```javascript
// 1. Send the DELETE request
const deleteRes = await request.delete('/api/users/123');

// 2. Verify DELETE response status (Standard is 204 No Content or 200 OK)
expect([200, 204]).toContain(deleteRes.status());

// 3. Verify it was actually deleted by sending a GET request
const getRes = await request.get('/api/users/123');
expect(getRes.status()).toBe(404); // Expect Not Found
```
**Why & How it aligns:** Real SDETs don't just trust a 200 OK on a DELETE request. They follow up with a GET request to ensure the backend actually purged or soft-deleted the record, returning a 404.

---

## 18.5. Practical API Status Code Handling & Validation
**Question:** Don't just explain the status codes theoretically. Show me practically how you validate each status code in your automation framework (2XX, 3XX, 4XX, 5XX).

**Practical Snippet & Answer:**

### 2XX: Success
```javascript
// 200 OK: Standard GET success
const getRes = await request.get('/api/users/1');
expect(getRes.status()).toBe(200);

// 201 Created: Standard POST success
const postRes = await request.post('/api/users', { data: { name: 'John' } });
expect(postRes.status()).toBe(201);
expect(await postRes.json()).toHaveProperty('id');

// 204 No Content: Successful DELETE (No response body)
const delRes = await request.delete('/api/users/1');
expect(delRes.status()).toBe(204);
// Ensuring there is actually no body returned
expect(await delRes.text()).toBe('');
```

### 3XX: Redirection & Caching
```javascript
// 301 Moved Permanently: Testing redirect handling
// Playwright follows redirects automatically. To validate the redirect happened, we check the final URL.
const redirectRes = await request.get('/old-api/endpoint');
expect(redirectRes.status()).toBe(200); // The final resolved status
expect(redirectRes.url()).toContain('/new-api/endpoint'); // Proves the 301 redirect worked

// 304 Not Modified: Testing Caching (ETag)
const firstRes = await request.get('/api/config');
const etag = firstRes.headers()['etag']; // Capture the cache tag

// Send request again passing the ETag
const cachedRes = await request.get('/api/config', {
    headers: { 'If-None-Match': etag }
});
expect(cachedRes.status()).toBe(304); // Proves the backend correctly cached it
```

### 4XX: Client Errors (Negative Testing)
```javascript
// 400 Bad Request: Sending invalid JSON or missing mandatory fields
const badRes = await request.post('/api/users', { data: { age: 'invalid_string' } });
expect(badRes.status()).toBe(400);
expect(await badRes.json()).toMatchObject({ error: 'Validation failed' });

// 401 Unauthorized: Missing or invalid Bearer token
const unauthRes = await request.get('/api/secure-data', {
    headers: { 'Authorization': 'Bearer INVALID_TOKEN' }
});
expect(unauthRes.status()).toBe(401);

// 403 Forbidden: Valid token, but insufficient role permissions
const forbidRes = await request.delete('/api/admin/delete-all', {
    headers: { 'Authorization': 'Bearer STANDARD_USER_TOKEN' } // User trying admin action
});
expect(forbidRes.status()).toBe(403);

// 404 Not Found: Requesting an ID that doesn't exist
const notFoundRes = await request.get('/api/users/999999');
expect(notFoundRes.status()).toBe(404);

// 409 Conflict: Creating a duplicate record
const duplicateRes = await request.post('/api/register', { data: { email: 'already_exists@test.com' } });
expect(duplicateRes.status()).toBe(409);
expect(await duplicateRes.json()).toHaveProperty('error', 'Email already in use');
```

### 5XX: Server Errors (Backend Resilience)
```javascript
// 500 Internal Server Error & 501 Not Implemented
// We mock these to test how the UI handles backend crashes (Resilience Testing)
await page.route('**/api/dashboard', route => {
    route.fulfill({ status: 500, body: 'Internal Server Error' });
});
await page.goto('/dashboard');
// Validate that the UI gracefully shows a generic error page instead of crashing
await expect(page.locator('.error-banner')).toContainText('Something went wrong on our end');

// 504 Gateway Timeout: Handled in K6 Performance Testing, not functional testing
import http from 'k6/http';
import { check } from 'k6';

export default function () {
    const res = http.get('https://api.example.com/heavy-query');
    // If the DB takes > 30 seconds, it throws 504. We validate it didn't happen.
    check(res, {
        'status is not 504': (r) => r.status !== 504,
        'response time < 2s': (r) => r.timings.duration < 2000
    });
}
```

---

## 18.6. Runtime API Script Synchronization (MCP AI Integration)
**Question:** You mentioned your MCP CLI updates API scripts dynamically. How exactly does your framework intercept the HTTP methods, headers, and payloads during UI execution to know *what* to patch in your Playwright API scripts?

**Answer & Explanation:**
When an interviewer asks this, they are verifying if you actually built the AI tool or if you just used a plugin. You must explain *how* the data is extracted from the network layer before the AI even touches it.

**The Mechanical Process:**
"The AI is only as good as the data it receives. During my Playwright UI execution, I don't just click buttons; I use Playwright's native `page.on('request')` listeners to build a live 'Network Map'. 

When a developer changes a `POST` request payload in the application, my Playwright listener captures it in real-time. I extract the HTTP Method (`req.method()`), the Endpoint (`req.url()`), and the JSON Payload (`req.postDataJSON()`). 

My MCP CLI then takes this live data and compares it against the AST (Abstract Syntax Tree) of my *existing* Playwright API test files. If the live payload has a new field (e.g., `shipping_method: 'express'`), the CLI flags a mismatch. The AI then proposes a code diff to inject that exact field into my API validation script."

**Practical Snippet (The Sniffer Logic):**
```javascript
// This runs in a global hook or fixture to sniff traffic during UI tests
page.on('request', async req => {
    // Only intercept our backend API calls, ignoring analytics/images
    if (req.url().includes('/api/v1/orders') && req.method() === 'POST') {
        const livePayload = req.postDataJSON();
        
        // MCP CLI validates the live payload against existing API test scripts
        const diff = await mcpValidator.compareWithExistingApiTest('CreateOrder', livePayload);
        
        if (diff.hasMismatch) {
            console.log(`[AI Alert] The UI sent a new payload structure for POST /orders.`);
            console.log(`Missing fields in your API script: ${diff.missingFields.join(', ')}`);
            
            // Generate the patch for the API test file for developer review
            await mcpValidator.generateApiScriptPatch(diff); 
        }
    }
});
```

**Why & How it aligns:** This proves deep architectural knowledge. It shows you understand how to extract the exact HTTP methods, payloads, and validation parameters from the Chromium network layer programmatically, and how to use that raw data to feed a custom AI engine.
