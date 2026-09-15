# Git & GitHub — Session 1

## 1. What is Version Control?

Version Control is a system that records changes made to files over time.

It helps us:
- Track every change
- See who changed something and why
- Go back to an earlier version
- Work safely with multiple people
- Avoid overwriting each other's work
- Recover from mistakes

Version control is useful not only for application code, but also for:
- Scripts
- Infrastructure code
- Configuration files
- Documentation
- DevOps automation

### Without Version Control

Problems:
- Multiple copies such as `final_v2_FINAL`
- Changes can be overwritten
- No proper history
- Difficult to know who changed something
- Difficult to recover old versions

### With Version Control

We have:
- One repository and history
- Complete change history
- Safe parallel development
- Ability to recover previous states


# 2. Generations of Version Control

## Local Version Control

History is stored on one machine.

Examples:
- RCS
- SCCS

Problem:
The history is not naturally shared with other developers.

## Centralized Version Control (CVCS)

A central server stores the complete history.

Examples:
- SVN
- Perforce
- TFS

Developers get a working copy and commit changes back to the central server.

## Distributed Version Control (DVCS)

Every developer gets a complete copy of the repository and its history.

Examples:
- Git
- Mercurial

Important:

In Git, most operations such as commit, branch, diff and log happen locally and do not require the network.


# 3. Centralized vs Distributed

| Centralized | Distributed |
|---|---|
| History mainly on central server | Complete history in every clone |
| Depends heavily on server | Most operations work offline |
| Network required for many operations | Local operations are fast |
| Branching can be expensive | Branching is cheap |
| Commit usually publishes to central server | Commit is local |
| SVN, TFS, Perforce | Git, Mercurial |

### Important Git Concept

`git commit` does NOT automatically send changes to GitHub.

Commit = save the change in your local Git repository.

Push = send your local commits to the remote repository.

So:

```text
git commit
    ↓
Local repository

git push
    ↓
GitHub / Remote repository
```


# 4. What is Git?

Git is a distributed version control system.

Git was created by Linus Torvalds in 2005 for Linux kernel development.

Important characteristics of Git:

1. Distributed
2. Fast
3. Cheap branching
4. Data integrity
5. Staging area
6. Recoverable history

Git works locally on our computer.

We can use Git without GitHub.


# 5. Git vs GitHub

## Git

Git is the version control tool.

It:
- Runs on our computer
- Tracks changes
- Creates commits
- Creates branches
- Merges branches
- Maintains history
- Works offline

## GitHub

GitHub is a hosting and collaboration platform for Git repositories.

It provides:
- Remote repositories
- Pull Requests
- Code Reviews
- Issues
- Permissions
- GitHub Actions
- Releases
- Security features

### Easy way to remember

```text
Git = Version Control Tool

GitHub = Platform that hosts Git repositories
        + Collaboration features
```

GitHub is not Git.

Other Git hosting platforms include:
- GitLab
- Bitbucket
- Azure Repos


# 6. Installing Git

After installing Git, verify:

```bash
git --version
```

Example:

```text
git version 2.x.x
```

Git can be installed on:
- Windows
- Linux
- macOS


# 7. First-Time Git Configuration

Configure your name:

```bash
git config --global user.name "Your Name"
```

Configure your email:

```bash
git config --global user.email "your@email.com"
```

Set the default branch name:

```bash
git config --global init.defaultBranch main
```

Check configuration:

```bash
git config --list
```

### Configuration levels

```text
--system
    ↓
All users on the machine

--global
    ↓
Your user account

--local
    ↓
Only the current repository
```

Local configuration can override global configuration.


# 8. Git Repository

A Git repository is a directory managed by Git.

When we run:

```bash
git init
```

Git creates a hidden `.git` directory.

Example:

```text
my-project/
├── app.py
├── README.md
└── .git/
```

The `.git` directory contains Git's repository data and history.

### Important

Do not delete `.git` unless you intentionally want to remove Git history from that directory.

