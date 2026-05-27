# Study Plan: CI/CD & Docker

---

## 1. CI/CD Pipeline Concepts (GitLab CI/YAML)
**Concept Used:** Continuous Integration, Infrastructure as Code (IaC), Automation.

**Context Story:**
Running tests locally is great for development, but in an enterprise, code must be tested before it is merged and deployed. CI/CD pipelines automate this. By writing a YAML file, we define instructions for a remote server (runner) to pull our code, install dependencies, run Playwright, and publish the HTML report.

**Snippet (GitLab CI `gitlab-ci.yml`):**
```yaml
stages:
  - e2e-tests

run_playwright_tests:
  stage: e2e-tests
  image: mcr.microsoft.com/playwright:v1.40.0-jammy # Pre-built Playwright image
  script:
    - npm ci # Clean install of dependencies
    - npx playwright test
  artifacts:
    when: always
    paths:
      - playwright-report/
```

**How to Explain Code in Interview:**
"This is a basic GitLab CI configuration. I define a stage called `e2e-tests`. The crucial part is specifying the `image`—I use Microsoft's official Playwright Docker image because it already contains the necessary browser binaries, avoiding flaky installations. Under `script`, I run `npm ci` and then the tests. Finally, I define `artifacts` with `when: always`, ensuring that even if tests fail, the HTML report is saved and downloadable for debugging."

**Questions that might be asked:**
- "What is the difference between `npm install` and `npm ci`?" *(Answer: `npm ci` is for CI environments; it strictly reads `package-lock.json` and deletes `node_modules` before installing, ensuring reproducible builds.)*
- "How do you trigger this pipeline to run only on Pull Requests?"
- "How do you pass secrets, like passwords, into the CI pipeline safely?" *(Answer: Using CI/CD environment variables, never hardcoding them in the repo).*

---

## 2. Dockerizing Tests
**Concept Used:** Containerization, Environment Consistency.

**Context Story:**
"It works on my machine" is the classic developer excuse. Docker solves this by packaging the code, runtime, and system libraries into a standard unit (a container). Dockerizing Playwright tests ensures that if it runs successfully on your laptop, it will run identically on the CI server or a colleague's laptop.

**Snippet (Dockerfile):**
```dockerfile
FROM mcr.microsoft.com/playwright:v1.40.0-jammy
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npx", "playwright", "test"]
```

**How to Explain Code in Interview:**
"In this Dockerfile, I start from the official Playwright base image. I set the working directory to `/app`. I first copy `package.json` and run `npm ci`. I do this *before* copying the rest of the code to leverage Docker's layer caching—if the package file hasn't changed, Docker won't reinstall dependencies, saving huge amounts of build time. Finally, the `CMD` specifies the default command to run when the container starts."

**Questions that might be asked:**
- "Why use the official Playwright image instead of a generic Node image?" *(Answer: The Playwright image includes all required OS dependencies and browser binaries).*
- "What is the purpose of `.dockerignore`?"
- "How would you run this Docker image locally and mount the test results to your host machine?"
