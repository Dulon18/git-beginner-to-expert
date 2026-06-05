# Day 2 — Branching & Merging

> **Goal:** Create and switch branches confidently, merge work back together, and resolve conflicts like a pro.

---

## 📋 Table of Contents

- [What is a Branch?](#what-is-a-branch)
- [Why Branches Matter](#why-branches-matter)
- [How Branches Work Internally](#how-branches-work-internally)
- [Working with Branches](#working-with-branches)
- [The HEAD Pointer](#the-head-pointer)
- [Merging](#merging)
  - [Fast-Forward Merge](#fast-forward-merge)
  - [Three-Way Merge](#three-way-merge)
  - [Merge Conflicts](#merge-conflicts)
- [Resolving Conflicts](#resolving-conflicts)
- [Branch Management](#branch-management)
- [Branching Strategies](#branching-strategies)
- [Summary](#summary)
- [What's Next](#whats-next)

---

## What is a Branch?

A **branch** is a lightweight, movable pointer to a specific commit. When you create a branch, Git doesn't copy your files — it just creates a new pointer. This makes branches nearly instant to create and almost free in terms of storage.

Think of your commit history as a timeline:

```
a3f2c91 ── b7e8d02 ── c9f1a34   ← main branch
```

When you create a new branch, you get a second pointer to the same timeline:

```
a3f2c91 ── b7e8d02 ── c9f1a34   ← main
                           ↑
                        feature   ← new branch (same commit for now)
```

As you commit on the new branch, it moves forward independently:

```
a3f2c91 ── b7e8d02 ── c9f1a34              ← main
                           └── d1e2f3 ── e4f5a6   ← feature
```

`main` stays exactly where it was. You can safely experiment on `feature` without touching `main`.

---

## Why Branches Matter

Branches let you:

| Use Case | Without Branches | With Branches |
|----------|-----------------|---------------|
| Add a new feature | Break working code | Work in isolation |
| Fix a bug | Can't deploy until feature is done | Fix and deploy immediately |
| Experiment | Risk losing stable code | Throw away the branch if it fails |
| Collaborate | Overwrite each other | Each person works on their own branch |
| Code review | Risky direct commits | PR from branch → review → merge |

---

## How Branches Work Internally

Branches are just files inside `.git/refs/heads/`. Each file contains the SHA hash of the commit it points to.

```bash
cat .git/refs/heads/main
# a3f2c91d8e7f...  (just a commit hash)
```

Creating a branch creates a new file. Deleting a branch deletes that file. No data is copied — just a 40-character hash.

```
.git/refs/heads/
├── main          ← contains commit hash
├── feature/login ← contains commit hash
└── bugfix/navbar ← contains commit hash
```

---

## Working with Branches

### Create a branch

```bash
git branch feature-login        # create branch at current commit
git branch                      # list all local branches
git branch -v                   # list with last commit info
```

### Switch to a branch

```bash
git switch feature-login        # modern syntax (Git 2.23+)
git checkout feature-login      # older syntax (still works)
```

### Create AND switch in one step

```bash
git switch -c feature-login     # recommended modern syntax
git checkout -b feature-login   # older syntax
```

### Rename a branch

```bash
git branch -m old-name new-name         # rename any branch
git branch -m new-name                  # rename the current branch
```

### Delete a branch

```bash
git branch -d feature-login     # safe delete (fails if unmerged)
git branch -D feature-login     # force delete (even if unmerged)
```

### See all branches

```bash
git branch                      # local branches
git branch -r                   # remote branches
git branch -a                   # all (local + remote)
```

---

## The HEAD Pointer

`HEAD` is a special pointer that always points to your **current position** — usually the tip of the current branch.

```
a3f2c91 ── b7e8d02 ── c9f1a34   ← main ← HEAD
```

When you switch branches, HEAD moves:

```bash
git switch feature-login
```

```
a3f2c91 ── c9f1a34   ← main
                └── d1e2f3   ← feature-login ← HEAD
```

When you commit, the current branch pointer AND HEAD both move forward:

```bash
git commit -m "Add login form"
```

```
a3f2c91 ── c9f1a34                     ← main
                └── d1e2f3 ── e4f5a6   ← feature-login ← HEAD
```

`git log --oneline --graph --all` is your best tool to visualize this.

---

## Merging

Merging brings changes from one branch into another. You always merge **into** the branch you're currently on.

```bash
git switch main           # go to the branch receiving changes
git merge feature-login   # bring in changes from feature-login
```

There are two main types of merges:

---

### Fast-Forward Merge

Happens when the target branch (e.g. `main`) has not diverged — it's simply behind the source branch. Git just moves the pointer forward.

**Before:**
```
a3f2c91 ── b7e8d02           ← main ← HEAD
                └── c9f1a34 ── d1e2f3   ← feature
```

**After `git merge feature`:**
```
a3f2c91 ── b7e8d02 ── c9f1a34 ── d1e2f3   ← main ← HEAD
                                            ← feature
```

No new commit is created. The branch pointer just "fast-forwards" to catch up.

```bash
git merge feature-login
# Updating b7e8d02..d1e2f3
# Fast-forward
#  login.html | 20 ++++++++++++++++++
#  1 file changed, 20 insertions(+)
```

---

### Three-Way Merge

Happens when both branches have new commits since they diverged. Git needs to combine the work and creates a new **merge commit**.

**Before:**
```
       ── b7e8d02 ── c9f1a34   ← main ← HEAD
      /
a3f2c91
      \
       ── d1e2f3 ── e4f5a6   ← feature
```

**After `git merge feature`:**
```
       ── b7e8d02 ── c9f1a34 ──┐
      /                         ├── f6a7b8 (merge commit)   ← main ← HEAD
a3f2c91                         │
      \                         │
       ── d1e2f3 ── e4f5a6 ──┘
```

Git looks at three commits:
1. The **common ancestor** (where the branches diverged)
2. The **tip of main**
3. The **tip of feature**

It combines changes from both sides.

```bash
git merge feature-login
# Merge made by the 'ort' strategy.
#  login.html | 20 ++++++++++++++++++
#  1 file changed, 20 insertions(+)
```

---

### Merge Conflicts

A conflict occurs when **both branches changed the same part of the same file**. Git can't decide which version to keep — it asks you to resolve it manually.

```bash
git merge feature-login
# Auto-merging index.html
# CONFLICT (content): Merge conflict in index.html
# Automatic merge failed; fix conflicts and then commit the result.
```

Git marks the conflict inside the file:

```html
<<<<<<< HEAD
<h1>Welcome to My Site</h1>
=======
<h1>Welcome! Login Below</h1>
>>>>>>> feature-login
```

| Marker | Meaning |
|--------|---------|
| `<<<<<<< HEAD` | Start of your current branch's version |
| `=======` | Divider between the two versions |
| `>>>>>>> feature-login` | End of the incoming branch's version |

---

## Resolving Conflicts

**Step-by-step conflict resolution:**

**1. See which files have conflicts:**
```bash
git status
# Both modified: index.html
```

**2. Open the conflicted file and find the markers:**
```html
<<<<<<< HEAD
<h1>Welcome to My Site</h1>
=======
<h1>Welcome! Login Below</h1>
>>>>>>> feature-login
```

**3. Edit the file to your desired final result** (remove ALL markers):
```html
<h1>Welcome! Login to My Site</h1>
```

**4. Stage the resolved file:**
```bash
git add index.html
```

**5. Complete the merge with a commit:**
```bash
git commit -m "Merge feature-login: resolve heading conflict"
```

**Abort a merge if you want to start over:**
```bash
git merge --abort
```

### Tips for avoiding conflicts

- Pull from `main` frequently into your feature branch
- Keep branches short-lived — merge often
- Communicate with teammates about which files you're changing
- Split large files into smaller, focused modules

---

## Branch Management

### Keeping a feature branch up to date with main

```bash
git switch feature-login
git merge main           # bring main's latest changes into your branch
```

This reduces conflicts when you eventually merge back.

### After merging, clean up

```bash
git switch main
git merge feature-login
git branch -d feature-login    # delete the merged branch
```

The commits are still in history — deleting a branch just removes the pointer.

### View merged and unmerged branches

```bash
git branch --merged       # branches already merged into current branch
git branch --no-merged    # branches with unmerged work
```

---

## Branching Strategies

### Feature Branch Workflow (most common)

```
main       ──────────────────────────────────────────► (always deployable)
               ↑                          ↑
feature/login  ──── commit ── commit ────┘
                                    ↑
feature/signup                      └── commit ── commit ──► (still in progress)
```

- `main` is always production-ready
- Each feature lives on its own branch
- Merge to `main` only when done and reviewed

### Branch naming conventions

```bash
feature/user-authentication     # new features
bugfix/login-error              # bug fixes
hotfix/critical-security-patch  # urgent production fixes
release/v1.2.0                  # release preparation
chore/update-dependencies       # maintenance
docs/api-reference              # documentation
```

---

## Summary

| Concept | What to remember |
|---------|-----------------|
| **Branch** | A lightweight pointer to a commit — nearly free to create |
| **HEAD** | Points to your current position in history |
| **Fast-forward** | Simple pointer move — no new commit needed |
| **Three-way merge** | Creates a merge commit combining diverged histories |
| **Conflict** | Two branches changed the same lines — you decide the result |
| **`git switch -c`** | Create and switch to a new branch in one command |
| **`git merge`** | Integrate another branch into the current one |

### The Branch Workflow Loop

```
git switch -c feature/x  →  make commits  →  git switch main  →  git merge feature/x  →  git branch -d feature/x
```

---

## What's Next

You know how to work locally. Tomorrow you'll connect to the world — pushing, pulling, and collaborating on **GitHub**.

➡️ **[Day 3 — Remote Repositories & GitHub Collaboration](../Day-03-Remote-Repositories/README.md)**

---

*📝 Ready to practice? Try the [exercises](./exercises.md) — solutions are in [solutions.md](./solutions.md)*
