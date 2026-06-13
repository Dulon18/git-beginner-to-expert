# Day 4 — Advanced Git

> **Goal:** Master power-user techniques — rebase, stash, cherry-pick, bisect, tags, and blame — to work faster and keep history clean.

---

## 📋 Table of Contents

- [Git Rebase](#git-rebase)
  - [Rebase vs Merge](#rebase-vs-merge)
  - [Interactive Rebase](#interactive-rebase)
- [Git Stash](#git-stash)
- [Cherry-Pick](#cherry-pick)
- [Git Bisect](#git-bisect)
- [Git Blame](#git-blame)
- [Git Tags](#git-tags)
- [Git Reflog](#git-reflog)
- [Undoing Things](#undoing-things)
- [Summary](#summary)
- [What's Next](#whats-next)

---

## Git Rebase

**Rebase** moves or replays commits from one branch onto another. Instead of creating a merge commit, it rewrites history to look like the work was done sequentially.

### Rebase vs Merge

```
BEFORE (both):
      A─B─C  (main)
         └─D─E  (feature)

AFTER git merge main:           AFTER git rebase main:
      A─B─C────M  (main)              A─B─C─D'─E'  (feature, then merge)
         └─D─E─┘                               ↑ replayed commits
         (merge commit M)              (linear, clean history)
```

| | Merge | Rebase |
|--|-------|--------|
| History | Preserves exact history, adds merge commit | Rewrites — looks like linear development |
| Safety | Always safe | Never rebase public/shared branches |
| Traceability | Clear when branches joined | Cleaner log, harder to see divergence |
| Use when | Merging into `main` | Keeping feature branch up to date |

### Basic Rebase

```bash
# Rebase your feature branch onto the latest main
git switch feature/my-task
git rebase main

# Step-by-step what happens:
# 1. Git finds the common ancestor of feature and main
# 2. Git temporarily removes your feature commits
# 3. Git moves the branch pointer to the tip of main
# 4. Git replays your commits one by one on top
```

If conflicts arise during rebase:
```bash
# Git pauses and tells you which commit caused the conflict
# Fix the conflict in the file, then:
git add <resolved-file>
git rebase --continue     # move to next commit

# OR abort the whole rebase
git rebase --abort
```

### Interactive Rebase

Interactive rebase (`-i`) lets you edit, reorder, squash, or drop commits before they become permanent. It's the most powerful history-editing tool in Git.

```bash
# Interactively edit the last 4 commits
git rebase -i HEAD~4
```

Git opens your editor with a list of commits:

```
pick a3f2c91 Add login form HTML
pick b7e8d02 Fix typo in login form
pick c9f1a34 Add login form CSS
pick d1e2f3a Add login form validation

# Commands:
# p, pick   = use commit as-is
# r, reword = use commit, but edit the message
# e, edit   = use commit, but pause to amend it
# s, squash = combine with previous commit (keep both messages)
# f, fixup  = combine with previous commit (discard this message)
# d, drop   = remove this commit entirely
```

**Common interactive rebase operations:**

```bash
# Squash 3 commits into 1 clean commit
pick a3f2c91 Add login form HTML
fixup b7e8d02 Fix typo in login form
fixup c9f1a34 Add login form CSS
pick d1e2f3a Add login form validation

# Result: 2 clean commits instead of 4 noisy ones

# Reorder commits (just move the lines)
pick c9f1a34 Add login form CSS
pick a3f2c91 Add login form HTML

# Drop a commit entirely
drop b7e8d02 Fix typo in login form

# Reword a commit message
reword a3f2c91 Add login form HTML
```

> ⚠️ **Golden Rule of Rebase:** Never rebase commits that have been pushed to a shared remote branch. Rebasing rewrites history — if others have based work on those commits, their history breaks.

---

## Git Stash

**Stash** saves your uncommitted changes to a temporary stack so you can switch context without committing half-done work.

```bash
# Save changes to stash
git stash                         # stash all tracked changes
git stash push -m "WIP: login"    # stash with a description
git stash -u                      # include untracked files too

# List stashes
git stash list
# stash@{0}: On feature/login: WIP: login
# stash@{1}: On main: WIP: quick experiment

# Apply stash
git stash pop                     # apply most recent stash AND remove from list
git stash apply                   # apply most recent stash, keep in list
git stash apply stash@{1}         # apply a specific stash

# Inspect a stash
git stash show                    # summary of most recent stash
git stash show -p                 # full diff of most recent stash
git stash show stash@{1} -p       # full diff of specific stash

# Delete stashes
git stash drop stash@{0}          # delete a specific stash
git stash clear                   # delete ALL stashes
```

**Real-world stash workflow:**

```bash
# You're in the middle of feature work
echo "half done code" >> feature.js
git status   # modified: feature.js (not committed)

# Urgent bug report comes in!
git stash push -m "WIP: feature in progress"

# Switch to main, fix the bug
git switch main
git switch -c hotfix/urgent-bug
echo "bug fix" >> app.js
git add . && git commit -m "Fix urgent bug"
git switch main && git merge hotfix/urgent-bug && git branch -d hotfix/urgent-bug

# Go back to your feature work
git switch feature/my-task
git stash pop    # your half-done work is restored
```

**Create a branch from a stash:**
```bash
git stash branch feature/from-stash stash@{0}
# Creates a new branch, checks it out, and applies the stash
```

---

## Cherry-Pick

**Cherry-pick** applies the changes from a specific commit onto your current branch — without merging the entire branch.

```bash
git cherry-pick <commit-hash>        # apply one commit
git cherry-pick a3f2c91              # example with real hash
git cherry-pick a3f2c91 b7e8d02      # apply multiple commits
git cherry-pick a3f2c91..d1e2f3a     # apply a range of commits
```

**When to use cherry-pick:**

```
main:    A─B─C─────────────────────►
              └─D─E─F  (feature branch)
                  ↑
                  Only commit E is the fix you need on main

git switch main
git cherry-pick E
main:    A─B─C─E'─►   (E' is a copy of E applied to main)
```

Use cases:
- A critical bug fix lives on a feature branch — you need it on `main` now
- You made a useful commit on the wrong branch
- Backporting a fix to an older release branch

```bash
# Cherry-pick without committing immediately (stage only)
git cherry-pick --no-commit a3f2c91

# Cherry-pick and edit the commit message
git cherry-pick --edit a3f2c91

# If conflict arises during cherry-pick
git add <resolved-file>
git cherry-pick --continue
# or
git cherry-pick --abort
```

---

## Git Bisect

**Bisect** uses binary search to find the exact commit that introduced a bug. Instead of manually checking commits one by one, Git halves the search space each step.

```bash
# Start bisect
git bisect start

# Tell Git what you know:
git bisect bad                   # current state is broken
git bisect good v1.0.0           # this tag/commit was working

# Git checks out a commit in the middle
# Test your code — is it broken?

# Tell Git the result:
git bisect bad                   # this commit is broken too (search later half)
git bisect good                  # this commit works (search earlier half)

# Repeat until Git says:
# "a3f2c91 is the first bad commit"

# End bisect session (returns to original branch)
git bisect reset
```

**Automate bisect with a test script:**

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0

# Provide a script that exits 0 (success) or 1 (failure)
git bisect run npm test
# Git automatically runs the test on each commit and finds the culprit
```

**Why bisect is powerful:**

If you have 1000 commits to search, manual checking takes up to 1000 steps. Binary search takes at most **10 steps** (log₂ 1000 ≈ 10). Bisect is one of Git's most underused but most time-saving features.

---

## Git Blame

**Blame** shows who last modified each line of a file — and in which commit. Essential for understanding why code is written a certain way.

```bash
git blame <file>                   # annotate every line
git blame README.md                # example
git blame -L 10,25 app.js          # only lines 10–25
git blame -w app.js                # ignore whitespace changes
git blame --since=3.weeks app.js   # only blame within last 3 weeks
```

**Reading blame output:**

```
a3f2c91 (Jane Smith  2024-01-15 14:32:01 +0000  1) # My Project
b7e8d02 (John Doe    2024-01-16 09:15:44 +0000  2) 
b7e8d02 (John Doe    2024-01-16 09:15:44 +0000  3) A hands-on Git project.
c9f1a34 (Jane Smith  2024-01-17 11:00:00 +0000  4)
c9f1a34 (Jane Smith  2024-01-17 11:00:00 +0000  5) ## Author
^a3f2c9 (Jane Smith  2024-01-15 14:32:01 +0000  6) Jane Smith
│          │              │                      │   │
hash    author          date                  line  content
```

> 💡 `git blame` is not about assigning fault — it's about finding context. When you see unexpected code, blame tells you the commit hash. From there, `git show <hash>` tells you WHY it was written.

```bash
# The typical workflow:
git blame app.js           # find the commit hash for a suspicious line
git show a3f2c91           # read the full commit to understand the why
```

---

## Git Tags

**Tags** mark specific points in history — usually release versions. Unlike branch pointers, tags don't move as new commits are made.

```bash
# Lightweight tag (just a pointer, no extra info)
git tag v1.0.0

# Annotated tag (recommended — includes author, date, message)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag a specific past commit
git tag -a v0.9.0 a3f2c91 -m "Beta release"

# List all tags
git tag
git tag -l "v1.*"          # list tags matching pattern

# Show tag details
git show v1.0.0

# Push tags to remote (tags are NOT pushed by default)
git push origin v1.0.0     # push one tag
git push origin --tags     # push all tags

# Delete a tag
git tag -d v1.0.0                    # delete local tag
git push origin --delete v1.0.0      # delete remote tag

# Checkout the code at a tag
git checkout v1.0.0        # detached HEAD — read-only view
```

**Semantic versioning with tags:**

```
v1.0.0  ← Major.Minor.Patch
  │  │  └─ Bug fixes only
  │  └──── New features, backward compatible
  └──────── Breaking changes
```

---

## Git Reflog

**Reflog** is Git's safety net — it records every time `HEAD` moves, including after resets, rebases, and checkouts. Even "deleted" commits stay in the reflog for 90 days.

```bash
git reflog                    # show full reflog of HEAD movements
git reflog show main          # reflog for a specific branch
```

Example output:
```
a3f2c91 (HEAD -> main) HEAD@{0}: commit: Add footer
b7e8d02                HEAD@{1}: commit: Add navbar
c9f1a34                HEAD@{2}: rebase: Add login form
d1e2f3a                HEAD@{3}: checkout: moving from feature to main
```

**Recover a "lost" commit after a hard reset:**

```bash
# Oops — you reset too far
git reset --hard HEAD~3
# Those 3 commits seem gone...

# Find them in the reflog
git reflog
# a3f2c91 HEAD@{1}: commit: The commit I need

# Restore to that commit
git reset --hard a3f2c91
# Your commits are back!
```

**Recover a deleted branch:**

```bash
# Branch was deleted
git branch -D feature/important-work

# Find the last commit of that branch
git reflog
# b7e8d02 HEAD@{4}: checkout: moving from feature/important-work to main

# Recreate the branch
git switch -c feature/important-work b7e8d02
```

---

## Undoing Things

A complete map of how to undo different situations:

### Before staging

```bash
git restore <file>            # discard changes in working directory
git restore .                 # discard ALL unstaged changes (careful!)
```

### After staging, before committing

```bash
git restore --staged <file>   # unstage (keep changes in working dir)
git restore --staged .        # unstage everything
```

### After committing (local only)

```bash
git commit --amend            # fix last commit message or content
git revert HEAD               # create new commit that undoes last commit (safe)
git reset --soft HEAD~1       # undo commit, keep changes staged
git reset --mixed HEAD~1      # undo commit, keep changes unstaged (default)
git reset --hard HEAD~1       # undo commit AND discard all changes (destructive)
```

### Understanding reset modes

```
Commit:    HEAD~1  →  HEAD
           [A─B─C]     [D]   ← reset undoes D

--soft:   D's changes go to staging area   (ready to re-commit)
--mixed:  D's changes go to working dir    (need to re-stage)
--hard:   D's changes are DELETED          (unrecoverable without reflog)
```

### After pushing (public commits)

```bash
# NEVER reset public commits — use revert instead
git revert HEAD               # creates an "undo commit" — safe for shared repos
git revert a3f2c91            # revert a specific past commit
git revert a3f2c91..HEAD      # revert a range of commits
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git rebase <branch>` | Replay current branch commits on top of another branch |
| `git rebase -i HEAD~N` | Interactively edit the last N commits |
| `git stash` | Save uncommitted work to a temporary stack |
| `git stash pop` | Restore the most recently stashed work |
| `git cherry-pick <hash>` | Apply a specific commit to the current branch |
| `git bisect` | Binary search through history to find a bug |
| `git blame <file>` | See who last changed each line of a file |
| `git tag -a v1.0 -m "msg"` | Create an annotated release tag |
| `git reflog` | View all HEAD movements — your safety net |
| `git reset --soft HEAD~1` | Undo a commit, keep changes staged |
| `git reset --hard HEAD~1` | Undo a commit and discard all changes |
| `git revert HEAD` | Create an undo commit (safe for shared repos) |

---

## What's Next

One more day — tomorrow you reach **expert level**: hooks, submodules, advanced workflows, and contributing to open source like a pro.

➡️ **[Day 5 — Expert Git](../Day-05-Git-Expert-Level/README.md)**

---

*📝 Ready to practice? Try the [exercises](./exercises.md) — solutions are in [solutions.md](./solutions.md)*
