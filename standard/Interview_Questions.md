# SDET Interview Questions (1-Hour Technical Round)

**Candidate Profile:** Sourabh Kulkarni (9+ Years, QE Lead, Playwright/TS, AI-assisted testing)
**Role:** Test Automation Engineer / SDET (5-8 Years, JS/Playwright, API automation, CI/CD)

## 1. Introduction & Architecture (10 mins)
*   **Question:** You mentioned architecting a scalable Playwright framework using POM and fixtures. Can you walk me through the directory structure and how fixtures manage the state across UI, API, and DB layers?
*   **Question:** The JD emphasizes hands-on coding and DevOps. In your recent role, how did you integrate your Playwright tests into Azure DevOps CI/CD pipelines, and what specific quality gates did you implement to block defect leakage?
*   **Question:** (Bonus/Curiosity) You've used an MCP CLI for AI-assisted test generation. How does that architecture securely parse your application context without compromising sensitive data?

## 2. JavaScript/TypeScript Coding & Logic (15 mins)
*(The JD explicitly states that knowing a tool isn't enough; logical coding is mandatory.)*
*   **Question (Promises & Async):** Write a function that takes an array of API URLs and fetches them concurrently. If one fails, it should still return the successful responses.
*   **Question (Data Manipulation):** Given an array of nested user objects (e.g., users with an array of permissions), write a script to extract a unique list of all permissions across all users.
*   **Question:** Explain the difference between `==` and `===` in JavaScript, and how TypeScript's strict typing improves the stability of your automation scripts.

## 3. API Automation Using Playwright (15 mins)
*(Focusing on deeper integration and control, avoiding Postman as per JD)*
*   **Question:** How do you handle authentication (e.g., Bearer tokens or Session Cookies) in Playwright API tests so you don't have to authenticate before every single test?
*   **Question:** Imagine a scenario where a downstream microservice is flaky. How would you use Playwright's `route.fulfill` or `route.abort` to mock the API response and test the UI's error handling behavior?
*   **Question:** How do you validate a complex, deeply nested JSON response schema using Playwright? Do you use any specific libraries (like `ajv` or `zod`)?

## 4. CI/CD, DevOps & Git (10 mins)
*   **Question:** You have experience with Azure DevOps, but the JD mentions GitLab CI/CD and Docker. Conceptually, how would you containerize your Playwright framework using Docker? What base image would you use?
*   **Question:** What is your branching strategy for test code? How do you ensure that test code is versioned and deployed alongside the application code?
*   **Question:** Explain how you achieved a 60% reduction in regression execution time via parallel test runs. Did you use Playwright's built-in sharding or CI-level matrix strategies?

## 5. Scenario & Problem Solving (10 mins)
*   **Question:** The JD mentions IoT devices and real-time data validation. If you had to test a web dashboard that receives real-time WebSocket updates from an IoT device, how would you automate this using Playwright?
*   **Question:** How do you approach NFR (Non-Functional Requirements) like performance testing? Have you integrated tools like K6 into your testing lifecycle?
