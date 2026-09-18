# Git & GitHub — Study Notes

**Study Date:** 18-09-2026  
**Session Video Date:** 10-09-2026

---

# 1. Git Reset

`git reset` moves the current branch pointer backward to another commit.

It is mainly used when commits are still local and have not been shared.

```bash
git reset HEAD~1
```

There are three important modes:

## --soft

```bash
git reset --soft HEAD~1
```

- Moves the branch pointer backward
- Keeps changes staged
- Useful when we want to undo the commit but keep the changes ready to commit again

```text
Commit
  ↓
Reset --soft
  ↓
Staging Area
```

## --mixed

```bash
git reset --mixed HEAD~1
```

or simply:

```bash
git reset HEAD~1
```

`--mixed` is the **default mode**.

- Moves branch pointer backward
- Keeps the file changes
- Removes the changes from staging
- Changes remain in the working directory

```text
Commit
  ↓
Reset --mixed
  ↓
Working Directory
```

## --hard

```bash
git reset --hard HEAD~1
```

- Moves branch pointer backward
- Removes changes from staging
- Removes the changes from the working directory
- Destructive for uncommitted work

```text
Commit
  ↓
Reset --hard
  ↓
Changes discarded
```

### Easy memory

```text
--soft
→ Keep changes staged

--mixed
→ Keep changes unstaged
→ DEFAULT

--hard
→ Remove changes
```

---

# 2. Reset vs Revert

## Reset

`reset` moves the branch pointer backward.

```text
Before:

A → B → C
        ↑
       HEAD

After reset:

A → B
    ↑
   HEAD
```

The old commit is no longer part of the normal branch history.

Use reset mainly for **local/unshared commits**.

## Revert

`revert` creates a new commit that reverses the changes from an earlier commit.

```text
Before:

A → B → C

After revert:

A → B → C → D
            ↑
       Revert C
```

The original commit remains in history.

### Rule

```text
Local / not shared
→ reset can be used

Already pushed / shared
→ revert is the safer approach
```

Do not rewrite shared history with reset unless the team explicitly agrees to it. fileciteturn0file0

---

# 3. Git Rebase

`git rebase` takes commits from one branch and replays them on top of another branch.

Example:

```bash
git switch feature/login
git rebase main
```

Before:

```text
      C → D
     /
A → B
     \
      E → F
```

After rebase:

```text
A → B → E' → F'
```

The feature commits are recreated on top of the latest base.

The recreated commits get **new commit IDs**.

---

# 4. Merge vs Rebase

## Merge

```bash
git merge feature/login
```

Merge preserves the branch history and may create a merge commit.

```text
A → B → C
     \   /
      D →
```

## Rebase

```bash
git rebase main
```

Rebase creates a more linear history.

```text
A → B → C' → D'
```

### Main difference

```text
MERGE
→ Combine histories
→ Preserves existing commits

REBASE
→ Replay commits
→ Creates new commit IDs
→ Produces linear history
```

### Golden Rule

> Never rebase commits that other people are already using.

A common safe workflow is rebasing your own feature branch onto an updated `main` before sharing it. fileciteturn0file0

---

# 5. Git Commit Show

To see the details of a commit:

```bash
git show <commit-id>
```

It displays information such as:

- Commit ID
- Author
- Date
- Commit message
- Changes introduced by the commit

Example:

```bash
git show a1b2c3d
```

To show the latest commit:

```bash
git show HEAD
```

### Difference

```text
git log
→ Shows commit history

git show <commit>
→ Shows details and changes of a particular commit
```

---

# 6. Branch Protection Rules

Branch protection rules are GitHub repository rules used to protect important branches such as `main`.

Example:

```text
Developer
    ↓
Feature Branch
    ↓
Pull Request
    ↓
Code Review
    ↓
CI/CD Checks
    ↓
Protected main
```

Branch protection can require things such as:

- Pull Request reviews
- Passing status checks
- Up-to-date branches before merging

This prevents developers from directly introducing unreviewed or failing changes into protected branches. fileciteturn0file0

### Example production rule

```text
main branch
    ↓
Direct push ❌
    ↓
Pull Request ✅
    ↓
Review ✅
    ↓
CI tests pass ✅
    ↓
Merge
```

This is very important in DevOps because `main` may be connected to a production deployment pipeline.

---

# 7. Webhook

A webhook allows one system to notify another system when an event occurs.

Simple flow:

```text
GitHub
   ↓
Webhook
   ↓
External System
```

Example:

```text
Developer pushes code
        ↓
GitHub
        ↓
Webhook event
        ↓
CI/CD system
        ↓
Build
        ↓
Test
        ↓
Deploy
```

A webhook is essentially an event-driven notification mechanism.

Examples of events:

- Push
- Pull Request
- Release

Webhooks are useful for connecting GitHub with external automation systems.

---

# 8. GitHub Actions

