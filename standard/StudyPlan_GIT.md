# Study Plan: Git & Version Control

---

## 1. Rebase vs Merge
**Concept Used:** Git History Management, Branch Integration.

**Context Story:**
When working on a feature branch, the main branch will inevitably get updated by other developers. You need to pull those changes in. `git merge` takes both branches and ties them together with a "merge commit". `git rebase` temporarily rewinds your commits, pulls the latest main, and plays your commits on top of it. Rebase keeps the project history linear and clean, which is highly preferred in enterprise environments.

**Snippet:**
```bash
# You are on feature/login-tests
# Fetch the latest from remote
git fetch origin

# Apply your commits ON TOP of the latest main
git rebase origin/main

# If conflicts occur, resolve them, then:
# git add .
# git rebase --continue
```

**How to Explain Code in Interview:**
"When I need to update my feature branch, I prefer `git rebase` over `merge`. The commands are `git fetch origin` followed by `git rebase origin/main`. If there are conflicts, Git pauses the rebase. I resolve the conflicts, run `git add`, and then `git rebase --continue`. I explain to the interviewer that this creates a perfectly linear Git history, making it much easier to trace bugs using `git bisect` later on."

**Questions that might be asked:**
- "What happens if you rebase a branch that you've already pushed to the remote?" *(Answer: You will rewrite history, which causes issues for anyone else who pulled that branch. Rule of thumb: never rebase public/shared branches).*
- "How do you squash multiple commits into a single commit before raising a PR?" *(Answer: Using interactive rebase `git rebase -i`)*

---

## 2. Resolving Merge Conflicts
**Concept Used:** Conflict Resolution, Collaboration.

**Context Story:**
Merge conflicts happen when two people edit the exact same line of code. As a senior SDET, you shouldn't panic when this happens. You need to confidently explain how you interpret Git's conflict markers.

**Snippet:**
```text
<<<<<<< HEAD
const timeout = 5000;
=======
const timeout = 10000;
>>>>>>> origin/main
```

**How to Explain Code in Interview:**
"In an interview, I explain that when a conflict occurs, Git injects conflict markers. `<<<<<<< HEAD` represents the changes in my current local branch, while `>>>>>>>` represents the incoming changes. My job is to manually edit the file, choose the correct line (or combine them), delete the conflict markers, save the file, and then commit the resolution. I might also mention using tools like VS Code's merge editor to make this visual and safer."

**Questions that might be asked:**
- "How do you prevent merge conflicts in automation frameworks?" *(Answer: Modularizing code, using Page Object Models, and having developers work on separate components).*
- "If you realize you made a mistake during a merge, how do you undo it?" *(Answer: `git merge --abort` before committing).*
