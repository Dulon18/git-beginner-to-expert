# Day 1 — Git Basics

> **Goal:** Understand what Git is, how it works, and make your first commits with confidence.

---

## 📋 Table of Contents

- [What is Version Control?](#what-is-version-control)
- [What is Git?](#what-is-git)
- [Installing & Configuring Git](#installing--configuring-git)
- [Core Concepts](#core-concepts)
  - [Repository](#1-repository)
  - [The .git Directory](#2-the-git-directory)
  - [The Three Areas](#3-the-three-areas)
  - [File States](#4-file-states)
  - [Commits](#5-commits)
- [Your First Repository](#your-first-repository)
- [Essential Commands](#essential-commands)
- [Working with .gitignore](#working-with-gitignore)
- [Viewing History](#viewing-history)
- [Summary](#summary)
- [What's Next](#whats-next)

---

## What is Version Control?

A **Version Control System (VCS)** tracks every change you make to files over time. It lets you:

- Go back to any earlier version of your project
- See exactly what changed, when, and who changed it
- Work on new features without breaking working code
- Collaborate with other developers without overwriting each other's work

**Without version control**, developers used to:
- Email zip files back and forth
- Name files `project_final_v2_FINAL_fixed.zip`
- Lose hours of work to accidental deletions

---

## What is Git?

**Git** is a free, open-source **distributed version control system** created by Linus Torvalds in 2005 to manage the Linux kernel source code.

**Key characteristics:**

| Feature | What it means |
|---------|---------------|
| Distributed | Every developer has a full copy of the entire project history |
| Fast | Most operations happen locally, no network needed |
| Secure | Every commit is cryptographically checksummed (SHA-1) |
| Non-linear | Supports thousands of parallel branches |
| Free | Open source, runs on all platforms |

> **Git ≠ GitHub.** Git is the tool that runs on your computer. GitHub is a website that hosts Git repositories online. You can use Git without GitHub.

---

## Installing & Configuring Git

### Install Git

| OS | Command |
|----|---------|
| **macOS** | `brew install git` or download from [git-scm.com](https://git-scm.com) |
| **Ubuntu/Debian** | `sudo apt install git` |
| **Windows** | Download [Git for Windows](https://gitforwindows.org) |

Verify installation:
```bash
git --version
# git version 2.44.0
```

### Configure Git (do this once)

Before making any commits, tell Git who you are:

```bash
# Set your name (shown in every commit you make)
git config --global user.name "Your Full Name"

# Set your email (should match your GitHub email)
git config --global user.email "you@example.com"

# Set your preferred text editor
git config --global core.editor "code --wait"    # VS Code
git config --global core.editor "nano"           # Nano
git config --global core.editor "vim"            # Vim

# Set default branch name to 'main'
git config --global init.defaultBranch main

# Make Git output colorful
git config --global color.ui auto
```

View your config at any time:
```bash
git config --list
git config user.name    # check a specific value
```

Config is stored in `~/.gitconfig` on your machine.

---

## Core Concepts

### 1. Repository

A **repository** (repo) is any folder that Git is tracking. It contains:
- Your project files (the stuff you work on)
- A hidden `.git/` folder (Git's database — don't touch this)

Two types of repositories:

```
Local Repo          Remote Repo
(your machine)  ←→  (GitHub, GitLab, etc.)
```

Create a new repo:
```bash
mkdir my-project
cd my-project
git init
# Initialized empty Git repository in /path/to/my-project/.git/
```

---

### 2. The .git Directory

When you run `git init`, Git creates a hidden `.git/` folder. This is the **entire history of your project** stored in a compact database.

```
.git/
├── HEAD          ← points to the current branch
├── config        ← repository-specific settings
├── index         ← the staging area (what's ready to commit)
├── objects/      ← every commit, file snapshot, and tree
└── refs/
    ├── heads/    ← branch pointers
    └── tags/     ← tag pointers
```

> ⚠️ **Never manually edit files inside `.git/`.** If you delete `.git/`, Git forgets all history. Your files survive, but version control is gone.

---

### 3. The Three Areas

Understanding these three areas is the key to understanding Git:

```
┌─────────────────┐    git add     ┌──────────────┐   git commit  ┌──────────────┐
│                 │ ─────────────► │              │ ────────────► │              │
│ Working         │                │ Staging Area │               │  Repository  │
│ Directory       │ ◄───────────── │  (Index)     │               │  (.git/)     │
│                 │   git restore  │              │               │              │
└─────────────────┘                └──────────────┘               └──────────────┘
   Your files                     Changes ready                  Permanent history
   (edit freely)                  to be committed                (immutable)
```

| Area | Description |
|------|-------------|
| **Working Directory** | Your actual project files. Edit freely here. |
| **Staging Area** | A preparation zone. You choose which changes to include in the next commit. |
| **Repository** | The permanent, immutable history of all your commits. |

---

### 4. File States

Every file in your repo is in one of four states:

```
Untracked ──── git add ────► Staged ──── git commit ────► Committed
                                                               │
                              ◄──────── (edit file) ──────────┘
                                           Modified
```

| State | Meaning |
|-------|---------|
| **Untracked** | New file Git has never seen. Not in history, not staged. |
| **Modified** | A tracked file that has changed since the last commit. |
| **Staged** | Change is in the staging area, ready for the next commit. |
| **Committed** | Snapshot saved permanently in the `.git` repository. |

Check file states at any time:
```bash
git status
```

Example output:
```
On branch main

Changes to be committed:        ← STAGED
  (use "git restore --staged <file>..." to unstage)
        new file:   index.html

Changes not staged for commit:  ← MODIFIED
  (use "git add <file>..." to update what will be committed)
        modified:   style.css

Untracked files:                ← UNTRACKED
  (use "git add <file>..." to include in what will be committed)
        notes.txt
```

---

### 5. Commits

A **commit** is a permanent snapshot of your staged changes. Each commit contains:

- A unique **SHA-1 hash** (e.g., `a3f2c91`) — its ID
- The **author** name and email
- A **timestamp**
- A **commit message** describing what changed
- A pointer to the **parent commit** (forming a chain)

```
a3f2c91 ← b7e8d02 ← c9f1a34 ← (current)
  │            │          │
"Initial    "Add CSS"  "Fix nav
  commit"              bug"
```

**Writing good commit messages:**

```bash
# ✅ Good — clear, imperative, specific
git commit -m "Add user login form with validation"
git commit -m "Fix broken image link on homepage"
git commit -m "Refactor database connection logic"

# ❌ Bad — vague, past tense, or useless
git commit -m "stuff"
git commit -m "fixed it"
git commit -m "changes"
git commit -m "asdfgh"
```

> 💡 **Rule of thumb:** Write commit messages in the imperative mood. Complete the sentence: *"If applied, this commit will… [your message]"*

---

## Your First Repository

Here is the complete workflow from scratch:

```bash
# Step 1: Create a new project folder
mkdir my-first-repo
cd my-first-repo

# Step 2: Initialize Git
git init
# Initialized empty Git repository in ./my-first-repo/.git/

# Step 3: Create a file
echo "# My First Project" > README.md

# Step 4: Check what Git sees
git status
# Untracked files: README.md

# Step 5: Stage the file
git add README.md

# Step 6: Check status again
git status
# Changes to be committed: new file: README.md

# Step 7: Make your first commit
git commit -m "Initial commit: add README"
# [main (root-commit) a3f2c91] Initial commit: add README
# 1 file changed, 1 insertion(+)

# Step 8: View your history
git log --oneline
# a3f2c91 (HEAD -> main) Initial commit: add README
```

---

## Essential Commands

### Staging Commands

```bash
git add <filename>        # stage a specific file
git add .                 # stage everything in current directory
git add *.html            # stage all HTML files
git add -p                # stage changes interactively (chunk by chunk)

git restore --staged <file>   # unstage a file (keep changes in working dir)
```

### Commit Commands

```bash
git commit -m "message"       # commit with an inline message
git commit                    # open editor to write a detailed message
git commit --amend            # fix the last commit (message or content)
```

> ⚠️ Only use `--amend` on commits that haven't been pushed yet.

### Inspection Commands

```bash
git status                    # show working dir and staging area state
git diff                      # show unstaged changes (working dir vs staging)
git diff --staged             # show staged changes (staging vs last commit)
git log                       # full commit history
git log --oneline             # compact one-line view
git log --oneline --graph     # visual branch graph
git show <hash>               # show a specific commit's details
git show HEAD                 # show the latest commit
```

### File Management

```bash
git rm <file>                 # delete file and stage the deletion
git mv <old> <new>            # rename/move a file and stage it
```

> Use `git rm` and `git mv` instead of regular `rm` and `mv` — they automatically stage the change.

---

## Working with .gitignore

A `.gitignore` file tells Git which files and folders to **never track**. Create it in your project root:

```bash
touch .gitignore
```

### Common patterns

```gitignore
# Dependencies
node_modules/
vendor/

# Environment & secrets — NEVER commit these
.env
.env.local
*.key
*.pem

# Build output
dist/
build/
*.min.js

# Logs
*.log
logs/

# OS metadata
.DS_Store        # macOS
Thumbs.db        # Windows
desktop.ini      # Windows

# Editor files
.vscode/
.idea/
*.swp            # Vim swap files
```

### Gitignore rules

| Pattern | Matches |
|---------|---------|
| `*.log` | Any file ending in `.log` |
| `logs/` | The entire `logs` folder |
| `!important.log` | Exception — do NOT ignore this file |
| `**/temp` | `temp` folder anywhere in the project |
| `doc/*.txt` | `.txt` files directly inside `doc/` |

After adding `.gitignore`, commit it:
```bash
git add .gitignore
git commit -m "Add .gitignore"
```

> 💡 Use [gitignore.io](https://www.toptal.com/developers/gitignore) to generate `.gitignore` files for your specific tech stack.

---

## Viewing History

```bash
# Basic log
git log

# Compact view
git log --oneline

# Show last 5 commits
git log -5

# Show commits by a specific author
git log --author="Your Name"

# Show commits that changed a specific file
git log -- README.md

# Show full diff for each commit
git log -p

# Visual graph (useful once you learn branches)
git log --oneline --graph --all
```

Example `git log --oneline` output:
```
c9f1a34 (HEAD -> main) Fix broken link in README
b7e8d02 Add style.css with base styles
a3f2c91 Initial commit: add README
```

Each line shows: `[short hash] [commit message]`

`HEAD` means "where you currently are" in the history.

---

## Summary

| Concept | What to remember |
|---------|-----------------|
| **Repository** | A folder Git is tracking, with a `.git/` database inside |
| **Staging area** | The preparation zone — you choose what goes into each commit |
| **Commit** | An immutable snapshot with a hash, author, message, and timestamp |
| **Working dir** | Your actual files — edit freely here |
| **`git status`** | Your best friend — run it constantly |
| **`.gitignore`** | List files Git should never track (node_modules, .env, etc.) |

### The Daily Git Loop

```
Edit files  →  git add  →  git commit  →  repeat
```

---

## What's Next

You've mastered the fundamentals. Tomorrow you'll learn how to work on **multiple things at once** using branches — one of Git's most powerful features.

➡️ **[Day 2 — Branching & Merging](../Day-02-Branching-and-Merging/README.md)**

---

*📝 Ready to practice? Try the [exercises](./exercises.md) — solutions are in [solutions.md](./solutions.md)*
