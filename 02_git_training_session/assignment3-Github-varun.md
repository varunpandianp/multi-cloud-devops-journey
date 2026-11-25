# Assignment 3: Version Control with Git and GitHub

**Name:** Varun Pandian\
**Module:** 4 -- Version Control with Git and GitHub\
**Project:** `invoice-service`\
**Platform:** Windows, IntelliJ IDEA Terminal (Git Bash), GitHub\
**Status:** Tasks 1--3 practised; Task 4 collaboration steps must be
completed/verified where marked pending.

> Replace every `TODO` placeholder with your actual repository URL, pull
> request URLs, partner details, screenshots, or terminal output. Do not
> submit invented URLs or evidence.

------------------------------------------------------------------------

## Before You Start

### Question: What Git identity and defaults must be configured?

**Answer / commands:**

``` bash
git config --global user.name "Varun Pandian"
git config --global user.email "YOUR_GITHUB_EMAIL"
git config --global init.defaultBranch main
git config --global core.editor "notepad"
```

Check the configuration:

``` bash
git config --global --list
```

The name and email are stored as author/committer metadata in commits.
The default branch is `main`. The editor setting selects the editor Git
opens for commit messages.

### Assignment working standards

-   Use imperative commit messages, with the first line under 50
    characters.
-   Make one logical change per commit.
-   Do not commit generated files, secrets, or binaries.
-   Keep `.env` out of Git.

------------------------------------------------------------------------

# Task 1 --- Start the Repository Properly

## Question 1: Create `~/assignment3/invoice-service` and initialize a repository.

**Commands:**

``` bash
mkdir -p ~/assignment3/invoice-service
cd ~/assignment3/invoice-service
git init
git status
```

**Answer:** `git init` creates the `.git` directory and initializes the
current directory as a Git repository. The repository tracks changes
after files are staged and committed.

## Question 2: Create a two-sentence README and commit it separately.

**`README.md`:**

``` markdown
# Invoice Service

Invoice Service is a small application designed to manage customer invoices and calculate invoice amounts. It helps organize invoice details and maintain accurate billing records.
```

**Commands:**

``` bash
git add README.md
git commit -m "Add invoice service README"
```

**Answer:** The README was committed independently so the project
description has its own logical commit.

## Question 3: Create `app.py` and `config.ini`, and commit them separately.

**`app.py`:**

``` python
def calculate_invoice(subtotal, tax_rate):
    tax = subtotal * tax_rate / 100
    total = subtotal + tax
    return total

invoice_total = calculate_invoice(1000, 18)
print(f"Invoice Total: {invoice_total}")
```

**Commands:**

``` bash
git add app.py
git commit -m "Add invoice calculation logic"
```

**`config.ini`:**

``` ini
[application]
name = invoice-service
environment = development
port = 8080
```

**Commands:**

``` bash
git add config.ini
git commit -m "Add application configuration"
```

**Answer:** Separate commits make it clear which commit introduced the
application code and which introduced configuration.

## Question 4: Create `.env` and `build/`.

**Commands:**

``` bash
echo "API_KEY=not-a-real-secret" > .env
mkdir -p build
echo "Build output sample" > build/output.txt
```

**Answer:** These are local files for the exercise. They should not be
committed.

## Question 5: Add `.gitignore` and commit it.

**`.gitignore`:**

``` gitignore
# Ignore environment variables and secrets
.env

# Ignore generated build files
build/
```

**Commands:**

``` bash
git add .gitignore
git commit -m "Add gitignore for local files"
```

**Answer:** `.gitignore` tells Git to ignore matching untracked files.
It does not automatically remove files that are already tracked.

## Question 6: Prove `.env` and `build/` are ignored.

**Commands:**

``` bash
git status --short
git check-ignore -v .env build/output.txt
```

**Expected result:** `git status --short` should not list `.env` or
files under `build/`. `git check-ignore -v` identifies the `.gitignore`
rule responsible for ignoring each path.

**Evidence to insert:** TODO --- paste your actual command output or
screenshot.

## Question 7: Modify README and compare the differences before and after staging.

Add this sentence to `README.md`:

``` markdown
The application currently supports invoice total calculation with tax.
```

**Commands, in order:**

``` bash
git diff
git add README.md
git diff
git diff --staged
```

**Answer:**

-   `git diff` shows changes in the working directory that have not been
    staged.
-   After `git add README.md`, `git diff` normally shows no output for
    that file because the working copy and staging area match.
-   `git diff --staged` shows changes in the staging area compared with
    the last commit (`HEAD`).

If you commit the README update:

``` bash
git commit -m "Document invoice calculation feature"
```

