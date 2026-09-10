🐧 Linux Fundamentals — Study Notes
📅 Date: 02-09-2026
Slides 14–26 — Permissions, Users, SSH & vi

1. Session 02 Overview
   Slides 15–26 introduce:
   Users
   ↓
   Groups
   ↓
   Sudo
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
   vi/vim
   **The main purpose of this session is to understand:
   Who can access what, how access is controlled, how we remotely access servers, and how we edit files on Linux servers.**

This is very important for DevOps/SRE, because production servers need controlled access and are usually managed remotely.
2. Slide 14 — Hands-on Lab 1
   The first lab combines the commands learned in Session 1.
   Tasks
   Explore
   pwd
   ls -lh /etc
   ls -a
   Create
   mkdir -p ~/lab1/logs
   touch ~/lab1/logs/file1.txt
   touch ~/lab1/logs/file2.txt
   touch ~/lab1/logs/file3.txt
   Inspect logs
   tail -n 20 /var/log/syslog
   Search for errors:
   grep -i "error" /var/log/syslog
   Count errors:
   grep -ic "error" /var/log/syslog
   Find configuration files:
   find /etc -name "*.conf"
   Check resources:
   df -h
   du -sh /var/*
   ps aux
   Important lesson
   When a Linux command fails:
   Read the error message first.

Linux errors are often very specific about what went wrong. Module2_Linux_Fundamentals.pptxPPTX
3. Slide 15 — Session 02
   Permissions, Users, SSH and vi
   This session focuses on three major areas:
**1. Users, Groups & sudo
2. Permissions & Ownership**
**3. SSH & vi**
   Think of a Linux server like an office:
   Users       → People
   Groups      → Teams
   Permissions → Access rules
   sudo        → Temporary admin authority
   SSH         → Remote entrance
   vi          → File editor
**4. Slide 16 — Users, Groups & Identity**
   Linux is a multi-user operating system.
   Different users can have different:
- Files
- Permissions
- Groups
- Privileges
- Home directories
  Three important types of users
**1. Root**
   UID = 0
   Root is the superuser.
   It has extremely broad privileges and can bypass normal permission restrictions.
   Best practice:
   Don't work directly as root unless necessary.

Instead:
sudo command
**2. Regular users**
   Created for people.
   Example:
   ec2-user
   ubuntu
   priya
   They normally have:
   /home/username
   and only the permissions explicitly granted to them.
**3. System accounts**
   Created for services/applications.
   Examples:
   nginx
   mysql
   These accounts generally aren't intended for interactive login.
   A service account may use:
   /usr/sbin/nologin
   to prevent interactive login.
**5. Important Linux Account Files**
   /etc/passwd
   Contains account information such as:
   username
   UID
   GID
   home directory
   login shell
   /etc/shadow
   Contains:
   password hashes
   password ageing information
   It should only be readable by privileged users.
   /etc/group
   Contains:
   group name
   GID
   group members
   /etc/sudoers
   Defines:
   Who can execute which commands using sudo.

Edit it using:
visudo
Never casually edit /etc/sudoers with a normal editor.
**6. Checking Your Identity**
   whoami
   Shows your current username.
   whoami
   Example:
   ec2-user
   id
   Shows UID, GID and groups:
   id
   groups
   Shows your group memberships:
   groups
   Quick memory
   whoami → Who am I?
   id     → Who am I + UID/GID/groups?
   groups → Which groups am I in?
**7. Why Groups Exist**
   Imagine 20 developers need access to:
   /opt/project
   Bad approach:
   Give permission individually to 20 users
   Better:
   developers group
   ↓
   /opt/project
   ↓
   All developers get required access
   So:
   Groups allow permissions to be assigned to a role/team instead of repeatedly configuring individual users.

This becomes extremely useful in real DevOps environments.
**8. Slide 17 — Creating & Managing Users**
   Create a user
   sudo useradd -m -s /bin/bash priya
   Breakdown:
   sudo       → administrative privilege
   useradd    → create user
   -m         → create home directory
   -s         → specify login shell
   /bin/bash  → Bash shell
   priya      → username
   Set password
   sudo passwd priya
   Create a group
   sudo groupadd developers
   Add user to group
   sudo usermod -aG developers priya
   🔥 Important: -aG
   -a → append
   -G → supplementary group
   Use:
   usermod -aG group user
   because without -a, existing supplementary groups can be replaced.
   Delete user
   sudo userdel -r priya
   -r also removes the user's home directory.
   Check password ageing
   sudo chage -l priya
   Switch user
   su - priya
**9. Typical User Onboarding**
   A real-world sequence:
   sudo useradd -m -s /bin/bash priya

sudo passwd priya

sudo groupadd developers

sudo usermod -aG developers priya

id priya
Conceptually:
Create user
↓
Create/set credentials
↓
Assign role/group
↓
Verify access
Professional habit
Don't just configure access.
Verify it as the actual user.
su - priya
Then test what that user can and cannot access.
**10. Slide 18 — Linux File Permissions**
    Linux permissions answer:
    Who can do what with this file/directory?

Run:
ls -l
Example:
-rwxr-xr-- 1 priya developers 4096 deploy.sh
Break it down:
-    rwx    r-x    r--
     │     │      │      │
     │     │      │      └── Others
     │     │      └───────── Group
     │     └──────────────── Owner
     └────────────────────── File type
**11. File Type**
    First character:
- → regular file
  d → directory
  l → symbolic link
  Example:
  -rwxr-xr--
- means regular file.
**12. Permission Groups**
    Linux applies permissions to three categories:
    u → user/owner
    g → group
    o → others
    And three permissions:
    r → read
    w → write
    x → execute
    Therefore:
    Owner | Group | Others
    rwx  |  r-x  |  r--
**13. Permission Meaning**
    For a file
    r → read contents
    w → modify contents
    x → execute as a program
    For a directory
    This is VERY important.
    r → list directory contents
    w → create/delete entries inside
    x → enter/traverse directory
    So x on a directory does not mean "execute the directory."
    It means:
    You can traverse/access it with commands such as cd.

**14. Numeric Permissions**
    Each permission has a value:
    r = 4
    w = 2
    x = 1
    Therefore:
    rwx = 4 + 2 + 1 = 7
    rw- = 4 + 2     = 6
    r-x = 4 + 1     = 5
    r-- = 4         = 4
    --- = 0
**15. chmod**
    chmod means:
    Change file permissions.

755
chmod 755 deploy.sh
Means:
Owner  → rwx = 7
Group  → r-x = 5
Others → r-x = 5
So:
rwxr-xr-x
644
chmod 644 index.html
Means:
Owner  → rw-
Group  → r--
Others → r--
600
chmod 600 ~/.ssh/id_rsa
Means:
Owner  → rw-
Group  → ---
Others → ---
This is commonly used for private SSH keys.
700
chmod 700 ~/.ssh
Means:
Owner  → rwx
Group  → ---
Others → ---
16. **chmod 777 ⚠️**
    chmod 777 file
    means:
    Owner  → rwx
    Group  → rwx
    Others → rwx
    Everyone gets full permissions.
   
17. **Professional rule
    Don't use chmod 777 as a solution to a permissions problem.

Instead ask:**
Who actually needs access?
Owner?
Group?
Others?
Then give only the required permission.
This is the principle of least privilege.

17. **Symbolic chmod**
    Instead of numbers:
    chmod u+x script.sh
    Give owner execute permission.
    chmod g-w file.txt
    Remove group write permission.
    chmod o= file.txt
    Remove all permissions from others.
    chmod a+r file.txt
    Give read permission to everyone.

18. **Ownership**
    Permissions aren't only about chmod.
    Files also have:
    Owner
    Group
    Change owner:
    sudo chown priya file.txt
    Change owner + group:
    sudo chown priya:developers file.txt
    Change group only:
    sudo chgrp developers file.txt
19. **umask**
    umask 022
    umask controls the default permissions removed when new files/directories are created.
    Think:
    Default permissions
    -
    umask
    ↓
    Actual permissions
    You don't normally memorize this by itself; understand that umask influences default permissions.
20. **Slide 20 — Special Permissions**
    There are three special permission bits.
    SUID
    4xxx
    Example:
    chmod u+s program
    The program can run with the file owner's privileges.
    passwd is a classic example.
    Security-sensitive.
    SGID
    2xxx
    On a directory:
    New files inherit the directory's group.

Very useful for shared team directories.
Example:
/opt/projectx
↓
projectx group
↓
new files inherit projectx group
Sticky Bit
1xxx
Used for shared directories.
Example:
/tmp
It prevents one user from deleting another user's files in the shared directory.
**21. Permission Patterns to Remember**
    These are worth memorizing:
    chmod 600 ~/.ssh/id_rsa
    Private SSH key.
    chmod 700 ~/.ssh
    SSH directory.
    chmod 644 index.html
    Normal web content.
    chmod 755 deploy.sh
    Executable script.
22. **Slide 21 — SSH**
    What is SSH?
    SSH = Secure Shell
    SSH allows you to securely connect to another machine and operate its terminal remotely.
    Example:
    ssh priya@192.168.1.20
    Architecture:
    Your laptop
    │
    │ SSH
    │
    ↓
    Remote Linux Server
    In cloud environments, this is fundamental because you normally don't physically access the server.
23. **SSH Components**
    Client
    Your machine runs:
    ssh
    Server
    Remote machine runs:
    sshd
    SSH server normally listens on:
    TCP 22
    Authentication
    Can use:
    Password
    OR
    Public/Private Key
    Key-based authentication is preferred.
    Encryption
    After the SSH handshake, communication is encrypted.
    So your:
    commands
    keystrokes
    data
    are protected in transit.
24. **Useful SSH Commands**
    Normal connection
    ssh user@host
    Custom port
    ssh -p 2222 user@host
    Specific private key
    ssh -i ~/.ssh/id_ed25519 user@host
    Exit
    exit
25. **AWS Connection**
    When connecting to an EC2 Linux instance, you might use:
    ssh -i my-key.pem ec2-user@PUBLIC_IP
    Conceptually:
    Your computer
    ↓
    SSH
    ↓
    Internet
    ↓
    AWS Security Group
    ↓
    EC2 :22
    ↓
    sshd
    ↓
    Linux user
    
26. If SSH doesn't work, don't immediately assume SSH itself is broken.
    Check:
1. Is EC2 running?
2. Is port 22 allowed by Security Group?
3. Is sshd running?
4. Is username correct?
5. Is the private key correct?
6. Are key permissions correct?
26. Slide 22 — SSH Key Authentication
    SSH key authentication uses two keys:
    Private Key 🔐
    Public Key 🔓
    Private key
    Stays on your machine.
    Never share it.
    Never:
    Email it
    Commit it to GitHub
    Paste it into chat
    Upload it
    Public key
    Can be placed on servers.
    The server stores it in:
    ~/.ssh/authorized_keys
27. How SSH Keys Work
    Your machine
    │
    │ Private key
    │
    ↓
    SSH authentication
    │
    ↓
    Remote server
    │
    │ Public key
    ↓
    authorized_keys
    The important idea:
    The private key proves that you are the person who owns the corresponding public key.

The private key itself isn't sent to the server as a password.
28. Generate SSH Key
    ssh-keygen -t ed25519 -C "priya@laptop"
    This creates a key pair.
    Conceptually:
    id_ed25519
    ↓
    Private key

id_ed25519.pub
↓
Public key
29. Copy Public Key
    ssh-copy-id priya@192.168.1.20
    This adds the public key to:
    ~/.ssh/authorized_keys
    Then:
    ssh priya@192.168.1.20
    can authenticate using the key.
30. Why Keys Are Better
    Key authentication:
- Avoids reusable passwords
- Helps resist password brute-force attacks
- Can be revoked by removing a key from authorized_keys
- Works well for automation
- Can be protected with a passphrase
  This is especially important in:
  DevOps
  CI/CD
  Automation
  Cloud
  Infrastructure management
31. Slide 23 — File Transfer
    SSH isn't only for terminal access.
    scp
    Copy file to remote server:
    scp file.txt user@host:/tmp/
    Copy file from remote server:
    scp user@host:/tmp/a.log .
    Copy directory:
    scp -r dir/ user@host:/opt/
    rsync
    rsync -avz dir/ host:/opt/
    Useful for synchronizing directories and transferring only differences.
    sftp
    sftp user@host
    Interactive file transfer session.
32. SSH Server Hardening
    Configuration file:
    /etc/ssh/sshd_config
    Important settings from the slide:
    PermitRootLogin no
    PasswordAuthentication no
    PubkeyAuthentication yes
    Conceptually:
    ❌ Direct root login
    ❌ Password authentication
    ✅ Public-key authentication
    The slide also shows:
    AllowUsers priya rochak
    to restrict SSH login to specified users.
33. SSH Troubleshooting
    If SSH connection is refused/fails:
    Check SSH service
    systemctl status sshd
    Check firewall/security group
    For AWS:
    Security Group
    ↓
    TCP 22
    ↓
    Your IP allowed?
    Check permissions
    chmod 600 ~/.ssh/id_rsa
    chmod 700 ~/.ssh
    Verbose SSH
    ssh -v user@host
    This shows where the connection/authentication process is failing.
34. 🔥 Professional SSH Rule
    The PPT gives an excellent production practice:
    Keep one SSH session open before changing SSH configuration.

For example, you're connected through:
Session 1
You modify:
/etc/ssh/sshd_config
Open:
Session 2
and test it before closing Session 1.
Why?
Because a bad SSH configuration can lock you out of the server.
This is a very important production habit.
35. Slide 24 — vi/vim
    vi is a terminal-based text editor.
    Why should a DevOps engineer know it?
    Because production/minimal Linux systems may not have graphical editors.
    You may only have:
    vi
    or:
    vim
36. vi Modes
    The most important concept:
    ┌──────────────┐
    │ Command Mode │
    └──────┬───────┘
    │
    i/a/o
    ↓
    ┌──────────────┐
    │ Insert Mode  │
    └──────┬───────┘
    │
    Esc
    ↓
    Command Mode
    │
    :
    ↓
    Last-line mode
    Command mode
    Default mode.
    Keys are commands.
    Insert mode
    Used to type/edit text.
    Enter with:
    i
    a
    o
    Last-line mode
    Press:
    :
    Then enter commands such as:
    :w
    :q
    :wq
37. Essential vi Commands
    Command	Meaning
    i	Insert before cursor
    a	Insert after cursor
    o	New line below
    Esc	Return to command mode
    dd	Delete current line
    yy	Copy current line
    p	Paste
    u	Undo
    :w	Save
    :q	Quit
    :wq	Save + quit
    :q!	Quit without saving
    /word	Search
    :set number	Show line numbers


38. ⭐ Most Important vi Escape
    If you get stuck inside vi:
    Esc
    :q!
    Enter
    Meaning:
    Esc   → command mode
    :q!   → quit without saving
    This is your escape hatch.
    Memorize this first.
39. Slide 25 — Hands-on Lab 2
    The lab combines everything from this session.
    Create
    Create user: trainee
    Create group: projectx
    Add trainee to projectx
    Secure
    Create:
    /opt/projectx
    Set group ownership and permissions so:
    Owner/group → required access
    Others      → no access
    Connect
    Generate SSH keys:
    ssh-keygen
    Copy public key:
    ssh-copy-id
    Connect without password.
    Edit
    Use:
    vi file.txt
    Add three lines, save, reopen, and delete one line.
    Finally:
    su - trainee
    and verify what the user can actually access.
    The key lesson:
    Configuration is not enough — test access as the user who will actually use it.

40. Slide 26 — Introduction to Shell Scripting
    This slide starts Session 03.
    The next session is:
    Shell Scripting
    The purpose is to turn commands you already know into repeatable automation.
    Topics:
    Variables
    ↓
    Input & arguments
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
    The important idea from the slide is:
    Anything you repeatedly type can potentially become a script.

🔥 DevOps Connection — Why This Session Matters
This entire session is foundational for your DevOps career.
Linux
│
├── Users & Groups
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
│   Automation / CI/CD
│
├── scp / rsync
│      ↓
│   File deployment
│
├── vi
│      ↓
│   Server configuration
│
└── Shell scripting
↓
Automation
Later, these concepts connect directly to:
AWS EC2
Docker
Jenkins
GitHub Actions
Terraform
Ansible
Kubernetes
CI/CD
SRE troubleshooting
🧠 Must-Remember Cheat Sheet
# Identity
whoami
id
groups

# User management
sudo useradd -m -s /bin/bash user
sudo passwd user
sudo groupadd developers
sudo usermod -aG developers user
su - user

# Permissions
ls -l
chmod 755 file
chmod 644 file
chmod 600 file
chmod 700 directory

# Ownership
chown user file
chown user:group file
chgrp group file

# SSH
ssh user@host
ssh -i key.pem user@host
ssh -p 2222 user@host

# SSH keys
ssh-keygen -t ed25519
ssh-copy-id user@host

# File transfer
scp file user@host:/tmp/
scp user@host:/tmp/file .
rsync -avz dir/ host:/opt/

# SSH troubleshooting
systemctl status sshd
ssh -v user@host

# vi
i       → insert
Esc     → command mode
dd      → delete line
yy      → copy line
p       → paste
u       → undo
:w      → save
:wq     → save & quit
:q!     → quit without saving

