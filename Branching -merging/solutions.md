# Day 2 — Solutions

> **Try the exercises yourself before reading these.**
>
> ← Back to [exercises.md](./exercises.md)

---

## Solution 1 — Create & Explore Branches

```bash
cd git-practice
git log --oneline
# c9f1a34 (HEAD -> main) Add .gitignore
# ...

# List branches — only main exists
git branch
# * main

# Create a new branch
git branch feature/about-page

# List again — new branch appears, HEAD still on main
git branch
# * main
#   feature/about-page

# Switch to the new branch
git switch feature/about-page
# Switched to branch 'feature/about-page'

# Confirm current branch
git branch
#   main
# * feature/about-page

# Both branches point to the same commit
git log --oneline --graph --all
# * c9f1a34 (HEAD -> feature/about-page, main) Add .gitignore
# * ...
```

**Answer: Why does the new branch point to the same commit?**

A branch is just a pointer. When you run `git branch feature/about-page`, Git creates a new pointer file at `.git/refs/heads/feature/about-page` containing the same commit hash as `main`. No files are copied, no new commits are made — it's just a second label on the same snapshot.

**Answer: What does `*` mean in `git branch`?**

The asterisk marks your **current branch** — where `HEAD` is pointing. When you're on `feature/about-page`, that branch gets the `*`.

---

## Solution 2 — Commit on a Branch

```bash
# Confirm current branch
git branch
# * feature/about-page

# Create about.html
echo "<h1>About Us</h1>" > about.html
echo "<p>We love Git.</p>" >> about.html

git add about.html
git commit -m "Add about page HTML"

echo "<p>Founded in 2024.</p>" >> about.html
git add about.html
git commit -m "Add founding year to about page"

# Visualize divergence
git log --oneline --graph --all
# * e4f5a67 (HEAD -> feature/about-page) Add founding year to about page
# * d1e2f3a Add about page HTML
# * c9f1a34 (main) Add .gitignore
# * ...

# Switch to main — about.html disappears!
git switch main
ls
# README.md  index.html  style.css  .gitignore
# (no about.html)

# Switch back — about.html reappears!
git switch feature/about-page
ls
# README.md  index.html  style.css  .gitignore  about.html
```

**Answer: Where did `about.html` go?**

Git doesn't "hide" the file — it literally updates your working directory to match the state of the branch you're switching to. On `main`, `about.html` has never been committed, so it doesn't exist. Git removes it from your folder. Switch back and Git restores it. Your filesystem reflects whichever branch `HEAD` is on.

**Answer: What happens with uncommitted changes when switching?**

If you have uncommitted changes that would be overwritten by switching branches, Git refuses:
```
error: Your local changes to the following files would be overwritten by checkout
```
Your options are: commit the changes, stash them (`git stash`), or discard them (`git restore`).

---

## Solution 3 — Fast-Forward Merge

```bash
git switch main

# Confirm main is behind
git log --oneline --graph --all
# * e4f5a67 (feature/about-page) Add founding year to about page
# * d1e2f3a Add about page HTML
# * c9f1a34 (HEAD -> main) Add .gitignore

# Merge
git merge feature/about-page
# Updating c9f1a34..e4f5a67
# Fast-forward
#  about.html | 3 +++
#  1 file changed, 3 insertions(+)

# main has moved forward to the same commit
git log --oneline --graph --all
# * e4f5a67 (HEAD -> main, feature/about-page) Add founding year to about page
# * d1e2f3a Add about page HTML
# * c9f1a34 Add .gitignore

ls
# about.html is now here on main

# Delete the merged branch
git branch -d feature/about-page
# Deleted branch feature/about-page (was e4f5a67).

git branch
# * main
```

**Answer: Was a new merge commit created?**

No. In a fast-forward merge, Git just moves the `main` pointer forward to where `feature/about-page` was. The history is perfectly linear — no new commit needed.

**Answer: What condition makes fast-forward possible?**

Fast-forward is possible when the target branch (`main`) has not received any new commits since the source branch was created from it. In other words, `main` is a **direct ancestor** of `feature/about-page`. There are no diverging paths to reconcile.

