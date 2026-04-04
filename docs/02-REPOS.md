# 02 — Repositories, Cloning & Collaboration

> **Phase 2 of 7** | Create repos, clone public and private repos on every environment, understand remotes, roles, forking, and pull requests.

---

## Table of Contents

1. [What is a Repository?](#1-what-is-a-repository)
2. [Create a Repository on GitHub](#2-create-a-repository-on-github)
3. [Clone a Repository (Public)](#3-clone-a-repository-public)
4. [Public vs Private Repos](#4-public-vs-private-repos)
5. [Clone a Private Repo via HTTPS (PAT)](#5-clone-a-private-repo-via-https-pat)
6. [Clone a Private Repo via SSH](#6-clone-a-private-repo-via-ssh)
7. [Troubleshooting Auth per Environment](#7-troubleshooting-auth-per-environment)
8. [The .git Folder — What's Inside](#8-the-git-folder--whats-inside)
9. [Remotes — Connecting Local to Remote](#9-remotes--connecting-local-to-remote)
10. [Upstream Remote — Deep Dive](#10-upstream-remote--deep-dive)
11. [Full Forking Contribution Workflow](#11-full-forking-contribution-workflow)
12. [Pull Requests — Deep Dive](#12-pull-requests--deep-dive)
13. [Complete OSS Contribution Cycle](#13-complete-oss-contribution-cycle)
14. [GitHub Repository Roles](#14-github-repository-roles)

---

## 1. What is a Repository?

A **repository (repo)** is a directory tracked by Git. It stores:
- Your project files
- The complete history of every change ever made
- Metadata: branches, tags, configuration

```
my-project/
â”œâ”€â”€ .git/          â† Git's database (don't touch this manually)
â”œâ”€â”€ src/
â”œâ”€â”€ README.md
â””â”€â”€ package.json
```

There are two types:
| Type | Description |
|---|---|
| **Local** | Lives on your machine — `~/.../my-project` |
| **Remote** | Lives on a server — `github.com/user/my-project` |

---

## 2. Create a Repository on GitHub

### Method A — GitHub UI (easiest)

1. Click the **+** button (top-right) → **New repository**
2. Fill in:
   - **Repository name**: `git-fork-heroku-fundamentals`
   - **Description**: `A comprehensive Git fundamentals learning repo`
   - **Visibility**: Public
   - ✅ Add a README file (optional — we'll add our own)
   - ✅ Add .gitignore → choose `Node`
   - **License**: MIT
3. Click **Create repository**

### Method B — GitHub CLI

```bash
# Install GitHub CLI first: https://cli.github.com
gh repo create git-fork-heroku-fundamentals --public --description "Git fundamentals to Heroku"
```

### Method C — From an existing local folder (what we do in this repo)

```bash
# 1. Initialize local repo
cd my-project
git init

# 2. Add and commit files
git add .
git commit -m "initial commit"

# 3. Add the remote
git remote add origin https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

# 4. Push
git branch -M main
git push -u origin main
```

---

## 3. Clone a Repository (Public)

`git clone` downloads a remote repo to your machine — including all history, branches, and tags.

```bash
# HTTPS
git clone https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

# SSH
git clone git@github.com:sarowar-alam/git-fork-heroku-fundamentals.git

# Clone into a specific folder name
git clone https://github.com/user/repo.git my-custom-folder-name

# Clone a specific branch only
git clone -b develop https://github.com/user/repo.git

# Shallow clone — only latest snapshot (faster, no full history)
git clone --depth 1 https://github.com/user/repo.git
```

### What happens when you clone

```
Remote: github.com/sarowar-alam/git-fork-heroku-fundamentals
           â†“ git clone
Local:   ~/projects/git-fork-heroku-fundamentals/
         â”œâ”€â”€ .git/
         â”‚   â”œâ”€â”€ config          â† remote "origin" is set automatically
         â”‚   â””â”€â”€ refs/remotes/origin/main
         â”œâ”€â”€ index.js
         â””â”€â”€ ...
```

Git automatically sets the clone source as the `origin` remote.

---

## 4. Public vs Private Repos

| | Public | Private |
|---|---|---|
| Cloning | No auth needed | Auth required (HTTPS PAT or SSH key) |
| Browsing on GitHub | Anyone | Only invited collaborators |
| Forking | Anyone | Only collaborators (with permission) |
| `git clone` HTTPS | Works without login | Requires PAT as password |
| `git clone` SSH | Works if SSH key added | Works if SSH key added |

---

## 5. Clone a Private Repo via HTTPS (PAT)

GitHub no longer accepts your account password for HTTPS operations. You need a **Personal Access Token (PAT)**.

### Create a PAT (do this once)

1. GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. **Generate new token (classic)**
3. Note: `CLI - Git access`
4. Expiration: 90 days (or custom)
5. Scopes: ✅ **repo** (full control of private repos)
6. Click **Generate token** → **copy it immediately**
   - It looks like: `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
   - You will NOT see it again

---

### Windows (PowerShell / CMD)

```powershell
# Clone — when prompted:
# Username: your-github-username
# Password: your-PAT (NOT your GitHub password)
git clone https://github.com/yourusername/private-repo.git
```

**Save credentials so you don't type every time:**
```powershell
# Windows Credential Manager (built-in — recommended)
git config --global credential.helper manager

# After running git clone once and entering PAT,
# Windows stores it. Future clones/pushes are automatic.
```

**Embed credentials in URL (quick method — use caution, URL stays in shell history):**
```powershell
git clone https://your-username:ghp_xxxYOURTOKENxxx@github.com/yourusername/private-repo.git
```

---

### Windows Git Bash

Git Bash uses the same Windows Credential Manager as PowerShell:

```bash
# In Git Bash terminal
git clone https://github.com/yourusername/private-repo.git
# Git Credential Manager opens a dialog — sign in to GitHub once
# All future operations are automatic

# Verify credential helper
git config --global credential.helper
# manager-core  (or manager)
```

**If the credential dialog doesn't appear (headless Git Bash):**
```bash
# Use the URL-embedded method
git clone https://your-username:ghp_xxxTOKENxxx@github.com/yourusername/private-repo.git

# Or configure store (saves token in plaintext ~/.git-credentials)
git config --global credential.helper store
git clone https://github.com/yourusername/private-repo.git
# Enter username + PAT once — saved permanently
```

---

### WSL (Ubuntu on Windows)

> WSL does NOT share the Windows Credential Manager by default. You need to set one up.

**Option A — Git Credential Manager from Windows (recommended)**

WSL can use the Windows Git Credential Manager installed on the Windows side:

```bash
# In WSL terminal — tell Git to use the Windows credential manager
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"

# If using Git for Windows 64-bit:
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-manager.exe"

# Test it
git clone https://github.com/yourusername/private-repo.git
# A browser window will open on Windows for sign-in — works with MFA!
```

**Option B — Store PAT in memory (cache)**
```bash
# Cache credentials for 3600 seconds (1 hour)
git config --global credential.helper 'cache --timeout=3600'

# Now clone — enter username + PAT once
git clone https://github.com/yourusername/private-repo.git
# Username: your-github-username
# Password: ghp_xxxTOKENxxx
```

**Option C — Store PAT on disk (plain text)**
```bash
git config --global credential.helper store
# Saves to ~/.git-credentials in format:
# https://username:ghp_TOKEN@github.com

git clone https://github.com/yourusername/private-repo.git
# Enter once — saved permanently
# Warning: file is plain text, readable if someone accesses your machine
```

**Option D — SSH (best for WSL — see Section 6)**

---

### Ubuntu / Linux (native)

Same options as WSL. SSH is the recommended approach on Linux. For HTTPS:

```bash
# Option 1: Cache (1 hour by default)
git config --global credential.helper 'cache --timeout=7200'

# Option 2: libsecret (uses system keychain — most secure on Linux)
sudo apt install libsecret-1-0 libsecret-1-dev
sudo make --directory=/usr/share/doc/git/contrib/credential/libsecret
git config --global credential.helper /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret

# Option 3: .netrc file (not recommended — plain text)
echo "machine github.com login yourusername password ghp_TOKEN" >> ~/.netrc
chmod 600 ~/.netrc
```

---

### macOS

```bash
# macOS Keychain (built-in — automatic)
git config --global credential.helper osxkeychain

# Clone — enter username + PAT once
git clone https://github.com/yourusername/private-repo.git
# Stored in macOS Keychain forever
```

---

## 6. Clone a Private Repo via SSH

SSH is the **recommended method on Linux/WSL** — no tokens to manage, no expiry, works everywhere.

---

### SSH on Windows Git Bash

**Step 1 — Check for existing SSH keys**
```bash
ls -la ~/.ssh
# Look for id_ed25519 and id_ed25519.pub
# If they don't exist, create them:
```

**Step 2 — Generate an SSH key**
```bash
ssh-keygen -t ed25519 -C "sarowar@example.com"
# Press Enter to accept default path: ~/.ssh/id_ed25519
# Enter a passphrase (or press Enter for none)
```

**Step 3 — Start the SSH agent and add your key**
```bash
# In Git Bash
eval "$(ssh-agent -s)"
# Agent pid 12345

ssh-add ~/.ssh/id_ed25519
# Identity added: /c/Users/Sarowar/.ssh/id_ed25519
```

**Step 4 — Add public key to GitHub**
```bash
# Copy the public key
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAAC3Nza... sarowar@example.com
```
1. GitHub → **Settings** → **SSH and GPG keys** → **New SSH key**
2. Title: `Git Bash on Windows`
3. Paste public key → **Add SSH key**

**Step 5 — Test and clone**
```bash
ssh -T git@github.com
# Hi sarowar-alam! You've successfully authenticated...

# Clone private repo via SSH
git clone git@github.com:yourusername/private-repo.git
```

---

### SSH on WSL / Ubuntu

WSL has its own separate SSH key from Windows. Set it up inside WSL:

**Step 1 — Check for existing keys (inside WSL)**
```bash
ls ~/.ssh
# If empty or no id_ed25519, generate a new key:
```

**Step 2 — Generate SSH key in WSL**
```bash
ssh-keygen -t ed25519 -C "sarowar@example.com"
# Default file: ~/.ssh/id_ed25519
# Add a passphrase for security (optional)
```

**Step 3 — Start SSH agent in WSL**

By default, the SSH agent is not automatically started in WSL. Add this to `~/.bashrc`:

```bash
# Open ~/.bashrc
nano ~/.bashrc

# Add these lines at the bottom:
eval "$(ssh-agent -s)" > /dev/null 2>&1
ssh-add ~/.ssh/id_ed25519 > /dev/null 2>&1
```

```bash
# Apply immediately
source ~/.bashrc
```

**Or use keychain (recommended for persistent WSL sessions):**
```bash
sudo apt install keychain
echo 'eval $(keychain --eval --quiet id_ed25519)' >> ~/.bashrc
source ~/.bashrc
```

**Step 4 — Copy public key and add to GitHub**
```bash
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAAC3Nza... sarowar@example.com
```
GitHub → **Settings** → **SSH and GPG keys** → **New SSH key**
Title: `WSL Ubuntu` → paste key → **Add SSH key**

> Note: This is a **different key** from your Windows Git Bash key — you must add it separately to GitHub.

**Step 5 — Test and clone**
```bash
ssh -T git@github.com
# Hi sarowar-alam! You've successfully authenticated...

git clone git@github.com:yourusername/private-repo.git
```

---

### SSH on macOS

```bash
# Generate key
ssh-keygen -t ed25519 -C "sarowar@example.com"

# Add to macOS Keychain (auto-loads on login)
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# Configure ~/.ssh/config for automatic loading
echo "Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519" >> ~/.ssh/config

# Test
ssh -T git@github.com

# Clone
git clone git@github.com:yourusername/private-repo.git
```

---

## 7. Troubleshooting Auth per Environment

### "Authentication failed" (HTTPS)

```bash
# Check what credential helper is active
git config --global credential.helper

# Reset stored credentials
git credential reject <<EOF
protocol=https
host=github.com
username=yourusername
EOF

# Or clear all Windows credentials:
# Control Panel → Credential Manager → Windows Credentials
# Remove GitHub entries → try again
```

### "Permission denied (publickey)" (SSH)

```bash
# Test with verbose output — see exactly what's failing
ssh -vT git@github.com

# Make sure ssh-agent is running and key is loaded
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l   # list loaded keys

# Verify the correct public key is on GitHub
cat ~/.ssh/id_ed25519.pub
# Compare with: GitHub → Settings → SSH keys

# Check file permissions (must be strict)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### SSH works but WSL disconnects / key lost on restart

```bash
# Add to ~/.bashrc to auto-load on every WSL session:
echo 'eval $(ssh-agent -s) > /dev/null' >> ~/.bashrc
echo 'ssh-add ~/.ssh/id_ed25519 2>/dev/null' >> ~/.bashrc
source ~/.bashrc
```

### "Could not resolve host: github.com" (WSL)

```bash
# DNS issue in WSL
cat /etc/resolv.conf   # check nameserver

# Quick fix:
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Permanent fix — prevent WSL from overwriting resolv.conf:
echo "[network]
generateResolvConf = false" | sudo tee -a /etc/wsl.conf

sudo rm /etc/resolv.conf
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

### Switching an existing repo from HTTPS to SSH (or vice versa)

```bash
# Check current remote URL
git remote -v
# origin  https://github.com/yourusername/repo.git (fetch)

# Switch to SSH
git remote set-url origin git@github.com:yourusername/repo.git

# Switch back to HTTPS
git remote set-url origin https://github.com/yourusername/repo.git

# Verify
git remote -v
```

---

## 8. The .git Folder — What's Inside

The `.git` folder IS the repository. Delete it and you lose all history.

```
.git/
â”œâ”€â”€ HEAD            â† points to current branch (e.g., "ref: refs/heads/main")
â”œâ”€â”€ config          â† repo-local Git config + remotes
â”œâ”€â”€ description     â† used by GitWeb (ignore for normal use)
â”œâ”€â”€ hooks/          â† scripts that run on git events (pre-commit, post-push, etc.)
â”œâ”€â”€ info/
â”‚   â””â”€â”€ exclude     â† like .gitignore, but not committed
â”œâ”€â”€ objects/        â† all data: commits, trees, blobs (your file contents)
â”‚   â”œâ”€â”€ pack/       â† packed objects for efficiency  
â”‚   â””â”€â”€ info/
â”œâ”€â”€ refs/
â”‚   â”œâ”€â”€ heads/      â† local branches → refs/heads/main = hash of latest commit
â”‚   â”œâ”€â”€ remotes/    â† remote-tracking branches
â”‚   â””â”€â”€ tags/       â† tags
â””â”€â”€ COMMIT_EDITMSG  â† last commit message (used by --amend)
```

**Everything in Git is just files in `.git/objects/`.** Git uses SHA-1/SHA-256 hashes as file names.

```bash
# See what HEAD points to
cat .git/HEAD
# ref: refs/heads/main

# See the commit hash of main
cat .git/refs/heads/main
# a3f9c2b1d4e8f7a6b5c4d3e2f1a0b9c8d7e6f5a4
```

---

## 9. Remotes — Connecting Local to Remote

A **remote** is a named reference to a URL of another repository (usually on GitHub).

```bash
# View current remotes
git remote -v
# origin  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (fetch)
# origin  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (push)

# Add a new remote
git remote add origin https://github.com/sarowar-alam/repo.git

# Add a second remote (common when working with forks)
git remote add upstream https://github.com/original-owner/repo.git

# Rename a remote
git remote rename origin old-origin

# Remove a remote
git remote remove upstream

# Change a remote's URL (e.g., switch from HTTPS to SSH)
git remote set-url origin git@github.com:sarowar-alam/repo.git

# Inspect a remote in detail
git remote show origin
```

### Convention: origin vs upstream

| Name | Points to | Used for |
|---|---|---|
| `origin` | Your fork or your repo | Push your changes |
| `upstream` | The original repo you forked from | Pull in upstream changes |

---

## 10. Upstream Remote — Deep Dive

### What “upstreamâ€ really means

```
github.com/ORIGINAL-OWNER/project    â† upstream
         â†“ fork
github.com/YOUR-USERNAME/project     â† origin
         â†“ clone
~/projects/project                   â† local
```

- `upstream` = the authoritative source. You never push to upstream (unless you’re a maintainer).
- `origin` = your fork. You push your feature branches here.

### Cannot push to upstream (normal behavior)

```bash
git push upstream main
# remote: Permission to ORIGINAL-OWNER/project.git denied to YOUR-USERNAME.
# fatal: unable to access 'https://github.com/ORIGINAL-OWNER/project.git/': ...
```

This is expected. You contribute via PRs, not direct pushes.

### Fetching specific upstream branches

```bash
# Fetch only the develop branch from upstream
git fetch upstream develop

# Create local tracking branch
git switch -c develop upstream/develop

# Or just inspect without creating a local branch
git log upstream/develop --oneline
git diff main upstream/main
```

### Multiple upstreams

On large projects you might have multiple important remotes:

```bash
git remote add upstream https://github.com/original/project.git
git remote add team https://github.com/ourteam/project.git
git remote add staging https://github.com/staging/project.git

git remote -v
# origin    git@github.com:yourname/project.git
# upstream  https://github.com/original/project.git
# team      https://github.com/ourteam/project.git
# staging   https://github.com/staging/project.git
```

---

## 11. Full Forking Contribution Workflow

This is the standard open-source contribution workflow — used to contribute to any project on GitHub.

### The Complete Cycle

```
1.  Fork the repo on GitHub
2.  Clone YOUR fork locally
3.  Add upstream remote (points to original)
4.  Create a feature branch from main
5.  Make changes → commit
6.  Sync with upstream before pushing (avoid conflicts)
7.  Push feature branch to YOUR fork (origin)
8.  Open a Pull Request to the original repo
9.  Respond to review feedback → push more commits
10. PR gets merged (or closed)
11. Delete your feature branch
12. Sync your fork’s main with upstream
13. Repeat
```

### Step-by-Step

```bash
# â”€â”€ PHASE 1: ONE-TIME SETUP â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

# 1. Fork on GitHub (click Fork button)

# 2. Clone YOUR fork
git clone git@github.com:YOUR-USERNAME/project.git
cd project

# 3. Add upstream remote
git remote add upstream https://github.com/ORIGINAL-OWNER/project.git

# Verify
git remote -v
# origin    git@github.com:YOUR-USERNAME/project.git (fetch)
# origin    git@github.com:YOUR-USERNAME/project.git (push)
# upstream  https://github.com/ORIGINAL-OWNER/project.git (fetch)
# upstream  https://github.com/ORIGINAL-OWNER/project.git (push)

# â”€â”€ PHASE 2: BEFORE EACH CONTRIBUTION â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

# 4. Pull latest from upstream to stay current
git switch main
git fetch upstream
git merge upstream/main
# or: git pull upstream main

# 5. Push synced main to your fork
git push origin main

# â”€â”€ PHASE 3: MAKE YOUR CHANGE â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

# 6. Create feature branch
git switch -c fix/typo-in-readme

# 7. Make changes
nano README.md

# 8. Commit
git add README.md
git commit -m "docs: fix typo in installation section"

# â”€â”€ PHASE 4: SYNC BEFORE PUSH (important!) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

# 9. Check if upstream has new commits since you branched
git fetch upstream
git log --oneline main..upstream/main
# If this shows commits, you need to rebase

# 10. Rebase your branch on latest upstream/main
git rebase upstream/main
# Resolve any conflicts that arise, then:
# git add .
# git rebase --continue

# â”€â”€ PHASE 5: PUSH AND OPEN PR â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

# 11. Push feature branch to YOUR fork
git push -u origin fix/typo-in-readme

# 12. Open Pull Request on GitHub
# Go to: github.com/ORIGINAL-OWNER/project
# You’ll see: "fix/typo-in-readme had recent pushes — Compare & pull request"

# â”€â”€ PHASE 6: RESPOND TO FEEDBACK â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

# 13. If reviewer requests changes:
git add .
git commit -m "fix: address review feedback"
git push    # updates the PR automatically

# 14. If upstream gets more commits while your PR is waiting:
git fetch upstream
git rebase upstream/main
git push --force-with-lease origin fix/typo-in-readme

# â”€â”€ PHASE 7: AFTER MERGE â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

# 15. Switch back to main
git switch main

# 16. Pull the merged change from upstream
git pull upstream main

# 17. Update your fork
git push origin main

# 18. Delete your local feature branch
git branch -d fix/typo-in-readme

# 19. Delete the remote feature branch on your fork
git push origin --delete fix/typo-in-readme
```

---

## 12. Pull Requests — Deep Dive

### PR Templates

A PR template auto-fills the PR description box on GitHub. Create it at `.github/pull_request_template.md`:

```markdown
## Summary
<!-- What does this PR do? One clear sentence. -->

## Type of change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Refactor
- [ ] Breaking change

## How to test
<!-- Steps for the reviewer to verify your changes -->
1. 
2. 

## Related issues
Closes #<!-- issue number -->

## Screenshots (if UI change)
<!-- Before / After -->

## Checklist
- [ ] Tested locally
- [ ] No new console errors or warnings
- [ ] Responsive on mobile (if UI change)
- [ ] Added/updated tests (if applicable)
```

Now every new PR opens pre-filled with your template.

---

### Draft PRs

A **Draft PR** signals: “I’m not done yet — do NOT merge, but feedback welcome.â€

**When to use:**
- Work in progress you want early feedback on
- Blocked waiting for another PR to merge
- Want CI to run but aren’t ready for review

**Create a Draft PR:**
1. Click **Compare & pull request**
2. Click the arrow next to **Create pull request**
3. Select **Create draft pull request**

**Or via GitHub CLI:**
```bash
gh pr create --draft --title "WIP: add dark mode" --body "Still working on mobile breakpoints"
```

**Convert Draft → Ready for review:**
- On the PR page → click **Ready for review** button
- This notifies assigned reviewers

---

### Linking Issues to PRs

When a PR fixes an issue, link them so the issue auto-closes when the PR merges.

**In the PR description:**
```markdown
Closes #42
Fixes #42
Resolves #42

# For multiple issues:
Closes #42, closes #58

# Closes issue in ANOTHER repo:
Closes org/repo#42
```

GitHub automatically closes issue #42 when the PR is merged into the default branch.

---

### Updating Your PR After Upstream Changes

If the original repo gets new commits while your PR is open:

```bash
# 1. Fetch latest from upstream
git fetch upstream

# 2. Make sure you're on your feature branch
git switch feature/my-change

# 3. Rebase on upstream/main
git rebase upstream/main
# If conflicts: fix them, then:
# git add .
# git rebase --continue

# 4. Force push (required after rebase — history changed)
git push --force-with-lease origin feature/my-change
```

> Use `--force-with-lease` not `--force`. It fails if someone else pushed to your branch since you last pulled.

---

### Creating PRs from the CLI

Install GitHub CLI: [https://cli.github.com](https://cli.github.com)

```bash
# Install on macOS
brew install gh

# Install on Ubuntu/WSL
sudo apt install gh

# Authenticate (MFA-safe — opens browser)
gh auth login

# Create a PR
gh pr create \
  --title "feat: add dark mode toggle" \
  --body "Adds dark mode. Closes #42." \
  --base main \
  --head feature/dark-mode \
  --reviewer teammate1 \
  --label enhancement \
  --assignee @me

# Create as draft
gh pr create --draft

# List PRs
gh pr list
gh pr list --state all

# View a PR
gh pr view 45
gh pr view --web

# Checkout a PR branch locally to test it
gh pr checkout 45

# Approve / request changes
gh pr review 45 --approve
gh pr review 45 --request-changes --body "Please fix the port issue"

# Merge
gh pr merge 45 --squash
gh pr merge 45 --rebase
gh pr merge 45 --auto   # auto-merge when CI passes

# Close / reopen
gh pr close 45
gh pr reopen 45
```

---

## 13. Complete OSS Contribution Cycle

A practical example contributing to this repo.

```bash
# â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
# SCENARIO: You found a bug in docs/03-COMMANDS.md
# â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

# 1. Fork on GitHub:
#    github.com/sarowar-alam/git-fork-heroku-fundamentals → Fork

# 2. Clone YOUR fork (SSH — recommended)
git clone git@github.com:YOUR-USERNAME/git-fork-heroku-fundamentals.git
cd git-fork-heroku-fundamentals

# 3. Add upstream
git remote add upstream https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

# 4. Sync with upstream
git pull upstream main
git push origin main

# 5. Create fix branch
git switch -c fix/commands-doc-typo

# 6. Fix the bug
nano docs/03-COMMANDS.md

# 7. Commit with clear message
git add docs/03-COMMANDS.md
git commit -m "docs: fix incorrect git reset flag in commands guide

The --mixed flag example was shown as --soft.
Corrected to --mixed and added clarifying explanation.

Closes #17"

# 8. Sync with upstream one more time before opening PR
git fetch upstream
git rebase upstream/main

# 9. Push to your fork
git push -u origin fix/commands-doc-typo

# 10. Open PR via CLI
gh pr create \
  --repo sarowar-alam/git-fork-heroku-fundamentals \
  --title "docs: fix incorrect git reset flag in commands guide" \
  --body "Fixes a documentation error in 03-COMMANDS.md. Closes #17." \
  --base main

# 11. Wait for review. If changes requested:
nano docs/03-COMMANDS.md
git add docs/03-COMMANDS.md
git commit -m "docs: address review feedback — add more context"
git push

# 12. PR is approved and merged ✅

# 13. Cleanup
git switch main
git pull upstream main
git push origin main
git branch -d fix/commands-doc-typo
git push origin --delete fix/commands-doc-typo

# â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
# Done! Your contribution is now part of the project.
# â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
```

---

## 14. GitHub Repository Roles

When you collaborate, you assign roles to team members. Each role grants specific permissions.

### The 5 Roles

| Role | Level | Description |
|---|---|---|
| **Read** | 1 — Lowest | Can view and clone public + private repos |
| **Triage** | 2 | Read + manage issues and PRs (no code access) |
| **Write** | 3 | Triage + push code to non-protected branches |
| **Maintain** | 4 | Write + manage settings (no destructive actions) |
| **Admin** | 5 — Highest | Full control including deletion and settings |

---

### Role: Read

**Who is it for?** Stakeholders, QA, designers, clients — people who need to see code but not modify it.

```
Can do:
  ✅ View all files and commits
  ✅ Clone the repository
  ✅ View issues and pull requests
  ✅ Open issues
  ✅ Comment on issues and PRs

Cannot do:
  âŒ Push any code
  âŒ Merge pull requests
  âŒ Change settings
```

**Example:** A client needs to review your code — give them **Read**.

---

### Role: Triage

**Who is it for?** Project managers, community moderators, scrum masters.

```
Can do everything Read can, plus:
  ✅ Apply labels, milestones, and assignees to issues
  ✅ Close and reopen issues
  ✅ Request PR reviews
  ✅ Mark comments as resolved
  ✅ Manage GitHub Discussions

Cannot do:
  âŒ Push code
  âŒ Merge PRs
```

**Example:** A scrum master organizes backlog issues — give them **Triage**.

---

### Role: Write

**Who is it for?** Active developers and contributors on the team.

```
Can do everything Triage can, plus:
  ✅ Push to non-protected branches (e.g., feature branches)
  ✅ Merge pull requests (when you are the author)
  ✅ Create and delete branches
  ✅ Manage releases
  ✅ Create GitHub Actions workflows

Cannot do:
  âŒ Push directly to protected branches (main, develop)
  âŒ Change branch protection rules
  âŒ Manage collaborators
```

**Example:** A junior developer joins your team — give them **Write**.

---

### Role: Maintain

**Who is it for?** Team leads, senior developers who manage the repo but shouldn't have full admin access.

```
Can do everything Write can, plus:
  ✅ Push to protected branches (if allowed by rules)
  ✅ Manage webhooks
  ✅ View traffic and insights
  ✅ Manage topics and description
  ✅ Archive the repository

Cannot do:
  âŒ Delete the repository
  âŒ Transfer ownership
  âŒ Manage team access
  âŒ Change visibility (public/private)
```

**Example:** A tech lead who manages releases but shouldn't be able to delete the repo — give them **Maintain**.

---

### Role: Admin

**Who is it for?** Repository owners and trusted leads.

```
Can do everything Maintain can, plus:
  ✅ Delete the repository
  ✅ Archive the repository
  ✅ Transfer ownership
  ✅ Change visibility (public â†” private)
  ✅ Add and remove collaborators
  ✅ Set and change branch protection rules
  ✅ Enable/disable features (Issues, Wiki, Discussions)
  ✅ Manage GitHub Secrets and Variables
  ✅ Bypass branch protection rules
```

**Example:** You and your co-founder on your startup's repo — both **Admin**.

---

### How to Assign Roles

**For a personal repo (collaborators):**
1. Repo → **Settings** → **Collaborators** → **Add people**
2. Search by username or email
3. Select their role → Send invitation

**For an organization repo (teams):**
1. Organization → **Teams** → Create a team (e.g., "Backend Devs")
2. Add members to the team
3. In repo → **Settings** → **Collaborators and teams** → Add team → Assign role

---

### Quick Decision Guide

```
Needs to SEE code only?           → Read
Manages issues and PRs?           → Triage
Writes code and opens PRs?        → Write
Manages releases and settings?    → Maintain
Full control, owns the repo?      → Admin
```

---

## Summary

- A **repository** stores your code and its entire history
- `git clone` downloads a full copy of a remote repo (public = no auth, private = PAT or SSH)
- Cloning private repos: use **HTTPS+PAT** (Windows default) or **SSH key** (recommended on Linux/WSL)
- Each environment (Windows, Git Bash, WSL, Ubuntu, macOS) has its own credential setup
- `.git/` is Git’s internal database — never delete it manually
- **Remotes** link your local repo to GitHub; `origin` = your repo, `upstream` = the original you forked from
- **Upstream deep dive**: never push to upstream — contribute via Pull Requests
- **Full forking cycle**: fork → clone → add upstream → branch → rebase → PR → cleanup
- **Pull Requests**: use templates, draft PRs, link issues (`Closes #42`), update via rebase, manage with `gh` CLI
- **Roles** control what each collaborator can do: Read → Triage → Write → Maintain → Admin

**Next:** [03 — Core Git Commands →](./03-COMMANDS.md)

---

## 🧑‍💻 Author

*Md. Sarowar Alam*  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/sarowar/

---