## Question 8: Show compact one-line history.

**Command:**

``` bash
git log --oneline
```

**Answer:** Each line displays a short commit hash and its commit
message.

**Evidence to insert:** TODO --- paste your actual `git log --oneline`
output.

## Task 1 --- Think About It

**Question:** If `.env` was committed before `.gitignore` was added,
what two things must be done, and which is not a Git operation?

**Answer:**

1.  Add `.env` to `.gitignore` so it is ignored in future.
2.  Remove `.env` from Git tracking, for example with
    `git rm --cached .env`, and commit that removal.

If the file contains a real secret, rotate/revoke the secret and replace
it. Rotating the secret with the service/provider is not a Git
operation. Removing a file from tracking does not erase it from previous
commits; sensitive history may also need careful cleanup.

------------------------------------------------------------------------

# Task 2 --- Work on Two Things at Once

## Question 1: Create `feature/currency-format` and make two commits.

**Commands:**

``` bash
git switch main
git switch -c feature/currency-format
```

The feature branch added a currency-formatting function to `app.py`:

``` python
def format_currency(amount):
    return f"₹{amount:,.2f}"
```

It also added this call:

``` python
print(format_currency(invoice_total))
```

First commit:

``` bash
git add app.py
git commit -m "Add currency formatting function"
```

A sentence was added to `README.md`:

``` markdown
The application formats invoice totals in Indian Rupees.
```

Second commit:

``` bash
git add README.md
git commit -m "Document currency formatting feature"
```

Verify:

``` bash
git log --oneline -2
git branch
```

## Question 2: Return to main and confirm feature changes are absent.

**Commands:**

``` bash
git switch main
git branch
```

Inspect `app.py` and `README.md`.

**Answer:** The feature changes are not present on `main` because they
were committed on a separate branch and had not yet been merged into
`main`.

## Question 3: Create `bugfix/rounding-error`, make one commit, and merge it.

**Commands:**

``` bash
git switch main
git switch -c bugfix/rounding-error
```

The invoice calculation function was changed to round the total to two
decimal places:

``` python
def calculate_invoice(subtotal, tax_rate):
    tax = subtotal * tax_rate / 100
    total = subtotal + tax
    return round(total, 2)
```

Commit and merge:

``` bash
git add app.py
git commit -m "Fix invoice amount rounding"
git switch main
git merge bugfix/rounding-error
```

## Question 4: Show the graph and identify the merge type.

**Command:**

``` bash
git log --oneline --graph --all
```

**Answer:** The bugfix merge was a fast-forward merge, provided `main`
had not moved forward after the bugfix branch was created. Git could
move the `main` pointer forward to the bugfix commit without creating a
separate merge commit.

**Evidence:** TODO --- paste your actual graph output.

## Question 5: Merge `feature/currency-format` into main and compare it with the previous merge.

**Commands:**

``` bash
git switch main
git merge feature/currency-format
```

Git opened Vim for the merge message. The merge was completed by
pressing `Esc`, entering `:wq`, and pressing `Enter`.

**Answer:** This feature merge created a merge commit because `main` had
received the bugfix commit after the feature branch had diverged. The
two branches therefore contained independent commits that needed to be
joined. The earlier bugfix merge was fast-forward because `main` had not
moved independently.

**Evidence:** TODO --- paste your actual
`git log --oneline --graph --all` output.

## Question 6: Delete both merged branches and prove they are gone.

**Commands:**

``` bash
git switch main
git branch -d bugfix/rounding-error
git branch -d feature/currency-format
git branch
git branch --list "bugfix/rounding-error" "feature/currency-format"
```

**Answer:** `git branch -d` safely deletes a branch that Git considers
merged. The final `--list` command should return no output for those two
branch names. Deleting branch references does not delete commits that
are reachable from `main`.

**Evidence:** TODO --- paste your actual output.

## Question 7: Delete an unmerged experimental branch.

**Commands used:**

``` bash
git switch main
git switch -c experiment/redesign
echo "Experimental invoice service redesign." > redesign.txt
git add redesign.txt
git commit -m "Add experimental redesign notes"
git switch main
git branch -d experiment/redesign
```

**Expected warning:**

``` text
error: The branch 'experiment/redesign' is not fully merged.
If you are sure you want to delete it, run 'git branch -D experiment/redesign'.
```

Force deletion for this deliberate disposable exercise:

``` bash
git branch -D experiment/redesign
git branch --list "experiment/redesign"
```

**Answer:** `-d` refuses to delete an unmerged branch to protect work
that may not exist elsewhere. `-D` force-deletes the branch reference
even when its commits are unmerged, so it should only be used when the
unmerged work is intentionally disposable or safely preserved elsewhere.

