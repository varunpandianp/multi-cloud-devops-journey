# Linux Fundamentals — Study Notes

**Date:** 02-09-2026  
**Topic:** Permissions, Users, SSH, and vi  
**Slides:** 14–26

---

## 1. Session Overview

This session covers:

```text
Users
   ↓
Groups
   ↓
sudo
   ↓
File Permissions
   ↓
Ownership
   ↓
SSH
   ↓
SSH Keys
   ↓
File Transfer
   ↓
vi / vim
```

### Main Purpose

Understand:

- Who can access what
- How access is controlled
- How to remotely access Linux servers
- How to edit files on Linux servers

This is important for DevOps and SRE because production servers need controlled access and are usually managed remotely.

---

## 2. Hands-on Lab 1

The first lab combines commands learned in Session 1.

### Explore

```bash
pwd
ls -lh /etc
ls -a
```

### Create

```bash
mkdir -p ~/lab1/logs
touch ~/lab1/logs/file1.txt
touch ~/lab1/logs/file2.txt
touch ~/lab1/logs/file3.txt
```

### Inspect Logs

```bash
tail -n 20 /var/log/syslog
```

Search for errors:

```bash
grep -i "error" /var/log/syslog
```

Count errors:

```bash
grep -ic "error" /var/log/syslog
```

Find configuration files:

```bash
find /etc -name "*.conf"
```

### Check Resources

```bash
df -h
du -sh /var/*
ps aux
```

### Important Lesson

When a Linux command fails:

```text
Read the error message first.
```

Linux errors are often specific about what went wrong.

---

## 3. Users, Groups, `sudo`, Permissions, SSH, and vi

This session focuses on three major areas:

1. Users, Groups, and `sudo`
2. Permissions and Ownership
3. SSH and `vi`

Think of a Linux server like an office:

| Linux Concept | Office Example |
|---|---|
| Users | People |
| Groups | Teams |
| Permissions | Access rules |
| `sudo` | Temporary admin authority |
| SSH | Remote entrance |
| `vi` | File editor |

---

# Users and Groups

## 4. Users, Groups, and Identity

Linux is a multi-user operating system.

Different users can have different:

- Files
- Permissions
- Groups
- Privileges
- Home directories

### Root User

```text
UID = 0
```

Root is the superuser. It has broad privileges and can bypass normal permission restrictions.

Best practice:

```text
Do not work directly as root unless necessary.
```

Instead, use:

```bash
sudo command
```

### Regular Users

Regular users are created for people.

Examples:

```text
ec2-user
ubuntu
priya
```

They normally have a home directory:

```text
/home/username
```

They receive only the permissions explicitly granted to them.

### System Accounts

System accounts are created for services and applications.

Examples:

```text
nginx
mysql
```

These accounts generally are not intended for interactive login.

A service account may use:

```text
/usr/sbin/nologin
```

to prevent interactive login.

---

## 5. Important Linux Account Files

### `/etc/passwd`

Contains account information:

```text
Username
UID
GID
Home directory
Login shell
```

### `/etc/shadow`

Contains:

```text
Password hashes
Password ageing information
```

It should only be readable by privileged users.

### `/etc/group`

Contains:

```text
Group name
GID
Group members
```

### `/etc/sudoers`

Defines who can execute commands using `sudo`.

Edit it using:

```bash
visudo
```

> Do not casually edit `/etc/sudoers` with a normal editor.

---

## 6. Checking Your Identity

```bash
whoami
```

Shows the current username.

Example:

```text
ec2-user
```

```bash
id
```

Shows UID, GID, and groups.

```bash
groups
```

Shows group memberships.

### Quick Memory

```text
whoami → Who am I?
id     → Who am I + UID/GID/groups?
groups → Which groups am I in?
```

---

## 7. Why Groups Exist

Imagine that 20 developers need access to:

```text
/opt/project
```

Bad approach:

```text
Give permission individually to 20 users.
```

Better approach:

```text
developers group
   ↓
/opt/project
   ↓
All developers get required access
```

Groups allow permissions to be assigned to a role or team instead of repeatedly configuring individual users.

---

## 8. Creating and Managing Users

### Create a User

```bash
sudo useradd -m -s /bin/bash priya
```

Breakdown:

```text
sudo       → Administrative privilege
useradd    → Create user
-m         → Create home directory
-s         → Specify login shell
/bin/bash  → Bash shell
priya      → Username
```

### Set Password

```bash
sudo passwd priya
```

### Create a Group

```bash
sudo groupadd developers
```

### Add a User to a Group

```bash
sudo usermod -aG developers priya
```

```text
-a → Append
-G → Supplementary group
```

Use:

```bash
usermod -aG group user
```

Without `-a`, existing supplementary groups can be replaced.

### Delete a User

