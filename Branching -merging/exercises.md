# Day 2 — Exercises

> Build on your Day 1 repo (`git-practice`) or create a fresh one. Try every step yourself before peeking at solutions.
>
> Solutions: [solutions.md](./solutions.md)

---

## Exercise 1 — Create & Explore Branches

**Goal:** Create branches and understand how they relate to commits.

### Tasks

1. Navigate into your `git-practice` folder (or create a fresh repo):
   ```bash
   cd git-practice
   git log --oneline   # confirm you have some commits
   ```

2. List your existing branches:
   ```bash
   git branch
   ```
   You should see `* main` (the `*` marks your current branch).

3. Create a new branch called `feature/about-page`:
   ```bash
   git branch feature/about-page
   ```

4. List branches again. What do you notice about `HEAD`?
   ```bash
   git branch
   ```

5. Switch to the new branch:
   ```bash
   git switch feature/about-page
   ```

6. Confirm you're on the new branch:
   ```bash
   git branch
   git status
   ```

7. Look at the log — notice both branches point to the same commit:
   ```bash
   git log --oneline --graph --all
   ```

### ✅ Success Criteria
- You can create and switch branches
- You understand that a new branch starts at the same commit as the branch it was created from

### 🤔 Think About It
- Why does the new branch point to the same commit as `main` initially?
- What does the `*` in `git branch` output indicate?

---

## Exercise 2 — Commit on a Branch

**Goal:** See how branches diverge as you make commits.

### Tasks

1. Make sure you're on `feature/about-page`:
   ```bash
   git branch   # * feature/about-page
   ```

2. Create an `about.html` file:
   ```bash
   echo "<h1>About Us</h1>" > about.html
   echo "<p>We love Git.</p>" >> about.html
   ```

3. Stage and commit it:
   ```bash
   git add about.html
   git commit -m "Add about page HTML"
   ```

4. Add more content and make a second commit:
   ```bash
   echo "<p>Founded in 2024.</p>" >> about.html
   git add about.html
   git commit -m "Add founding year to about page"
   ```

5. Visualize the branch history:
   ```bash
   git log --oneline --graph --all
   ```
   Notice how `feature/about-page` is now **ahead** of `main`.

6. Switch back to `main` and check what files exist:
   ```bash
   git switch main
   ls
   ```
   Is `about.html` there?

7. Switch back to `feature/about-page` and check again:
   ```bash
   git switch feature/about-page
   ls
   ```

### ✅ Success Criteria
- `about.html` appears and disappears as you switch branches
- `git log --oneline --graph --all` shows the branches diverging

### 🤔 Think About It
- Where did `about.html` go when you switched to `main`?
- What would happen if you tried to switch branches with uncommitted changes?

---

## Exercise 3 — Fast-Forward Merge

**Goal:** Merge a branch when the target hasn't diverged (fast-forward).

### Tasks

1. Switch to `main`:
   ```bash
   git switch main
   ```

2. Check the log — confirm `main` is behind `feature/about-page`:
   ```bash
   git log --oneline --graph --all
   ```

3. Merge `feature/about-page` into `main`:
   ```bash
   git merge feature/about-page
   ```
   Read the output. Does it say `Fast-forward`?

4. Check the log again:
   ```bash
   git log --oneline --graph --all
   ```
   Where is `main` now?

5. Confirm `about.html` now exists on `main`:
   ```bash
   ls
   cat about.html
   ```

6. Since the feature is merged, delete the branch:
   ```bash
   git branch -d feature/about-page
   ```

7. List branches to confirm it's gone:
   ```bash
   git branch
   ```

### ✅ Success Criteria
- The merge output says `Fast-forward`
- `main` now points to the same commit as `feature/about-page` did
- `about.html` is accessible on `main`
- The feature branch is deleted

### 🤔 Think About It
- Was a new merge commit created? Why or why not?
- What condition must be true for a fast-forward merge to happen?

---

## Exercise 4 — Three-Way Merge

**Goal:** Experience a merge when both branches have diverged.

### Tasks

1. Create and switch to a new feature branch:
   ```bash
   git switch -c feature/contact-page
   ```

2. Create a `contact.html` file and commit it:
   ```bash
   echo "<h1>Contact Us</h1>" > contact.html
   echo "<p>Email: hello@example.com</p>" >> contact.html
   git add contact.html
   git commit -m "Add contact page"
   ```

3. **Now switch back to `main` and make a commit there too** (this causes divergence):
   ```bash
   git switch main
   echo "footer { text-align: center; }" >> style.css
   git add style.css
   git commit -m "Update footer styles"
   ```

4. Visualize — you should see the branches have diverged:
   ```bash
   git log --oneline --graph --all
   ```

5. Merge `feature/contact-page` into `main`:
   ```bash
   git merge feature/contact-page
   ```
   Your editor may open for a merge commit message. Save and close it.

6. Check the log — you should see a merge commit:
   ```bash
   git log --oneline --graph --all
   ```

7. Clean up the feature branch:
   ```bash
   git branch -d feature/contact-page
   ```

### ✅ Success Criteria
- `git log --graph` shows a diamond shape (two lines diverge and rejoin)
- A merge commit was created automatically
- Both `contact.html` and the updated `style.css` exist on `main`

### 🤔 Think About It
- How many parent commits does the merge commit have?
- Why didn't Git fast-forward this time?

---

## Exercise 5 — Create a Merge Conflict

**Goal:** Deliberately cause a conflict and understand what Git is telling you.

### Tasks

1. Create a new branch:
   ```bash
   git switch -c feature/update-heading
   ```

