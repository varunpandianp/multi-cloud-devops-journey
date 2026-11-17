# Linux Fundamentals — Study Notes

**Date:** 01-09-2026  
**Session:** 1  
**Topic:** Linux Overview, Architecture, and Basic Commands

---

## 1. What Is Linux?

- Linux is a free and open-source operating system kernel.
- Linus Torvalds originally released Linux in 1991.
- Linux combined with system tools, libraries, and a shell forms a complete operating system.
- Linux is widely used in:
    - Servers
    - Cloud instances
    - Containers
    - Routers
    - Embedded systems

---

## 2. Why Linux Is Important for DevOps

```text
DevOps
   ↓
Linux Servers
   ↓
Cloud Instances
   ↓
Containers
   ↓
CI/CD and Automation
```

Linux is the working environment for many servers, containers, and cloud instances.

---

## 3. Linux Architecture

```text
Applications and Utilities
           ↓
         Shell
           ↓
        Kernel
           ↓
       Hardware
```

### Kernel

The kernel is the core of Linux. It manages:

- Processes
- Memory
- Filesystems
- Devices
- Networking
- System calls

### Shell

The shell is a command interpreter.

Example:

```bash
ls
```

Command flow:

```text
User
   ↓
Shell
   ↓
System Call
   ↓
Kernel
   ↓
Filesystem
   ↓
Result
   ↓
Terminal
```

**Important:** Shell, kernel, and Linux operating system are not the same thing.

---

## 4. Linux Distributions

A Linux distribution packages the Linux kernel with tools, package managers, configuration, and support.

### Debian Family

```text
Debian
Ubuntu
Linux Mint
```

Package management:

```bash
apt
dpkg
```

### Red Hat Family

```text
RHEL
Rocky
Fedora
CentOS Stream
```

Package management:

```bash
dnf
yum
rpm
```

### Other Important Distributions

```text
Amazon Linux → AWS-optimized
Alpine       → Small and common in containers
SUSE         → Uses zypper
Arch         → Rolling release
```

---

## 5. Linux Filesystem

Linux uses one filesystem tree starting from `/`.

| Directory | Purpose |
|---|---|
| `/` | Root of the filesystem |
| `/home` | User home directories |
| `/root` | Root user’s home directory |
| `/etc` | Configuration files |
| `/var` | Logs, cache, and variable data |
| `/tmp` | Temporary files |
| `/usr` | Programs, libraries, and documentation |
| `/opt` | Optional or third-party software |
| `/dev` | Device files |
| `/proc` | Kernel and process information |
| `/sys` | Devices and drivers |

### Important for DevOps

```text
/etc      → Configuration
/var/log  → Logs
/home     → User files
/opt      → Optional or third-party software
```

Linux paths are case-sensitive:

```text
/etc/hosts
```

and

```text
/etc/Hosts
```

are different paths.

---

## 6. Anatomy of a Linux Command

Example:

```bash
ls -lh /var/log
```

```text
ls        → Command
-lh       → Options
/var/log  → Argument
```

### Important Shell Habits

```text
Tab       → Autocomplete
Up Arrow  → Command history
$         → Normal user
#         → Root user
~         → Current user’s home directory
```

Commands and filenames are case-sensitive.

---

## 7. Navigation Commands

```bash
pwd
```

Shows the current directory.

```bash
ls
```

Lists files and directories.

```bash
ls -l
```

Shows a long listing.

```bash
ls -a
```

Shows hidden files.

```bash
ls -lh
```

Shows a long listing with human-readable sizes.

```bash
cd /var/log
```

Moves using an absolute path.

```bash
cd logs
```

Moves using a relative path.

```bash
cd ..
```

Moves to the parent directory.

```bash
cd ~
```

Moves to the home directory.

```bash
cd -
```

Returns to the previous directory.

### Absolute and Relative Paths

**Absolute path:**

```text
/var/log
```

Starts from `/`, the root filesystem.

**Relative path:**

```text
logs
```

Depends on the current location.

---

## 8. Creating, Copying, and Removing Files

```bash
touch file.txt
```

Creates an empty file.

```bash
mkdir project
```

Creates a directory.