```bash
sudo userdel -r priya
```

`-r` also removes the user’s home directory.

### Check Password Ageing

```bash
sudo chage -l priya
```

### Switch User

```bash
su - priya
```

---

## 9. Typical User Onboarding

```bash
sudo useradd -m -s /bin/bash priya
sudo passwd priya
sudo groupadd developers
sudo usermod -aG developers priya
id priya
```

```text
Create user
   ↓
Create or set credentials
   ↓
Assign role or group
   ↓
Verify access
```

### Professional Habit

Do not only configure access. Verify it as the actual user.

```bash
su - priya
```

Then test what that user can and cannot access.

---

# Linux Permissions and Ownership

## 10. Linux File Permissions

Linux permissions answer:

```text
Who can do what with this file or directory?
```

Run:

```bash
ls -l
```

Example:

```text
-rwxr-xr-- 1 priya developers 4096 deploy.sh
```

Breakdown:

```text
-    rwx    r-x    r--
│     │      │      │
│     │      │      └── Others
│     │      └───────── Group
│     └──────────────── Owner
└────────────────────── File type
```

---

## 11. File Type

The first character in a permission string shows the file type.

```text
- → Regular file
d → Directory
l → Symbolic link
```

Example:

```text
-rwxr-xr--
```

The first `-` means it is a regular file.

---

## 12. Permission Groups

Linux applies permissions to three categories:

```text
u → User / Owner
g → Group
o → Others
```

The three permissions are:

```text
r → Read
w → Write
x → Execute
```

Example:

```text
Owner | Group | Others
rwx   | r-x   | r--
```

---

## 13. Permission Meaning

### For a File

```text
r → Read file contents
w → Modify file contents
x → Execute as a program
```

### For a Directory

```text
r → List directory contents
w → Create or delete entries inside
x → Enter or traverse the directory
```

`x` on a directory does not mean “execute the directory.”

It means you can access it using commands such as:

```bash
cd directory-name
```

---

## 14. Numeric Permissions

```text
r = 4
w = 2
x = 1
```

Examples:

```text
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2     = 6
r-x = 4 + 1     = 5
r-- = 4         = 4
--- = 0
```

---

## 15. `chmod`

`chmod` means:

```text
Change file permissions.
```

### Permission `755`

```bash
chmod 755 deploy.sh
```

```text
Owner  → rwx = 7
Group  → r-x = 5
Others → r-x = 5
```

Result:

```text
rwxr-xr-x
```

### Permission `644`

```bash
chmod 644 index.html
```

```text
Owner  → rw-
Group  → r--
Others → r--
```

### Permission `600`

```bash
chmod 600 ~/.ssh/id_rsa
```

```text
Owner  → rw-
Group  → ---
Others → ---
```

Commonly used for private SSH keys.

### Permission `700`

```bash
chmod 700 ~/.ssh
```

```text
Owner  → rwx
Group  → ---
Others → ---
```

---

## 16. `chmod 777`

```bash
chmod 777 file
```

Means:

```text
Owner  → rwx
Group  → rwx
Others → rwx
```

Everyone receives full permissions.

### Professional Rule

Do not use `chmod 777` as a solution to a permissions problem.

Instead, ask:

```text
Who actually needs access?
Owner?
Group?
Others?
```

Give only the required permission.

This is the principle of least privilege.

---

## 17. Symbolic `chmod`

```bash
chmod u+x script.sh
```

Gives the owner execute permission.

```bash
chmod g-w file.txt
```

Removes group write permission.

```bash
chmod o= file.txt
```

Removes all permissions from others.

```bash
chmod a+r file.txt
```

Gives read permission to everyone.

---

## 18. Ownership

Files have both an owner and a group.

Change owner:

```bash
sudo chown priya file.txt
```

Change owner and group:

```bash
sudo chown priya:developers file.txt
```

Change group only:

```bash
sudo chgrp developers file.txt
```

---

## 19. `umask`

```bash
umask 022
```

`umask` controls default permissions removed when new files and directories are created.

```text
Default permissions
   -
umask
   ↓
Actual permissions
```

The key idea is that `umask` influences default permissions.

---

## 20. Special Permissions

There are three special permission bits.

### SUID

```text
4xxx
```

Example:

```bash
chmod u+s program
```

The program runs with the file owner’s privileges.

`passwd` is a classic example.

SUID is security-sensitive.

### SGID

```text
2xxx
```

On a directory, new files inherit the directory’s group.

Useful for shared team directories.

```text
/opt/projectx
   ↓
projectx group
   ↓
New files inherit projectx group
```

### Sticky Bit

```text
1xxx
```

Used for shared directories.

Example:

```text
/tmp
```

It prevents one user from deleting another user’s files in a shared directory.

---

## 21. Permission Patterns to Remember

