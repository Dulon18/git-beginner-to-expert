# Day 3 — Remote Repositories & GitHub Collaboration

> **Goal:** Connect your local Git repo to GitHub, push and pull changes, collaborate with others using forks and pull requests, and understand the full open-source contribution workflow.

---

## 📋 Table of Contents

- [What is a Remote Repository?](#what-is-a-remote-repository)
- [Local vs Remote](#local-vs-remote)
- [Setting Up GitHub](#setting-up-github)
- [Connecting to a Remote](#connecting-to-a-remote)
- [Cloning a Repository](#cloning-a-repository)
- [Pushing Changes](#pushing-changes)
- [Fetching and Pulling](#fetching-and-pulling)
- [Remote Branches & Tracking](#remote-branches--tracking)
- [Collaboration Workflows](#collaboration-workflows)
  - [Shared Repository Workflow](#shared-repository-workflow)
  - [Fork & Pull Request Workflow](#fork--pull-request-workflow)
- [Pull Requests](#pull-requests)
- [Handling Remote Conflicts](#handling-remote-conflicts)
- [SSH vs HTTPS](#ssh-vs-https)
- [Summary](#summary)
- [What's Next](#whats-next)

---

## What is a Remote Repository?

A **remote repository** is a version of your project hosted on the internet or a network — somewhere other than your local machine. It acts as the shared hub where all collaborators sync their work.

```
Your Machine          GitHub (Remote)          Teammate's Machine
─────────────         ───────────────          ──────────────────
local repo   ──push─► remote repo ◄──clone──  local repo
             ◄─pull──             ──push──►
```

Remotes give you:
- **Backup** — your work is safe even if your laptop dies
- **Collaboration** — multiple people sharing one source of truth
- **Deployment** — many services deploy directly from GitHub
- **Open source** — contribute to projects worldwide

---

## Local vs Remote

| | Local Repository | Remote Repository |
|--|-----------------|-------------------|
| **Location** | Your machine | Server (GitHub, GitLab, etc.) |
| **Speed** | Instant | Network dependent |
| **Access** | Just you | Anyone with permission |
| **Commands** | `commit`, `branch`, `merge` | `push`, `pull`, `fetch`, `clone` |
| **Offline** | Works fine | Requires internet |

> Git is a **distributed** system. Your local repo is a full copy — you have the entire history locally. You don't need the remote to work; you only need it to share.

---

## Setting Up GitHub

### Create a GitHub account
Go to [github.com](https://github.com) and sign up.

### Authenticate: SSH Keys (Recommended)

SSH keys let you connect to GitHub without typing a password every time.

```bash
# Step 1: Generate an SSH key
ssh-keygen -t ed25519 -C "you@example.com"
# Press Enter to accept defaults, optionally set a passphrase

# Step 2: Copy the public key
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAAC3Nza... you@example.com

# Step 3: Add to GitHub
# GitHub → Settings → SSH and GPG keys → New SSH key → paste it

# Step 4: Test the connection
ssh -T git@github.com
# Hi username! You've successfully authenticated.
```

### Authenticate: Personal Access Token (HTTPS alternative)

```bash
# GitHub → Settings → Developer settings → Personal access tokens → Generate
# Use the token as your password when Git prompts for credentials
```

---

## Connecting to a Remote

### Add a remote to an existing local repo

```bash
git remote add origin https://github.com/username/repo-name.git
# or with SSH:
git remote add origin git@github.com:username/repo-name.git
```

`origin` is the conventional name for the primary remote. You can name it anything, but `origin` is the universal standard.

### View remotes

```bash
git remote              # list remote names
git remote -v           # list remotes with their URLs
# origin  git@github.com:username/repo.git (fetch)
# origin  git@github.com:username/repo.git (push)
```

### Manage remotes

```bash
git remote rename origin upstream       # rename a remote
git remote set-url origin <new-url>     # change a remote's URL
git remote remove origin               # remove a remote
```

---

## Cloning a Repository

`git clone` downloads a complete copy of a remote repo — all files, all history, all branches.

```bash
git clone https://github.com/username/repo-name.git
# Cloning into 'repo-name'...

# Clone into a specific folder name
git clone https://github.com/username/repo-name.git my-folder

# Clone with SSH
git clone git@github.com:username/repo-name.git
```

After cloning:
- The remote is automatically named `origin`
- You're on the default branch (usually `main`)
- All remote branches are available as `origin/branch-name`

---

## Pushing Changes

`git push` uploads your local commits to the remote.

```bash
# First push — set the upstream tracking branch
git push -u origin main
# -u sets the upstream so future pushes can just be: git push

# Subsequent pushes
git push

# Push a specific branch
git push origin feature/login

# Push all branches
git push --all origin

# Push tags
git push origin --tags
```

### What happens when you push

```
Local:   a──b──c──d   (main)
                 ↑
Remote:  a──b──c       (origin/main)

After git push:
Remote:  a──b──c──d   (origin/main) ✓
```

### Push rejected?

```
! [rejected] main -> main (non-fast-forward)
```

This means someone else pushed commits you don't have. You must pull first:
```bash
git pull
# resolve any conflicts
git push
```

---

## Fetching and Pulling

### git fetch

Downloads changes from the remote **without** modifying your working directory or local branches. Safe to run anytime.

```bash
git fetch origin              # fetch all changes from origin
git fetch origin main         # fetch only the main branch
git fetch --all               # fetch from all remotes
```

After fetching, changes are in `origin/main` — a remote-tracking branch. Your local `main` is untouched:

```
Local main:        a──b──c
Remote origin/main: a──b──c──d──e   (new commits from teammates)
```

Inspect what was fetched before merging:
```bash
git log main..origin/main --oneline   # commits on remote not on local
git diff main origin/main             # see exact changes
```

### git pull

Downloads AND integrates remote changes into your current branch. It's `fetch` + `merge` in one step.

```bash
git pull                    # pull from tracked upstream
git pull origin main        # explicit: pull main from origin
git pull --rebase           # pull using rebase instead of merge
```

```
Before pull:
Local main:        a──b──c
Remote origin/main: a──b──c──d──e

After git pull:
Local main:        a──b──c──d──e   ✓
```

### fetch vs pull — when to use which

| | `git fetch` | `git pull` |
|--|-------------|------------|
| Downloads remote changes | ✅ | ✅ |
| Updates local branch | ❌ | ✅ |
| Safe to run anytime | ✅ | ⚠️ can create conflicts |
| Good for | Reviewing before integrating | Quick sync when you trust the remote |

> 💡 **Pro habit:** `git fetch` first, review with `git log`, then `git merge origin/main`. This gives you control. `git pull` is fine for solo projects; teams prefer `fetch` + inspect + merge.

---

## Remote Branches & Tracking

### Remote-tracking branches

When you fetch, Git creates **remote-tracking branches** like `origin/main`. These are read-only snapshots of the remote's state — they update only when you fetch or pull.

```bash
git branch -r                 # list remote-tracking branches
# origin/main
# origin/feature/login
# origin/HEAD -> origin/main

git branch -a                 # all: local + remote-tracking
```

### Tracking relationships

A **tracking relationship** links a local branch to a remote branch. This is what makes `git push` and `git pull` (without arguments) work.

```bash
# Set up tracking when pushing for the first time
git push -u origin feature/login

# Set tracking on an existing branch
git branch --set-upstream-to=origin/main main

# Check tracking relationships
git branch -vv
# * main     c9f1a34 [origin/main] Add .gitignore
#   feature  a1b2c3d [origin/feature/login: ahead 2] Add login form
```

`ahead 2` means you have 2 local commits not yet pushed.
`behind 3` means the remote has 3 commits you haven't pulled.

### Check out a remote branch locally

```bash
git switch feature/login              # Git auto-tracks origin/feature/login
# or explicitly:
git switch -c feature/login origin/feature/login
```

---

## Collaboration Workflows

### Shared Repository Workflow

Everyone has push access to the same repository. Common in small teams and companies.

```
         GitHub: main
              │
     ┌────────┴────────┐
     │                 │
  Alice              Bob
  (clones)          (clones)
     │                 │
  feature/A         feature/B
     │                 │
  git push          git push
     │                 │
     └────────┬────────┘
         GitHub: main
         (after PRs merged)
```

**Typical flow:**

```bash
# Start of day — get latest
git switch main
git pull

# Create your feature branch
git switch -c feature/my-feature

# Do your work, commit
git add .
git commit -m "Add my feature"

# Push your branch
git push -u origin feature/my-feature

# Open a Pull Request on GitHub
# Get it reviewed and merged

# Clean up locally
git switch main
git pull
git branch -d feature/my-feature
```

---

### Fork & Pull Request Workflow

Used for open-source projects where contributors don't have push access to the main repo.

```
Original Repo (upstream)          Your Fork (origin)
─────────────────────────         ──────────────────
  maintainer/project       fork►   you/project
         │                              │
         │                           clone
         │                              │
         │                        Your Machine
         │                              │
         │ ◄──── Pull Request ─────── push
         │                         (feature branch)
```

**Step-by-step:**

```bash
# 1. Fork on GitHub (click Fork button)

# 2. Clone your fork
git clone git@github.com:you/project.git
cd project

# 3. Add the original repo as upstream
git remote add upstream git@github.com:maintainer/project.git

# 4. Verify remotes
git remote -v
# origin    git@github.com:you/project.git (fetch)
# origin    git@github.com:you/project.git (push)
# upstream  git@github.com:maintainer/project.git (fetch)
# upstream  git@github.com:maintainer/project.git (push)

# 5. Keep your fork in sync with upstream
git fetch upstream
git switch main
git merge upstream/main
git push origin main        # update your fork too

# 6. Create a feature branch
git switch -c fix/typo-in-readme

# 7. Make changes and commit
git add README.md
git commit -m "Fix typo in README introduction"

# 8. Push to YOUR fork
git push -u origin fix/typo-in-readme

# 9. Open a Pull Request on GitHub:
#    Your fork → fix/typo-in-readme  →  maintainer/project → main
```

---

## Pull Requests

A **Pull Request (PR)** is a GitHub feature that lets you propose changes, get code review, and merge branches collaboratively.

### Anatomy of a good PR

```
Title:       Fix: correct typo in README introduction

Description:
  ## What changed
  Fixed "recieve" → "receive" in the Getting Started section.

  ## Why
  Typo was confusing new contributors.

  ## How to test
  Read the README — the spelling should be correct now.

  ## Related issues
  Closes #42
```

### PR review workflow

```
Author pushes branch → Opens PR → Reviewers assigned
       ↓
Reviewer leaves comments
       ↓
Author addresses feedback (pushes more commits to same branch)
       ↓
Reviewer approves
       ↓
PR merged → branch deleted
```

### After your PR is merged

```bash
git switch main
git pull                        # get the merged changes
git branch -d feature/my-pr    # delete local branch
git push origin --delete feature/my-pr   # delete remote branch
```

---

## Handling Remote Conflicts

When two people push to the same branch, conflicts arise.

### Scenario: Push rejected

```bash
git push origin main
# ! [rejected] main -> main (non-fast-forward)
# error: failed to push some refs to 'origin'
# hint: Updates were rejected because the remote contains work
# hint: that you do not have locally.
```

### Resolution

```bash
# 1. Fetch and see what's there
git fetch origin

# 2. Compare
git log main..origin/main --oneline

# 3. Pull (merge remote into local)
git pull origin main
# If conflicts appear, resolve them as in Day 2

# 4. Push again
git push origin main
```

### Pull with rebase (cleaner history)

```bash
git pull --rebase origin main
```

Instead of creating a merge commit, this replays your local commits on top of the remote commits:

```
Before:
Remote: a──b──c
Local:  a──b──d (your commit)

After git pull --rebase:
Local:  a──b──c──d  (your commit replayed on top — clean line)
```

---

## SSH vs HTTPS

| | SSH | HTTPS |
|--|-----|-------|
| **URL format** | `git@github.com:user/repo.git` | `https://github.com/user/repo.git` |
| **Auth method** | SSH key pair | Username + token |
| **Password prompt** | Never (after setup) | Every time (unless cached) |
| **Setup effort** | One-time key setup | Minimal |
| **Recommended** | ✅ For daily use | For quick/one-time access |

**Check which protocol you're using:**
```bash
git remote -v
# SSH:   git@github.com:user/repo.git
# HTTPS: https://github.com/user/repo.git

# Switch from HTTPS to SSH
git remote set-url origin git@github.com:username/repo.git
```

---

## Summary

| Concept | What to remember |
|---------|-----------------|
| **Remote** | A repo hosted on GitHub/GitLab — the shared hub |
| **`git clone`** | Download a full copy of a remote repo |
| **`git push`** | Upload local commits to the remote |
| **`git fetch`** | Download remote changes without touching local branches |
| **`git pull`** | Fetch + merge in one step |
| **`origin`** | Conventional name for your primary remote |
| **`upstream`** | Conventional name for the original repo you forked from |
| **PR** | A proposal to merge a branch — where code review happens |
| **Tracking branch** | Links local branch to remote counterpart |

### The Daily Remote Workflow

```
git pull  →  git switch -c feature/x  →  commits  →  git push -u origin feature/x  →  open PR
```

---

## What's Next

You now know how to collaborate with the world. Tomorrow you'll learn the power tools — rebase, stash, cherry-pick, and bisect — that separate good developers from great ones.

➡️ **[Day 4 — Advanced Git](../Day-04-Advanced-Git/README.md)**

---

*📝 Ready to practice? Try the [exercises](./exercises.md) — solutions are in [solutions.md](./solutions.md)*
