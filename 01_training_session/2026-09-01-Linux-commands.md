Linux Fundamentals — Study Notes
📅 01-09-2026 — Session 1
Topic: Linux Overview, Architecture & Basic Commands
1. What is Linux?
- Linux is a free and open-source operating system kernel.
- Originally released by Linus Torvalds in 1991.
- Linux combined with system tools, libraries and a shell forms a complete operating system.
- Linux is widely used in:
    - Servers
    - Cloud instances
    - Containers
    - Routers
    - Embedded systems
2. Why Linux is important for DevOps
   DevOps
   ↓
   Linux Servers
   ↓
   Cloud Instances
   ↓
   Containers
   ↓
   CI/CD & Automation
   Linux is the working environment for many servers, containers and cloud instances.
3. Linux Architecture
   The basic architecture:
   Applications & Utilities
   ↓
   Shell
   ↓
   Kernel
   ↓
   Hardware
   Kernel
   The kernel is the core of Linux.
   It manages:
- Processes
- Memory
- Filesystems
- Devices
- Networking
- System calls
  Shell
  The shell is a command interpreter.
  Example:
  ls
  Flow:
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
  Important: Shell ≠ Kernel ≠ Linux operating system.
4. Linux Distributions
   A distribution packages the Linux kernel with tools, package managers, configuration and support.
   Debian family
   Debian
   Ubuntu
   Linux Mint
   Package management:
   apt
   dpkg
   Red Hat family
   RHEL
   Rocky
   Fedora
   CentOS Stream
   Package management:
   dnf
   yum
   rpm
   Other important distributions:
   Amazon Linux → AWS optimized
   Alpine       → Small, common in containers
   SUSE         → zypper
   Arch         → Rolling release
5. Linux Filesystem
   Linux uses one filesystem tree starting from /.
   Important directories:
   Directory	Purpose
   /	Root of filesystem
   /home	User home directories
   /root	Root user's home
   /etc	Configuration files
   /var	Logs, cache, variable data
   /tmp	Temporary files
   /usr	Programs, libraries, documentation
   /opt	Optional/third-party software
   /dev	Device files
   /proc	Kernel/process information
   /sys	Devices and drivers


Must remember for DevOps
/etc       → Configuration
/var/log   → Logs
/home      → User files
/opt       → Optional/third-party software
Linux paths are case-sensitive.
/etc/hosts
and
/etc/Hosts
are different paths.
6. Anatomy of a Linux Command
   Example:
   ls -lh /var/log
   ls          → Command
   -lh         → Options
   /var/log    → Argument
   Important shell habits:
   Tab       → Autocomplete
   Up Arrow  → Command history
   $         → Normal user
#         → Root user
~         → Current user's home directory
Commands and filenames are case-sensitive.

**7. Navigation Commands**
   pwd
   Shows current directory.
   ls
   Lists files/directories.
   ls -l
   Long listing.
   ls -a
   Shows hidden files.
   ls -lh
   Long listing with human-readable sizes.
   cd /var/log
   Move using absolute path.
   cd logs
   Move using relative path.
   cd ..
   Move to parent directory.
   cd ~
   Go to home directory.
   cd -
   Return to previous directory.
   Important concept
   Absolute path:
   /var/log
   Starts from /.
   Relative path:
   logs
   Depends on your current location.

**8. Creating, Copying & Removing**
   touch file.txt
   Create an empty file.
   mkdir project
   Create directory.
   mkdir -p a/b/c
   Create nested directories.
   cp src.txt dst.txt
   Copy file.
   cp -r dir1 dir2
   Copy directory.
   mv old.txt new.txt
   Rename/move.
   rm file.txt
   Delete file.
   rm -r dir
   Delete directory recursively.
   rmdir dir
   Remove empty directory.
   ⚠️ Important
   Linux has no recycle bin for rm.
   Be especially careful with:
   rm -rf

**9. Reading Files & Logs ⭐**
   cat file.txt
   Display complete file.
   less file.txt
   Read large files page by page.
   head file.txt
   First 10 lines.
   tail file.txt
   Last 10 lines.
   tail -f app.log
   Follow a log in real time.
   wc -l file.txt
   Count lines.
   diff a.txt b.txt
   Show differences.
   DevOps important command
   tail -f /var/log/syslog
   Used to watch logs while troubleshooting.

**10. Search, Pipes & Redirection ⭐**
    Search
    grep "error" app.log
    Search for error.
    grep -i "error" app.log
    Case-insensitive search.
    grep -r "TODO" .
    Search recursively.
    find /var -name "*.log"
    Find .log files.
    which python3
    Find the executable being used.
    Pipes
    A pipe sends one command's output to another command.
    cat app.log | grep ERROR | wc -l
    Think:
    Command 1
    ↓ output
    Command 2
    ↓ output
    Command 3
    This is one of the most powerful concepts in Linux.
    Redirection
    cmd > out.txt
    Write/overwrite output.
    cmd >> out.txt
    Append output.
    cmd < in.txt
    Read input from file.
    cmd 2> err.txt
    Redirect errors.
    cmd > all.txt 2>&1
    Redirect output + errors.

**11. Processes, Resources & Services**
    Processes
    ps aux
    View running processes.
    top
    Live CPU/memory/process information.
    kill 1234
    Ask process to terminate.
    kill -9 1234
    Force termination — use as a last resort.
    Resources
    df -h
    Check disk space.
    du -sh /var/log
    Check directory size.
    free -h
    Check memory.
    uptime
    System uptime/load information.

**12. Services with systemctl**
    Example: Nginx
    systemctl status nginx
    Check service.
    systemctl start nginx
    Start service.
    systemctl stop nginx
    Stop service.
    systemctl restart nginx
    Restart service.
    systemctl enable nginx
    Start automatically at boot.
    journalctl -u nginx
    View service logs.
    Important distinction
    start
    ↓
    Starts service NOW

enable
↓
Starts service automatically after reboot

**13. Basic Troubleshooting Flow ⭐**
    The slide gives a very useful first-line troubleshooting sequence:
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
This is worth remembering as a DevOps/SRE troubleshooting habit.
🧠 Quick Revision
Linux = OS/kernel + system tools + shell

Architecture:
Application → Shell → Kernel → Hardware

Filesystem:
/etc      → Configuration
/var/log  → Logs
/home     → User files
/opt      → Optional software

Navigation:
pwd → ls → cd

Files:
touch → mkdir → cp → mv → rm

Logs:
cat → less → head → tail -f

Search:
grep → find → which

Automation power:
Pipes | + Redirection >

Troubleshooting:
systemctl → journalctl → df → free → top