# Git & GitHub — Session 2

## 1. Session 2 Topics

- Undoing changes
- `git restore`
- `git revert`
- `git reset`
- `git stash`
- Branching
- Branching strategies
- Merging
- Merge conflicts
- Rebase
- Squashing
- `git reflog`

---

# 2. Undoing Changes

The Git undo command depends on **where the change currently exists**.

```text
Working Directory
        ↓
     Staging
        ↓
     Commit
        ↓
      Push
```

Different situations require different commands.


# 3. git restore

Used to restore files in the working directory.

### Discard changes in a file

```bash
git restore file.txt
```

This restores the file to its last committed state.

### Unstage a file

```bash
git restore --staged file.txt
```

This removes the file from staging but keeps the changes in the working directory.

Remember:

```text
git restore file.txt
        ↓
Discard working changes

git restore --staged file.txt
        ↓
Unstage changes
```


# 4. git commit --amend

Used when the **last commit** needs to be corrected.

Example:

```bash
git commit --amend
```

Useful for:
- Correcting the last commit message
- Adding a forgotten change to the last commit

It rewrites the most recent commit, so use it carefully if the commit has already been shared.


# 5. git revert

`git revert` safely undoes a previous commit by creating a **new opposite commit**.

Example:

```bash
git revert <commit-id>
```

History remains:

```text
Commit A
   ↓
Commit B
   ↓
Revert B
```

The original commit is not deleted.

### Important rule

If a commit has already been pushed and other people may have pulled it:

```text
Use git revert
```

Do not normally use reset on shared history.


# 6. git reset

`git reset` moves the branch pointer to another commit.

There are three important modes.

## --soft

```bash
git reset --soft HEAD~1
```

- Moves the branch pointer
- Keeps changes staged

## --mixed

```bash
git reset HEAD~1
```

Default mode.

- Moves branch pointer
- Keeps file changes
- Changes become unstaged

## --hard

```bash
git reset --hard HEAD~1
```

- Moves branch pointer
- Removes changes from working directory
- Destructive for uncommitted work

### Remember

```text
--soft
Keep staged changes

--mixed
Keep changes, unstage them

--hard
Discard changes
```

### Important

Avoid resetting commits that have already been shared with others.

Use `git revert` for shared history.


# 7. git stash

`git stash` temporarily stores uncommitted changes so that the working directory becomes clean.

Useful when:

> You are working on one task but suddenly need to switch to another task.

Example:

```bash
git stash
```

Include untracked files:

```bash
git stash -u
```

View stashes:

```bash
git stash list
```

Apply and remove the latest stash:

```bash
git stash pop
```

Apply a specific stash:

```bash
git stash apply stash@{1}
```

Delete a stash:

```bash
git stash drop
```

Delete all stashes:

```bash
git stash clear
```

### Typical workflow

```bash
git stash -u
git switch main

# Fix urgent issue
git add .
git commit -m "Fix urgent issue"

git switch feature/login
git stash pop
```

Stashes are:
- Local
- Not automatically pushed
- Easy to forget

For work lasting longer than a few hours, a branch is usually better.


# 8. git reflog

`git reflog` records where `HEAD` has been.

```bash
git reflog
```

It can help recover commits after operations such as:

```text
reset
branch deletion
rebase
```

Important idea:

> `reflog` is a safety net for recovering lost Git history.

Example:

```bash
git reflog
```

Find the required commit ID and recover it when necessary.


# 9. What is a Branch?

A branch is a **movable pointer to a commit**.

It allows us to work on a separate line of development without disturbing the main branch.

Example:

```text
main
 |
 A
 |
 B
 |\
 | \
 |  C
 |  D
 |  feature/login
 |
```

Typical branches:

```text
main
feature/login
feature/payment
bugfix/api-error
```


# 10. HEAD

`HEAD` points to the branch/commit we currently have checked out.

Example:

```text
HEAD
 ↓
feature/login
 ↓
Current commit
```

When we create a new commit, the branch pointer moves forward.


# 11. Branch Commands

List branches:

```bash
git branch
```

List local and remote branches:

