# Day 4 — Exercises

> These exercises build on your `git-practice` repo from Days 1–2. Make sure you have several commits to work with.
>
> Solutions: [solutions.md](./solutions.md)

---

## Exercise 1 — Interactive Rebase: Squash Messy Commits

**Goal:** Clean up a noisy commit history by squashing related commits into one.

### Setup

1. Make sure you're in `git-practice` and on `main`:
   ```bash
   cd git-practice
   git switch main
   git log --oneline
   ```

2. Create a branch and make several small, messy commits:
   ```bash
   git switch -c feature/cleanup-test

   echo "<nav>Home | About</nav>" >> index.html
   git add index.html && git commit -m "Add nav"

   echo "/* nav styles */" >> style.css
   git add style.css && git commit -m "wip"

   echo "nav { display: flex; }" >> style.css
   git add style.css && git commit -m "fix nav"

   echo "nav a { color: black; }" >> style.css
   git add style.css && git commit -m "fix nav again"

   echo "nav a:hover { color: blue; }" >> style.css
   git add style.css && git commit -m "ok done now"
   ```

3. View the messy history:
   ```bash
   git log --oneline
   ```

### Tasks

4. Start an interactive rebase on the last 5 commits:
   ```bash
   git rebase -i HEAD~5
   ```

5. In the editor, change the plan so that commits 2–5 are squashed into commit 1:
   ```
   pick   <hash> Add nav
   squash <hash> wip
   squash <hash> fix nav
   squash <hash> fix nav again
   squash <hash> ok done now
   ```
   Save and close. A second editor opens for the combined commit message — write:
   `Add navigation bar with styles`

6. View the cleaned-up history:
   ```bash
   git log --oneline
   ```
   You should now have **one** clean commit instead of five.

7. Merge back to main and clean up:
   ```bash
   git switch main
   git merge feature/cleanup-test
   git branch -d feature/cleanup-test
   ```

### ✅ Success Criteria
- Five noisy commits become one clean commit
- The final commit message is clear and descriptive

### 🤔 Think About It
- Why is squashing useful before merging a PR?
- What is the difference between `squash` and `fixup` in interactive rebase?

---

## Exercise 2 — Interactive Rebase: Reword & Drop

**Goal:** Edit commit messages and remove an unwanted commit.

### Setup

1. Create a branch with some commits including a bad one:
   ```bash
   git switch -c feature/reword-test

   echo "# Projects" > projects.html
   git add projects.html && git commit -m "projcts page"

   echo "<p>temp debug line</p>" >> projects.html
   git add projects.html && git commit -m "debug: remove before merge"

   echo "<p>See my work below.</p>" >> projects.html
   git add projects.html && git commit -m "add projects description"
   ```

### Tasks

2. Open interactive rebase for the last 3 commits:
   ```bash
   git rebase -i HEAD~3
   ```

3. Set the plan:
   - `reword` the first commit (fix the typo "projcts" → "projects")
   - `drop` the second commit (remove the debug line entirely)
   - `pick` the third commit (keep as-is)

   Save and close. An editor opens for the reworded message — fix it to:
   `Add projects page`

4. Check the result:
   ```bash
   git log --oneline
   cat projects.html
   ```
   The debug line should be gone. You should have 2 clean commits.

5. Merge back and clean up:
   ```bash
   git switch main
   git merge feature/reword-test
   git branch -d feature/reword-test
   ```

### ✅ Success Criteria
- The typo in the commit message is corrected
- The debug commit is completely removed from history
- `projects.html` does NOT contain the debug line

---

## Exercise 3 — Git Stash: Context Switching

**Goal:** Use stash to save work-in-progress and switch tasks cleanly.

### Tasks

