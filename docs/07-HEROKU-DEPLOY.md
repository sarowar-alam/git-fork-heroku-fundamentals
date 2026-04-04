# 07 — Deploy to Heroku

> **Phase 7 of 7** | Ship your Node.js app live. MFA-safe deployment using API Key authentication.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Install Heroku CLI](#2-install-heroku-cli)
3. [Authenticate with Heroku (MFA-Safe)](#3-authenticate-with-heroku-mfa-safe)
4. [Push Repo to GitHub First](#4-push-repo-to-github-first)
5. [Create and Deploy the Heroku App](#5-create-and-deploy-the-heroku-app)
6. [Verify the Deployment](#6-verify-the-deployment)
7. [Heroku Logs & Debugging](#7-heroku-logs--debugging)
8. [Re-Deploy After Changes](#8-re-deploy-after-changes)
9. [Connect GitHub for Auto-Deploy](#9-connect-github-for-auto-deploy)
10. [Environment Variables on Heroku](#10-environment-variables-on-heroku)
11. [Common Errors & Fixes](#11-common-errors--fixes)

---

## 1. Prerequisites

Before deploying, make sure you have:

- [x] A [Heroku account](https://signup.heroku.com/) (free tier available)
- [x] MFA enabled on Heroku (you already have this)
- [x] `index.js` — your Express server
- [x] `package.json` — with a `"start"` script: `"start": "node index.js"`
- [x] `Procfile` — containing: `web: node index.js`
- [x] `.gitignore` — excluding `node_modules/`
- [x] A GitHub repo with your code pushed

**Verify your app files:**

```bash
# Check Procfile
cat Procfile
# web: node index.js

# Check package.json start script
cat package.json | grep '"start"'
# "start": "node index.js"
```

---

## 2. Install Heroku CLI

### Windows

```powershell
# Option A — winget
winget install --id Heroku.HerokuCLI

# Option B — direct download
# Go to: https://devcenter.heroku.com/articles/heroku-cli
# Download the Windows installer (.exe)
```

### macOS

```bash
brew tap heroku/brew && brew install heroku
```

### Linux (Ubuntu/Debian)

```bash
# Option A — Snap (easiest)
sudo snap install --classic heroku

# Option B — curl installer
curl https://cli-assets.heroku.com/install.sh | sh

# Option C — npm
npm install -g heroku
```

### Verify

```bash
heroku --version
# heroku/9.x.x linux-x64 node-v20.x.x
```

---

## 3. Authenticate with Heroku (MFA-Safe)

> **Why normal login fails with MFA on Ubuntu/WSL:**
> `heroku login` opens a browser window. On headless terminals (Ubuntu server, WSL without GUI), there is no browser — the command hangs or fails.
> `heroku login -i` is blocked by Heroku when MFA is enabled.

### ✅ The Solution: API Key Authentication

This works everywhere — Ubuntu, WSL, CI/CD, servers — regardless of MFA.

#### Step 1 — Get your API Key from Heroku Dashboard

1. Log in to [https://dashboard.heroku.com](https://dashboard.heroku.com) from your browser
2. Click your **profile avatar** (top-right) → **Account settings**
3. Scroll down to **API Key** section
4. Click **Reveal** → copy the key (looks like: `01234567-89ab-cdef-0123-456789abcdef`)

#### Step 2 — Set the API Key in your terminal

**For the current terminal session only:**
```bash
export HEROKU_API_KEY=01234567-89ab-cdef-0123-456789abcdef
```

**Permanently (add to `~/.bashrc` or `~/.zshrc`):**
```bash
echo 'export HEROKU_API_KEY=01234567-89ab-cdef-0123-456789abcdef' >> ~/.bashrc
source ~/.bashrc
```

**Windows (PowerShell — current session):**
```powershell
$env:HEROKU_API_KEY = "01234567-89ab-cdef-0123-456789abcdef"
```

**Windows (permanent — via System Environment Variables):**
1. Search "Environment Variables" in Start Menu
2. Add new User variable: `HEROKU_API_KEY` = `your-api-key`

#### Step 3 — Verify authentication

```bash
heroku auth:whoami
# sarowar@example.com
```

If you see your email, you're authenticated. 

---

## 4. Push Repo to GitHub First

The repo needs to exist on GitHub before deploying to Heroku.

```bash
# In your project directory
cd git-fork-heroku-fundamentals

# Initialize if not already done
git init

# Add all files
git add .

# Initial commit
git commit -m "initial commit: app + documentation"

# Set the main branch name
git branch -M main

# Add remote (replace with your actual repo URL)
git remote add origin https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

# Push to GitHub
git push -u origin main
```

Go to `https://github.com/sarowar-alam/git-fork-heroku-fundamentals` to confirm.

---

## 5. Create and Deploy the Heroku App

```bash
# Make sure HEROKU_API_KEY is exported (from step 3)

# Create the Heroku app with your chosen name
heroku create ostad-devops
# Creating â¬¢ ostad-devops... done
# https://ostad-devops.herokuapp.com/ | https://git.heroku.com/ostad-devops.git

# Verify a 'heroku' remote was added
git remote -v
# heroku  https://git.heroku.com/ostad-devops.git (fetch)
# heroku  https://git.heroku.com/ostad-devops.git (push)
# origin  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (fetch)
# origin  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (push)

# Deploy by pushing to the heroku remote
git push heroku main
```

You should see output like:

```
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Compressing objects: 100% (9/9), done.
Writing objects: 100% (12/12), 4.50 KiB | 4.50 MiB/s, done.

remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Building on the Heroku-22 stack
remote: -----> Determining which buildpack to use for this app
remote: -----> Node.js app detected
remote:
remote: -----> Creating runtime environment
remote:        NPM_CONFIG_LOGLEVEL=error
remote:        NODE_ENV=production
remote:
remote: -----> Installing binaries
remote:        engines.node (package.json):  >=18.x
remote:        engines.npm (package.json):   unspecified (use default)
remote:        Resolving node version >=18.x...
remote:        Downloading and installing node 20.x.x...
remote:
remote: -----> Installing dependencies
remote:        Installing node modules (package.json + package-lock.json)
remote:        added 65 packages in 2s
remote:
remote: -----> Build succeeded!
remote:
remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote:
remote: -----> Compressing...
remote:        Done: 23.0M
remote:
remote: -----> Launching...
remote:        Released v3
remote:        https://ostad-devops.herokuapp.com/ deployed to Heroku
remote:
To https://git.heroku.com/ostad-devops.git
 * [new branch]      main -> main
```

---

## 6. Verify the Deployment

```bash
# Open your live app in the browser
heroku open
# Opens: https://ostad-devops.herokuapp.com

# Or just open it manually in your browser:
# https://ostad-devops.herokuapp.com
```

If the page loads — your app is live on the internet!

---

## 7. Heroku Logs & Debugging

```bash
# View recent logs
heroku logs

# Stream live logs (tail mode — Ctrl+C to stop)
heroku logs --tail

# View last N log lines
heroku logs -n 200

# Filter logs by process type
heroku logs --ps web

# View build logs specifically
heroku builds:info
heroku builds:output

# Check dyno status
heroku ps
# web.1: up 2026/04/04 10:00:00 +0000 (~ 5m ago)

# Restart the dyno (if app is frozen)
heroku restart
heroku restart web.1   # restart specific dyno
```

**Common log patterns:**
```
2026-04-04T10:00:00 app[web.1]: Server running on http://localhost:3000
        â†‘ timestamp   â†‘ source    â†‘ your app's console.log output
```

---

## 8. Re-Deploy After Changes

Every time you update your code, re-deploy by pushing to the `heroku` remote.

```bash
# Make your changes
git add .
git commit -m "feat: update landing page hero text"

# Deploy to Heroku
git push heroku main

# Also push to GitHub to keep in sync
git push origin main
```

Heroku automatically rebuilds and redeploys on every push.

---

## 9. Connect GitHub for Auto-Deploy

Instead of manually running `git push heroku main`, set up automatic deployment from GitHub — every push to `main` deploys automatically.

### Steps

1. Go to **[Heroku Dashboard](https://dashboard.heroku.com)** → select your app `ostad-devops`
2. Click the **Deploy** tab
3. Under **Deployment method** → click **GitHub**
4. Click **Connect to GitHub** → authorize Heroku
5. Search for your repo: `git-fork-heroku-fundamentals`
6. Click **Connect**

### Enable Automatic Deploys

7. Under **Automatic deploys** → select branch: `main`
8. Optional: ✅ **Wait for CI to pass before deploy** (recommended with GitHub Actions)
9. Click **Enable Automatic Deploys**

Now `git push origin main` → triggers a Heroku build automatically.

### Manual Deploy from Dashboard

10. Under **Manual deploy** → **Deploy Branch** (useful for testing)

---

## 10. Environment Variables on Heroku

Never commit secrets (API keys, database URLs) to Git. Use Heroku config vars instead.

```bash
# Set a config variable
heroku config:set SECRET_KEY=abc123
heroku config:set NODE_ENV=production

# View all config variables
heroku config

# Get a single variable value
heroku config:get SECRET_KEY

# Remove a variable
heroku config:unset SECRET_KEY
```

In your code (`index.js`), access them the same as local environment variables:

```javascript
const secretKey = process.env.SECRET_KEY;
const port = process.env.PORT || 3000;
```

**For local development**, create a `.env` file (already in `.gitignore`):
```bash
# .env
SECRET_KEY=abc123
PORT=3000
```

Install `dotenv` to load `.env` locally:
```bash
npm install dotenv
```

```javascript
// index.js — at the top
if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config();
}
```

---

## 11. Common Errors & Fixes

### Error: `heroku: command not found`
```bash
# Re-install Heroku CLI
# Check your PATH includes the Heroku bin directory
which heroku         # should output a path
echo $PATH           # verify heroku's bin is in path

# On Ubuntu after snap install, restart terminal or:
source ~/.bashrc
```

---

### Error: `Authentication required` / API key not working
```bash
# Check the variable is set
echo $HEROKU_API_KEY
# Should print your key

# If empty, set it again
export HEROKU_API_KEY=your-key-here

# Verify
heroku auth:whoami
```

---

### Error: `No default language could be detected`
Your `package.json` is missing, not committed, or in a subdirectory.
```bash
# Ensure package.json is in the ROOT of your repo
ls package.json
# package.json

# Make sure it's committed
git status
# If it shows untracked: git add package.json && git commit -m "add package.json"
```

---

### Error: `Procfile not found` or app doesn't start
```bash
# Check Procfile exists and is correct
cat Procfile
# Must be: web: node index.js

# Check it's committed and at root, not inside a folder
git ls-files Procfile
# Should output: Procfile
```

---

### Error: App crashes — `H10` in logs
```bash
heroku logs --tail
# Look for the error message before H10
```

Common causes:
- `npm start` fails (missing dependency)
- Port not bound correctly — Express must use `process.env.PORT`:
  ```javascript
  const PORT = process.env.PORT || 3000;  // â† must include process.env.PORT
  app.listen(PORT, ...);
  ```
- Missing dependency in `package.json` (was installed globally locally but not listed)

---

### App name already taken on Heroku
```bash
heroku create ostad-devops
# Name is already taken

# Try a different unique name
heroku create ostad-devops-2026
heroku create ostad-sarowar-devops
# Or leave blank — Heroku auto-generates a name
heroku create
# Creating â¬¢ tranquil-meadow-12345... done
```

---

## Deployment Checklist

Before every production deploy:

- [ ] `package.json` has a `"start"` script
- [ ] `Procfile` contains `web: node index.js`
- [ ] `node_modules/` is in `.gitignore`
- [ ] App uses `process.env.PORT` for the port
- [ ] All dependencies are in `dependencies` (not `devDependencies`)
- [ ] No hardcoded secrets (use `heroku config:set` instead)
- [ ] `git push heroku main` completes without errors
- [ ] `heroku open` shows the live app

---

## Summary: Full Deploy Flow

```bash
# 1. Authenticate (Ubuntu/WSL/MFA-safe)
export HEROKU_API_KEY=your-api-key-from-dashboard
heroku auth:whoami               # verify

# 2. Ensure code is committed
git status                       # should be clean
git log --oneline -5             # confirm latest commits

# 3. Create Heroku app (first time only)
heroku create ostad-devops

# 4. Deploy
git push heroku main

# 5. Verify
heroku open                      # opens browser
heroku logs --tail               # watch live logs

# 6. Re-deploy after changes
git add .
git commit -m "feat: update"
git push heroku main
git push origin main             # also push to GitHub
```

---

**Congratulations!** Your app is live at:

```
https://ostad-devops.herokuapp.com
```

You've completed the full journey: **GitHub account → Git CLI → Repositories → Commands → Branches → Protection → Fork & Collaborate → Deploy**.

---

## 🧑‍💻 Author

*Md. Sarowar Alam*  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/sarowar/

---