If `.git` is deleted:
- Your normal files remain
- Git history is removed from that repository


# 9. Creating a New Repository

Create a directory:

```bash
mkdir my-project
```

Move into it:

```bash
cd my-project
```

Initialize Git:

```bash
git init
```

Check the repository:

```bash
git status
```

Now the directory is a Git repository.


# 10. Clone an Existing Repository

If a repository already exists remotely:

```bash
git clone <repository-url>
```

Example:

```bash
git clone https://github.com/user/project.git
```

Clone downloads the repository including its history.

It also automatically creates a connection to the remote, normally named:

```text
origin
```


# 11. The Git File Workflow

This is the most important concept in Session 1.

```text
Working Directory
        |
        | git add
        ↓
Staging Area
        |
        | git commit
        ↓
Local Repository
        |
        | git push
        ↓
Remote Repository
        |
      GitHub
```

## Working Directory

The actual files we edit on our computer.

## Staging Area

The changes selected for the next commit.

## Local Repository

The committed Git history stored locally inside `.git`.

## Remote Repository

The shared repository, commonly hosted on GitHub.


# 12. Why Do We Need the Staging Area?

Suppose we changed five files:

```text
app.py
database.py
README.md
config.py
test.py
```

But we only want to commit:

```text
README.md
```

We can do:

```bash
git add README.md
git commit -m "Update documentation"
```

The staging area allows us to select exactly what belongs in a commit.

Therefore:

> Staging helps us create clean, logical commits instead of committing unrelated changes together.


# 13. Four States of a File

A file can commonly be understood through four states:

```text
Untracked
    ↓
Modified
    ↓
Staged
    ↓
Committed
```

## Untracked

Git sees the file, but Git is not tracking it yet.

Example:

```text
notes.txt
```

## Modified

A tracked file was changed after the last commit.

## Staged

The changes have been selected for the next commit using:

```bash
git add
```

## Committed

The change has been saved in the local Git repository.

### Most important command

```bash
git status
```

`git status` tells us what is happening with our files.


# 14. git status

Use:

```bash
git status
```

It can show:

```text
Changes to be committed
```

Meaning:

```text
STAGED
```

It can show:

```text
Changes not staged for commit
```

Meaning:

```text
MODIFIED
```

It can show:

```text
Untracked files
```

Meaning:

```text
NEW FILE NOT TRACKED BY GIT
```

### Best habit

Run:

```bash
git status
```

frequently.

Think of it as the **map of your Git repository**.


# 15. git add

Stage a specific file:

```bash
git add app.py
```

Stage multiple files:

```bash
git add app.py README.md
```

Stage everything:

```bash
git add .
```

Interactive staging:

```bash
git add -p
```

`git add` does NOT create a commit.

It only moves selected changes into the staging area.


# 16. git commit

Create a commit:

```bash
git commit -m "Add application code"
```

A commit saves the staged changes into the local Git repository.

Important:

```text
git add
    ↓
Select changes

git commit
    ↓
Save selected changes locally
```

A commit does NOT automatically push to GitHub.


# 17. Good Commit Messages

A good commit should represent one logical change.

Good:

```text
Add retry logic to payment client
```

Bad:

```text
fix
updates
asdf
changes
final version
```

### Good practices

- One logical change per commit
- Use imperative language
- Keep the summary concise
- Explain why when necessary

Example:

```bash
git commit -m "Add retry logic to payment client"
```

Good commit history makes troubleshooting and collaboration easier.


# 18. git diff

See changes that are modified but NOT staged:

```bash
git diff
```

See changes that ARE staged:

```bash
git diff --staged
```

Important difference:

```text
git diff
    ↓
Working changes vs last committed state

git diff --staged
    ↓
Staged changes vs last committed state
```


# 19. Reading Git History

View history:

```bash
git log
```

Compact history:

```bash
git log --oneline
```

Visual history:

```bash
git log --oneline --graph --all
```

