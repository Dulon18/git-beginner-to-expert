# Day 3 Exercises — Remote Repositories & GitHub Collaboration

## Objective

By the end of these exercises, you should be able to:

* Create and manage GitHub repositories
* Connect local repositories to GitHub
* Use clone, push, pull, and fetch
* Work with remote branches
* Understand Fork workflows
* Create and manage Pull Requests
* Resolve remote conflicts

---

# Exercise 1 — GitHub Setup

## Task 1.1

Create a GitHub account (if you don't already have one).

## Task 1.2

Generate an SSH key and connect it to GitHub.

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
cat ~/.ssh/id_ed25519.pub
```

Add the public key to GitHub.

## Verification

```bash
ssh -T git@github.com
```

Expected:

```text
Hi username! You've successfully authenticated.
```

---

# Exercise 2 — Create Your First Remote Repository

## Task 2.1

Create a new GitHub repository:

```text
git-practice-day3
```

Settings:

* Public Repository
* No README
* No .gitignore
* No License

---

## Task 2.2

Create a local repository.

```bash
mkdir git-practice-day3
cd git-practice-day3

git init
```

---

## Task 2.3

Create a README file and commit it.

```bash
echo "# Git Practice Day 3" > README.md

git add .
git commit -m "Initial commit"
```

---

# Exercise 3 — Connect Local Repository to GitHub

## Task 3.1

Add GitHub as a remote.

```bash
git remote add origin git@github.com:YOUR_USERNAME/git-practice-day3.git
```

---

## Task 3.2

Verify the remote.

```bash
git remote -v
```

Expected output:

```text
origin (fetch)
origin (push)
```

---

# Exercise 4 — First Push

## Task 4.1

Push your local repository to GitHub.

```bash
git push -u origin main
```

---

## Verification

Refresh GitHub.

Expected:

```text
README.md appears in the repository.
```

---

# Exercise 5 — Clone a Repository

## Task 5.1

Clone the repository into another folder.

```bash
cd ..

git clone git@github.com:YOUR_USERNAME/git-practice-day3.git cloned-repo
```

---

## Verification

```bash
cd cloned-repo

git log --oneline
```

You should see the initial commit.

---

# Exercise 6 — Push Additional Changes

## Task 6.1

Update the README.

```bash
echo "Learning Remote Repositories" >> README.md
```

Commit the change.

```bash
git add .
git commit -m "Update README"
```

Push it.

```bash
git push
```

---

## Verification

Refresh GitHub and verify the updated content.

---

# Exercise 7 — Fetch vs Pull

## Task 7.1

Edit README directly from GitHub.

Add:

```text
Edited from GitHub
```

Commit the change.

---

## Task 7.2

In your local repository:

```bash
git fetch
```

Check what changed:

```bash
git log main..origin/main --oneline
```

---

## Task 7.3

Merge the remote changes.

```bash
git pull
```

Verify:

```bash
cat README.md
```

---

# Exercise 8 — Working with Remote Branches

## Task 8.1

Create a feature branch.

```bash
git switch -c feature/login
```

---

## Task 8.2

Create a new file.

```bash
touch login.txt
```

Commit it.

```bash
git add .
git commit -m "Add login feature"
```

---

## Task 8.3

Push the branch.

```bash
git push -u origin feature/login
```

---

## Verification

```bash
git branch -vv
```

Confirm the tracking branch is configured.

---

# Exercise 9 — Pull Request Workflow

## Task 9.1

Create a Pull Request on GitHub.

Title:

```text
Add login feature
```

Description:

```text
Added login feature file for Git practice.
```

---

## Task 9.2

Merge the Pull Request.

---

## Task 9.3

Sync your local repository.

```bash
git switch main

git pull
```

---

## Task 9.4

Delete the feature branch.

```bash
git branch -d feature/login

git push origin --delete feature/login
```

---

# Exercise 10 — Fork Workflow

## Task 10.1

Fork a public repository.

Example:

```text
octocat/Spoon-Knife
```

---

## Task 10.2

Clone your fork.

```bash
git clone <your-fork-url>
```

---

## Task 10.3

Add the original repository as upstream.

```bash
git remote add upstream <original-repo-url>
```

---

## Verification

```bash
git remote -v
```

Expected:

```text
origin
upstream
```

---

# Exercise 11 — Sync a Fork

```bash
git fetch upstream

git switch main

git merge upstream/main

git push origin main
```

---

# Exercise 12 — Simulate a Remote Conflict

## Machine A

```bash
echo "Version A" > conflict.txt

git add .
git commit -m "Version A"

git push
```

---

## Machine B

```bash
echo "Version B" > conflict.txt

git add .
git commit -m "Version B"

git push
```

Expected:

```text
Push Rejected
```

---

## Resolve the Conflict

```bash
git pull
```

Fix the conflict manually.

Then:

```bash
git add .

git commit

git push
```

---

# Final Practical Challenge

Complete the following workflow from start to finish without referring to notes:

1. Create a GitHub Repository
2. Clone the Repository
3. Create a Feature Branch
4. Make Changes
5. Commit Changes
6. Push the Branch
7. Open a Pull Request
8. Merge the Pull Request
9. Pull Latest Changes
10. Delete the Feature Branch

---

# Self-Assessment Questions

Answer these without looking at notes:

1. What does `git clone` do?
2. What does `git push` do?
3. What is the difference between `git fetch` and `git pull`?
4. What is `origin`?
5. What is `upstream`?
6. What is a tracking branch?
7. Why do teams use Pull Requests?
8. When should you use a Fork workflow?
9. Why can a push be rejected?
10. How do you resolve a remote conflict?

---

# Completion Criteria

You can mark Day 3 complete when you can:

* Create and connect repositories
* Push and pull changes confidently
* Work with remote branches
* Create Pull Requests
* Resolve basic conflicts
* Understand Fork workflows

```text
Day 3 Completed ✅
Remote Repositories & GitHub Collaboration
```
