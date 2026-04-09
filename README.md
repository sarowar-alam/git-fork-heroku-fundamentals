# git-fork-heroku-fundamentals - change form Owner

> A complete, zero-to-deploy Git learning repository.  
> Covers every Git concept from account creation to live Heroku deployment — with a production-quality Node.js landing page as the deliverable.

**Live demo:** `https://ostad-devops.herokuapp.com`  
**GitHub:** `https://github.com/sarowar-alam/git-fork-heroku-fundamentals`  
**Author:** Sarowar Alam · Ostad Batch 11, Module 02

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [Repository Structure](#3-repository-structure)
4. [Prerequisites](#4-prerequisites)
5. [Local Development Setup](#5-local-development-setup)
6. [Running the Application](#6-running-the-application)
7. [Environment Variables](#7-environment-variables)
8. [Branch Strategy](#8-branch-strategy)
9. [Contributing — Making Changes Safely](#9-contributing--making-changes-safely)
10. [Deployment](#10-deployment)
    - [Heroku (primary)](#heroku-primary)
    - [Re-deploying after changes](#re-deploying-after-changes)
    - [Auto-deploy via GitHub](#auto-deploy-via-github)
11. [Release Workflow](#11-release-workflow)
12. [Operational Tasks](#12-operational-tasks)
13. [Learning Guides](#13-learning-guides)
14. [Dependencies](#14-dependencies)
15. [Reliability Considerations](#15-reliability-considerations)
16. [Troubleshooting](#16-troubleshooting)

---

## 1. Project Overview

This repository serves two purposes simultaneously:

| Purpose | Description |
|---|---|
| **Learning resource** | 7 structured guides covering Git fundamentals — from account setup through deployment |
| **Deployed application** | A Node.js/Express server that hosts a gradient SaaS-style landing page linking all guides |

The guides are the documentation. The app is the delivery mechanism. The repo itself demonstrates everything it teaches — branching, PRs, protection rules, and Heroku deployment.

**What this is NOT:**
- A framework-heavy app — it is intentionally minimal (Express + static files, no build step)
- A database-backed service — no persistence layer
- A test suite — educational focus; tests are out of scope at this stage

---

## 2. Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     GitHub                              │
│  sarowar-alam/git-fork-heroku-fundamentals              │
│  ├── Branch: main  (protected — PRs required)           │
│  └── Branch: develop (integration, optional)            │
└────────────────────┬────────────────────────────────────┘
                     │ git push heroku main
                     ▼
┌─────────────────────────────────────────────────────────┐
│                    Heroku                               │
│  App: ostad-devops                                      │
│  Stack: heroku-22 (Ubuntu 22.04)                        │
│  Runtime: Node.js ≥18.x (auto-detected via package.json)│
│  Process: web dyno — node index.js                      │
│  Port: process.env.PORT (set by Heroku automatically)   │
└────────────────────┬────────────────────────────────────┘
                     │ HTTP
                     ▼
┌─────────────────────────────────────────────────────────┐
│             Express Server (index.js)                   │
│  app.use(express.static('public/'))                     │
│  GET /  → public/index.html                             │
│  *      → 404 handler                                   │
└────────────────────┬────────────────────────────────────┘
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   public/index.html      public/style.css
   (SaaS landing page)    (gradient theme)
```

**Design decisions:**

| Decision | Rationale |
|---|---|
| No frontend build step | Keeps deployment simple — `git push` is the full pipeline |
| `process.env.PORT` binding | Required by Heroku; falls back to `3000` for local dev |
| Static file serving via Express | Single dependency, no Nginx config needed |
| `Procfile` declares `web:` process | Explicit over implicit — Heroku knows exactly how to start the app |
| Node ≥18.x in `engines` field | Pins minimum runtime; Heroku respects this during slug compilation |

---

## 3. Repository Structure

```
git-fork-heroku-fundamentals/
│
├── index.js                  # Express server — entry point
├── package.json              # Dependencies, start script, engine constraint
├── package-lock.json         # Locked dependency tree (committed — ensures reproducible installs)
├── Procfile                  # Heroku process declaration: web: node index.js
├── .gitignore                # Excludes: node_modules/, .env, logs, coverage
│
├── public/                   # Static assets served by Express
│   ├── index.html            # Landing page — links all guides, includes cheatsheet
│   └── style.css             # Dark gradient theme, responsive, no framework
│
└── docs/                     # Structured learning guides (Markdown)
    ├── 01-SETUP.md           # GitHub account · Git install (Win/Mac/Linux) · SSH/HTTPS auth
    ├── 02-REPOS.md           # Create/clone repos · private repo auth · remotes · upstream ·
    │                         #   forking workflow · PRs · roles (14 sections)
    ├── 03-COMMANDS.md        # 25+ commands with examples — init through reflog
    ├── 04-BRANCHES.md        # Branching · merge · rebase · cherry-pick · Git Flow
    ├── 05-BRANCH-PROTECTION.md  # Restrict deletion · block force push · require PRs · CI checks
    ├── 06-FORK-COLLABORATE.md   # Fork · upstream · sync · PR · code review · merge strategies
    └── 07-HEROKU-DEPLOY.md      # MFA-safe deploy · CLI · logs · env vars · auto-deploy · errors
```

**Key files at a glance:**

| File | What to change it for |
|---|---|
| `index.js` | Add routes, middleware, API endpoints |
| `public/index.html` | Update landing page content, links, sections |
| `public/style.css` | Modify theme colours, layout, responsive breakpoints |
| `package.json` | Add dependencies, change Node version requirement |
| `Procfile` | Change process type or start command for Heroku |
| `.gitignore` | Add file patterns to exclude from version control |

---

## 4. Prerequisites

### Required

| Tool | Version | Install |
|---|---|---|
| **Git** | ≥ 2.39 | [git-scm.com](https://git-scm.com/downloads) |
| **Node.js** | ≥ 18.x | [nodejs.org](https://nodejs.org) or `nvm install 20` |
| **npm** | ≥ 9.x (comes with Node) | Included with Node.js |

### For deployment

| Tool | Notes |
|---|---|
| **Heroku CLI** | [Install guide](https://devcenter.heroku.com/articles/heroku-cli) · Run `heroku --version` to verify |
| **Heroku account** | [signup.heroku.com](https://signup.heroku.com) · MFA users: use API key auth (see §10) |
| **GitHub account** | [github.com/signup](https://github.com/signup) · See [docs/01-SETUP.md](docs/01-SETUP.md) |

### Verify your environment

```bash
git --version        # git version 2.x.x
node --version       # v20.x.x
npm --version        # 10.x.x
heroku --version     # heroku/9.x.x  (if deploying)
```

---

## 5. Local Development Setup

```bash
# 1. Clone the repository
git clone https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git
cd git-fork-heroku-fundamentals

# 2. Install dependencies
npm install

# 3. (Optional) Create a local environment file
cp .env.example .env     # if .env.example exists
# Or create manually:
echo "PORT=3000" > .env
```

> **Note:** `.env` is in `.gitignore` and will never be committed. Do not add secrets to any other file.

**If you are contributing (forked workflow):**

```bash
# After forking on GitHub, clone YOUR fork
git clone https://github.com/YOUR-USERNAME/git-fork-heroku-fundamentals.git
cd git-fork-heroku-fundamentals

# Add the original repo as upstream
git remote add upstream https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

# Verify remotes
git remote -v
# origin    https://github.com/YOUR-USERNAME/git-fork-heroku-fundamentals.git
# upstream  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git

npm install
```

---

## 6. Running the Application

### Start the server

```bash
npm start
# > node index.js
# Server running on http://localhost:3000
```

Open `http://localhost:3000` in your browser.

### Stopping the server

Press `Ctrl + C` in the terminal.

### What the app does

| Route | Response |
|---|---|
| `GET /` | Serves `public/index.html` — the landing page |
| `GET /style.css` | Served automatically by `express.static` |
| Any other route | 404 page with a link back to `/` |

There are **no build steps**, **no compilation**, and **no watch mode** needed. Edit `public/index.html` or `public/style.css` and refresh the browser — changes are immediate.

---

## 7. Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `PORT` | No | `3000` | Port the Express server listens on. **Heroku sets this automatically** — do not hardcode it. |
| `NODE_ENV` | No | `development` | Set to `production` on Heroku automatically. |

**Local development** — create a `.env` file (never commit it):
```bash
PORT=3000
```

**Heroku** — set via CLI:
```bash
heroku config:set NODE_ENV=production
heroku config                          # list all vars
heroku config:get PORT                 # view a single var
heroku config:unset MY_VAR             # remove a var
```

> Adding future secrets (API keys, DB URLs): always use `heroku config:set` — never commit them to the repository.

---

## 8. Branch Strategy

This repository follows **GitHub Flow** — simple, continuous delivery friendly.

```
main ──────────────────────────────────────────────► (always deployable)
         │              │              │
feature/xxx          fix/yyy       docs/zzz
(branch off main, PR back to main when done)
```

| Branch | Purpose | Direct push? |
|---|---|---|
| `main` | Production. Every commit here is deployed. | ❌ PRs only |
| `feature/*` | New features | ✅ (your own branch) |
| `fix/*` | Bug fixes | ✅ (your own branch) |
| `docs/*` | Documentation updates | ✅ (your own branch) |
| `hotfix/*` | Urgent production patches | ✅ (your own branch) |

**Branch protection on `main`** (configured in GitHub Settings → Branches):
- ✅ Restrict branch deletion
- ✅ Block force pushes (history is permanent)
- ✅ Require pull request before merging
- ✅ Require at least 1 approval

See [docs/05-BRANCH-PROTECTION.md](docs/05-BRANCH-PROTECTION.md) for full setup instructions.

---

## 9. Contributing — Making Changes Safely

Follow this workflow for every change, no matter how small.

```bash
# Step 1 — Sync with latest main before branching
git switch main
git pull origin main        # or: git pull upstream main (if forked)

# Step 2 — Create a descriptive branch
git switch -c feature/add-new-guide
# Naming: feature/ | fix/ | docs/ | hotfix/ | refactor/ | chore/

# Step 3 — Make changes
# Edit files...

# Step 4 — Stage and commit (follow conventional commits)
git add .
git commit -m "feat: add render deployment guide to docs"
# Types: feat | fix | docs | style | refactor | test | chore | perf

# Step 5 — Push to remote
git push -u origin feature/add-new-guide

# Step 6 — Open a Pull Request on GitHub
# Base: main  ←  Compare: feature/add-new-guide
# Fill in: title, description, linked issue (Closes #N)

# Step 7 — After PR is merged, clean up
git switch main
git pull origin main
git branch -d feature/add-new-guide
git push origin --delete feature/add-new-guide
```

### Commit message convention

```
<type>(<optional scope>): <short summary>

<optional body — explain WHY not WHAT>

<optional footer — Closes #N, Breaking Change: ...>
```

Examples:
```
feat: add dark mode toggle to navbar
fix: correct PORT binding to use process.env.PORT
docs: update heroku deploy guide with MFA workaround
chore: upgrade express from 4.18.2 to 4.19.0
```

### What NOT to do

```bash
# ❌ Never push directly to main
git push origin main

# ❌ Never force push to main
git push --force origin main

# ❌ Never commit sensitive values
echo "API_KEY=secret" >> index.js && git commit  # don't do this
```

---

## 10. Deployment

### Heroku (primary)

#### One-time setup

```bash
# 1. Authenticate — MFA-safe method (works on Ubuntu/WSL/any terminal)
#    Get your API key from: Heroku Dashboard → Account Settings → API Key → Reveal
export HEROKU_API_KEY=your-api-key-here

# Verify authentication
heroku auth:whoami

# 2. Create the Heroku app (first time only)
heroku create ostad-devops

# 3. Confirm the heroku remote was added automatically
git remote -v
# heroku  https://git.heroku.com/ostad-devops.git (fetch)
# heroku  https://git.heroku.com/ostad-devops.git (push)
# origin  https://github.com/sarowar-alam/git-fork-heroku-fundamentals.git (fetch)

# 4. Deploy
git push heroku main

# 5. Open the live app
heroku open
```

#### What Heroku does on `git push heroku main`

```
1. Receives the git push
2. Detects Node.js buildpack (via package.json in root)
3. Runs: npm install --production
4. Reads Procfile → finds: web: node index.js
5. Starts a web dyno running node index.js
6. Binds to $PORT (assigned by Heroku's routing layer)
7. App becomes live at https://ostad-devops.herokuapp.com
```

> **Why `process.env.PORT` matters:** Heroku assigns a random port at runtime. If you hardcode `3000`, the dyno starts but Heroku's router can never reach it → H20 error. Always use `const PORT = process.env.PORT || 3000`.

---

### Re-deploying after changes

```bash
# Make and commit your changes
git add .
git commit -m "feat: update landing page hero section"

# Deploy to Heroku
git push heroku main

# Also keep GitHub in sync
git push origin main
```

The deployment is complete when you see:
```
remote: -----> Build succeeded!
remote: https://ostad-devops.herokuapp.com deployed to Heroku
```

---

### Auto-deploy via GitHub

Eliminate the manual `git push heroku main` step:

1. **Heroku Dashboard** → your app → **Deploy** tab
2. **Deployment method** → **GitHub** → Connect
3. Search and connect repo: `git-fork-heroku-fundamentals`
4. **Automatic deploys** → select branch `main` → **Enable Automatic Deploys**

Now every merged PR to `main` triggers a Heroku build automatically.

---

## 11. Release Workflow

This project uses **continuous deployment** — there are no versioned releases or release branches. Every merge to `main` is a release.

```
Feature branch
     │
     ▼
  Pull Request
     │ review + approval
     ▼
Merge to main
     │ triggers
     ▼
Heroku auto-deploy (if connected)
     │  OR
git push heroku main (manual)
     │
     ▼
Live at https://ostad-devops.herokuapp.com
```

**Tagging (optional, for milestones):**
```bash
# Tag a significant checkpoint
git tag -a v1.0.0 -m "Initial deployment — all 7 guides complete"
git push origin v1.0.0
```

---

## 12. Operational Tasks

### View live logs

```bash
# Stream live logs (press Ctrl+C to stop)
heroku logs --tail --app ostad-devops

# View last 200 lines
heroku logs -n 200 --app ostad-devops
```

### Check dyno status

```bash
heroku ps --app ostad-devops
# web.1: up 2026/04/04 10:00:00 +0000 (~ 5m ago)
```

### Restart the application

```bash
heroku restart --app ostad-devops
```

### Run a one-off command on Heroku

```bash
heroku run node --version --app ostad-devops
heroku run npm list --app ostad-devops
```

### Manage environment variables

```bash
heroku config --app ostad-devops               # list all
heroku config:set KEY=value --app ostad-devops # add/update
heroku config:unset KEY --app ostad-devops     # remove
```

### Scale dynos

```bash
# Scale up (paid tier)
heroku ps:scale web=2 --app ostad-devops

# Scale down to 0 (stop the app)
heroku ps:scale web=0 --app ostad-devops

# Back to 1 (free/eco tier)
heroku ps:scale web=1 --app ostad-devops
```

---

## 13. Learning Guides

The `docs/` folder contains 7 sequential guides. Read them in order — each builds on the last.

| Guide | Topics |
|---|---|
| [01-SETUP.md](docs/01-SETUP.md) | Create GitHub account · Install Git (Windows/macOS/Linux) · First-time config · SSH & HTTPS auth |
| [02-REPOS.md](docs/02-REPOS.md) | Create & clone repos · Public vs private · HTTPS PAT per OS · SSH per OS · Troubleshooting auth · Remotes · Upstream deep dive · Full forking cycle · PRs · GitHub repository roles |
| [03-COMMANDS.md](docs/03-COMMANDS.md) | Every Git command — init through bisect, reflog, worktree, submodule |
| [04-BRANCHES.md](docs/04-BRANCHES.md) | Branches · merge (FF + 3-way) · rebase · interactive rebase · cherry-pick · conflict resolution · Git Flow · trunk-based dev |
| [05-BRANCH-PROTECTION.md](docs/05-BRANCH-PROTECTION.md) | Restrict deletion · block force push · require PRs · require CI · admin bypass · GitHub Rulesets |
| [06-FORK-COLLABORATE.md](docs/06-FORK-COLLABORATE.md) | Fork · clone fork · upstream remote · sync · PR lifecycle · code review · merge strategies |
| [07-HEROKU-DEPLOY.md](docs/07-HEROKU-DEPLOY.md) | Install Heroku CLI · MFA-safe API key auth · create app · deploy · logs · env vars · auto-deploy · common errors |

---

## 14. Dependencies

### Runtime dependencies

| Package | Version | Why it's here |
|---|---|---|
| `express` | `^4.18.2` | HTTP server and static file serving |

### No dev dependencies

This project intentionally has no build tools, test runners, linters, or transpilers. The goal is minimal setup friction for learners.

### Node.js engine

```json
"engines": { "node": ">=18.x" }
```

Node 18+ is required for native `fetch`, improved performance, and long-term support. Heroku respects this field and installs the matching runtime.

### Dependency security

```bash
# Check for known vulnerabilities
npm audit

# Auto-fix low-risk issues
npm audit fix

# View outdated packages
npm outdated
```

`package-lock.json` is committed intentionally — it locks the full dependency tree, ensuring `npm install` produces the same result on every machine and on Heroku.

---

## 15. Reliability Considerations

| Consideration | Current state | Notes |
|---|---|---|
| **Port binding** | `process.env.PORT \|\| 3000` | Correct — required for Heroku |
| **404 handling** | Custom handler in `index.js` | Prevents Express default error output |
| **Static assets** | Served by `express.static` | No CDN — acceptable for learning project |
| **Process crash recovery** | Heroku auto-restarts crashed dynos | Built-in platform behaviour |
| **Zero-downtime deploy** | Not configured | Heroku replaces dyno on push — brief downtime possible |
| **HTTPS** | Enforced by Heroku's routing layer | All traffic to `.herokuapp.com` is HTTPS automatically |
| **Secrets in repo** | None — `.env` in `.gitignore` | Never commit `.env` or API keys |
| **Dependency lock** | `package-lock.json` committed | Ensures reproducible installs |
| **Node version pinned** | `engines.node >= 18.x` | Prevents runtime mismatch on Heroku |
| **Logging** | `heroku logs --tail` | No structured logging — add `morgan` if needed |
| **Health check** | None | Add `GET /health → 200 OK` if integrating uptime monitors |

---

## 16. Troubleshooting

### App crashes on Heroku (H10 error)

```bash
heroku logs --tail --app ostad-devops
```

Most common causes:
1. **Port hardcoded** — change to `process.env.PORT || 3000`
2. **Missing `start` script** — `package.json` must have `"start": "node index.js"`
3. **Missing `Procfile`** — must exist at repo root with `web: node index.js`
4. **Dependency not in `dependencies`** — don't put required packages under `devDependencies`

---

### `heroku: command not found`

```bash
# Re-source your shell profile
source ~/.bashrc       # Linux/WSL
source ~/.zshrc        # macOS

# Or verify the install
which heroku
echo $PATH
```

---

### `HEROKU_API_KEY` not working (MFA users)

```bash
# Check the variable is set
echo $HEROKU_API_KEY     # should print your key

# Get a fresh key from: https://dashboard.heroku.com/account
# Under "API Key" → Reveal → copy

export HEROKU_API_KEY=your-fresh-key
heroku auth:whoami       # should print your email
```

---

### `git push heroku main` rejected

```bash
# Check if the heroku remote exists
git remote -v

# If missing, add it
heroku git:remote -a ostad-devops

# If wrong URL
git remote set-url heroku https://git.heroku.com/ostad-devops.git
```

---

### `npm install` fails

```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

---

### Permission denied (SSH) when cloning

```bash
# Test SSH connection
ssh -T git@github.com

# Check key is loaded
ssh-add -l

# If not loaded
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Verify correct public key is on GitHub
cat ~/.ssh/id_ed25519.pub
# GitHub → Settings → SSH keys → compare
```

Full auth troubleshooting: [docs/02-REPOS.md §7](docs/02-REPOS.md#7-troubleshooting-auth-per-environment)

---

## 🧑‍💻 Author

*Md. Sarowar Alam*  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/sarowar/

---