Last five commits:

```bash
git log -5
```

History of a particular file:

```bash
git log -- file.txt
```

Find commits where a particular string was added or removed:

```bash
git log -S "password"
```

See who last changed each line:

```bash
git blame file.txt
```

Show a particular commit:

```bash
git show <commit-id>
```


# 20. HEAD

`HEAD` represents where we currently are in the Git history.

For example:

```text
HEAD
 ↓
Current commit
```

Previous commits can be referenced as:

```text
HEAD~1
HEAD~2
HEAD~3
```

Meaning:

```text
HEAD
 ↓
Current commit

HEAD~1
 ↓
One commit before HEAD

HEAD~2
 ↓
Two commits before HEAD
```


# 21. .gitignore

`.gitignore` tells Git which files/directories should not be tracked.

Example:

```gitignore
node_modules/
__pycache__/
dist/

.env
*.pem
credentials.json

.vscode/
.DS_Store
*.log
```

Common things to ignore:

- Dependencies
- Build output
- Environment files
- Secrets
- Credentials
- Editor files
- Operating-system files
- Log files


# 22. Important .gitignore Warning

`.gitignore` does NOT remove a file that has already been committed.

For example, if `.env` was already committed, adding:

```gitignore
.env
```

does not remove it from Git tracking.

Use:

```bash
git rm --cached .env
```

Then commit the change.

### Security rule

If a secret/token/password has already been committed:

> Treat the secret as compromised and rotate/revoke it.

Simply deleting the file later is not enough because it may remain in Git history.




# 23. Git Daily Workflow

The basic workflow to remember:

```text
1. Edit files
       ↓
2. git status
       ↓
3. git diff
       ↓
4. git add
       ↓
5. git diff --staged
       ↓
6. git commit
       ↓
7. git log
```

Later, when GitHub is introduced:

```text
git commit
    ↓
git push
    ↓
GitHub
```


# 25. DevOps/SRE Importance

Git is one of the most important foundations for DevOps.

A typical DevOps workflow looks like:

```text
Developer / DevOps Engineer
          ↓
       Git repo
          ↓
       GitHub
          ↓
     Pull Request
          ↓
      Code Review
          ↓
        CI/CD
          ↓
       Build/Test
          ↓
      Artifact/Image
          ↓
       Deployment
          ↓
        AWS / K8s
          ↓
      Monitoring
```

For a DevOps/SRE engineer, Git is used for:

- Infrastructure as Code
- Terraform
- Kubernetes manifests
- Helm charts
- Shell scripts
- Python automation
- CI/CD pipelines
- Configuration
- Documentation
- Application source code

The goal is not to memorize Git commands.

The goal is to understand the **Git workflow and use it confidently in production-style work**.


# Session 1 — Commands to Memorize

```bash
git --version

git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --list

git init
git clone <url>

git status

git add <file>
git add .

git commit -m "message"

git diff
git diff --staged

git log
git log --oneline
git log --oneline --graph --all

git show <commit>
git blame <file>

git rm --cached <file>
```

# Key Interview Questions

### What is Git?

Git is a distributed version control system used to track changes, manage history, and support parallel development.

### What is GitHub?

GitHub is a cloud-based hosting and collaboration platform for Git repositories.

### What is the difference between Git and GitHub?

Git is the version control tool; GitHub hosts Git repositories and provides collaboration features such as Pull Requests, reviews, issues and Actions.

### What is staging?

Staging is the intermediate area where we select changes that will be included in the next commit.

### Does git commit push code to GitHub?

No.

`git commit` saves changes locally.

`git push` sends local commits to the remote repository.

### What does git status do?

It shows the current state of files in the working directory and staging area.

### Why do we use .gitignore?

To prevent unwanted files such as secrets, dependencies, build output and editor/OS files from being tracked.

### Does .gitignore remove an already committed file?

No. If it is already tracked, it must first be removed from tracking, for example:

```bash
git rm --cached <file>
```