---

## Solution 4 — Three-Way Merge

```bash
git switch -c feature/contact-page

echo "<h1>Contact Us</h1>" > contact.html
echo "<p>Email: hello@example.com</p>" >> contact.html
git add contact.html
git commit -m "Add contact page"

# Create divergence by committing on main too
git switch main
echo "footer { text-align: center; }" >> style.css
git add style.css
git commit -m "Update footer styles"

# Visualize diverged history
git log --oneline --graph --all
# * f6a7b89 (HEAD -> main) Update footer styles
# | * a5b4c3d (feature/contact-page) Add contact page
# |/
# * e4f5a67 Add founding year to about page
# ...

# Merge — Git opens editor for merge commit message
git merge feature/contact-page
# Merge made by the 'ort' strategy.
#  contact.html | 2 ++
#  1 file changed, 2 insertions(+)

git log --oneline --graph --all
# *   g8h9i0j (HEAD -> main) Merge branch 'feature/contact-page'
# |\
# | * a5b4c3d (feature/contact-page) Add contact page
# * | f6a7b89 Update footer styles
# |/
# * e4f5a67 Add founding year to about page

git branch -d feature/contact-page
```

**Answer: How many parent commits does the merge commit have?**

Two. Every merge commit has two (or more) parent commits — one from each branch being merged. You can see this with:
```bash
git show HEAD
# Merge: f6a7b89 a5b4c3d
```

**Answer: Why didn't Git fast-forward?**

`main` had a new commit (`Update footer styles`) that `feature/contact-page` didn't have. The two branches diverged from a common ancestor. Git cannot simply move a pointer — it must create a new commit that combines both histories.

---

## Solution 5 — Create a Merge Conflict

```bash
git switch -c feature/update-heading

echo '<h1>My Awesome Git Project</h1>' > index.html
git add index.html
git commit -m "Update heading on feature branch"

git switch main
echo '<h1>Git Practice Repository</h1>' > index.html
git add index.html
git commit -m "Update heading on main"

git merge feature/update-heading
# Auto-merging index.html
# CONFLICT (content): Merge conflict in index.html
# Automatic merge failed; fix conflicts and then commit the result.

git status
# On branch main
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#   (use "git merge --abort" to abort the merge)
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#         both modified:   index.html

cat index.html
# <<<<<<< HEAD
# <h1>Git Practice Repository</h1>
# =======
# <h1>My Awesome Git Project</h1>
# >>>>>>> feature/update-heading
```

**Answer: Why did Git fail to merge automatically?**

Both branches modified the **exact same line** of the same file. Git records every change as line-level diffs. When both sides changed line 1 of `index.html` to different values, there's no algorithmic way to decide which one wins — that's a human decision. Git stops and asks you.

**Answer: What does `=======` separate?**

The `=======` divides:
- Everything **above** it (between `<<<<<<< HEAD` and `=======`) — the version from your **current branch** (`HEAD`, i.e. `main`)
- Everything **below** it (between `=======` and `>>>>>>> branch-name`) — the version from the **incoming branch** (`feature/update-heading`)

---

## Solution 6 — Resolve the Conflict

```bash
# Current state of index.html:
# <<<<<<< HEAD
# <h1>Git Practice Repository</h1>
# =======
# <h1>My Awesome Git Project</h1>
# >>>>>>> feature/update-heading

# Choose a resolution — edit to whatever you want
echo '<h1>Git Practice Repository — Awesome!</h1>' > index.html

# Verify clean (no conflict markers)
cat index.html
# <h1>Git Practice Repository — Awesome!</h1>

# Stage the resolved file
git add index.html

git status
# On branch main
# All conflicts fixed but you are still merging.
#   (use "git commit" to conclude merge)

# Complete the merge
git commit -m "Merge feature/update-heading: combine heading text"
# [main k1l2m3n] Merge feature/update-heading: combine heading text

git log --oneline --graph --all
# *   k1l2m3n (HEAD -> main) Merge feature/update-heading: combine heading text
# |\
# | * h9i0j1k (feature/update-heading) Update heading on feature branch
# * | j0k1l2m Update heading on main
# |/
# * ...

git branch -d feature/update-heading
```