2. Edit `index.html` — change the heading:
   ```bash
   echo '<h1>My Awesome Git Project</h1>' > index.html
   git add index.html
   git commit -m "Update heading on feature branch"
   ```

3. Switch to `main` and edit the **same line** differently:
   ```bash
   git switch main
   echo '<h1>Git Practice Repository</h1>' > index.html
   git add index.html
   git commit -m "Update heading on main"
   ```

4. Try to merge:
   ```bash
   git merge feature/update-heading
   ```
   You should see a conflict message.

5. Check the status:
   ```bash
   git status
   ```

6. Open `index.html` and look at the conflict markers:
   ```bash
   cat index.html
   ```
   Identify the `<<<<<<<`, `=======`, and `>>>>>>>` markers.

### ✅ Success Criteria
- You see `CONFLICT (content): Merge conflict in index.html`
- You can identify and read the conflict markers in the file

### 🤔 Think About It
- Why did Git fail to merge automatically?
- What does the `=======` line separate?

---

## Exercise 6 — Resolve the Conflict

**Goal:** Manually resolve the conflict from Exercise 5 and complete the merge.

### Tasks

1. Open `index.html`. It looks like this:
   ```
   <<<<<<< HEAD
   <h1>Git Practice Repository</h1>
   =======
   <h1>My Awesome Git Project</h1>
   >>>>>>> feature/update-heading
   ```

2. Edit the file to keep a combined version (remove ALL conflict markers):
   ```bash
   echo '<h1>Git Practice Repository — Awesome!</h1>' > index.html
   ```

3. Verify the markers are gone:
   ```bash
   cat index.html
   ```
   No `<<<<<<<`, `=======`, or `>>>>>>>` should remain.

4. Stage the resolved file:
   ```bash
   git add index.html
   ```

5. Check status — it should say "All conflicts fixed but you are still merging":
   ```bash
   git status
   ```

6. Complete the merge with a descriptive commit message:
   ```bash
   git commit -m "Merge feature/update-heading: combine heading text"
   ```

7. View the final history:
   ```bash
   git log --oneline --graph --all
   ```

8. Clean up:
   ```bash
   git branch -d feature/update-heading
   ```

### ✅ Success Criteria
- The merge is complete with no conflict markers in `index.html`
- `git log --graph` shows the merge commit connecting both branches

---

## Exercise 7 — Abort a Merge (Bonus)

**Goal:** Learn to safely cancel a merge in progress.

### Tasks

1. Create a conflict scenario:
   ```bash
   git switch -c feature/conflict-test
   echo '<h2>Section from feature</h2>' >> about.html
   git add about.html
   git commit -m "Add section on feature branch"

   git switch main
   echo '<h2>Section from main</h2>' >> about.html
   git add about.html
   git commit -m "Add section on main"
   ```

2. Start the merge — it will conflict:
   ```bash
   git merge feature/conflict-test
   ```

3. Check the status:
   ```bash
   git status
   ```

4. Decide you're not ready to resolve it yet — **abort the merge**:
   ```bash
   git merge --abort
   ```

5. Check status and the file — everything should be back to the pre-merge state:
   ```bash
   git status
   cat about.html
   ```

6. Clean up the test branch:
   ```bash
   git branch -D feature/conflict-test
   ```

### ✅ Success Criteria
- After `git merge --abort`, `git status` shows a clean working tree
- `about.html` is restored to its pre-merge state on `main`

---

## Exercise 8 — Branch Strategy Practice (Bonus)

**Goal:** Simulate a real-world feature branch workflow.

### Scenario
You're building a personal website. While working on a new "Projects" page, a bug is reported on the homepage. You need to fix the bug without mixing it with unfinished feature work.

### Tasks

1. Start working on the projects feature:
   ```bash
   git switch -c feature/projects-page
   echo "<h1>My Projects</h1>" > projects.html
   git add projects.html
   git commit -m "Start projects page"
   ```

2. Urgent bug reported! Switch to `main` and create a hotfix branch:
   ```bash
   git switch main
   git switch -c hotfix/fix-typo
   ```

3. Fix the "bug" (add a line to README):
   ```bash
   echo "Fixed: corrected spelling in header" >> README.md
   git add README.md
   git commit -m "Hotfix: correct spelling in header"
   ```

4. Merge the hotfix to `main` and delete it:
   ```bash
   git switch main
   git merge hotfix/fix-typo
   git branch -d hotfix/fix-typo
   ```

5. Go back to the feature and finish it:
   ```bash
   git switch feature/projects-page
   echo "<p>See my work below.</p>" >> projects.html
   git add projects.html
   git commit -m "Complete projects page content"
   ```

6. Merge the feature to `main`:
   ```bash
   git switch main
   git merge feature/projects-page
   git branch -d feature/projects-page
   ```

7. View the full picture:
   ```bash
   git log --oneline --graph --all
   ```

### ✅ Success Criteria
- You successfully switched contexts between a feature and a hotfix
- Both changes made it into `main` independently
- `git log --graph` shows both branch paths

---

## 🏁 Day 2 Complete!

You can now:

- [x] Create, switch, and delete branches
- [x] Understand how HEAD moves with you
- [x] Perform fast-forward merges
- [x] Perform three-way merges
- [x] Cause, read, and resolve merge conflicts
- [x] Abort a merge in progress
- [x] Apply a real-world feature branch workflow

➡️ **[Check your work in solutions.md](./solutions.md)**

➡️ **Ready for Day 3? [Remote Repositories & GitHub →](../Day-03-Remote-Repositories/README.md)**