```bash
mkdir -p a/b/c
```

Creates nested directories.

```bash
cp src.txt dst.txt
```

Copies a file.

```bash
cp -r dir1 dir2
```

Copies a directory.

```bash
mv old.txt new.txt
```

Renames or moves a file.

```bash
rm file.txt
```

Deletes a file.

```bash
rm -r dir
```

Deletes a directory recursively.

```bash
rmdir dir
```

Removes an empty directory.

> ⚠️ Linux has no recycle bin for `rm`. Be especially careful with `rm -rf`.

---

## 9. Reading Files and Logs

```bash
cat file.txt
```

Displays the complete file.

```bash
less file.txt
```

Reads a large file page by page.

```bash
head file.txt
```

Displays the first 10 lines.

```bash
tail file.txt
```

Displays the last 10 lines.

```bash
tail -f app.log
```

Follows a log file in real time.

```bash
wc -l file.txt
```

Counts lines in a file.

```bash
diff a.txt b.txt
```

Shows differences between two files.

### Important DevOps Command

```bash
tail -f /var/log/syslog
```

Used to monitor logs while troubleshooting.

---

## 10. Search, Pipes, and Redirection

### Search Commands

```bash
grep "error" app.log
```

Searches for `error`.

```bash
grep -i "error" app.log
```

Searches without case sensitivity.

```bash
grep -r "TODO" .
```

Searches recursively from the current directory.

```bash
find /var -name "*.log"
```

Finds `.log` files under `/var`.

```bash
which python3
```

Shows the executable being used.

### Pipes

A pipe sends the output of one command to another command.

```bash
cat app.log | grep ERROR | wc -l
```

```text
Command 1
   ↓ output
Command 2
   ↓ output
Command 3
```

### Redirection

```bash
cmd > out.txt
```

Writes output to a file and overwrites existing content.

```bash
cmd >> out.txt
```

Appends output to a file.

```bash
cmd < in.txt
```

Reads input from a file.

```bash
cmd 2> err.txt
```

Redirects errors to a file.

```bash
cmd > all.txt 2>&1
```

Redirects both normal output and errors to one file.

---

## 11. Processes, Resources, and Services

### Processes

```bash
ps aux
```

Shows running processes.

```bash
top
```

Shows live CPU, memory, and process information.

```bash
kill 1234
```

Requests a process to terminate.

```bash
kill -9 1234
```

Forcefully terminates a process. Use this as a last resort.

### Resources

```bash
df -h
```

Checks disk space.

```bash
du -sh /var/log
```

Checks directory size.

```bash
free -h
```

Checks memory usage.

```bash
uptime
```

Shows system uptime and load information.

---

## 12. Services with `systemctl`

Example: Nginx

```bash
systemctl status nginx
```

Checks the status of the service.

```bash
systemctl start nginx
```

Starts the service.

```bash
systemctl stop nginx
```

Stops the service.

```bash
systemctl restart nginx
```

Restarts the service.

```bash
systemctl enable nginx
```

Configures the service to start automatically after reboot.

```bash
journalctl -u nginx
```

Shows logs for the Nginx service.

### Important Difference

```text
start
   ↓
Starts the service now

enable
   ↓
Starts the service automatically after reboot
```

---

## 13. Basic Troubleshooting Flow

```text
Is the service running?
   ↓
systemctl status

What does the service say?
   ↓
journalctl

Is disk full?
   ↓
df -h

Is memory exhausted?
   ↓
free -h

What is consuming CPU?
   ↓
top
```

This is a useful first-line troubleshooting flow for DevOps and SRE work.

---

# Quick Revision

```text
Linux = Kernel + system tools + shell
```

```text
Architecture:
Application → Shell → Kernel → Hardware
```

```text
Filesystem:
/etc      → Configuration
/var/log  → Logs
/home     → User files
/opt      → Optional software
```

```text
Navigation:
pwd → ls → cd
```

```text
Files:
touch → mkdir → cp → mv → rm
```

```text
Logs:
cat → less → head → tail -f
```

```text
Search:
grep → find → which
```

```text
Automation:
Pipes | + Redirection >
```

```text
Troubleshooting:
systemctl → journalctl → df → free → top
```