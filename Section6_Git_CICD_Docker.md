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