```bash
chmod 600 ~/.ssh/id_rsa
```

Private SSH key.

```bash
chmod 700 ~/.ssh
```

SSH directory.

```bash
chmod 644 index.html
```

Normal web content.

```bash
chmod 755 deploy.sh
```

Executable script.

---

# SSH: Secure Shell

## 22. What Is SSH?

SSH means:

```text
Secure Shell
```

SSH allows you to securely connect to another machine and operate its terminal remotely.

Example:

```bash
ssh priya@192.168.1.20
```

```text
Your Laptop
   │
   │ SSH
   ↓
Remote Linux Server
```

SSH is fundamental in cloud environments because you normally do not physically access the server.

---

## 23. SSH Components

### Client

Your machine runs:

```bash
ssh
```

### Server

The remote machine runs:

```bash
sshd
```

The SSH server normally listens on:

```text
TCP port 22
```

### Authentication

SSH can use:

```text
Password
or
Public/private key
```

Key-based authentication is preferred.

### Encryption

After the SSH handshake, communication is encrypted.

This protects:

```text
Commands
Keystrokes
Data
```

---

## 24. Useful SSH Commands

Normal connection:

```bash
ssh user@host
```

Custom port:

```bash
ssh -p 2222 user@host
```

Specific private key:

```bash
ssh -i ~/.ssh/id_ed25519 user@host
```

Exit an SSH session:

```bash
exit
```

---

## 25. AWS EC2 SSH Connection

Example:

```bash
ssh -i my-key.pem ec2-user@PUBLIC_IP
```

Connection flow:

```text
Your Computer
   ↓
SSH
   ↓
Internet
   ↓
AWS Security Group
   ↓
EC2 port 22
   ↓
sshd
   ↓
Linux User
```

If SSH does not work, check:

1. Is the EC2 instance running?
2. Is port 22 allowed in the Security Group?
3. Is `sshd` running?
4. Is the username correct?
5. Is the private key correct?
6. Are key permissions correct?

---

# SSH Key Authentication

## 26. Public and Private Keys

SSH key authentication uses two keys:

```text
Private Key 🔐
Public Key 🔓
```

### Private Key

The private key stays on your machine.

Never:

```text
Email it
Commit it to GitHub
Paste it into chat
Upload it
```

### Public Key

The public key can be placed on servers.

The server stores it in:

```text
~/.ssh/authorized_keys
```

---

## 27. How SSH Keys Work

```text
Your Machine
   │
   │ Private Key
   ↓
SSH Authentication
   ↓
Remote Server
   │
   │ Public Key
   ↓
authorized_keys
```

The private key proves that you own the corresponding public key.

The private key itself is not sent to the server as a password.

---

## 28. Generate an SSH Key

```bash
ssh-keygen -t ed25519 -C "priya@laptop"
```

This creates a key pair:

```text
id_ed25519
   ↓
Private Key

id_ed25519.pub
   ↓
Public Key
```

---

## 29. Copy the Public Key

```bash
ssh-copy-id priya@192.168.1.20
```

This adds the public key to:

```text
~/.ssh/authorized_keys
```

Then connect using:

```bash
ssh priya@192.168.1.20
```

---

## 30. Why SSH Keys Are Better

SSH keys:

- Avoid reusable passwords
- Help resist password brute-force attacks
- Can be revoked by removing a key from `authorized_keys`
- Work well for automation
- Can be protected with a passphrase

They are important for:

```text
DevOps
CI/CD
Automation
Cloud
Infrastructure management
```

---

# File Transfer and SSH Security

## 31. File Transfer

### `scp`

Copy a file to a remote server:

```bash
scp file.txt user@host:/tmp/
```

Copy a file from a remote server:

```bash
scp user@host:/tmp/a.log .
```

Copy a directory:

```bash
scp -r dir/ user@host:/opt/
```

### `rsync`

```bash
rsync -avz dir/ host:/opt/
```

Useful for synchronizing directories and transferring only differences.

### `sftp`

```bash
sftp user@host
```

Starts an interactive file transfer session.

---

## 32. SSH Server Hardening

SSH server configuration file:

```text
/etc/ssh/sshd_config
```

Important settings:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers priya rochak
```

Conceptually:

```text
❌ Direct root login
❌ Password authentication
✅ Public-key authentication
```

`AllowUsers` restricts SSH login to specified users.

---

## 33. SSH Troubleshooting

Check SSH service:

```bash
systemctl status sshd
```

Check the firewall or security group:

```text
Security Group
   ↓
TCP port 22
   ↓
Your IP allowed?
```

Check permissions:

```bash
chmod 600 ~/.ssh/id_rsa
chmod 700 ~/.ssh
```

Use verbose SSH output:

```bash
ssh -v user@host
```

This helps show where the connection or authentication process is failing.

---

## 34. Professional SSH Rule

Keep one SSH session open before changing SSH configuration.

Example:

```text
Session 1: Connected to server
   ↓
