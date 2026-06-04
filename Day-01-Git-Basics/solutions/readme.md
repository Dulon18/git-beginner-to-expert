# Day 1 — Solutions

> **Try the exercises yourself before reading these.** The struggle is where learning happens.
>
> ← Back to [exercises.md](./exercises.md)

---

## Solution 1 — Setup & Verify

```bash
# Check Git is installed
git --version
# Expected: git version 2.x.x

# Set your identity
git config --global user.name "Jane Smith"
git config --global user.email "jane@example.com"
git config --global init.defaultBranch main

# Verify
git config --list
# Expected output includes:
# user.name=Jane Smith
# user.email=jane@example.com
# init.defaultbranch=main
```

**Where is this stored?**

Your global config lives at `~/.gitconfig`. You can view it directly:
```bash
cat ~/.gitconfig
```

---

## Solution 2 — Your First Repository

```bash
# Create and enter the folder
mkdir git-practice
cd git-practice

# Initialize
git init
# Initialized empty Git repository in /path/to/git-practice/.git/

# See the hidden .git folder
ls -la
# drwxr-xr-x  .git

# Check status on a fresh repo
git status
# On branch main
# No commits yet
# nothing to commit (create/copy files and use "git add" to track)

# Peek inside .git/
ls .git/
# HEAD  config  description  hooks/  info/  objects/  refs/
```

**Answer: What if you deleted `.git/?`**

```bash
rm -rf .git
git status
# fatal: not a git repository
```

Git would have no history, no configuration, and no knowledge of the project. Your files would still exist as plain files, but all version control would be gone. That's why you should never delete `.git/` accidentally.

**Answer: Why is `.git/` hidden?**

Folders starting with a dot (`.`) are hidden on Unix systems. This convention keeps the project root clean — you work in the project, and Git's internals stay out of the way.

---

## Solution 3 — Stage and Commit

```bash
# Create three files
echo "# Git Practice Project" > README.md
echo "body { margin: 0; }" > style.css
echo "<h1>Hello Git</h1>" > index.html

# Check status — all three are Untracked
git status
# Untracked files:
#   index.html
#   README.md
#   style.css

# Stage only README.md
git add README.md

# Status now shows README staged, others still untracked
git status
# Changes to be committed:
#   new file: README.md
# Untracked files:
#   index.html
#   style.css

# First commit
git commit -m "Initial commit: add README"
# [main (root-commit) a3f2c91] Initial commit: add README

# Stage and commit index.html
git add index.html
git commit -m "Add homepage HTML structure"

# Stage and commit style.css
git add style.css
git commit -m "Add base CSS styles"

# Verify history
git log --oneline
# c9f1a34 (HEAD -> main) Add base CSS styles
# b7e8d02 Add homepage HTML structure
# a3f2c91 Initial commit: add README
```

**Answer: Why commit each file separately?**

Commits should represent one logical change. Grouping unrelated changes makes history harder to read and makes it harder to revert a specific change without affecting others. For example:

- ✅ `"Add homepage HTML structure"` — clear, one purpose
- ❌ `"Add HTML, CSS, and README"` — hard to revert CSS without reverting HTML

**Answer: What does `HEAD -> main` mean?**

- `HEAD` is a pointer to "where you are right now" in history
- `main` is the name of your current branch
- `HEAD -> main` means "I am currently on the `main` branch, at this commit"

---

## Solution 4 — Track Changes with diff

```bash
# Add content to README.md
echo "" >> README.md
echo "A hands-on Git learning project." >> README.md
echo "" >> README.md
echo "## Author" >> README.md
echo "Your Name" >> README.md

# See unstaged changes
git diff
# diff --git a/README.md b/README.md
# --- a/README.md
# +++ b/README.md
# @@ -1 +1,5 @@
#  # Git Practice Project
# +
# +A hands-on Git learning project.
# +
# +## Author
# +Your Name

# Stage the file
git add README.md

# git diff now shows nothing (no UNstaged changes)
git diff
# (empty output)

# git diff --staged shows what IS staged
git diff --staged
# (same output as before, showing the staged changes)

# Commit
git commit -m "Update README: add description and author"

git log --oneline
# 4 commits shown
```

**Answer: `git diff` vs `git diff --staged`**

| Command | Compares |
|---------|----------|
| `git diff` | Working directory vs staging area (shows what is NOT yet staged) |
| `git diff --staged` | Staging area vs last commit (shows what IS staged and ready to commit) |

Think of it this way:
- `git diff` = "what have I changed but NOT prepared for committing yet?"
- `git diff --staged` = "what is about to go into my next commit?"

---

## Solution 5 — The .gitignore File

```bash
# Create fake sensitive/junk files
echo "SECRET_KEY=abc123" > .env
echo "debug info" > debug.log
mkdir node_modules
echo "a module" > node_modules/some-package.js

# Status shows everything (bad!)
git status
# Untracked files:
#   .env
#   debug.log
#   node_modules/

# Create .gitignore
touch .gitignore
```