```bash
git branch -a
```

Create a branch:

```bash
git branch feature/login
```

Switch branch:

```bash
git switch feature/login
```

Create and switch:

```bash
git switch -c feature/login
```

Older equivalent:

```bash
git checkout -b feature/login
```

Delete merged branch:

```bash
git branch -d feature/login
```

Force delete:

```bash
git branch -D feature/login
```

### Typical workflow

```bash
git switch main
git pull
git switch -c feature/search
```

Work:

```bash
git add .
git commit -m "Add search feature"
```


# 12. Branch Naming

Use meaningful names:

```text
feature/user-login
bugfix/null-pointer
hotfix/payment-timeout
release/2.4.0
```

Good branch names:
- Describe the work
- Use hyphens
- Use a consistent prefix


# 13. Branching Strategies

## GitHub Flow

```text
main
 ↓
feature branch
 ↓
Pull Request
 ↓
main
```

Best for:

> Simple continuous delivery.

## Git Flow

Uses:

```text
main
develop
feature
release
hotfix
```

Best for:

> Structured/versioned releases.

## GitLab Flow

Uses GitHub Flow-like development with environment branches such as:

```text
staging
production
```

## Trunk-Based Development

Developers integrate changes into `main` frequently.

Incomplete features can be controlled using feature flags.

### Common principles

Regardless of strategy:

- Keep branches short-lived
- Merge into main frequently


# 14. Merging

Merging brings changes from one branch into another.

Example:

```bash
git switch main
git pull
git merge feature/login
```

After successful merge:

```bash
git branch -d feature/login
```


# 15. Fast-Forward Merge

If `main` has not changed since the feature branch was created, Git can simply move the main pointer forward.

```text
Before:

main
 ↓
A
 \
  B
  ↑
feature


After merge:

A → B
    ↑
   main
```

No merge commit is required.


# 16. Three-Way Merge

If both branches contain new commits, Git combines the histories.

Example:

```text
       C
      /
A → B
      \
       D
```

Git creates a merge commit containing both histories.

```bash
git merge feature/login
```


# 17. Useful Merge Commands

Always update main first:

```bash
git switch main
git pull
```

Merge:

```bash
git merge feature/login
```

Always create a merge commit:

```bash
git merge --no-ff feature/login
```

Combine branch changes without immediately creating the commit:

```bash
git merge --squash feature/login
```

Abort a conflicted merge:

```bash
git merge --abort
```

View history:

```bash
git log --graph --all
```


# 18. Merge Conflict

A merge conflict happens when Git cannot automatically combine changes.

A common case:

```text
Branch A:
port = 8080

Branch B:
port = 9090
```

Both branches changed the same line.

Git marks the conflict:

```text
<<<<<<< HEAD
port = 8080
=======
port = 9090
>>>>>>> feature/login
```

This is normal.

Git is asking us to decide what the final content should be.


# 19. Resolving a Conflict

### Step 1

Check the conflict:

```bash
git status
```

### Step 2

Open the conflicted file.

### Step 3

Choose the correct final content.

Remove the conflict markers:

```text
<<<<<<<
=======
>>>>>>>
```

### Step 4

Stage the resolved file:

```bash
git add file.txt
```

### Step 5

Complete the merge:

```bash
git commit
```

Or cancel the merge:

```bash
git merge --abort
```

### Conflict workflow

```text
Merge
  ↓
Conflict
  ↓
Edit file
  ↓
git add
  ↓
git commit
```


# 20. How to Reduce Merge Conflicts

- Pull frequently
- Keep branches short-lived
- Keep commits small
- Avoid long-running branches
- Communicate when editing shared files
- Merge changes frequently


# 21. Rebase

`git rebase` moves/replays your commits on top of another branch.

Example:

```bash
git switch feature/login
git rebase main
```

Conceptually:

### Before

```text
      C → D
     /
A → B
     \
      E → F
```

### After rebase

```text
A → B → E' → F'
```

The feature commits are replayed on top of the latest main.


# 22. Merge vs Rebase

