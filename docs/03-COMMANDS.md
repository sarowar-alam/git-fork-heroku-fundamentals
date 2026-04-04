# 03 — Core Git Commands

> **Phase 3 of 7** | Every essential Git command with real examples.

---

## Table of Contents

1. [Setup & Init](#1-setup--init)
2. [Staging & Committing](#2-staging--committing)
3. [Viewing Status & History](#3-viewing-status--history)
4. [Diff & Comparison](#4-diff--comparison)
5. [Remote Operations](#5-remote-operations)
6. [Undoing Changes](#6-undoing-changes)
7. [Stashing](#7-stashing)
8. [Tagging](#8-tagging)
9. [Advanced Commands](#9-advanced-commands)

---

## 1. Setup & Init

### `git config`

Configure Git identity and preferences.

```bash
# Set name and email (required before first commit)
git config --global user.name "Sarowar Alam"
git config --global user.email "sarowar@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set default editor
git config --global core.editor "code --wait"

# Enable color output
git config --global color.ui auto

# Show all settings
git config --list

# Show single value
git config user.email
```

---

### `git init`

Initialize a new empty Git repository in the current directory.

```bash
# Initialize in current directory
cd my-project
git init
# Initialized empty Git repository in /home/user/my-project/.git/

# Initialize with a specific branch name
git init -b main

# Initialize in a new directory
git init new-project
```

What happens: Git creates a hidden `.git/` folder to track your project.

---

### `git clone`

Download a remote repository to your local machine.

```bash
# Clone via HTTPS
git clone https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

# Clone via SSH
git clone git@github.com:sarowar-alam/git-fork-heroku-fundamentals.git

# Clone into specific folder
git clone https://github.com/user/repo.git my-folder

# Clone specific branch
git clone -b develop https://github.com/user/repo.git

# Shallow clone (latest snapshot only — fast, no full history)
git clone --depth 1 https://github.com/user/repo.git

# Clone including submodules
git clone --recurse-submodules https://github.com/user/repo.git
```

---

## 2. Staging & Committing

### `git add`

Move changes from the working directory into the staging area (index).

```bash
# Stage a specific file
git add index.js

# Stage multiple files
git add index.js package.json

# Stage all changes in current directory (and subdirectories)
git add .

# Stage all tracked files (even in parent directories)
git add -A

# Stage part of a file interactively (choose which hunks to stage)
git add -p index.js

# Stage a directory
git add src/
```

**Staging area concept:**
```
Working Directory  →  git add  →  Staging Area  →  git commit  →  Repository
(your edits)                      (what will be                  (permanent history)
                                   in next commit)
```

---

### `git commit`

Save a snapshot of staged changes into the repository history.

```bash
# Commit with inline message
git commit -m "feat: add user authentication"

# Open editor for detailed message
git commit

# Stage all tracked files AND commit in one step
git commit -am "fix: correct typo in README"
# (Note: -a does NOT stage new untracked files)

# Amend the last commit (change message or add forgotten files)
git add forgotten-file.js
git commit --amend -m "feat: add user authentication and config"
# WARNING: Never amend commits that have been pushed to shared branches

# Commit with multiple message lines (header + body)
git commit -m "feat: add user login" -m "Validates email and password. Redirects to dashboard on success."

# Empty commit (useful to trigger CI/CD pipelines)
git commit --allow-empty -m "chore: trigger CI"

# Commit with a specific date
git commit --date="2026-01-01T12:00:00" -m "chore: backdate test commit"
```

**Good commit message conventions:**
```
type(scope): short summary (max 72 chars)

Longer description if needed. Explain WHY not WHAT.
The code shows what — the message shows why.

Refs: #123
```

Types: `feat` `fix` `docs` `style` `refactor` `test` `chore` `perf` `ci`

---

### `git rm`

Remove files from tracking (and optionally from disk).

```bash
# Remove file from Git AND from disk
git rm old-file.txt
git commit -m "chore: remove old-file.txt"

# Remove file from Git tracking only (keep file on disk)
git rm --cached secret.env
git commit -m "chore: stop tracking secret.env"
# Then add to .gitignore so it stays untracked

# Remove a directory
git rm -r old-folder/
```

---

### `git mv`

Rename or move a tracked file.

```bash
# Rename a file
git mv old-name.js new-name.js

# Move to a subdirectory
git mv index.js src/index.js

# This is equivalent to:
mv old-name.js new-name.js
git rm old-name.js
git add new-name.js
```

---

## 3. Viewing Status & History

### `git status`

Show what has changed — staged, unstaged, and untracked files.

```bash
# Full status output
git status

# Short/compact output
git status -s
# M  index.js    â† staged modification
#  M style.css   â† unstaged modification
# ?? new-file.js â† untracked file
# A  added.js    â† new file staged

# Show branch info + status
git status -sb
```

---

### `git log`

Show commit history.

```bash
# Full log (press q to exit)
git log

# One commit per line
git log --oneline
# a3f9c2b feat: add user auth
# b4d8e1f fix: correct redirect URL
# c5e7f2a Initial commit

# Show last N commits
git log -5

# Show commits by a specific author
git log --author="Sarowar"

# Show commits between dates
git log --after="2026-01-01" --before="2026-03-01"

# Show commits that changed a specific file
git log -- index.js

# Show commits matching a message pattern
git log --grep="feat"

# Show commits introducing a string change (pickaxe search)
git log -S "function login"

# Visual branch graph
git log --oneline --graph --all --decorate
# * a3f9c2b (HEAD -> main, origin/main) feat: add user auth
# | * b4d8e1f (origin/develop) feat: add dark mode
# |/
# * c5e7f2a Initial commit

# Show stats (files changed per commit)
git log --stat

# Show full diff per commit
git log -p

# Formatted output
git log --pretty=format:"%h %an %ar — %s"
# a3f9c2b Sarowar Alam 2 hours ago — feat: add user auth
```

---

### `git show`

Show details of a specific commit, tag, or tree.

```bash
# Show latest commit
git show

# Show a specific commit
git show a3f9c2b

# Show a specific file at a commit
git show a3f9c2b:index.js

# Show a tag
git show v1.0.0

# Short summary only
git show --stat a3f9c2b
```

---

### `git blame`

Show who last modified each line of a file — and in which commit.

```bash
# Blame a file
git blame index.js
# a3f9c2b (Sarowar Alam 2026-03-01 10:00 +0000  1) const express = require('express');
# a3f9c2b (Sarowar Alam 2026-03-01 10:00 +0000  2) const app = express();

# Blame specific line range
git blame -L 10,25 index.js

# Hide lines moved from another file
git blame -M index.js

# Ignore whitespace changes
git blame -w index.js
```

---

### `git reflog`

Show the history of HEAD — including commits you reset or deleted. Your safety net.

```bash
# Show reflog
git reflog
# a3f9c2b HEAD@{0}: commit: feat: add auth
# b4d8e1f HEAD@{1}: reset: moving to HEAD~1
# c5e7f2a HEAD@{2}: commit: add login form

# Show reflog for a specific branch
git reflog show feature/login

# Recover a commit you accidentally reset
git reset --hard HEAD@{2}
# or
git checkout -b recovery-branch HEAD@{2}
```

> **Golden rule:** `git reflog` can save you from almost any mistake. Even after `git reset --hard`.

---

## 4. Diff & Comparison

### `git diff`

Show what changed between the working directory, staging area, and commits.

```bash
# Unstaged changes (working dir vs staging area)
git diff

# Staged changes (staging area vs last commit)
git diff --staged
git diff --cached   # same thing

# Changes between two commits
git diff a3f9c2b b4d8e1f

# Changes between two branches
git diff main develop

# Changes in a specific file between commits
git diff a3f9c2b..HEAD -- index.js

# Show only changed file names (no diff content)
git diff --name-only main develop

# Show short stat summary
git diff --stat main develop
# index.js | 10 +++++-----
# 1 file changed, 5 insertions(+), 5 deletions(-)

# Word-level diff (more readable for prose)
git diff --word-diff
```

---

## 5. Remote Operations

### `git remote`

Manage connections to remote repositories.

```bash
# List remotes
git remote -v
# origin  https://github.com/sarowar-alam/repo.git (fetch)
# origin  https://github.com/sarowar-alam/repo.git (push)

# Add a remote
git remote add origin https://github.com/sarowar-alam/repo.git

# Add upstream (for forks)
git remote add upstream https://github.com/original-owner/repo.git

# Remove a remote
git remote remove upstream

# Rename
git remote rename origin backup

# Change URL
git remote set-url origin git@github.com:sarowar-alam/repo.git

# Inspect a remote
git remote show origin
```

---

### `git fetch`

Download changes from remote — but DON'T merge them into your local branch.

```bash
# Fetch from default remote (origin)
git fetch

# Fetch from all remotes
git fetch --all

# Fetch a specific remote
git fetch upstream

# Fetch a specific branch
git fetch origin develop

# Fetch and prune deleted remote branches
git fetch --prune
git fetch -p    # shorthand
```

After fetching, use `git log origin/main` to see what changed before merging.

---

### `git pull`

Fetch AND merge remote changes into current branch. (`git pull` = `git fetch` + `git merge`)

```bash
# Pull from tracked remote + branch
git pull

# Pull from specific remote and branch
git pull origin main

# Pull using rebase instead of merge (cleaner history)
git pull --rebase

# Pull and prune deleted remote tracking branches
git pull --prune

# Set default to always rebase on pull
git config --global pull.rebase true
```

---

### `git push`

Upload local commits to a remote repository.

```bash
# Push current branch to its tracked remote
git push

# Push to specific remote + branch
git push origin main

# Push and set upstream tracking (first push of a new branch)
git push -u origin feature/login

# Push all branches
git push --all origin

# Push all tags
git push --tags

# Push a specific tag
git push origin v1.0.0

# Delete a remote branch
git push origin --delete feature/old-branch
git push origin :feature/old-branch   # same

# Force push (DANGEROUS — rewrites remote history)
# Only use on your own private feature branches
git push --force-with-lease   # safer: fails if remote has new commits
git push -f                   # dangerous: no safety check
```

---

## 6. Undoing Changes

### `git restore`

Discard changes in working directory or unstage files. (Modern replacement for parts of `git checkout`)

```bash
# Discard unstaged changes to a file (restores from last commit)
git restore index.js

# Discard ALL unstaged changes
git restore .

# Unstage a staged file (keep changes in working dir)
git restore --staged index.js

# Unstage ALL staged files
git restore --staged .

# Restore a file to what it was in a specific commit
git restore --source a3f9c2b index.js
```

---

### `git reset`

Move the HEAD and branch pointer backward. Can also unstage files.

```bash
# Unstage a file (keep changes in working dir) — same as restore --staged
git reset HEAD index.js

# Undo last commit — keep changes STAGED
git reset --soft HEAD~1

# Undo last commit — keep changes in WORKING DIRECTORY (unstaged)
git reset HEAD~1         # default is --mixed
git reset --mixed HEAD~1

# Undo last commit — DISCARD all changes (DESTRUCTIVE)
git reset --hard HEAD~1

# Reset to a specific commit
git reset --hard a3f9c2b

# Undo last 3 commits but keep work
git reset --soft HEAD~3
```

**The 3 reset modes:**
```
--soft   → moves HEAD only. Changes stay staged.
--mixed  → moves HEAD + unstages changes. Changes in working dir.
--hard   → moves HEAD + discards all changes. CANNOT UNDO without reflog.
```

---

### `git revert`

Create a new commit that undoes a previous commit. **Safe — never rewrites history.**

```bash
# Revert the last commit
git revert HEAD

# Revert a specific commit by hash
git revert a3f9c2b

# Revert but don't auto-commit (stage the revert for you to commit)
git revert --no-commit a3f9c2b

# Revert a range of commits (oldest to newest)
git revert a3f9c2b..HEAD
```

> **Rule:** Use `revert` on shared/public branches. Use `reset` only on private local branches.

---

### `git clean`

Remove untracked files and directories from the working directory.

```bash
# Dry run — show what WOULD be deleted (safe to run first)
git clean -n

# Delete untracked files
git clean -f

# Delete untracked files AND directories
git clean -fd

# Include ignored files (e.g., build artifacts, node_modules)
git clean -fdx

# Interactive mode
git clean -i
```

---

## 7. Stashing

Stash temporarily shelves work in progress so you can switch context.

### `git stash`

```bash
# Save current changes to stash
git stash
git stash push   # same

# Stash with a label
git stash push -m "WIP: login form validation"

# Stash including untracked files
git stash push -u

# Stash including ignored files too
git stash push -a

# List all stashes
git stash list
# stash@{0}: WIP on main: a3f9c2b feat: add auth
# stash@{1}: WIP: login form validation

# Apply the latest stash (keep stash)
git stash apply

# Apply and remove from stash list (pop)
git stash pop

# Apply a specific stash
git stash apply stash@{1}

# Drop (delete) a specific stash
git stash drop stash@{0}

# Drop all stashes
git stash clear

# Create a new branch from a stash
git stash branch feature/login stash@{0}

# Show what's in a stash
git stash show
git stash show -p stash@{0}   # full diff
```

**Typical stash workflow:**
```bash
# You're on main, realize you made changes in the wrong branch
git stash              # shelve your work
git switch feature/login  # go to the right branch
git stash pop          # restore your work there
```

---

## 8. Tagging

Tags mark specific commits — typically used for release versions.

```bash
# List all tags
git tag

# Filter tags
git tag -l "v1.*"

# Create a lightweight tag (just a pointer)
git tag v1.0.0

# Create an annotated tag (recommended — has metadata)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag a specific past commit
git tag -a v0.9.0 a3f9c2b -m "Beta release"

# Show tag details
git show v1.0.0

# Push a specific tag to remote
git push origin v1.0.0

# Push ALL tags to remote
git push origin --tags

# Delete a local tag
git tag -d v1.0.0

# Delete a remote tag
git push origin --delete v1.0.0
git push origin :refs/tags/v1.0.0   # same
```

**Semantic versioning convention:** `vMAJOR.MINOR.PATCH`
- `MAJOR` — breaking changes
- `MINOR` — new features (backward compatible)
- `PATCH` — bug fixes

---

## 9. Advanced Commands

### `git cherry-pick`

Apply a specific commit from one branch onto another.

```bash
# Pick a single commit
git cherry-pick a3f9c2b

# Pick multiple commits
git cherry-pick a3f9c2b b4d8e1f

# Pick a range of commits
git cherry-pick a3f9c2b..c5e7f2a

# Pick without committing (stage only)
git cherry-pick --no-commit a3f9c2b

# If conflict occurs:
git cherry-pick --continue   # after resolving conflict
git cherry-pick --abort      # cancel cherry-pick
```

---

### `git bisect`

Binary search through commit history to find which commit introduced a bug.

```bash
# Start bisect session
git bisect start

# Mark current commit as bad (has the bug)
git bisect bad

# Mark a known good commit (before the bug existed)
git bisect good v1.0.0

# Git checks out the middle commit automatically
# Test your app, then tell Git:
git bisect good    # if this commit is fine
git bisect bad     # if this commit has the bug

# Git narrows down and finds the culprit commit
# When done:
git bisect reset   # return to original branch

# Automated bisect with a test script
git bisect run npm test
```

---

### `git archive`

Export a snapshot of your repo as a zip/tar — without the `.git/` folder.

```bash
# Export main branch as a zip
git archive --format=zip HEAD > project.zip

# Export a specific tag as tar.gz
git archive --format=tar.gz --prefix=v1.0.0/ v1.0.0 > v1.0.0.tar.gz

# Export only a subdirectory
git archive HEAD:src/ --format=zip > src.zip
```

---

### `git submodule`

Include another Git repo as a subdirectory of your repo.

```bash
# Add a submodule
git submodule add https://github.com/user/lib.git lib/

# Initialize submodules after cloning a repo that has them
git submodule init
git submodule update
# Or in one step:
git submodule update --init --recursive

# Pull latest changes for all submodules
git submodule update --remote

# Remove a submodule
git submodule deinit lib/
git rm lib/
```

---

### `git worktree`

Check out multiple branches simultaneously in separate directories — without cloning again.

```bash
# Add a worktree for a branch
git worktree add ../hotfix-branch hotfix/payment-bug

# List all worktrees
git worktree list

# Remove a worktree
git worktree remove ../hotfix-branch
```

---

### Complete Command Summary Table

| Command | Category | What it does |
|---|---|---|
| `git config` | Setup | Configure Git settings |
| `git init` | Setup | Initialize new repo |
| `git clone` | Setup | Copy a remote repo |
| `git add` | Stage | Stage changes |
| `git commit` | Stage | Save snapshot |
| `git rm` | Stage | Remove tracked files |
| `git mv` | Stage | Move/rename tracked files |
| `git status` | Inspect | Show working tree status |
| `git log` | Inspect | Show commit history |
| `git show` | Inspect | Show commit details |
| `git blame` | Inspect | Show who changed each line |
| `git reflog` | Inspect | Show HEAD movement history |
| `git diff` | Compare | Show what changed |
| `git fetch` | Remote | Download (no merge) |
| `git pull` | Remote | Download + merge |
| `git push` | Remote | Upload to remote |
| `git remote` | Remote | Manage remote connections |
| `git restore` | Undo | Discard working dir changes |
| `git reset` | Undo | Move HEAD backward |
| `git revert` | Undo | Safe undo via new commit |
| `git clean` | Undo | Remove untracked files |
| `git stash` | Stash | Shelve work in progress |
| `git tag` | Tag | Mark release points |
| `git cherry-pick` | Advanced | Copy specific commit |
| `git bisect` | Advanced | Find bug-introducing commit |
| `git archive` | Advanced | Export repo snapshot |
| `git submodule` | Advanced | Embed another repo |
| `git worktree` | Advanced | Multiple checkouts |

---

**Next:** [04 — Branching Deep Dive →](./04-BRANCHES.md)

---

## 🧑‍💻 Author

*Md. Sarowar Alam*  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/sarowar/

---
