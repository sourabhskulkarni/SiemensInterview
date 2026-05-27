# DevOps, Git & Docker Cheat Sheet

A quick reference for the infrastructure and CI/CD concepts requested in the SDET role.

---

## 1. Git Version Control

**Core Commands:**
```bash
git fetch            # Download latest changes from remote, DO NOT merge
git pull             # Download AND merge changes from remote
git stash            # Temporarily save uncommitted changes
git cherry-pick <id> # Apply a specific commit from another branch
```

**Rebase vs Merge:**
- `git merge main`: Combines branches, creating a "merge commit" (messy history).
- `git rebase main`: Applies your branch commits ON TOP of main (clean, linear history).

**Conflict Resolution:**
If you get a merge conflict during rebase:
1. Fix markers (`<<<< HEAD`) in the conflicting file.
2. `git add .`
3. `git rebase --continue`

---

## 2. CI/CD Pipeline (GitLab YAML)
Automating test execution on a remote runner.

```yaml
# .gitlab-ci.yml
stages:
  - test                  # Define pipeline stages

e2e_tests:                # Job name
  stage: test
  image: mcr.microsoft.com/playwright:v1.40.0-jammy # Microsoft's Playwright Docker image
  script:
    - npm ci              # Clean install dependencies (Reads package-lock.json strictly)
    - npx playwright test # Execute tests
  artifacts:              # Save output (reports/traces) even if tests fail
    when: always
    paths:
      - playwright-report/
```

**Sharding (Parallelism):**
Splitting tests across multiple runners to reduce execution time.
`npx playwright test --shard=1/3`

---

## 3. Docker (Containerization)
Package tests and dependencies into a standard unit so they run identically everywhere ("Works on my machine" -> "Works everywhere").

```dockerfile
# Dockerfile

# 1. Use official image (pre-installed browsers + OS dependencies)
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

# 2. Set working directory in container
WORKDIR /app

# 3. Copy package.json and install first (Leverages Docker layer caching for speed)
COPY package*.json ./
RUN npm ci

# 4. Copy the rest of the framework code
COPY . .

# 5. Default command to execute
CMD ["npx", "playwright", "test"]
```