**Evidence:** TODO --- paste the actual warning and deletion output.

------------------------------------------------------------------------

# Task 3 --- Break Things, Then Recover

## Part A --- Resolve a Conflict

### Question 1: On main, set line 1 of `config.ini` to production and commit.

The file was edited so its first line became:

``` ini
environment=production
```

Commit recorded in the terminal history:

``` text
64fecd2 changed environment to production
```

### Question 2: Create `feature/staging-config` from the previous commit, change the same line, and commit.

The previous commit used as the branch base was `8c66d8d`.

**Commands:**

``` bash
git switch -c feature/staging-config 8c66d8d
```

The first line of `config.ini` was changed to:

``` ini
environment=staging
```

Commit recorded:

``` text
326b49e Set staging environment
```

### Question 3: Merge the branch into main and trigger a conflict.

**Commands:**

``` bash
git switch main
git merge feature/staging-config
```

**Observed output:**

``` text
Auto-merging config.ini
CONFLICT (content): Merge conflict in config.ini
Automatic merge failed; fix conflicts and then commit the result.
```

### Question 4: Record the conflict markers exactly.

The expected conflict block for the two versions was:

``` ini
<<<<<<< HEAD
environment=production
=======
environment=staging
>>>>>>> feature/staging-config
```

**Note:** Confirm this against the exact conflict block captured in your
IntelliJ file. Preserve the actual markers in your final evidence.

### Question 5: Resolve the conflict by keeping a combination of both versions, stage, and complete the merge.

The resolved `config.ini` was:

``` ini
[application]
name = invoice-service

[production]
environment = production
port = 8080

[staging]
environment = staging
port = 8081
```

**Commands:**

``` bash
git add config.ini
git commit
```

If Vim opens for the merge message: press `Esc`, type `:wq`, and press
`Enter`.

**Answer:** The conflict markers were removed and both configurations
were retained in separate sections. The merge commit has **two
parents**: the previous `main` commit and the staging branch commit.

**Evidence:** TODO --- paste the resulting merge commit and graph:

``` bash
git log --oneline --graph -5
```

## Part B --- Undo, Four Ways (Five Operations)

### Question 1: Modify README without staging, then discard the change.

**Commands:**

``` bash
git diff
git restore README.md
git status
```

**Answer:** `git restore README.md` discards unstaged modifications in
the working tree and restores the file from the index. Since the change
had not been staged, the index still contained the committed version.

### Question 2: Stage a change by mistake, then unstage it while keeping the edit.

**Commands:**

``` bash
git add README.md
git restore --staged README.md
git status
git diff
```

**Answer:** `git restore --staged README.md` removes the change from the
staging area but preserves the modification in the working directory.

### Question 3: Make a commit with a wrong message, then correct it without creating a second commit.

The deliberately incorrect commit message was:

``` bash
git commit -m "wrong message"
```

It was corrected using:

``` bash
git commit --amend -m "Document Git unstage practice"
```

**Answer:** `git commit --amend` replaces the latest commit with an
amended commit, including the corrected message. It changes the commit
ID. Do not amend commits already shared with others unless history
rewriting has been coordinated.

### Question 4: Commit a bug and undo it with a new commit.

The bug was introduced by changing the calculation return statement to:

``` python
return 0
```

The bug commit was created with:

``` bash
git add app.py
git commit -m "Introduce invoice calculation bug"
```

Undo the bug using:

``` bash
git revert <bug-commit-hash>
```

**Answer:** `git revert` creates a new commit that reverses the earlier
commit. This is the safe collaborative option when a faulty commit has
already been pushed to a shared branch and other developers may have
based work on it, because it preserves shared history rather than
rewriting it.

**Evidence:** TODO --- paste the actual original and revert commits:

``` bash
git log --oneline -4
```

### Question 5: Make two commits, then move the branch pointer back two commits while keeping changes staged.

Two practice commits were created:

``` text
089e418 update the app.py addition problem for reset example
1df8e0e update readme for reset command practice
```

The soft reset command was:

``` bash
git reset --soft HEAD~2
```

**Before reset:** The working tree was clean after the two commits.

**After reset:** The two commits were removed from the current branch
tip, while their combined changes remained staged.

Verification commands:

``` bash
git status
git diff --staged
git log --oneline -3
```

**Answer:** `--soft` moves the branch pointer but leaves the index and
working directory unchanged. The changes from the two commits remain
staged and ready to be committed again.

**Evidence:** TODO --- paste actual `git status` before and after the
reset.

