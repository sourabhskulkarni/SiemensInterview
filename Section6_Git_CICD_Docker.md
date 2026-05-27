# Section 6: Git / GitLab CI/CD / Docker

## 21. Git Commands
**Question:** Explain git commands.

**Practical Snippet & Answer:**
```bash
# Pulls latest changes from remote and merges them into current branch
git pull 

# Downloads changes from remote but does NOT merge them. Safe way to see what changed.
git fetch 

# Re-applies your local commits on top of the latest main branch (keeps linear history)
git rebase main 

# Temporarily saves your modified tracked files without committing them
git stash 

# Takes a specific commit from another branch and applies it to your current branch
git cherry-pick <commit-hash>
```

**Why & How it aligns with the question:**
SDETs work with developers and must manage version control.
*Cross Question Answers:* Merge creates a new commit that ties two branches together, leaving a messy history. Rebase rewrites history to be a single straight line. Merge conflicts are resolved by opening the file, choosing the correct `<<<< HEAD` or `>>>> branch` block, deleting the markers, and committing.

---

## 22. GitLab CI Pipeline
**Question:** Explain GitLab YAML structure.

**Practical Snippet & Answer:**
```yaml
stages:
  - test # Line 1: Define execution stages

playwright-test: # Line 2: Job Name
  stage: test # Line 3: Assign to stage
  image: mcr.microsoft.com/playwright:v1.40.0-jammy # Line 4: Use Docker image containing browsers
  script:
    - npm ci # Line 5: Clean install dependencies
    - npx playwright test # Line 6: Run tests
  artifacts: # Line 7: Save output files
    when: always # Line 8: Save even if tests fail
    paths:
      - playwright-report/ # Line 9: Folder to save
```

**Why & How it aligns with the question:**
Shows CI automation expertise.
*Cross Question Answers:* Artifacts are zipped files saved by GitLab after a job finishes, crucial for downloading HTML test reports to see why tests failed. Parallel jobs can be achieved by using the `parallel:` keyword in GitLab, which spins up multiple runners and splits the tests via Playwright's `--shard` command.

---

## 23. Docker for Playwright
**Practical Snippet & Answer:**
```dockerfile
# Line 1: Base image provided by Microsoft containing OS, Node, and browsers
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

# Line 2: Set the working directory inside the container
WORKDIR /app

# Line 3: Copy package files first to leverage Docker cache
COPY package*.json ./

# Line 4: Install NPM dependencies
RUN npm ci

# Line 5: Copy the rest of your framework files
COPY . .

# Line 6: Define the default command to run tests
CMD ["npx", "playwright", "test"]
```

**Why & How it aligns with the question:**
Containerization guarantees the test runs exactly the same on CI as it does on your laptop.
*Cross Question Answers:* Docker ensures there are no "It works on my machine" issues. The Microsoft base image manages browser dependencies natively, so you don't have to write scripts to download Chrome/Webkit binaries on Linux servers.

---

## 24. CI/CD Stages Explanation
**Question:** Explain the typical stages of a CI/CD pipeline where your automation suite is integrated.

**Answer & Explanation:**
In an enterprise environment (like GitLab CI or Azure DevOps), a pipeline isn't just one step. It's broken into logical stages.

**Typical Stages:**
1. **Build (`npm ci` / `docker build`):** Compiles the application code and installs test dependencies.
2. **Lint & Static Analysis (`eslint` / `SonarQube`):** Runs static code checks before any tests execute to catch basic syntax/formatting errors.
3. **Unit Tests (Jest / Mocha):** Executes developer unit tests. Fast and isolated.
4. **Deploy to Lower Env (Dev/QA):** Deploys the built artifact to a staging or QA server.
5. **E2E Automation (Playwright):** *This is our stage.* Once the code is deployed to QA, the pipeline triggers our Playwright smoke/regression suite against the live QA environment.
6. **Performance (K6):** If E2E passes, load testing runs to ensure no performance degradation.
7. **Deploy to Prod:** (Usually requires a manual approval gate in ADO/GitLab).

*Cross Question Answers:* If E2E fails, the pipeline immediately halts (fails the build), preventing the bad code from moving to the Production deployment stage.

---

## 25. CI/CD Security Gates
**Question:** How are security gates implemented and validated in your CI/CD pipeline?

**Answer & Explanation:**
Security isn't just an afterthought; it's a mandatory pipeline gate. If security checks fail, the pipeline stops.

**Practical Implementation:**
1. **Secret Scanning:** Using tools like GitLab's native secret detection or `git-secrets` to ensure developers (or QA) haven't hardcoded passwords or API keys in the code.
2. **Dependency Vulnerability (SCA):** Running `npm audit` or tools like *Snyk*. If a third-party NPM package used in our framework has a known CVE (vulnerability), the pipeline fails.
3. **DAST (Dynamic Application Security Testing):** Tools like *OWASP ZAP* can be run alongside Playwright. While Playwright navigates the UI, ZAP proxies the network traffic and scans for SQL Injection or XSS vulnerabilities dynamically.

**Practical Snippet (GitLab CI):**
```yaml
security_scan:
  stage: test
  script:
    - npm audit --audit-level=high # Fails pipeline if HIGH vulnerabilities exist
    - snyk test # Third-party security gate
  allow_failure: false # Pipeline STOPS if this fails
```

*Cross Question Answers:* "How do you manage tokens safely in automation?" We never hardcode them. We inject them into the Playwright tests dynamically at runtime using CI/CD Masked Variables (`process.env.API_TOKEN`), ensuring they never appear in Git logs, console outputs, or HTML reports.