GitHub Actions is GitHub's automation/CI/CD platform.

It can automatically run workflows when repository events occur.

Example:

```text
Developer
   ↓
git push
   ↓
GitHub
   ↓
GitHub Actions
   ↓
Build
   ↓
Test
   ↓
Security checks
   ↓
Deploy
```

Typical CI/CD tasks:

```text
Code
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Package
 ↓
Deploy
```

GitHub Actions can automate these steps.

---

# 9. GitHub Actions and CI/CD

## CI — Continuous Integration

CI automatically validates code changes.

Example:

```text
Pull Request
     ↓
GitHub Actions
     ↓
Build
     ↓
Unit Tests
     ↓
Security Checks
```

If tests fail:

```text
CI ❌
```

The team can fix the problem before merging.

## CD — Continuous Delivery / Deployment

After successful validation, the pipeline can continue toward deployment.

Example:

```text
Code
 ↓
Build
 ↓
Test
 ↓
Docker Image
 ↓
Container Registry
 ↓
AWS / Kubernetes
```

GitHub Actions can automate this workflow.

---

# 10. GitHub Workflow

A common GitHub development workflow:

```text
main
 ↓
Create feature branch
 ↓
Make changes
 ↓
git add
 ↓
git commit
 ↓
git push
 ↓
Create Pull Request
 ↓
Code Review
 ↓
GitHub Actions
 ↓
CI checks
 ↓
Approval
 ↓
Merge into main
 ↓
Deployment
```

The GitHub platform adds collaboration features around Git, including Pull Requests, reviews, Issues, Actions, releases and security tooling. fileciteturn0file0

---

# 11. GitLab Flow

GitLab Flow is another workflow model.

It can combine feature branches with environment branches.

Example:

```text
feature
   ↓
main
   ↓
staging
   ↓
production
```

It is useful when applications move through multiple environments.

The important point:

> Git commands remain Git commands. GitHub and GitLab mainly provide different platforms and workflow features around Git. fileciteturn0file0

---

# 12. GitHub vs GitLab

Both can provide:

- Git repositories
- Pull/Merge Requests
- Code review
- CI/CD
- Issues
- Permissions
- Branch protection
- Automation

Conceptually:

```text
Git
 ↓
Version Control

GitHub / GitLab
 ↓
Hosting + Collaboration + Automation
```

---

# 13. Production DevOps Workflow

A production-style workflow can look like:

```text
Developer
    ↓
Feature Branch
    ↓
Git Commit
    ↓
Push
    ↓
Pull Request
    ↓
Code Review
    ↓
Branch Protection
    ↓
GitHub Actions
    ↓
Build
    ↓
Test
    ↓
Security Scan
    ↓
Artifact / Docker Image
    ↓
Deploy
    ↓
AWS / Kubernetes
```

This is where Git becomes a foundation for DevOps.

---

# 14. Important Commands

```bash
# Reset
git reset HEAD~1
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1

# Revert
git revert <commit-id>

# Rebase
git rebase main

# Interactive rebase
git rebase -i HEAD~3

# Show commit
git show <commit-id>
git show HEAD

# History
git log
git log --oneline
git log --oneline --graph --all

# Branch
git branch
git switch main
git switch -c feature/login

# Merge
git merge feature/login
git merge --abort
```

---

# 15. Quick Revision

```text
git reset
→ Move branch pointer backward

--soft
→ Keep changes staged

--mixed
→ Keep changes unstaged
→ DEFAULT

--hard
→ Discard changes

git revert
→ Undo a commit using a NEW commit

git rebase
→ Replay commits on a new base

git show
→ Show details and changes of a commit

Branch Protection
→ Protect important branches

Webhook
→ Send event notification to another system

GitHub Actions
→ Automate CI/CD workflows

GitHub/GitLab
→ Git hosting + collaboration + automation
```

# 16. Key Interview Questions

### What is the default mode of git reset?

`--mixed` is the default.

### Difference between reset --soft and reset --mixed?

`--soft` keeps the changes staged.

`--mixed` keeps the changes but unstages them.

### What does reset --hard do?

It moves the branch pointer and makes the working directory match that commit, potentially discarding uncommitted changes.

### Reset vs revert?

Reset moves the branch pointer and can rewrite local history.

Revert creates a new commit that reverses an earlier commit.

### When should you use revert?

When undoing a commit that has already been pushed/shared.

### What is rebase?

Rebase replays commits on top of another base commit, creating a linear history with new commit IDs.

### What is branch protection?

Rules that restrict how protected branches such as `main` can be changed, for example requiring reviews and passing checks.

### What is a webhook?

A mechanism for sending event notifications from GitHub to another system.

### What is GitHub Actions?

GitHub's automation platform used to execute workflows such as CI/CD pipelines.

### What is CI/CD?

CI automatically builds/tests/validates changes. CD automates delivering or deploying validated changes.