| Merge | Rebase |
|---|---|
| Preserves branch history | Creates a linear history |
| Can create merge commit | Usually no merge commit |
| Existing commit IDs remain | Commit IDs are rewritten |
| Safe for shared history | Avoid on shared history |
| Shows actual branching | Makes history look sequential |

### Golden Rule

> Never rebase commits that other people are already using.

A common safe pattern:

```bash
git switch feature/login
git rebase main
```

before sharing the feature branch.


# 23. Squashing

Squashing combines multiple commits into one.

Example:

```text
Before:

Add login
Fix login
Fix typo
Fix login again
```

After squash:

```text
Add login feature
```

This creates cleaner shared history.


# 24. Interactive Rebase

Use:

```bash
git rebase -i HEAD~3
```

Git opens the last three commits.

Common options:

```text
pick
squash
fixup
reword
drop
```

### pick

Keep the commit.

### squash

Combine it with the previous commit and keep/edit messages.

### fixup

Combine it with the previous commit and discard its message.

### reword

Keep the commit but change its message.

### drop

Remove the commit.


# 25. When to Squash

Squash your own messy/private commits before sharing when appropriate.

Example:

```text
Add login
Fix login
Fix typo
Fix test
Final fix
```

can become:

```text
Add login feature
```

But don't blindly squash everything.

Different logical changes can deserve separate commits.


# 26. Important Session 2 Mental Model

When something goes wrong, first ask:

```text
Where is my change?
```

### Unstaged working change

```bash
git restore
```

### Staged change

```bash
git restore --staged
```

### Last local commit

```bash
git reset
```

### Already shared/pushed commit

```bash
git revert
```

### Temporary unfinished work

```bash
git stash
```

### Lost/reset history

```bash
git reflog
```

This decision-making ability is more important than memorizing commands.


# 27. Session 2 Practical Lab

## Branch

```bash
git switch -c feature/greeting
```

Make changes and create two commits.

Return:

```bash
git switch main
```

View branches:

```bash
git branch
```

## Merge

```bash
git merge feature/greeting
```

Delete branch:

```bash
git branch -d feature/greeting
```

View history:

```bash
git log --oneline --graph --all
```

## Conflict

Create changes to the same line on two branches.

Merge them and resolve the conflict.

```bash
git status
git add .
git commit
```

## Revert

Create a bad commit:

```bash
git commit -m "Add bad change"
```

Then:

```bash
git revert <commit-id>
```

## Stash

```bash
git stash
git stash list
git stash pop
```

## Reflog

```bash
git reflog
```

Practice recovering a reset commit.


# 28. Session 2 Commands

```bash
git restore file.txt
git restore --staged file.txt

git commit --amend

git revert <commit-id>

git reset --soft HEAD~1
git reset HEAD~1
git reset --hard HEAD~1

git stash
git stash -u
git stash list
git stash pop
git stash apply
git stash drop

git reflog

git branch
git branch -a
git branch feature/login
git switch feature/login
git switch -c feature/login
git branch -d feature/login
git branch -D feature/login

git merge feature/login
git merge --no-ff feature/login
git merge --squash feature/login
git merge --abort

git rebase main
git rebase -i HEAD~3
```


# 29. Interview Questions

### What is git revert?

It creates a new commit that reverses the changes introduced by an earlier commit.

### What is git reset?

It moves the current branch pointer to another commit and can change the staging/working state depending on the reset mode.

### Revert vs Reset?

```text
Reset
→ Rewrites local history

Revert
→ Creates a new commit to undo previous changes
```

For shared history, prefer `revert`.

### What is git stash?

It temporarily stores uncommitted changes so the working directory can be cleaned for another task.

### What is a branch?

A branch is a movable pointer to a commit.

### What is a merge conflict?

A conflict occurs when Git cannot automatically combine changes, commonly because different branches changed the same lines.

### Merge vs Rebase?

Merge combines histories and can create a merge commit.

Rebase replays commits on a new base and creates rewritten commits with new IDs.

### What is git reflog?

It records movements of `HEAD` and can help recover commits after operations such as reset or branch deletion.

### What is squash?

Squashing combines multiple commits into a smaller number of commits, often one logical commit.