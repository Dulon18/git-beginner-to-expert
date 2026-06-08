# Day 3 Solutions — Remote Repositories & GitHub Collaboration

## Exercise 1 — GitHub Setup

### Solution

Generate an SSH key:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Add the key to GitHub:

```text
GitHub
→ Settings
→ SSH and GPG Keys
→ New SSH Key
→ Paste Public Key
```

Test the connection:

```bash
ssh -T git@github.com
```

Expected:

```text
Hi username! You've successfully authenticated.
```

---

## Exercise 2 — Create First Remote Repository

### Solution

Create local repository:

```bash
mkdir git-practice-day3
cd git-practice-day3

git init
```

Create README:

```bash
echo "# Git Practice Day 3" > README.md
```

Create first commit:

```bash
git add .
git commit -m "Initial commit"
```

Verify:

```bash
git log --oneline
```

Expected:

```text
<commit-id> Initial commit
```

---

## Exercise 3 — Connect Local Repository to GitHub

### Solution

Add remote:

```bash
git remote add origin git@github.com:YOUR_USERNAME/git-practice-day3.git
```

Verify:

```bash
git remote -v
```

Expected:

```text
origin git@github.com:YOUR_USERNAME/git-practice-day3.git (fetch)
origin git@github.com:YOUR_USERNAME/git-practice-day3.git (push)
```

---

## Exercise 4 — First Push

### Solution

Rename branch if needed:

```bash
git branch -M main
```

Push repository:

```bash
git push -u origin main
```

Explanation:

```text
-u creates an upstream tracking relationship.

Future pushes can be done using:

git push
```

Verify on GitHub:

```text
README.md visible in repository
```

---

## Exercise 5 — Clone Repository

### Solution

Clone:

```bash
cd ..

git clone git@github.com:YOUR_USERNAME/git-practice-day3.git cloned-repo
```

Move into cloned repository:

```bash
cd cloned-repo
```

Verify:

```bash
git log --oneline
```

Expected:

```text
Initial commit appears
```

---

## Exercise 6 — Push Additional Changes

### Solution

Update README:

```bash
echo "Learning Remote Repositories" >> README.md
```

Commit:

```bash
git add .
git commit -m "Update README"
```

Push:

```bash
git push
```

Verify:

```bash
git log --oneline
```

Expected:

```text
Update README
Initial commit
```

---

## Exercise 7 — Fetch vs Pull

### Solution

After editing README directly on GitHub:

Fetch:

```bash
git fetch
```

View remote commits:

```bash
git log main..origin/main --oneline
```

Example:

```text
a1b2c3d Edited from GitHub
```

Merge changes:

```bash
git pull
```

Verify:

```bash
cat README.md
```

Expected:

```text
# Git Practice Day 3
Learning Remote Repositories
Edited from GitHub
```

### Key Difference

```text
git fetch
    Downloads changes only

git pull
    Downloads + merges changes
```

---

## Exercise 8 — Working with Remote Branches

### Solution

Create branch:

```bash
git switch -c feature/login
```

Create file:

```bash
touch login.txt
```

Commit:

```bash
git add .
git commit -m "Add login feature"
```

Push:

```bash
git push -u origin feature/login
```

Verify:

```bash
git branch -vv
```

Example:

```text
* feature/login abc123 [origin/feature/login]
main          xyz789 [origin/main]
```

Explanation:

```text
feature/login is now tracking origin/feature/login
```

---

## Exercise 9 — Pull Request Workflow

### Solution

Create PR on GitHub.

Title:

```text
Add login feature
```

Description:

```text
Added login feature file for Git practice.
```

Merge the PR.

Sync local repository:

```bash
git switch main
git pull
```

Delete local branch:

```bash
git branch -d feature/login
```

Delete remote branch:

```bash
git push origin --delete feature/login
```

Verify:

```bash
git branch
```

Expected:

```text
main
```

---

## Exercise 10 — Fork Workflow

### Solution

Fork:

```text
octocat/Spoon-Knife
```

Clone your fork:

```bash
git clone git@github.com:YOUR_USERNAME/Spoon-Knife.git
```

Add upstream:

```bash
git remote add upstream git@github.com:octocat/Spoon-Knife.git
```

Verify:

```bash
git remote -v
```

Expected:

```text
origin
upstream
```

Explanation:

```text
origin   = your fork
upstream = original repository
```

---

## Exercise 11 — Sync a Fork

### Solution

Fetch latest changes:

```bash
git fetch upstream
```

Merge upstream changes:

```bash
git switch main

git merge upstream/main
```

Update your fork:

```bash
git push origin main
```

Verify:

```bash
git status
```

Expected:

```text
Your branch is up to date.
```

---

## Exercise 12 — Simulate a Remote Conflict

### Situation

Machine A:

```bash
echo "Version A" > conflict.txt

git add .
git commit -m "Version A"

git push
```

Machine B:

```bash
echo "Version B" > conflict.txt

git add .
git commit -m "Version B"

git push
```

Expected:

```text
Push rejected
```

---

### Resolution

Fetch latest changes:

```bash
git pull
```

Conflict appears:

```text
<<<<<<< HEAD
Version B
=======
Version A
>>>>>>> commit-id
```

Choose final version:

```text
Version A
Version B
```

or

```text
Version B
```

Remove conflict markers.

Save file.

Commit:

```bash
git add conflict.txt

git commit -m "Resolve conflict"
```

Push:

```bash
git push
```

---

# Final Challenge Solution Flow

```bash
# Clone repository
git clone <repo>

# Create branch
git switch -c feature/task

# Make changes
echo "Hello Git" > demo.txt

# Commit
git add .
git commit -m "Add demo file"

# Push
git push -u origin feature/task

# Open PR on GitHub

# Merge PR

# Update local main
git switch main
git pull

# Delete branch
git branch -d feature/task

git push origin --delete feature/task
```

---

# Self-Assessment Answers

### What does git clone do?

Creates a complete local copy of a remote repository.

### What does git push do?

Uploads local commits to a remote repository.

### Difference between git fetch and git pull?

```text
fetch = download only

pull = download + merge
```

### What is origin?

Default name of the repository you cloned from or push to.

### What is upstream?

Original repository used in a fork workflow.

### What is a tracking branch?

A local branch linked to a remote branch.

### Why use Pull Requests?

For code review, collaboration, discussion, and controlled merging.

### When should Fork Workflow be used?

When contributors do not have direct write access to the original repository.

### Why can push be rejected?

Because the remote contains commits that do not exist locally.

### How do you resolve a remote conflict?

```text
1. Pull latest changes
2. Resolve conflicts manually
3. git add
4. git commit
5. git push
```

---

# Completion Checklist

```text
☑ Connected local repository to GitHub
☑ Pushed commits
☑ Pulled remote changes
☑ Used fetch
☑ Created remote branches
☑ Created Pull Requests
☑ Understood fork workflow
☑ Resolved merge conflicts

Day 3 Completed ✅
```