### Undo operations summary table

  ----------------------------------------------------------------------------------
  Situation               Command used                       What it touched
  ----------------------- ---------------------------------- -----------------------
  Discard unstaged README `git restore README.md`            Working tree file;
  edit                                                       restores from index

  Unstage accidental      `git restore --staged README.md`   Staging area; keeps
  staging                                                    working-tree edit

  Correct latest commit   `git commit --amend -m "..."`      Replaces latest commit;
  message                                                    changes commit ID

  Reverse a committed bug `git revert <hash>`                Adds a new commit that
  safely                                                     reverses an earlier
                                                             commit

  Move branch back two    `git reset --soft HEAD~2`          Branch pointer; leaves
  commits but keep                                           index and working tree
  changes staged                                             intact
  ----------------------------------------------------------------------------------

## Part C --- Lose Work and Get It Back

### Question 1: Make an important commit and note its message.

The important commit was:

``` text
0d14769 Add important recovery checkpoint
```

The README addition was:

``` markdown
## Important Project Recovery Point

This commit contains important project documentation for Git reflog recovery practice.
```

### Question 2: Run a hard reset and confirm the commit is gone from normal history.

Command:

``` bash
git reset --hard HEAD~1
```

After reset, the normal log began with:

``` text
412a562 (HEAD -> main) updated task 3 part A answer and commands
```

The important commit `0d14769` was no longer visible in the normal
`git log`.

### Question 3: Find the discarded commit in reflog and restore it.

Command:

``` bash
git reflog
```

The relevant reflog entry was:

``` text
0d14769 HEAD@{2}: commit: Add important recovery checkpoint
```

A recovery branch was created:

``` bash
git branch recovery/important-checkpoint 0d14769
```

Verify the recovered commit:

``` bash
git show --stat recovery/important-checkpoint
git log recovery/important-checkpoint --oneline -3
git show recovery/important-checkpoint:README.md
```

**Answer:** The recovery branch points to the discarded commit without
moving `main`.

### Question 4: Explain why the commit was recoverable and name work that would not have been.

**Two-sentence answer:**

The commit was recoverable because Git's reflog retained a reference to
the previous HEAD, allowing me to locate the commit even after
`git reset --hard`. Uncommitted changes that were discarded by a hard
reset, such as uncommitted file edits, would not necessarily be
recoverable through the reflog.

**Evidence:** The actual reflog entry and the recovery branch/log output
should be included as screenshots or pasted terminal output.

------------------------------------------------------------------------

# Task 4 --- Collaborate Through GitHub

> Task 4 requires a partner repository and two pull requests. Complete
> the pending actions below and replace every TODO with real evidence.

## Part A --- Publish

### Question 1: Create an empty GitHub repository named `invoice-service`.

**Answer:** Create the repository on GitHub with the name
`invoice-service`. Do not initialize it with a README, `.gitignore`, or
license because the local repository already exists.

Repository URL: **TODO --- paste actual URL**

### Question 2: Add remote origin, push main, and set branch tracking with one flag.

Commands:

``` bash
git remote add origin https://github.com/<YOUR-USERNAME>/invoice-service.git
git remote -v
git push -u origin main
```

**Answer:** `origin` is the conventional name for the remote repository.
The `-u` flag sets the upstream tracking branch, linking local `main`
with `origin/main`. Later, `git push` and `git pull` can be used without
specifying the remote and branch.

### Question 3: Confirm history is visible and `.env` is not on GitHub.

Commands:

``` bash
git log --oneline
git check-ignore -v .env
git status
```

**Answer:** Verify the GitHub repository's commit history and file list
in the browser. `.env` should not appear in the repository.
`git check-ignore -v .env` identifies the ignore rule. Add a screenshot
of the GitHub file list and commit history.

**Evidence:** TODO --- add screenshot.

### Question 4: Add your partner as a collaborator.

**Answer / evidence:** TODO --- add the partner using GitHub repository
Settings → Collaborators (or the current repository access interface).
Record the partner's GitHub username and a screenshot showing the
invitation/access status.

## Part B --- Contribute to Your Partner's Repository

### Question 1: Clone the partner repository into a separate directory.

``` bash
cd ..
git clone https://github.com/<PARTNER-USERNAME>/<PARTNER-REPOSITORY>.git
cd <PARTNER-REPOSITORY>
```

**Answer:** Cloning creates a separate local copy of the partner's
repository, keeping it separate from the user's own `invoice-service`
working directory.

Partner repository URL: **TODO**

### Question 2: Create `feature/<yourname>-validation`, make two commits, and push it.

