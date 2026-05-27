# Section 9: Performance & NFR Testing (K6 Integration via MCP)

## 28. Advanced NFR Testing Integration (The MCP CLI Approach)
**Question:** The JD mentions K6 and Sitespeed.io. How do you handle K6 performance testing in your framework without creating duplicate scripts every time the API changes?

**Answer & Explanation:**
Standard practice is to use Playwright to record a `.har` file and convert it using `har-to-k6`. However, this creates a maintenance nightmare: every time a developer changes an API endpoint, you have to re-record the HAR and generate a completely new K6 script, leading to duplicated test code and rot.

**The Advanced Master Solution:**
"Instead of generating new scripts from scratch every time, I use my custom MCP CLI tool. During my Playwright UI execution, the framework actively sniffs all outgoing network traffic. It compares the real-time API payloads against our *existing* K6 script repository."

**Practical Conceptual Snippet:**
```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests must complete below 500ms
  },
};

// This K6 script is automatically maintained by the MCP CLI
export default function () {
    // The MCP CLI detected a change in the '/checkout' endpoint payload during UI execution
    // and automatically proposed an AST patch to update this exact JSON body:
    const payload = JSON.stringify({
        itemId: '12345',
        quantity: 1,
        new_dynamic_field: 'added_by_AI_patch' // AI detected this new field and patched the K6 script!
    });

    const params = { headers: { 'Content-Type': 'application/json' } };
    const res = http.post('https://api.example.com/checkout', payload, params);

    // Asserting Functional Status inside Performance Test
    check(res, {
        'status is 200': (r) => r.status === 200,
    });
}
```

**How to explain in interview:**
"If the UI payload changes, my AI-driven MCP CLI detects the difference. It parses the existing K6 `.js` file using an Abstract Syntax Tree (AST), pinpoints the exact `payload` variable that needs updating, and proposes a code diff. Once I approve it, the existing K6 script is patched automatically. 

After the scripts are synced, the CI/CD pipeline triggers K6 to simulate 1,000 concurrent users. If the `p(95)` response time exceeds 500ms, the `thresholds` block causes K6 to exit with an error, instantly failing the GitLab CI pipeline and preventing degraded performance from reaching production. For frontend rendering NFRs (like Largest Contentful Paint), I integrate Sitespeed.io."

*Cross Question Answers:* How do you define NFR benchmarks? Benchmarks are defined by the business (e.g., SLA requires 500ms max latency). These are hardcoded into the K6 `options.thresholds` object so they act as automated quality gates in the CI/CD pipeline.