Contents of `.gitignore`:
```
# Environment variables (never commit secrets)
.env

# Log files
*.log

# Dependencies folder
node_modules/
```

```bash
# Status now ignores the files
git status
# Untracked files:
#   .gitignore

# Stage and commit .gitignore itself
git add .gitignore
git commit -m "Add .gitignore: ignore .env, logs, node_modules"

# Final check
git status
# On branch main
# nothing to commit, working tree clean
```

**Answer: Why is committing `.env` dangerous?**

`.env` files typically contain:
- API keys
- Database passwords
- Secret tokens
- Private credentials

Once pushed to a public GitHub repository, these secrets are **permanently exposed** even if you delete them in a later commit (the history still contains them). Attackers scan GitHub for leaked credentials automatically. This is why `.env` should always be in `.gitignore` before your first commit.

**Answer: What if you already committed a file?**

Adding it to `.gitignore` won't help — Git already tracks it. You need to remove it from tracking:
```bash
git rm --cached .env        # stop tracking, keep file on disk
git add .gitignore
git commit -m "Remove .env from tracking"
```
After this, `.env` stays on your machine but Git ignores future changes to it.

---

## Solution 6 — Explore History

```bash
# Compact history
git log --oneline
# f2a1b3c (HEAD -> main) Add .gitignore: ignore .env, logs, node_modules
# d8e7f91 Update README: add description and author
# c9f1a34 Add base CSS styles
# b7e8d02 Add homepage HTML structure
# a3f2c91 Initial commit: add README

# Full log with details
git log
# commit f2a1b3c...
# Author: Jane Smith <jane@example.com>
# Date:   Mon Jan 1 12:00:00 2024 +0000
#
#     Add .gitignore: ignore .env, logs, node_modules

# Inspect the first commit (use your actual hash)
git show a3f2c91
# commit a3f2c91...
# Author: Jane Smith <jane@example.com>
# Date:   ...
#
#     Initial commit: add README
#
# diff --git a/README.md b/README.md
# new file mode 100644
# +# Git Practice Project

# Last 3 commits
git log -3 --oneline

# Commits that touched README.md
git log --oneline -- README.md
# d8e7f91 Update README: add description and author
# a3f2c91 Initial commit: add README

# Visual graph
git log --oneline --graph --all
# * f2a1b3c (HEAD -> main) Add .gitignore
# * d8e7f91 Update README
# * c9f1a34 Add base CSS styles
# * b7e8d02 Add homepage HTML structure
# * a3f2c91 Initial commit: add README
```

**Reading `git log` output:**

```
commit f2a1b3c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4   ← full SHA hash (unique ID)
Author: Jane Smith <jane@example.com>               ← who made the commit
Date:   Mon Jan 1 12:00:00 2024 +0000              ← when

    Add .gitignore: ignore .env, logs, node_modules  ← the message
```

---

## Solution 7 — Unstage & Amend (Bonus)

**Part A: Unstage a file**

```bash
# Make a change
echo "h1 { color: red; }" >> style.css

# Accidentally stage everything
git add .

# Check what's staged
git status
# Changes to be committed:
#   modified: style.css

# Unstage style.css — changes stay in working directory
git restore --staged style.css

# Confirm it's unstaged
git status
# Changes not staged for commit:
#   modified: style.css
```

The change to `style.css` is still there — `restore --staged` only removes it from the staging area, it does NOT discard your edits.

**Part B: Fix the last commit message**

```bash
# Make a commit with a typo
echo "# new section" >> README.md
git add README.md
git commit -m "addd new section to README"

# Check the typo is there
git log --oneline
# 7b3a9f2 (HEAD -> main) addd new section to README   ← typo!

# Fix it
git commit --amend -m "Add new section to README"

# Verify — same hash position, corrected message
git log --oneline
# 9c4b1e3 (HEAD -> main) Add new section to README    ← fixed!
```

> ⚠️ **Important:** Notice the commit hash changed (`7b3a9f2` → `9c4b1e3`). `--amend` creates a **new commit** that replaces the old one. This is why you should **never amend commits that have already been pushed** to a shared repository — it rewrites history and causes problems for collaborators.

---

## 📊 Day 1 Command Cheatsheet

```bash
# Setup
git config --global user.name "Name"
git config --global user.email "email"

# Start
git init                    # new repo
git status                  # check state

# Stage
git add <file>              # stage specific file
git add .                   # stage everything
git restore --staged <file> # unstage

# Commit
git commit -m "message"     # commit with message
git commit --amend -m "fix" # fix last commit

# Inspect
git diff                    # unstaged changes
git diff --staged           # staged changes
git log --oneline           # compact history
git show <hash>             # inspect a commit

# Ignore
# create .gitignore with patterns
```

---

*➡️ Ready for Day 2? [Branching & Merging →](../Day-02-Branching-and-Merging/README.md)*