``` bash
git switch -c feature/varun-validation
```

Make two small, logical validation changes in the partner repository.
Stage and commit each separately:

``` bash
git add <first-file>
git commit -m "Add input validation"
git add <second-file>
git commit -m "Document validation behaviour"
git push -u origin feature/varun-validation
```

**Answer:** The feature branch isolates the contribution. Two separate
commits provide a readable history of the two logical changes.

**Evidence:** TODO --- paste the two actual commit messages and push
output.

### Question 3: Open a pull request against partner main.

**Answer:** On GitHub, open a pull request from
`feature/varun-validation` into the partner repository's `main`. Explain
what changed and why in the PR description.

First pull request URL: **TODO --- paste actual PR URL**

### Question 4: Review the pull request received and leave a genuine question.

**Answer:** Open the pull request from the partner, inspect the changes,
and leave at least one genuine question about the implementation. Do not
use a meaningless placeholder comment.

**Evidence:** TODO --- add screenshot of the review comment thread.

### Question 5: Respond to the comment on your own PR.

**Answer:** Respond to the reviewer's question by making and pushing a
further commit to your feature branch. Then request review again on
GitHub.

**Evidence:** TODO --- paste the follow-up commit and screenshot of the
updated review conversation.

### Question 6: Merge the PR received and pull the result into local main.

After the partner's PR is approved and merged, update the local
partner-repository clone:

``` bash
git switch main
git pull origin main
```

**Answer:** The local `main` branch is updated with the merged
contribution from the partner's GitHub repository.

**Evidence:** TODO --- paste pull output and updated log.

## Part C --- Tidy History Before Merging

### Question 1: Create a new branch and make three small commits, two being trivial fixes to the first.

In your own repository:

``` bash
git switch main
git pull origin main
git switch -c feature/varun-history-cleanup
```

Make three small, logical commits. The second and third should be
trivial fixes to the first change.

**Evidence:** TODO --- paste the three actual commit messages and
hashes.

### Question 2: Squash the three commits into one using interactive rebase.

Start the rebase:

``` bash
git rebase -i HEAD~3
```

In the interactive rebase editor, keep the first commit as `pick` and
change the next two from `pick` to `squash` (or `s`). Save and close the
editor, then provide a clear final commit message.

**Answer:** Interactive rebase combines the selected commits into one
commit. The final history is shorter and easier to review.

**Editor lines changed:** TODO --- paste the actual interactive rebase
lines before and after editing.

### Question 3: Make a commit directly on main, then rebase the feature branch onto updated main.

``` bash
git switch main
```

Make a small, separate change on `main`, then:

``` bash
git add <file>
git commit -m "Update main project documentation"
git switch feature/varun-history-cleanup
git rebase main
```

**Answer:** Rebasing replays the feature branch's commit(s) on top of
the updated `main` history.

### Question 4: Explain what happened to commit IDs and state the rule about rebasing.

**Two-sentence answer:**

Rebasing creates new commits with new commit IDs because each commit is
replayed on a different parent/base history. Never rebase commits that
have already been pushed to and shared with others, unless the team has
explicitly coordinated the history rewrite.

### Question 5: Push the branch, open a PR, and compare its graph with a merge.

``` bash
git push -u origin feature/varun-history-cleanup
git log --oneline --graph --all
```

Open a pull request from `feature/varun-history-cleanup` into your own
repository's `main`.

Second pull request URL: **TODO --- paste actual PR URL**

**Answer:** A rebase produces a linear feature history by replaying
commits on the updated base. A merge can preserve the two branch
histories and add a merge commit when the branches have diverged.

**Evidence:** TODO --- add history graph before and after rebase and the
PR URL.

------------------------------------------------------------------------

# Final Submission Checklist

-   [ ] Save this document as `assignment3-varun.md`.
-   [ ] Add the actual GitHub repository URL.
-   [ ] Add the partner-contribution pull request URL.
-   [ ] Add the own-repository pull request URL.
-   [ ] Include command output or screenshots for every Verify box in
    Tasks 1--4.
-   [ ] Include the Task 3 Part B undo table.
-   [ ] Include the exact conflict markers and resolved `config.ini`.
-   [ ] Include the reflog entry and evidence that the discarded commit
    was recovered.
-   [ ] Include the review comment thread screenshot.
-   [ ] Include the history graph before and after rebase.
-   [ ] Remove all TODO placeholders before submission.

## Submission

Submit the single Markdown file:

`assignment3-varun.md`

**Reminder:** Do not claim Task 4 is complete until the partner
collaboration, review, merge, squash, rebase, and both pull requests
have actually been completed and evidenced.
