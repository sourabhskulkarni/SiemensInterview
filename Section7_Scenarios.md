# Section 7: Scenario-Based Questions

## 24. Lift Synchronization Testing
**Question:** How to test lift (elevator) synchronization?

**Answer & Explanation:**
Testing IoT/hardware synchronization is a classic system design test.
**Practical Approach:**
1. **Concurrent Requests:** Lift receives calls from Floor 1 (Up) and Floor 5 (Down) simultaneously. Write tests to verify the routing algorithm prioritizes the most efficient path based on current floor and direction.
2. **Emergency Handling:** Trigger an API payload simulating a fire alarm. Assert that the lift overrides all user inputs and immediately proceeds to the ground floor.
3. **API Simulation:** Since you cannot automate physical lifts, you automate the API endpoints that the lift's sensors communicate with.

*Cross Question Answers:* Automate this by using Playwright's `request` context to bombard the lift controller API with concurrent requests (using `Promise.all()`) and verify the JSON response state. Race conditions are validated by ensuring the backend correctly locks states (e.g., doors cannot open while moving).

---

## 25. CAPTCHA Automation
**Question:** How to automate CAPTCHA?

**Answer & Explanation:**
**Practical Approach:**
You **DO NOT** automate CAPTCHA. The entire purpose of a CAPTCHA is to block automation. 
Instead, the industry standard approaches are:
1. **Disable in lower environments:** Work with developers to disable the CAPTCHA widget entirely on QA/Staging environments.
2. **Static Token:** Have developers configure a backdoor bypass token (e.g., if header `X-Bypass-Captcha: true` is present, allow login without captcha).
3. **Test API Directly:** Test the login logic via API, bypassing the UI completely.

*Cross Question Answers:* OCR (Optical Character Recognition) limitations: OCR is flaky, slow, and Google reCAPTCHA v3 uses behavioral analysis (mouse movements) which Playwright will trigger as a bot. Security concerns: Never put bypass tokens into production code.