1. Start working on a new feature (don't commit):
   ```bash
   git switch -c feature/new-header
   echo "<header><h1>New Header Design</h1></header>" >> index.html
   echo "header { background: navy; color: white; }" >> style.css
   ```

2. Check status — changes are unstaged:
   ```bash
   git status
   ```

3. Urgent bug! You need to switch to `main` but can't commit half-done work:
   ```bash
   git stash push -m "WIP: new header design"
   ```

4. Confirm the stash saved and working directory is clean:
   ```bash
   git status
   git stash list
   ```

5. Fix the "bug" on main:
   ```bash
   git switch main
   echo "<!-- bug fix: corrected meta tag -->" >> index.html
   git add index.html
   git commit -m "Fix: correct meta tag in index"
   ```

6. Go back to your feature and restore your stashed work:
   ```bash
   git switch feature/new-header
   git stash pop
   ```

7. Verify your changes are back:
   ```bash
   git status
   cat index.html
   ```

8. Now commit the feature work properly:
   ```bash
   git add .
   git commit -m "Add new header design"
   git switch main
   git merge feature/new-header
   git branch -d feature/new-header
   ```

### ✅ Success Criteria
- You switched context without committing half-done work
- Your WIP was fully restored by `git stash pop`
- Both the bugfix and feature are in `main`

---

## Exercise 4 — Git Stash: Multiple Stashes

**Goal:** Manage multiple stashes and apply them selectively.

### Tasks

1. Create two different stashes:
   ```bash
   # First stash
   echo "/* experiment A */" >> style.css
   git stash push -m "Experiment: dark mode styles"

   # Second stash
   echo "<!-- draft section -->" >> index.html
   git stash push -m "Draft: new section markup"
   ```

2. List both stashes:
   ```bash
   git stash list
   ```

3. Inspect the first stash without applying it:
   ```bash
   git stash show stash@{1} -p
   ```

4. Apply ONLY the dark mode stash (it's `stash@{1}` — the older one):
   ```bash
   git stash apply stash@{1}
   ```

5. Check that the change is applied but the stash still exists:
   ```bash
   git stash list
   cat style.css
   ```

6. Commit the applied stash:
   ```bash
   git add style.css
   git commit -m "Add dark mode styles experiment"
   ```

7. Drop the stash you applied, and also pop the remaining one:
   ```bash
   git stash drop stash@{1}
   git stash pop
   git stash list   # should be empty
   ```

### ✅ Success Criteria
- You managed two independent stashes
- You applied a specific stash by index without removing the other

---

## Exercise 5 — Cherry-Pick: Apply a Specific Commit

**Goal:** Move a useful commit from one branch to another without merging the whole branch.

### Setup

1. Create a branch with multiple commits, one of which is a bug fix you need on `main`:
   ```bash
   git switch -c feature/mixed-work

   echo "/* new animation */" >> style.css
   git add style.css && git commit -m "Add animation styles (WIP)"

   echo "<!-- fixed broken link -->" >> index.html
   git add index.html && git commit -m "Fix: correct broken navigation link"

   echo "/* more animation */" >> style.css
   git add style.css && git commit -m "Continue animation work (WIP)"
   ```

2. Find the hash of the bug fix commit (the middle one):
   ```bash
   git log --oneline
   ```
   Copy the hash of `"Fix: correct broken navigation link"`.

### Tasks

3. Switch to `main` and cherry-pick only the bug fix:
   ```bash
   git switch main
   git cherry-pick <hash-of-fix-commit>
   ```

4. Verify the fix is now on `main` but the WIP commits are not:
   ```bash
   git log --oneline
   cat index.html   # should have the fix
   cat style.css    # should NOT have animation styles
   ```

5. The feature branch continues independently — it still has all 3 commits:
   ```bash
   git log --oneline feature/mixed-work
   ```

### ✅ Success Criteria
- The bug fix commit appears on `main`
- The WIP animation commits do NOT appear on `main`
- The feature branch still has all 3 original commits

### 🤔 Think About It
- What is the hash of the cherry-picked commit on `main` compared to the original? Are they the same?
- When would cherry-pick be better than merging the whole branch?

---

## Exercise 6 — Git Bisect: Find the Bug

**Goal:** Use binary search to find which commit introduced a bug.

### Setup

Run this script to create a repo with a "bug" introduced somewhere in 7 commits:

```bash
mkdir bisect-practice && cd bisect-practice
git init && git switch -c main

# Commit 1: works fine
echo 'result = 2 + 2' > calc.py
echo 'print(f"2 + 2 = {result}")' >> calc.py
git add . && git commit -m "Initial calculator"

# Commit 2: works fine
echo '# basic calculator app' >> calc.py
git add . && git commit -m "Add comment"

# Commit 3: BUG INTRODUCED HERE
echo 'result = 2 + 3' > calc.py
echo 'print(f"2 + 2 = {result}")' >> calc.py
git add . && git commit -m "Refactor calculation"

# Commit 4: works on other things
echo '# version 1.0' >> calc.py
git add . && git commit -m "Add version comment"

# Commit 5: more features
echo 'print("Calculator ready")' >> calc.py
git add . && git commit -m "Add ready message"
```

### Tasks

1. See the history:
   ```bash
   git log --oneline
   ```

2. Start bisect. Mark the current state as bad, and the first commit as good:
   ```bash
   git bisect start
   git bisect bad HEAD
   git bisect good HEAD~4    # or the hash of commit 1
   ```

3. Git checks out a middle commit. Test if the bug exists:
   ```bash
   cat calc.py
   # Is result = 2 + 2 or result = 2 + 3?
   # If result = 2 + 3, it's BAD. If result = 2 + 2, it's GOOD.
   git bisect bad   # or git bisect good
   ```

4. Repeat until Git identifies the first bad commit.

5. End bisect:
   ```bash
   git bisect reset
   ```

6. Return to your `git-practice` folder:
   ```bash
   cd ../git-practice
   ```

### ✅ Success Criteria
- Git correctly identifies commit 3 ("Refactor calculation") as the first bad commit
- You understand the binary search pattern

---

## Exercise 7 — Git Blame & Show

**Goal:** Trace the origin of code using blame and show.

### Tasks

1. View the blame output for `index.html`:
   ```bash
   git blame index.html
   ```

2. Find the commit hash next to a specific line you added (e.g. the nav line from Exercise 1).

3. Use `git show` to see the full context of that commit:
   ```bash
   git show <hash>
   ```

4. See blame for only a range of lines:
   ```bash
   git blame -L 1,5 index.html
   ```

5. See which commits last touched `style.css`:
   ```bash
   git blame style.css
   ```

6. Find the commit that added the most lines to `style.css`:
   ```bash
   git log --oneline --stat -- style.css
   ```

### ✅ Success Criteria
- You can trace any line of code back to its commit
- You can use `git show` to read the full context of any commit

---

## Exercise 8 — Git Tags: Mark a Release

**Goal:** Create annotated tags to mark version milestones.

### Tasks

1. View your current commit history:
   ```bash
   git log --oneline
   ```

2. Tag the very first commit as `v0.1.0`:
   ```bash
   # Get the hash of your first commit
   git log --oneline | tail -1
   git tag -a v0.1.0 <first-commit-hash> -m "Initial release: basic project structure"
   ```

3. Tag the current commit (HEAD) as `v1.0.0`:
   ```bash
   git tag -a v1.0.0 -m "Version 1.0.0: complete day 4 exercises"
   ```

4. List all tags:
   ```bash
   git tag
   ```

5. Show details of the `v1.0.0` tag:
   ```bash
   git show v1.0.0
   ```

6. See tags in the log:
   ```bash
   git log --oneline --decorate
   ```

7. Checkout the code as it was at `v0.1.0`:
   ```bash
   git checkout v0.1.0
   git log --oneline   # notice HEAD is detached
   ```

8. Go back to main:
   ```bash
   git switch main
   ```

### ✅ Success Criteria
- Two annotated tags exist: `v0.1.0` and `v1.0.0`
- `git show v1.0.0` displays the tagger name, date, and message
- You can checkout a tag and return to a branch

---

## Exercise 9 — Reflog: Recover Lost Work (Bonus)

**Goal:** Use reflog to recover commits after a hard reset.

### Tasks

1. Create some commits to "lose":
   ```bash
   echo "important feature A" >> index.html
   git add . && git commit -m "Add important feature A"

   echo "important feature B" >> index.html
   git add . && git commit -m "Add important feature B"
   ```

2. Note the current commit hash:
   ```bash
   git log --oneline -3
   ```

3. Simulate accidentally resetting too far:
   ```bash
   git reset --hard HEAD~2
   ```

4. Confirm the commits appear gone:
   ```bash
   git log --oneline -3
   ```

5. Open the reflog — find the commits:
   ```bash
   git reflog
   ```
   Look for `HEAD@{1}: commit: Add important feature B`

6. Recover by resetting to the lost commit:
   ```bash
   git reset --hard HEAD@{1}
   ```

7. Verify both commits are back:
   ```bash
   git log --oneline -3
   ```

### ✅ Success Criteria
- You recovered commits that appeared to be permanently deleted
- You understand that `git reset --hard` is not truly permanent (within 90 days)

---

## 🏁 Day 4 Complete!

You can now:

- [x] Squash, reword, reorder, and drop commits with interactive rebase
- [x] Save and restore work-in-progress with stash
- [x] Apply specific commits across branches with cherry-pick
- [x] Find bug-introducing commits with bisect
- [x] Trace code history with blame and show
- [x] Mark releases with annotated tags
- [x] Recover "lost" commits using reflog

➡️ **[Check your work in solutions.md](./solutions.md)**

➡️ **Ready for Day 5? [Expert Git →](../Day-05-Git-Expert-Level/README.md)**