Modify /etc/ssh/sshd_config
   ↓
Open Session 2
   ↓
Test login
   ↓
Close Session 1 only after successful test
```

A bad SSH configuration can lock you out of the server.

---

# vi / vim Editor

## 35. What Is vi?

`vi` is a terminal-based text editor.

DevOps engineers should know `vi` because production or minimal Linux systems may not have graphical editors.

You may only have:

```text
vi
```

or:

```text
vim
```

---

## 36. vi Modes

```text
Command Mode
   │
   │ i / a / o
   ↓
Insert Mode
   │
   │ Esc
   ↓
Command Mode
   │
   │ :
   ↓
Last-line Mode
```

### Command Mode

Default mode. Keys are commands.

### Insert Mode

Used to type or edit text.

Enter using:

```text
i
a
o
```

### Last-line Mode

Press:

```text
:
```

Then enter commands such as:

```text
:w
:q
:wq
```

---

## 37. Essential vi Commands

| Command | Meaning |
|---|---|
| `i` | Insert before cursor |
| `a` | Insert after cursor |
| `o` | Open a new line below |
| `Esc` | Return to command mode |
| `dd` | Delete current line |
| `yy` | Copy current line |
| `p` | Paste |
| `u` | Undo |
| `:w` | Save |
| `:q` | Quit |
| `:wq` | Save and quit |
| `:q!` | Quit without saving |
| `/word` | Search |
| `:set number` | Show line numbers |

### Most Important vi Escape

If you get stuck inside `vi`:

```text
Esc
:q!
Enter
```

```text
Esc  → Command mode
:q!  → Quit without saving
```

This is the `vi` escape hatch.

---

# Hands-on Lab 2

## 38. Set Up a Colleague’s Access

### Create

```text
Create user: trainee
Create group: projectx
Add trainee to projectx
```

### Secure

Create:

```text
/opt/projectx
```

Set group ownership and permissions:

```text
Owner and group → Required access
Others          → No access
```

### Connect

Generate SSH keys:

```bash
ssh-keygen
```

Copy the public key:

```bash
ssh-copy-id
```

Connect without a password.

### Edit

```bash
vi file.txt
```

- Add three lines.
- Save the file.
- Reopen the file.
- Delete one line.

Finally:

```bash
su - trainee
```

Verify what the user can actually access.

### Key Lesson

```text
Configuration is not enough.
Test access as the user who will actually use it.
```

---

# Shell Scripting Introduction

## 39. Session 03 Preview

The next session covers shell scripting.

Shell scripting turns repeated commands into repeatable automation.

Topics:

```text
Variables
   ↓
Input and arguments
   ↓
Exit codes
   ↓
Conditionals
   ↓
Loops
   ↓
Functions
   ↓
Reliable automation
```

Important idea:

```text
Anything you repeatedly type can potentially become a script.
```

---

# DevOps Connection

```text
Linux
│
├── Users and Groups
│      ↓
│   Access control
│
├── Permissions
│      ↓
│   Security
│
├── SSH
│      ↓
│   Remote server management
│
├── SSH Keys
│      ↓
│   Automation and CI/CD
│
├── scp and rsync
│      ↓
│   File deployment
│
├── vi
│      ↓
│   Server configuration
│
└── Shell Scripting
       ↓
    Automation
```

These concepts connect directly to:

```text
AWS EC2
Docker
Jenkins
GitHub Actions
Terraform
Ansible
Kubernetes
CI/CD
SRE Troubleshooting
```

---

# Must-Remember Cheat Sheet

## Identity

```bash
whoami
id
groups
```

## User Management

```bash
sudo useradd -m -s /bin/bash user
sudo passwd user
sudo groupadd developers
sudo usermod -aG developers user
su - user
```

## Permissions

```bash
ls -l
chmod 755 file
chmod 644 file
chmod 600 file
chmod 700 directory
```

## Ownership

```bash
chown user file
chown user:group file
chgrp group file
```

## SSH

```bash
ssh user@host
ssh -i key.pem user@host
ssh -p 2222 user@host
```

## SSH Keys

```bash
ssh-keygen -t ed25519
ssh-copy-id user@host
```

## File Transfer

```bash
scp file user@host:/tmp/
scp user@host:/tmp/file .
rsync -avz dir/ host:/opt/
```

## SSH Troubleshooting

```bash
systemctl status sshd
ssh -v user@host
```

## vi / vim

```text
i       → Insert
Esc     → Command mode
dd      → Delete line
yy      → Copy line
p       → Paste
u       → Undo
:w      → Save
:wq     → Save and quit
:q!     → Quit without saving
```