**Key rules when resolving conflicts:**

1. Remove ALL conflict markers — `<<<<<<<`, `=======`, `>>>>>>>`
2. The final file should contain exactly what you want the merged result to look like
3. You can keep one side, the other, both, or write something entirely new
4. `git add` signals to Git that the conflict is resolved
5. `git commit` finalizes the merge

---

## Solution 7 — Abort a Merge (Bonus)

```bash
git switch -c feature/conflict-test
echo '<h2>Section from feature</h2>' >> about.html
git add about.html
git commit -m "Add section on feature branch"

git switch main
echo '<h2>Section from main</h2>' >> about.html
git add about.html
git commit -m "Add section on main"

git merge feature/conflict-test
# CONFLICT (content): Merge conflict in about.html
# Automatic merge failed; fix conflicts and then commit the result.

git status
# You have unmerged paths.

# Not ready to resolve — abort
git merge --abort

# Clean state restored
git status
# On branch main
# nothing to commit, working tree clean

cat about.html
# Back to the main branch version — no conflict markers

git branch -D feature/conflict-test
```

**When to use `git merge --abort`:**

- You weren't expecting conflicts and need time to understand the changes
- The incoming branch needs to be updated before merging
- You need to coordinate with a teammate about the right resolution
- You merged the wrong branch by accident

`--abort` completely resets to the pre-merge state. Nothing is lost.

---

## Solution 8 — Branch Strategy Practice (Bonus)

```bash
# Start feature work
git switch -c feature/projects-page
echo "<h1>My Projects</h1>" > projects.html
git add projects.html
git commit -m "Start projects page"

# Context switch: urgent hotfix
git switch main
git switch -c hotfix/fix-typo
echo "Fixed: corrected spelling in header" >> README.md
git add README.md
git commit -m "Hotfix: correct spelling in header"

# Merge hotfix immediately
git switch main
git merge hotfix/fix-typo
# Fast-forward
git branch -d hotfix/fix-typo

# Back to feature
git switch feature/projects-page
echo "<p>See my work below.</p>" >> projects.html
git add projects.html
git commit -m "Complete projects page content"

# Merge feature
git switch main
git merge feature/projects-page
git branch -d feature/projects-page

git log --oneline --graph --all
# *   p4q5r6s (HEAD -> main) Merge branch 'feature/projects-page'
# |\
# | * n2o3p4q Complete projects page content
# | * m1n2o3p Start projects page
# * | l0m1n2o (hotfix was here) Hotfix: correct spelling in header
# |/
# * ...
```

**Why this workflow is powerful:**

The hotfix was completely isolated. It went directly from `main` → fix → back to `main` without ever touching the unfinished `feature/projects-page` work. This is the core value of branches: you can switch contexts instantly, fix things cleanly, and never mix unrelated work.

---

## 📊 Day 2 Command Cheatsheet

```bash
# Create & switch
git branch <name>            # create branch
git switch <name>            # switch to branch
git switch -c <name>         # create AND switch (one step)
git checkout -b <name>       # older equivalent of switch -c

# Info
git branch                   # list local branches (* = current)
git branch -v                # list with last commit
git branch -a                # list all (local + remote)
git log --oneline --graph --all  # visual history

# Merge
git switch main              # go to receiving branch
git merge <branch>           # merge branch into current
git merge --abort            # cancel in-progress merge

# Cleanup
git branch -d <name>         # delete merged branch
git branch -D <name>         # force delete unmerged branch
git branch --merged          # see which branches are merged

# Conflict resolution
# 1. Edit conflicted files (remove <<<, ===, >>>)
# 2. git add <resolved-file>
# 3. git commit
```

---

*➡️ Ready for Day 3? [Remote Repositories & GitHub Collaboration →](../Day-03-Remote-Repositories/README.md)*
