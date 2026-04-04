# 05 — Branch Protection Rules

> **Phase 5 of 7** | Restrict branch deletion, block history rewrites, require Pull Requests and reviews.

---

## Table of Contents

1. [Why Protect Branches?](#1-why-protect-branches)
2. [Setting Up Branch Protection on GitHub](#2-setting-up-branch-protection-on-github)
3. [Restrict Branch Deletion](#3-restrict-branch-deletion)
4. [Block Force Pushes (History Protection)](#4-block-force-pushes-history-protection)
5. [Require Pull Requests Before Merging](#5-require-pull-requests-before-merging)
6. [Require Review Approvals](#6-require-review-approvals)
7. [Require Status Checks (CI)](#7-require-status-checks-ci)
8. [Admin Bypass Options](#8-admin-bypass-options)
9. [Protecting the develop Branch](#9-protecting-the-develop-branch)
10. [Rulesets (GitHub's Modern System)](#10-rulesets-githubs-modern-system)

---

## 1. Why Protect Branches?

Without protection, any collaborator with Write access can:
- `git push --force` → overwrite and rewrite history of `main`
- `git push origin --delete main` → delete the production branch
- Push directly to `main`, skipping code review

Branch protection prevents all of this.

---

## 2. Setting Up Branch Protection on GitHub

> You must be a **repository Admin** or **Owner** to configure protection rules.

**Navigation:**
```
GitHub Repo → Settings → Branches → "Add branch ruleset" or "Add classic branch protection rule"
```

There are two systems:
- **Branch protection rules** (classic) — per branch, simple
- **Rulesets** (new, recommended) — more powerful, supports multiple branches

We'll cover both, starting with the classic system.

---

## 3. Restrict Branch Deletion

**What it does:** Prevents anyone from deleting the protected branch — even users with Admin role (unless they enable "Allow deletions" bypass).

### Enable it (Classic Rules)

1. Go to **Settings → Branches → Add rule**
2. In **Branch name pattern**, type: `main`
3. Scroll down to find ✅ **Do not allow deletions**
4. Click **Create** (or **Save changes**)

### What happens when someone tries to delete

```bash
git push origin --delete main
# remote: error: GH006: Protected branch update failed for refs/heads/main.
# remote: error: Cannot delete protected branch.
# To github.com:sarowar-alam/repo.git
#  ! [remote rejected] main (protected branch hook declined)
# error: failed to push some refs to 'github.com:sarowar-alam/repo.git'
```

### Verify it's working

1. Go to the repo → **Branches**
2. Try to click the trash icon next to `main` — it will be grayed out or blocked

---

## 4. Block Force Pushes (History Protection)

**What it does:** Prevents `git push --force` (and `--force-with-lease`) to the protected branch. This ensures history is never rewritten.

Why force push is dangerous:
```bash
# Developer A rewrites history:
git rebase -i HEAD~5   # squash 5 commits into 1
git push --force origin main   # overwrites remote history

# Developer B, who already pulled those 5 commits, is now out of sync
# Their local main has commits that no longer exist on origin
# git pull will create a tangled, confusing merge
```

### Enable it (Classic Rules)

1. **Settings → Branches → Add/edit rule for `main`**
2. Scroll to ✅ **Block force pushes**
3. Save

### What happens when blocked

```bash
git push --force origin main
# remote: error: GH006: Protected branch update failed for refs/heads/main.
# remote: error: Cannot force-push to a protected branch.
# To github.com:sarowar-alam/repo.git
#  ! [remote rejected] main (protected branch hook declined)
```

Even `--force-with-lease` is blocked:
```bash
git push --force-with-lease origin main
# Same error as above — force is force on protected branches
```

---

## 5. Require Pull Requests Before Merging

**What it does:** No one can push commits directly to the protected branch. All changes must come through a Pull Request.

### Enable it (Classic Rules)

1. **Settings → Branches → Add/edit rule for `main`**
2. ✅ **Require a pull request before merging**
3. Save

### What happens when someone tries to push directly

```bash
git push origin main
# remote: error: GH006: Protected branch update failed for refs/heads/main.
# remote: error: At least 1 approving review is required by reviewers with write access.
# (if reviews required) OR
# remote: error: Changes must be made through a pull request.
```

### The correct workflow now

```bash
# 1. Create a feature branch
git switch -c feature/add-profile

# 2. Make changes and commit
git add .
git commit -m "feat: add user profile page"

# 3. Push the feature branch
git push -u origin feature/add-profile

# 4. Open a Pull Request on GitHub UI
# GitHub will prompt: "Compare & pull request"
# Fill title, description, assign reviewers

# 5. Reviewer approves + merges via GitHub UI
# (or you merge after approval)
```

---

## 6. Require Review Approvals

**What it does:** A PR can only be merged after a minimum number of team members have approved it.

### Enable it

1. After enabling "Require pull request before merging"
2. ✅ **Require approvals**
3. Set **Required number of approvals before merging**: `1` (or more for large teams)
4. Optional: ✅ **Dismiss stale pull request approvals when new commits are pushed**
   - This means if you push more commits after approval, the approval is dismissed and someone must re-review
5. Optional: ✅ **Require review from Code Owners**

### How code review works

```
Developer A:
1. Push feature branch
2. Open PR on GitHub

Reviewer (Developer B):
1. Opens the PR
2. Reviews the "Files changed" tab
3. Adds inline comments on specific lines
4. Clicks "Review changes" →
   - "Comment" — leave comments only
   - "Approve" — approve the PR ✅
   - "Request changes" — block merge until addressed âŒ

Developer A:
5. Addresses feedback, pushes more commits
6. Re-requests review

Developer B:
7. Re-reviews → Approves ✅

Now PR can be merged.
```

---

## 7. Require Status Checks (CI)

**What it does:** A PR can only be merged if a CI pipeline (like GitHub Actions) passes. This prevents broken code from reaching `main`.

### Enable it

1. ✅ **Require status checks to pass before merging**
2. ✅ **Require branches to be up to date before merging** (prevents merging stale branches)
3. In the search box, type the name of your CI check (e.g., `test`, `build`, `lint`)
4. The check must run at least once before you can add it here

### Example: GitHub Actions CI setup

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install
      - run: npm test
```

Now every PR runs this check. If tests fail → merge is blocked automatically.

---

## 8. Admin Bypass Options

Some rules can be bypassed by Admins. Here's how to control that:

### Classic rules — include/exclude admins

1. ✅ **Do not allow bypassing the above settings**
   - When checked: even Admins cannot bypass (strongest protection)
   - When unchecked: Admins can bypass all rules

**Recommendation:** Check this for `main` in production repos. It forces Admins to follow the same process as everyone else.

### Rulesets — fine-grained bypass

With Rulesets (GitHub's newer system), you can set specific bypass actors:
- Specific teams (e.g., only "release-team" can merge without review)
- Specific app (e.g., your CI bot can force push to deploy)
- Repository roles: Admin, Maintain, Write

---

## 9. Protecting the develop Branch

Apply similar rules to `develop` — the integration branch.

**Recommended settings for `develop`:**

| Rule | Main | Develop |
|---|---|---|
| Restrict deletions | ✅ | ✅ |
| Block force pushes | ✅ | ✅ |
| Require PR | ✅ | ✅ |
| Required approvals | 2 | 1 |
| Require CI to pass | ✅ | ✅ |
| Dismiss stale reviews | ✅ | ✅ |
| Admin bypass | âŒ (none) | ✅ (admin only) |

**Steps to add protection for `develop`:**2

1. **Settings → Branches → Add rule**
2. Branch name pattern: `develop`
3. Apply desired settings (less strict than `main`)
4. Save

---

## 10. Rulesets (GitHub's Modern System)

Rulesets are the new, more powerful replacement for classic branch protection rules.

**Advantages over classic rules:**
- Apply to multiple branches with a single pattern
- Fine-grained bypass control per actor
- Apply to tag pushes too
- Can be at the organization level (protects all repos)
- Supports evaluation mode (warn before enforcing)

### Create a Ruleset

1. **Settings → Rules → Rulesets → New branch ruleset**
2. **Name**: `Protect main and develop`
3. **Enforcement status**: Active (or Evaluate to test without blocking)
4. **Bypass list**: Add teams/roles that can bypass
5. **Target branches**: Add patterns:
   - `main`
   - `develop`
   - `release/**` (all release branches)
6. **Rules to enable**:
   - ✅ Restrict deletions
   - ✅ Require a pull request
   - ✅ Block force pushes
   - ✅ Require status checks
7. Click **Create**

### Pattern examples

| Pattern | Matches |
|---|---|
| `main` | Exactly `main` |
| `release/**` | `release/v1.0`, `release/v1.2.3` |
| `feature/**` | All feature branches |
| `**` | All branches |

---

## Summary

| Protection | What it prevents |
|---|---|
| Restrict deletions | Accidental or malicious branch deletion |
| Block force pushes | History rewriting (`git push --force`) |
| Require PR | Direct pushes, bypassing review |
| Require approvals | Unreviewed code reaching production |
| Require status checks | Broken/untested code reaching production |
| Admin bypass disabled | Admins being exempt from all of the above |

**Next:** [06 — Fork & Collaborate →](./06-FORK-COLLABORATE.md)

---

## 🧑‍💻 Author

*Md. Sarowar Alam*  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/sarowar/

---
