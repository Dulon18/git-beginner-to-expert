# Day 1 — Exercises

> Complete these exercises in order. Each one builds on the previous. **Try before looking at solutions!**
>
> Solutions: [solutions.md](./solutions.md)

---

## Exercise 1 — Setup & Verify

**Goal:** Make sure Git is installed and configured correctly.

### Tasks

1. Open your terminal and check your Git version:
   ```bash
   git --version
   ```
   You should see something like `git version 2.44.0`. If you get an error, [install Git](https://git-scm.com/downloads) first.

2. Configure your global identity:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "you@example.com"
   ```

3. Set `main` as your default branch name:
   ```bash
   git config --global init.defaultBranch main
   ```

4. Verify your config is saved:
   ```bash
   git config --list
   ```

### ✅ Success Criteria
- `git --version` outputs a version number
- `git config --list` shows your name and email

---

## Exercise 2 — Your First Repository

**Goal:** Initialize a repository and understand what `git init` creates.

### Tasks

1. Create a new folder called `git-practice` and navigate into it:
   ```bash
   mkdir git-practice
   cd git-practice
   ```

2. Initialize a Git repository:
   ```bash
   git init
   ```

3. List all files including hidden ones to see the `.git` folder:
   ```bash
   ls -la
   ```

4. Run `git status` and read the output carefully. What does it tell you?
   ```bash
   git status
   ```

5. Look inside the `.git` directory (just look, don't edit anything):
   ```bash
   ls .git/
   ```

### ✅ Success Criteria
- You can see a `.git/` folder in your project
- `git status` says `On branch main` and `No commits yet`

### 🤔 Think About It
- What would happen if you deleted the `.git/` folder?
- Why is the `.git/` folder hidden (starts with a dot)?

---

## Exercise 3 — Stage and Commit

**Goal:** Go through the full add → commit workflow.

### Tasks

1. Inside `git-practice`, create three files:
   ```bash
   echo "# Git Practice Project" > README.md
   echo "body { margin: 0; }" > style.css
   echo "<h1>Hello Git</h1>" > index.html
   ```

2. Check the status — what state are these files in?
   ```bash
   git status
   ```

3. Stage **only** `README.md`:
   ```bash
   git add README.md
   ```

4. Run `git status` again. What changed?

5. Commit `README.md` with a clear message:
   ```bash
   git commit -m "Initial commit: add README"
   ```

6. Now stage and commit `index.html` separately:
   ```bash
   git add index.html
   git commit -m "Add homepage HTML structure"
   ```

7. Stage and commit `style.css`:
   ```bash
   git add style.css
   git commit -m "Add base CSS styles"
   ```

8. View your commit history:
   ```bash
   git log --oneline
   ```

### ✅ Success Criteria
- `git log --oneline` shows exactly **3 commits**
- Each commit has a clear, descriptive message

### 🤔 Think About It
- Why did we commit each file separately instead of all at once?
- What does `HEAD -> main` mean in the log output?

---

## Exercise 4 — Track Changes with diff

**Goal:** See exactly what changed before staging and committing.

### Tasks

1. Open `README.md` and add some content:
   ```bash
   echo "" >> README.md
   echo "A hands-on Git learning project." >> README.md
   echo "" >> README.md
   echo "## Author" >> README.md
   echo "Your Name" >> README.md
   ```

2. Before staging, inspect what changed:
   ```bash
   git diff
   ```
   Read the output. Lines starting with `+` are additions, `-` are removals.

3. Stage the change:
   ```bash
   git add README.md
   ```

4. Run `git diff` again. What do you notice?

5. Now run:
   ```bash
   git diff --staged
   ```
   What's the difference between `git diff` and `git diff --staged`?

6. Commit the change:
   ```bash
   git commit -m "Update README: add description and author"
   ```

7. Run `git log --oneline`. You should now have 4 commits.

### ✅ Success Criteria
- You can explain the difference between `git diff` and `git diff --staged`
- `git log --oneline` shows 4 commits

---

## Exercise 5 — The .gitignore File

**Goal:** Tell Git which files to ignore.

### Tasks

1. Simulate creating some files you don't want to track:
   ```bash
   echo "SECRET_KEY=abc123" > .env
   echo "debug info" > debug.log
   mkdir node_modules
   echo "a module" > node_modules/some-package.js
   ```

2. Check status — notice Git sees all these unwanted files:
   ```bash
   git status
   ```

3. Create a `.gitignore` file:
   ```bash
   touch .gitignore
   ```

4. Open `.gitignore` in your editor and add these rules:
   ```
   # Environment variables (never commit secrets)
   .env

   # Log files
   *.log

   # Dependencies folder
   node_modules/
   ```

5. Check status again:
   ```bash
   git status
   ```
   The ignored files should no longer appear.

6. Stage and commit `.gitignore`:
   ```bash
   git add .gitignore
   git commit -m "Add .gitignore: ignore .env, logs, node_modules"
   ```

7. Verify the ignored files are truly ignored:
   ```bash
   git status
   # Should show: nothing to commit, working tree clean
   ```

### ✅ Success Criteria
- `.env`, `debug.log`, and `node_modules/` do NOT appear in `git status`
- `.gitignore` is committed to the repo

### 🤔 Think About It
- Why is it dangerous to commit a `.env` file to a public GitHub repo?
- What happens if you already committed a file and then add it to `.gitignore`?

---

## Exercise 6 — Explore History

**Goal:** Navigate and inspect your commit history like a pro.

### Tasks

1. View the compact history:
   ```bash
   git log --oneline
   ```
   You should have 5 commits total.

2. View the full log with details:
   ```bash
   git log
   ```
   Notice the full SHA hash, author, date, and message.

3. Copy the hash of your **first commit** (the shortest version works). Show its details:
   ```bash
   git show <first-commit-hash>
   ```

4. Show only the last 3 commits:
   ```bash
   git log -3 --oneline
   ```

5. Show which commits touched `README.md`:
   ```bash
   git log --oneline -- README.md
   ```

6. Show a visual log (more useful after Day 2 with branches):
   ```bash
   git log --oneline --graph --all
   ```

### ✅ Success Criteria
- You can read and interpret `git log` output
- You can use `git show` to inspect any individual commit

---

## Exercise 7 — Unstage & Amend (Bonus)

**Goal:** Recover from common mistakes.

### Tasks

**Part A: Unstage a file**

1. Make a change to `style.css`:
   ```bash
   echo "h1 { color: red; }" >> style.css
   ```

2. Accidentally stage everything:
   ```bash
   git add .
   ```

3. Realize you didn't want to stage `style.css` yet. Unstage it:
   ```bash
   git restore --staged style.css
   ```

4. Confirm only what you want is staged:
   ```bash
   git status
   ```

**Part B: Fix the last commit message**

1. Make a deliberate typo in a commit message:
   ```bash
   echo "# new section" >> README.md
   git add README.md
   git commit -m "addd new section to README"
   ```

2. Fix the typo using amend:
   ```bash
   git commit --amend -m "Add new section to README"
   ```

3. Check the log — the commit message should be corrected:
   ```bash
   git log --oneline
   ```

### ✅ Success Criteria
- You can unstage files without losing your changes
- You can fix a commit message with `--amend`

---

## 🏁 Day 1 Complete!

If you've finished all exercises, you now know how to:

- [x] Install and configure Git
- [x] Initialize a repository
- [x] Stage and commit changes
- [x] Inspect diffs before committing
- [x] Ignore files with `.gitignore`
- [x] Navigate commit history
- [x] Fix common mistakes

➡️ **[Check your work in solutions.md](./solutions.md)**

➡️ **Ready for Day 2? [Branching & Merging →](../Day-02-Branching-and-Merging/README.md)**
