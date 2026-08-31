**28-08-2026 // OS Linux operating System**


Topics Covered

Operating System (OS) Basics

Linux File System

Linux Directories

Basic Linux Commands

What I Learned

Operating System Basics

Understood what an Operating System is and why it is required.

Learned that the OS acts as an interface between hardware, applications, and users.

Understood the basic responsibilities of an OS:

Process management

Memory management

File management

Device management

User and permission management

Networking

Linux File System

Learned that Linux uses a single hierarchical file system starting from the root directory:

/

Unlike Windows, Linux does not use drive letters such as C: or D: in the same way.

Basic structure:

/
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── opt
├── proc
├── root
├── tmp
├── usr
└── var

Important Directories

/ → Root of the entire Linux filesystem

/home → Home directories of normal users

/root → Home directory of the root user

/etc → System and application configuration files

/var → Variable data such as logs and application data

/tmp → Temporary files

/usr → User-space programs, libraries and shared resources

/opt → Optional/additional software

/dev → Device files

/proc → Virtual filesystem exposing process and kernel information

/boot → Files required for booting Linux

Basic Linux Commands

Practiced basic commands for navigating and working with the filesystem:

pwd
ls
cd
mkdir
touch
cp
mv
rm
cat
less
head
tail
find

Also learned commands for understanding the current system/environment:

whoami
hostname
date
uname

Important Concepts

Absolute path starts from /.

/home/user/project

Relative path starts from the current directory.

./project

. represents the current directory.

.. represents the parent directory.

/ represents the root directory.

DevOps Relevance

Linux is fundamental to DevOps and SRE because many production workloads run on Linux.

Understanding the filesystem and basic commands is required for:

Managing EC2 servers

Working with Docker containers

Kubernetes nodes and containers

Managing application configuration

Reading application/system logs

Troubleshooting production issues

Writing shell scripts

Managing permissions

Working with CI/CD systems

Managing Nginx and other services

Key Takeaway

Linux organizes everything under a single hierarchical filesystem starting at /. Understanding directories, paths, and basic commands is the foundation for working with Linux servers in DevOps.

Next Topics to Learn