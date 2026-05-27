# Section 8: Agile, Test Management & Strategy

## 26. Agile Methodologies & Shift-Left
**Question:** How do you embed shift-left quality gates into Agile sprints?

**Answer & Explanation:**
In a senior role, the focus shifts from just "writing scripts" to "orchestrating quality."
**Practical Approach:**
Shift-left means testing earlier in the lifecycle. In my framework, I achieved this by:
1. **Pre-Commit Hooks:** Enforcing linting and unit tests via Husky before code can even be pushed.
2. **PR Quality Gates:** Integrating Playwright smoke tests into Azure DevOps (ADO) Pull Request pipelines. The PR cannot be merged into `main` unless the Playwright tests pass, preventing defect leakage completely.
3. **JIRA Dashboards:** I built automated reporting that updates JIRA Xray/Zephyr with real-time pass/fail metrics, enabling product owners to make risk-based Go/No-Go decisions on release day.

*Cross Question Answers:* For JIRA/ADO integration, you use REST APIs or native ADO plugins to post Playwright HTML reports directly to the JIRA ticket. Risk-based go/no-go means prioritizing critical path tests (like Checkout) over low-risk tests (like UI color checks) when time is short.

---

## 27. Defect Lifecycle & Root Cause
**Question:** Explain how you leveraged GCP log monitoring for defect root cause analysis.

**Answer & Explanation:**
When a test fails on CI/CD, the hardest part is knowing *why*.
**Practical Approach:**
If an order creation API fails in my Playwright test, I don't just report "Test Failed." I log into Google Cloud Platform (GCP) Logs Explorer. By passing a unique `X-Correlation-ID` header in my Playwright API requests, I can trace exactly which microservice crashed in GCP. This allows me to attach the exact backend stack trace to the JIRA bug, reducing developer debugging time drastically and helping meet the 48-hour SLA.

*Cross Question Answers:* Differentiating frontend vs backend: If the API returns 500, it's a backend issue. If the API returns 200 but the UI doesn't display the data, it's a frontend React/Angular parsing issue. GCP logs help prove it's a backend issue immediately.
