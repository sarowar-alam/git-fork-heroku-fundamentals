# 01 — Setup & Installation

> **Phase 1 of 7** | Start here if you're completely new to Git.

---

## Table of Contents

1. [Create a GitHub Account](#1-create-a-github-account)
2. [Install Git CLI](#2-install-git-cli)
   - [Windows](#windows)
   - [macOS](#macos)
   - [Linux (Ubuntu/Debian)](#linux-ubuntudebian)
3. [Verify Installation](#3-verify-installation)
4. [First-Time Git Configuration](#4-first-time-git-configuration)
5. [Connect Git to GitHub](#5-connect-git-to-github)
   - [Option A: HTTPS (recommended for beginners)](#option-a-https-recommended-for-beginners)
   - [Option B: SSH Key (recommended for daily use)](#option-b-ssh-key-recommended-for-daily-use)

---

## 1. Create a GitHub Account

GitHub is where you host, share, and collaborate on Git repositories.

### Steps

1. Go to **[https://github.com/signup](https://github.com/signup)**
2. Enter your **email address** → click **Continue**
3. Create a **password** (min 15 chars, or 8 chars + number + lowercase)
4. Choose a **username** — this is your public handle (e.g., `sarowar-alam`)
5. Solve the puzzle verification → click **Create account**
6. Check your email → enter the **8-digit launch code**
7. Answer the onboarding questions (or skip)
8. Choose the **Free plan** → click **Continue for free**

### After signing up — do these

| Setting | Where | Why |
|---|---|---|
| Verify email | Emails → click link | Required to create repos |
| Add profile photo | Settings → Profile | Builds credibility |
| Enable 2FA (MFA) | Settings → Password and authentication | **Strongly recommended** |
| Set public profile | github.com/settings/profile | Visible to employers |

> **Tip:** Your GitHub username appears in all your repo URLs, e.g., `github.com/sarowar-alam`. Choose carefully.

---

## 2. Install Git CLI

### Windows

**Option A — winget (Windows 10/11, recommended)**
```powershell
winget install --id Git.Git -e --source winget
```

**Option B — Direct installer**
1. Download from **[https://git-scm.com/download/win](https://git-scm.com/download/win)**
2. Run the installer
3. Recommended settings to change from defaults:
   - **Default editor** → choose VS Code (if installed)
   - **Initial branch name** → select `main` (instead of `master`)
   - **Adjusting PATH** → "Git from the command line and also from 3rd-party software"
   - Keep all other defaults

**Verify (PowerShell or Git Bash):**
```powershell
git --version
# git version 2.45.2.windows.1
```

> **WSL (Windows Subsystem for Linux):** If you use Ubuntu on WSL, follow the Linux section below inside your WSL terminal.

---

### macOS

**Option A — Homebrew (recommended)**
```bash
# Install Homebrew first if you don't have it
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Then install Git
brew install git
```

**Option B — Xcode Command Line Tools**
```bash
xcode-select --install
# A dialog will appear — click Install
```

**Verify:**
```bash
git --version
# git version 2.45.0
```

---

### Linux (Ubuntu/Debian)

```bash
# Update package list
sudo apt update

# Install Git
sudo apt install git -y

# Verify
git --version
# git version 2.43.0
```

**Other distros:**
```bash
# Fedora / RHEL / CentOS
sudo dnf install git -y

# Arch Linux
sudo pacman -S git

# Alpine Linux
apk add git
```

---

## 3. Verify Installation

Run this on any OS:

```bash
git --version
```

Expected output format: `git version 2.x.x`

Also check where Git is installed:
```bash
# Linux / macOS
which git
# /usr/bin/git

# Windows (PowerShell)
where.exe git
# C:\Program Files\Git\cmd\git.exe
```

---

## 4. First-Time Git Configuration

Before making any commits, tell Git who you are. This info is attached to every commit you make.

```bash
# Set your name (use your real name or GitHub display name)
git config --global user.name "Sarowar Alam"

# Set your email (use the SAME email as your GitHub account)
git config --global user.email "sarowar@example.com"

# Set default branch name to "main" (GitHub's default)
git config --global init.defaultBranch main

# Set VS Code as default editor (optional but recommended)
git config --global core.editor "code --wait"

# Set diff/merge tool to VS Code (optional)
git config --global diff.tool vscode
git config --global merge.tool vscode
```

### View your config

```bash
# View all config
git config --list

# View a specific value
git config user.name
# Sarowar Alam
```

### Config file location

| Scope | File | What it affects |
|---|---|---|
| `--global` | `~/.gitconfig` | All repos for your user |
| `--local` | `.git/config` | Only the current repo |
| `--system` | `/etc/gitconfig` | All users on the machine |

Local overrides global, which overrides system.

---

## 5. Connect Git to GitHub

You need to authenticate to push/pull from GitHub. Two methods:

---

### Option A: HTTPS (recommended for beginners)

With HTTPS you use a **Personal Access Token (PAT)** — NOT your GitHub password.

#### Create a PAT
1. GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. Click **Generate new token (classic)**
3. Note: `Git CLI access`
4. Expiration: 90 days (or No expiration)
5. Scopes: check ✅ **repo** (full control)
6. Click **Generate token** → **copy it NOW** (it won't show again)

#### Use the PAT

The first time you push, Git will ask for credentials:
```
Username: sarowar-alam          â† your GitHub username
Password: ghp_xxxxxxxxxxxx      â† paste your PAT here (not your password!)
```

#### Save credentials (so you don't type every time)

**Windows (Credential Manager — built-in):**
```bash
git config --global credential.helper manager
```

**macOS (Keychain — built-in):**
```bash
git config --global credential.helper osxkeychain
```

**Linux:**
```bash
# Cache in memory for 1 hour
git config --global credential.helper 'cache --timeout=3600'

# Or store permanently (plain text — less secure)
git config --global credential.helper store
```

---

### Option B: SSH Key (recommended for daily use)

SSH keys are more secure and you never type passwords.

#### Step 1 — Generate an SSH key pair

```bash
# Replace with YOUR GitHub email
ssh-keygen -t ed25519 -C "sarowar@example.com"

# When prompted for file location, press Enter (use default)
# When prompted for passphrase, enter one (or press Enter for none)
```

This creates two files:
- `~/.ssh/id_ed25519` — **private key** (never share this)
- `~/.ssh/id_ed25519.pub` — **public key** (give this to GitHub)

#### Step 2 — Add the public key to GitHub

```bash
# Display your public key
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAAC3Nza... sarowar@example.com
```

1. Copy the entire output
2. GitHub → **Settings** → **SSH and GPG keys** → **New SSH key**
3. Title: `My Laptop` (or whatever machine this is)
4. Paste key → click **Add SSH key**

#### Step 3 — Test the connection

```bash
ssh -T git@github.com
# Hi sarowar-alam! You've successfully authenticated...
```

#### Step 4 — Use SSH URLs when cloning

Instead of:
```bash
git clone https://github.com/sarowar-alam/repo.git
```

Use:
```bash
git clone git@github.com:sarowar-alam/repo.git
```

To switch an existing repo from HTTPS to SSH:
```bash
git remote set-url origin git@github.com:sarowar-alam/repo.git
```

---

## Summary

At this point you should have:

- [x] A GitHub account with email verified
- [x] Git installed on your machine
- [x] `git --version` returning a version number
- [x] `user.name` and `user.email` configured
- [x] Authentication set up (HTTPS PAT or SSH key)

**Next:** [02 — Repositories & Roles →](./02-REPOS.md)

---

## 🧑‍💻 Author

*Md. Sarowar Alam*  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/sarowar/

---
