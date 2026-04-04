# 06 — Fork & Collaborate

> **Phase 6 of 7** | Fork repos, sync with upstream, open Pull Requests, and do code reviews.

---

## Table of Contents

1. [What is a Fork?](#1-what-is-a-fork)
2. [Fork a Repository](#2-fork-a-repository)
3. [Clone Your Fork](#3-clone-your-fork)
4. [Add the Upstream Remote](#4-add-the-upstream-remote)
5. [Keep Your Fork in Sync](#5-keep-your-fork-in-sync)
6. [Make Changes and Push](#6-make-changes-and-push)
7. [Open a Pull Request](#7-open-a-pull-request)
8. [Code Review Process](#8-code-review-process)
9. [Merge Strategies](#9-merge-strategies)
10. [After the PR is Merged](#10-after-the-pr-is-merged)
11. [Complete Collaboration Workflow Summary](#11-complete-collaboration-workflow-summary)

---

## 1. What is a Fork?

A **fork** is a personal copy of someone else's repository, hosted under your own GitHub account.

```
Original repo:    github.com/original-owner/project
                          â†“ fork
Your fork:        github.com/sarowar-alam/project
                          â†“ clone
Your machine:     ~/projects/project
```

Unlike cloning, a fork:
- Lives on GitHub (not just your machine)
- Is linked to the original ("upstream")
- Lets you open Pull Requests back to the original
- Lets you experiment freely without affecting the original

**Fork vs Clone:**
| | Fork | Clone |
|---|---|---|
| Lives on | GitHub (your account) | Your machine |
| Linked to original? | Yes — can open PRs | No — independent copy |
| Used for | Contributing to OSS, team collaboration | Working on your own repo |

---

## 2. Fork a Repository

### On GitHub UI

1. Go to the repo you want to fork (e.g., `github.com/original/project`)
2. Click the **Fork** button (top-right)
3. Select **your personal account** as the owner
4. Optionally rename it or add a description
5. Check/uncheck "Copy the `main` branch only" as needed
6. Click **Create fork**

GitHub creates `github.com/sarowar-alam/project` — your own full copy.

### Using GitHub CLI

```bash
gh repo fork original-owner/project
# Clones it to your machine automatically

gh repo fork original-owner/project --clone=false
# Fork only, don't clone
```

---

## 3. Clone Your Fork

Always clone YOUR fork, not the original.

```bash
# HTTPS
git clone https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

# SSH
git clone git@github.com:sarowar-alam/git-fork-heroku-fundamentals.git

# Move into the project
cd git-fork-heroku-fundamentals
```

The `origin` remote now points to YOUR fork:
```bash
git remote -v
# origin  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (fetch)
# origin  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (push)
```

---

## 4. Add the Upstream Remote

The **upstream** remote points to the original repo. You need this to pull in any updates the original owner makes.

```bash
# Add upstream
git remote add upstream https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

# Verify
git remote -v
# origin    https://github.com/YOUR-USERNAME/git-fork-heroku-fundamentals.git (fetch)
# origin    https://github.com/YOUR-USERNAME/git-fork-heroku-fundamentals.git (push)
# upstream  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (fetch)
# upstream  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (push)
```

**Convention:**
- `origin` → your fork on GitHub
- `upstream` → the original repo you forked from

---

## 5. Keep Your Fork in Sync

As the original repo gets new commits, your fork falls behind. Sync it regularly.

```bash
# Step 1: Fetch changes from upstream (downloads, doesn't merge)
git fetch upstream

# Step 2: Switch to your main branch
git switch main

# Step 3: Merge upstream/main into your local main
git merge upstream/main

# Step 4: Push the updated main to your fork on GitHub
git push origin main
```

**One-liner (fetch + merge):**
```bash
git pull upstream main
git push origin main
```

**Using rebase (cleaner, linear history):**
```bash
git fetch upstream
git switch main
git rebase upstream/main
git push --force-with-lease origin main
# (force-with-lease is needed because rebase rewrites history)
```

**Via GitHub UI (easier for one-off syncing):**
1. Go to your fork on GitHub
2. Click **Sync fork** → **Update branch**

---

## 6. Make Changes and Push

Always work on a **feature branch** — never directly on `main`.

```bash
# 1. Ensure you're on main and it's up-to-date
git switch main
git pull upstream main

# 2. Create a feature branch
git switch -c feature/add-dark-mode

# 3. Make your changes
# edit files...

# 4. Stage and commit
git add .
git commit -m "feat: add dark mode toggle"

# 5. Push the feature branch to YOUR fork
git push -u origin feature/add-dark-mode
```

After pushing, GitHub will show a banner:
> "Your recently pushed branches: feature/add-dark-mode — **Compare & pull request**"

---

## 7. Open a Pull Request

A **Pull Request (PR)** proposes your changes to be merged into a branch of the original (or another person's) repo.

### Steps

1. On GitHub, click **Compare & pull request** (after pushing)
   — OR go to the original repo → **Pull requests** → **New pull request**

2. Set:
   - **Base repository**: `sarowar-alam/git-fork-heroku-fundamentals`
   - **Base branch**: `main` (or `develop`)
   - **Head repository**: `your-username/git-fork-heroku-fundamentals`
   - **Compare branch**: `feature/add-dark-mode`

3. Fill in the PR form:

**Title:** `feat: add dark mode toggle`

**Description** (use this template):
```markdown
## What does this PR do?
Adds a dark mode toggle button to the navigation bar.
Users can switch between light and dark themes. Preference is saved in localStorage.

## Why?
Addresses issue #42 — dark mode request from users.

## How to test?
1. Open the app
2. Click the moon icon in the nav
3. Verify the theme switches and persists on refresh

## Screenshots
[attach before/after screenshots]

## Checklist
- [x] Tested locally
- [x] No console errors
- [x] Responsive on mobile
```

4. Assign **Reviewers**, **Labels** (e.g., `enhancement`), **Milestone**
5. Click **Create pull request**

---

## 8. Code Review Process

### As a Reviewer

1. Open the PR → go to **Files changed** tab
2. Review each file's diff (red = deleted, green = added)
3. **Leave inline comments**: click the `+` on any line
4. **Suggest changes**: in a comment, use:
   ````
   ```suggestion
   const PORT = process.env.PORT || 3000;
   ```
   ````
   The author can apply your suggestion with one click.

5. When done reviewing → click **Review changes**:
   - **Comment**: general feedback, no block
   - **Approve**: ✅ changes look good, ready to merge
   - **Request changes**: âŒ must address issues before merging

### As the PR Author

```bash
# Apply reviewer feedback: make changes and push more commits
git add .
git commit -m "fix: address review feedback — use env var for PORT"
git push
# PR updates automatically — reviewer is notified

# Re-request review after addressing feedback
# (click "Re-request review" button next to reviewer's name on GitHub)
```

### Resolve Conversations

Once a review comment is addressed:
- Click **Resolve conversation** on the comment thread
- This collapses it so reviewers can focus on outstanding issues

---

## 9. Merge Strategies

When merging a PR, GitHub offers three strategies. Choose based on your branch strategy.

### Create a Merge Commit (default)

```
main: A â”€â”€ B â”€â”€ C â”€â”€ M
                     â†‘ (merge commit with 2 parents)
feature:      D â”€â”€ E â”˜
```

- Preserves full branch history
- Shows exactly when branches diverged and merged
- Good for: Git Flow, tracking feature integration dates

```bash
# Equivalent git command
git merge --no-ff feature/add-dark-mode
```

---

### Squash and Merge

```
main: A â”€â”€ B â”€â”€ C â”€â”€ S
                     â†‘ (single commit — all feature commits squashed)
feature:      D â”€â”€ E (D + E squashed into S)
```

- All feature commits become one clean commit on main
- Cleaner history — no WIP commits
- Feature branch history is discarded after merge
- Good for: GitHub Flow, teams that want clean main history

```bash
git merge --squash feature/add-dark-mode
git commit -m "feat: add dark mode toggle (#45)"
```

---

### Rebase and Merge

```
main: A â”€â”€ B â”€â”€ C â”€â”€ D' â”€â”€ E'
                      â†‘ replayed commits (D and E rebased)
```

- Each feature commit is replayed on top of main
- Linear history — no merge commit
- Commits get new hashes
- Good for: trunk-based development, linear history preference

```bash
git rebase main feature/add-dark-mode
git switch main
git merge feature/add-dark-mode   # fast-forward
```

---

**Which to use?**
| Strategy | History | Team size | Recommended when |
|---|---|---|---|
| Merge commit | Bushy | Any | You want to see branch history |
| Squash | Clean | Small-medium | Each PR = one logical unit |
| Rebase | Linear | Advanced | Everyone understands rebase |

---

## 10. After the PR is Merged

```bash
# 1. Switch back to main
git switch main

# 2. Pull the latest main (which now has your merged changes)
git pull upstream main

# 3. Update your fork
git push origin main

# 4. Delete your local feature branch (it's merged, no longer needed)
git branch -d feature/add-dark-mode

# 5. Delete the remote feature branch on your fork
git push origin --delete feature/add-dark-mode
```

---

## 11. Complete Collaboration Workflow Summary

```bash
# â”€â”€ SETUP (one time) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
git clone git@github.com:sarowar-alam/project.git
cd project
git remote add upstream https://github.com/original/project.git

# â”€â”€ BEFORE EACH FEATURE â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
git switch main
git fetch upstream
git merge upstream/main          # sync with latest
git push origin main             # update your fork

# â”€â”€ WORKING ON A FEATURE â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
git switch -c feature/my-feature
# ... make changes ...
git add .
git commit -m "feat: describe the feature"
git push -u origin feature/my-feature
# → Go to GitHub → Open Pull Request

# â”€â”€ DURING REVIEW (if feedback) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
# ... make fixes ...
git add .
git commit -m "fix: address review comments"
git push
# → Re-request review on GitHub

# â”€â”€ AFTER PR IS MERGED â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
git switch main
git pull upstream main
git push origin main
git branch -d feature/my-feature
git push origin --delete feature/my-feature

# â”€â”€ REPEAT for next feature â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
```

---

**Next:** [07 — Deploy to Heroku →](./07-HEROKU-DEPLOY.md)

---

## 🧑‍💻 Author

*Md. Sarowar Alam*  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/sarowar/

---
