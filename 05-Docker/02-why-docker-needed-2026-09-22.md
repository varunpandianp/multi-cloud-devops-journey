# Virtualization, Hypervisor and Why Docker Is Needed

## 1. Why Can't We Run Every Application Directly on One Server?

A single physical server can run multiple applications, but problems can occur when different applications require different:

- Operating systems
- Runtime versions
- Libraries
- Dependencies
- System packages
- Configurations

For example:

```text
Application A → Requires Windows
Application B → Requires Linux
Application C → Requires Python 3.9
Application D → Requires Python 3.12
```

If all applications are installed directly on the same operating system, dependency and environment conflicts can occur.

---

# 2. Example Problem

Suppose we have one physical server:

```text
Physical Server
       |
       ↓
    Linux OS
       |
 ┌─────┼─────┐
 ↓     ↓     ↓
App A App B App C
```

Now suppose:

```text
App A → Python 3.9
App B → Python 3.12
App C → Requires Windows
```

App C cannot simply run on the Linux operating system.

Even when applications use the same OS, different versions of libraries and runtimes can create dependency conflicts.

Example:

```text
App A → Requires Library X version 1.0
App B → Requires Library X version 2.0
```

Installing one version globally may break the other application.

---

# 3. What Is Virtualization?

**Virtualization** allows one physical server to be divided into multiple isolated virtual machines.

Each virtual machine can have its own:

- Operating system
- Libraries
- Dependencies
- Applications
- Configuration

Example:

```text
                 Physical Server
                        |
                        ↓
                    Hypervisor
                  /           \
                 /             \
                ↓               ↓
              VM 1            VM 2
            Windows           Linux
               |                |
             App A            App B
```

Now:

```text
VM 1 → Windows → App A
VM 2 → Linux   → App B
```

Both applications can run on the same physical server because they have separate virtual machines.

---

# 4. What Is a Hypervisor?

A **hypervisor** is software that creates and manages virtual machines.

It allows multiple virtual machines to share the physical server's:

- CPU
- Memory
- Storage
- Network

Example:

```text
Physical Server
       |
       ↓
   Hypervisor
       |
 ┌─────┴─────┐
 ↓           ↓
VM 1        VM 2
Windows     Linux
```

The hypervisor manages the resources given to each VM.

For example:

```text
Physical Server
CPU     → 16 Cores
RAM     → 64 GB
Storage → 1 TB

        ↓

Hypervisor

        ↓

VM 1 → 8 CPU + 32 GB RAM + Windows
VM 2 → 8 CPU + 32 GB RAM + Linux
```

---

# 5. Why Do We Need Virtualization?

Virtualization is useful when different applications require different operating systems or isolated environments.

Example:

```text
Physical Server
       ↓
   Hypervisor
       ↓
 ┌──────────────┬──────────────┐
 │ VM 1         │ VM 2         │
 │ Windows      │ Linux        │
 │              │              │
 │ App A        │ App B        │
 └──────────────┴──────────────┘
```

Without virtualization:

```text
One Physical Server
       ↓
One Operating System
       ↓
Applications
```

With virtualization:

```text
One Physical Server
       ↓
Hypervisor
       ↓
Multiple Virtual Machines
       ↓
Different Operating Systems
       ↓
Applications
```

---

# 6. What Problem Does Virtualization Solve?

Virtualization mainly solves the **operating-system-level isolation problem**.

For example:

```text
Application A → Windows
Application B → Linux
```

We can create:

```text
VM 1 → Windows → Application A
VM 2 → Linux   → Application B
```

Therefore, both applications can run on the same physical server.

---

# 7. But Virtual Machines Have Overhead

Each VM requires a complete operating system.

For example:

```text
Physical Server
       ↓
Hypervisor
       ↓
VM 1
 ├── Guest OS
 ├── Libraries
 ├── Dependencies
 └── Application

VM 2
 ├── Guest OS
 ├── Libraries
 ├── Dependencies
 └── Application
```

If we have many applications, running a separate VM for every application can consume significant:

- CPU
- Memory
- Storage
- Startup time

This leads to the need for a lighter form of isolation.

---

# 8. Docker and Containers

Docker uses **containers** to isolate applications and their dependencies.

Example:

```text
Physical Server
       ↓
Host Operating System
       ↓
Docker Engine
       ↓
 ┌──────────┬──────────┬──────────┐
 ↓          ↓          ↓
App A      App B      App C
Python3.9  Python3.12 Node.js
```

Containers share the host operating system kernel.

They package the application together with its required dependencies.

---

# 9. Virtual Machine vs Container

## Virtual Machine

```text
Physical Server
       ↓
Hypervisor
       ↓
VM
       ↓
Guest OS
       ↓
Application
```

Each VM has its own complete operating system.

## Container

```text
Physical Server
       ↓
Host OS
       ↓
Docker Engine
       ↓
Container
       ↓
Application + Dependencies
```

Containers share the host OS kernel.

---

# 10. Simple Comparison

```text
Virtual Machine:

Server
  ↓
Hypervisor
  ↓
Windows VM → App A
Linux VM   → App B
```

```text
Docker:

Server
  ↓
Linux Host
  ↓
Docker
  ↓
Container → App A + Dependencies
Container → App B + Dependencies
Container → App C + Dependencies
```

---

# 11. Important Difference

### Virtualization

Virtualization isolates the **entire operating system**.

```text
VM = Application + Dependencies + Complete Guest OS
```

### Containerization

Containerization isolates the **application and its dependencies**.

```text
Container = Application + Dependencies
```

Containers share the host operating system kernel.

---

# 12. Important Example

Suppose we have:

```text
App A → Windows
App B → Linux
```

Virtual machines are suitable:

```text
Physical Server
       ↓
Hypervisor
       ↓
 ┌──────────────┬──────────────┐
 │ Windows VM   │ Linux VM     │
 │ App A        │ App B        │
 └──────────────┴──────────────┘
```

Now suppose:

```text
App A → Linux + Python 3.9
App B → Linux + Python 3.12
App C → Linux + Node.js
```

Containers can provide lightweight isolation:

```text
Linux Host
    ↓
Docker
    ↓
 ┌────────────┬────────────┬────────────┐
 │ Container  │ Container  │ Container  │
 │ App A      │ App B      │ App C      │
 │ Python3.9  │ Python3.12 │ Node.js    │
 └────────────┴────────────┴────────────┘
```

---

# 13. Important Concept to Remember

The main reason for virtualization is **not simply that a server cannot run multiple applications**.

A single server can run many applications.

The problem is:

> Different applications may require different operating systems, runtimes, libraries, dependencies, and configurations.

Virtualization provides **isolated virtual machines with their own operating systems**.

Containerization provides **lighter application-level isolation while sharing the host kernel**.

---

# 14. Final Mental Model

```text
                    Physical Server
                           |
            ┌──────────────┴──────────────┐
            ↓                             ↓
      Virtualization                Containerization
            ↓                             ↓
       Hypervisor                    Docker Engine
            ↓                             ↓
    Multiple Virtual Machines       Multiple Containers
            ↓                             ↓
   Each has its own OS          Share Host OS Kernel
            ↓                             ↓
      Heavyweight                    Lightweight
```

## Easy Way to Remember

```text
Virtual Machine
→ Virtualize the Operating System

Container
→ Isolate the Application and Dependencies
```

### One-Line Summary

> **Virtualization allows multiple operating systems to run on one physical server using a hypervisor, while Docker containers provide lightweight application isolation by sharing the host operating system kernel.**