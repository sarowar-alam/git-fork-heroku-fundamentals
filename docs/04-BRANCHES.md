# 04 — Branching Deep Dive

> **Phase 4 of 7** | Master branches, merging, rebasing, and workflows.

---

## Table of Contents

1. [What is a Branch?](#1-what-is-a-branch)
2. [Branch Commands](#2-branch-commands)
3. [Merging](#3-merging)
4. [Conflict Resolution](#4-conflict-resolution)
5. [Rebasing](#5-rebasing)
6. [Cherry-Pick](#6-cherry-pick)
7. [Branch Naming Conventions](#7-branch-naming-conventions)
8. [Branching Strategies](#8-branching-strategies)
9. [Modern Git Commands: switch & restore](#9-modern-git-commands-switch--restore)

---

## 1. What is a Branch?

A **branch** is a lightweight, movable pointer to a commit. Creating a branch costs nothing — it's just a 41-byte file in `.git/refs/heads/`.

```
main:     A â† B â† C â† D (HEAD)
                    â†‘
feature:            E â† F (HEAD of feature)
```

Each team member works on their own branch. When done, branches are merged back.

**Why use branches?**
- Isolate work — main stays clean and deployable at all times
- Enable parallel development — multiple features at once
- Safe experimentation — try things without risk
- Required for Pull Requests on GitHub

---

## 2. Branch Commands

### Create & Switch

```bash
# List all local branches (* = current branch)
git branch
# * main
#   feature/login
#   develop

# List all branches including remote-tracking
git branch -a
# * main
#   feature/login
#   remotes/origin/main
#   remotes/origin/develop

# List remote branches only
git branch -r

# Create a new branch (does NOT switch to it)
git branch feature/login

# Switch to an existing branch
git switch feature/login        # modern (Git 2.23+)
git checkout feature/login      # classic (still works)

# Create AND switch in one command
git switch -c feature/login     # modern
git checkout -b feature/login   # classic

# Create a branch from a specific commit or tag
git switch -c hotfix/payment v1.2.0
git checkout -b hotfix/payment a3f9c2b

# Create a branch from a remote branch
git switch -c develop origin/develop
git checkout -b develop origin/develop

# Switch to the previous branch
git switch -
git checkout -
```

---

### Rename & Delete

```bash
# Rename current branch
git branch -m new-name

# Rename a specific branch
git branch -m old-name new-name

# Delete a branch (only if fully merged)
git branch -d feature/login

# Force delete (even if not merged — careful!)
git branch -D feature/abandoned

# Delete a remote branch
git push origin --delete feature/login
git push origin :feature/login   # same (older syntax)

# Prune stale remote-tracking references
git fetch --prune
git remote prune origin
```

---

### Inspect Branches

```bash
# Show last commit on each branch
git branch -v
# * main           a3f9c2b feat: add user auth
#   feature/login  b4d8e1f WIP: login form
#   develop        c5e7f2a feat: add profile page

# Show branches that ARE merged into current branch
git branch --merged

# Show branches NOT yet merged into current branch
git branch --no-merged

# Show which remote a local branch tracks
git branch -vv
# * main  a3f9c2b [origin/main] feat: add user auth
```

---

## 3. Merging

Merging brings changes from one branch into another.

### Fast-Forward Merge

If the target branch has no new commits since the branch point, Git just moves the pointer forward — no merge commit created.

```
Before:
main:     A â† B â† C
                   â†‘
feature:           D â† E

After (fast-forward):
main:     A â† B â† C â† D â† E
```

```bash
git switch main
git merge feature/login
# Updating c5e7f2a..b4d8e1f
# Fast-forward
#  login.js | 30 +++++++++++++++
```

---

### 3-Way Merge

If both branches have diverged (both have new commits), Git creates a **merge commit** that has two parents.

```
Before:
main:     A â† B â† C â† D
                   â†‘
feature:           E â† F

After (3-way merge):
main:     A â† B â† C â† D â† M (merge commit, 2 parents)
                   â†‘       â†‘
feature:           E â† F â”€â”€â”˜
```

```bash
git switch main
git merge feature/login
# Merge made by the 'ort' strategy.
# login.js  | 30 +++++++++
# 1 file changed, 30 insertions(+)
```

---

### Merge Options

```bash
# Merge with a merge commit always (even if fast-forward is possible)
git merge --no-ff feature/login

# Squash all feature commits into one staged change (you commit it manually)
git merge --squash feature/login
git commit -m "feat: add login functionality"

# Abort a merge that has conflicts
git merge --abort

# Continue after resolving conflicts
git merge --continue
```

---

## 4. Conflict Resolution

A **conflict** occurs when the same line of the same file was changed on both branches.

### Example conflict

You're merging `feature/login` into `main`. Both changed `app.js` line 5:

```
<<<<<<< HEAD (main)
const PORT = 3000;
=======
const PORT = process.env.PORT || 3000;
>>>>>>> feature/login
```

**What each part means:**
- `<<<<<<< HEAD` — your current branch's version
- `=======` — separator
- `>>>>>>> feature/login` — incoming branch's version

### Step-by-step conflict resolution

```bash
# 1. Start the merge (conflict occurs)
git merge feature/login
# CONFLICT (content): Merge conflict in app.js

# 2. See all conflicted files
git status
# both modified: app.js

# 3. Open the file and resolve manually
# Edit app.js to the correct final state:
# const PORT = process.env.PORT || 3000;  â† keep the better version

# 4. Stage the resolved file
git add app.js

# 5. Complete the merge
git commit
# (Git pre-fills the merge commit message)
```

### VS Code Conflict Resolution (easier)

When VS Code detects a conflict it shows buttons:
- **Accept Current Change** — keep main's version
- **Accept Incoming Change** — keep feature's version
- **Accept Both Changes** — keep both
- **Compare Changes** — see side-by-side diff

### Tools for conflict resolution

```bash
# Open merge tool (if configured)
git mergetool

# Configure VS Code as merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

---

## 5. Rebasing

Rebase rewrites commit history by replaying commits on top of a different base.

### Basic Rebase

```
Before rebase:
main:    A â† B â† C â† D
                  â†‘
feature:          E â† F

After: git rebase main (while on feature)
main:    A â† B â† C â† D
                       â†‘
feature:               E' â† F'  (new commits, same changes)
```

```bash
# Rebase current branch onto main
git switch feature/login
git rebase main

# Rebase a specific branch onto main
git rebase main feature/login

# If conflict during rebase:
# 1. Resolve conflict in the file
# 2. Stage the file
git add app.js
# 3. Continue
git rebase --continue

# Abort rebase entirely
git rebase --abort
```

---

### Interactive Rebase

Interactively rewrite, squash, reorder, or edit commits. **Most powerful Git feature.**

```bash
# Rebase last 3 commits interactively
git rebase -i HEAD~3
```

This opens an editor with:
```
pick a3f9c2b feat: add user auth
pick b4d8e1f fix: typo
pick c5e7f2a chore: update deps
```

Change `pick` to:
| Command | What it does |
|---|---|
| `pick` | Keep commit as-is |
| `reword` (r) | Keep commit, edit message |
| `edit` (e) | Stop to amend the commit |
| `squash` (s) | Merge into previous commit |
| `fixup` (f) | Like squash but discard message |
| `drop` (d) | Delete the commit entirely |
| `exec` (x) | Run a shell command |

**Example — squash 3 commits into 1:**
```
pick a3f9c2b feat: add user auth
squash b4d8e1f fix: typo            â† will merge into above
squash c5e7f2a chore: lint fix      â† will merge into above
```

---

### Rebase vs Merge — When to Use Each

| | Merge | Rebase |
|---|---|---|
| **History** | Preserves true history with merge commits | Creates linear, clean history |
| **Use for** | Public/shared branches, release merges | Private feature branches, before PR |
| **Safety** | Always safe | Never rebase shared/public branches |
| **Traceability** | Shows when branches diverged | Harder to trace original timing |

> **Golden Rule:** Never rebase branches that other people have based their work on.

---

## 6. Cherry-Pick

Apply a specific commit to the current branch.

```bash
# Pick a single commit from another branch
git cherry-pick a3f9c2b

# Pick without committing (stage only)
git cherry-pick --no-commit a3f9c2b

# Pick multiple commits
git cherry-pick a3f9c2b b4d8e1f

# If conflict:
git cherry-pick --continue   # after resolving
git cherry-pick --abort      # cancel

# Example: hotfix applied to both main and develop
git switch main
git cherry-pick hotfix-commit-hash
git switch develop
git cherry-pick hotfix-commit-hash
```

---

## 7. Branch Naming Conventions

Consistent naming makes branches easy to understand at a glance.

| Prefix | Purpose | Example |
|---|---|---|
| `feature/` | New features | `feature/user-authentication` |
| `fix/` or `bugfix/` | Bug fixes | `fix/login-redirect-error` |
| `hotfix/` | Critical fix for production | `hotfix/payment-crash` |
| `release/` | Release preparation | `release/v1.2.0` |
| `docs/` | Documentation only | `docs/update-api-guide` |
| `chore/` | Maintenance tasks | `chore/upgrade-dependencies` |
| `test/` | Adding/fixing tests | `test/add-auth-unit-tests` |
| `refactor/` | Code restructuring | `refactor/split-user-service` |
| `experiment/` | Throwaway experiments | `experiment/new-cache-strategy` |

**Rules:**
- Use lowercase and hyphens only (no spaces, no underscores)
- Keep names short but descriptive
- Include ticket/issue number if applicable: `feature/JIRA-123-user-auth`

---

## 8. Branching Strategies

### Git Flow

The classic strategy for teams with scheduled releases.

```
main          â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º (production)
                \                           /
hotfix           â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
                  \
release/v1.0       â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
                     \                  /
develop          â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º (integration)
                  \          /    \        /
feature/login      â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€      â”€â”€â”€â”€â”€â”€â”€â”€
feature/profile
```

**Branches:**
| Branch | Long-lived | Purpose |
|---|---|---|
| `main` | Yes | Production code. Always deployable. |
| `develop` | Yes | Integration branch. Feature branches merge here. |
| `release/*` | Short | Prepare releases from develop. Bug fixes only. |
| `hotfix/*` | Short | Urgent patches direct from main. |
| `feature/*` | Short | New features from develop. |

**Workflow:**
```bash
# Start a new feature
git switch -c feature/login develop

# Complete feature
git switch develop
git merge --no-ff feature/login
git branch -d feature/login

# Start a release
git switch -c release/v1.0.0 develop
# bump version, last-minute fixes
git switch main
git merge --no-ff release/v1.0.0
git tag v1.0.0
git switch develop
git merge --no-ff release/v1.0.0
git branch -d release/v1.0.0
```

---

### Trunk-Based Development (TBD)

Simpler, used by large tech companies (Google, Facebook). Everyone commits to `main` frequently.

```
main    â”€â”€ A â”€â”€ B â”€â”€ C â”€â”€ D â”€â”€ E â”€â”€â–º  (always deployable)
            â†‘       â†‘
feature  â”€â”€â”€â”˜       â””â”€â”€â”€ (short-lived, <2 days)
```

**Key practices:**
- Short-lived branches (max 1-2 days)
- Merge to main multiple times per day
- Feature flags hide incomplete features
- Requires strong CI/CD pipeline

```bash
# Short-lived feature branch
git switch -c feature/add-button
# make changes, tests pass
git switch main
git merge feature/add-button    # fast-forward
git branch -d feature/add-button
git push

# Or commit directly to main (only for tiny changes with CI)
git switch main
git commit -am "fix: correct button label"
git push
```

---

### GitHub Flow (simplified)

Good for teams that deploy continuously.

1. `main` is always deployable
2. Branch from `main` for any work
3. Open a PR with clear description
4. Review and discuss
5. Deploy from branch to staging, test
6. Merge to `main`

---

## 9. Modern Git Commands: switch & restore

Git 2.23 (2019) split the overloaded `git checkout` into two focused commands.

### `git switch` — change branches

```bash
# Switch to a branch
git switch main                    # was: git checkout main

# Create and switch
git switch -c feature/new          # was: git checkout -b feature/new

# Switch to previous branch
git switch -                       # was: git checkout -

# Create branch from a tag/commit
git switch -c hotfix v1.2.0        # was: git checkout -b hotfix v1.2.0
```

### `git restore` — restore files

```bash
# Discard working directory changes
git restore index.js               # was: git checkout -- index.js

# Unstage a file
git restore --staged index.js      # was: git reset HEAD index.js

# Restore file to state at a commit
git restore --source a3f9c2b index.js
```

> `git checkout` still works — these are just clearer alternatives.

---

**Next:** [05 — Branch Protection Rules →](./05-BRANCH-PROTECTION.md)

---

## 🧑‍💻 Author

*Md. Sarowar Alam*  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/sarowar/

---
