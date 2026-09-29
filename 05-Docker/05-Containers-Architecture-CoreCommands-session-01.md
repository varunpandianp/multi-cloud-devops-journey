# Module 07 — Containerization with Docker

## 📚 Course / Module

**Module:** Containerization with Docker  
**Session:** Session 1 — Containers, Architecture and Core Commands  
**Duration:** 2 Hours  
**Slides Covered:** 3–19

---

# 🎯 Learning Objectives

By the end of this session, I should be able to:

- Understand why containers exist.
- Explain the "It works on my machine" problem.
- Understand virtualization vs containerization.
- Compare virtual machines and containers.
- Explain what a Docker container actually is.
- Understand namespaces, cgroups, and union filesystems.
- Understand Docker architecture.
- Understand Docker Client, Docker Daemon, Images, Containers, and Registries.
- Understand Docker Hub and image references.
- Understand Docker image layers.
- Understand image tags and digests.
- Install and verify Docker.
- Understand the Docker container lifecycle.
- Work with Docker images.
- Run and manage containers.
- Inspect containers and images.
- View container logs.
- Execute commands inside containers.
- Understand attached, detached, and interactive modes.
- Understand restart policies.
- Understand Docker port binding.
- Run and test Nginx containers.

---

# Session 1 — Containers, Architecture and Core Commands

## 📌 Topics Covered

1. Why "It Works on My Machine" Happens
2. From Physical Servers to Containers
3. Virtual Machines vs Containers
4. What a Container Actually Is
5. Namespaces
6. Control Groups — cgroups
7. Union Filesystem
8. Docker Architecture
9. Docker Client
10. Docker Daemon
11. Docker Images
12. Docker Containers
13. Docker Registry
14. Docker Hub
15. Image References
16. Image Layers
17. Image Tags and Digests
18. Installing Docker
19. Docker Post-Installation and Verification
20. Container Lifecycle
21. Working with Docker Images
22. Running and Managing Containers
23. Looking Inside a Running Container
24. Docker Debugging Sequence
25. Attached Mode
26. Detached Mode
27. Interactive Mode
28. Restart Policies
29. Port Binding
30. Hands-on Lab 1

---

# 1. Why "It Works on My Machine" Happens

## What is the problem?

An application is never just its source code.

An application depends on:

- Runtime
- Libraries
- System packages
- Configuration
- Environment variables
- Specific software versions

Every machine can have a slightly different environment.

Therefore, an application may work on the developer's machine but fail on another machine.

### Example

    Developer Machine
            ↓
        Java 17

    Test Server
            ↓
        Java 11

    Production Server
            ↓
        Java 17.0.2 + undocumented patch

The application may behave differently because the environments are different.

---

## 1.1 Version Drift

### What is it?

Version drift happens when different environments have different versions of software.

### Example

    Developer  → Java 17
    Testing    → Java 11
    Production → Java 17.0.2

The developer tests against one version while production uses another version.

This can cause unexpected application behavior.

---

## 1.2 Dependency Conflicts

Two applications running on the same server may require different versions of the same library.

### Example

    Application A
         ↓
    Library Version 1

    Application B
         ↓
    Library Version 2

If only one version is installed globally, one of the applications may fail.

---

## 1.3 Snowflake Servers

### What is a Snowflake Server?

A server that has been configured manually over many years can become difficult to reproduce.

Example:

    Server
     ├── Manually installed packages
     ├── Manual configuration changes
     ├── Different software versions
     ├── Old patches
     └── Unknown dependencies

Nobody may know exactly how the server was configured.

Therefore:

- Nobody can easily rebuild it.
- Nobody wants to make risky changes.
- Troubleshooting becomes difficult.

This type of server is commonly called a **Snowflake Server**.

---

## 1.4 Slow Onboarding

A new developer may need to spend significant time installing:

- Database
- Cache
- Runtime
- Libraries
- System packages
- Correct software versions
- Configuration

The presentation gives an example of a new developer spending two days installing the correct database, cache, and runtime versions.

Containers help package the required environment and make setup more consistent.

---

## 1.5 Environment Mismatch

An application may pass testing but fail in production.

### Example

    Development → Works
    Testing     → Works
    Staging     → Works
    Production  → Fails

The environments may not actually be identical.

---

## 1.6 Wasteful Isolation

A traditional solution is:

    Application A → Virtual Machine 1
    Application B → Virtual Machine 2
    Application C → Virtual Machine 3

This provides isolation, but every VM carries a complete operating system.

Therefore:

- More memory is required.
- More disk space is required.
- More OS overhead exists.
- Fewer applications can fit on the same host.

---

## How Containers Solve This Problem

A container ships the application together with its environment.

    Application
         +
    Libraries
         +
    Dependencies
         +
    Configuration
         ↓
      Container

The environment becomes less of a variable.

### Key Idea

> A container ships the application together with its environment.

---

# 2. From Physical Servers to Containers

The presentation explains the evolution:

    Physical Servers
           ↓
    Virtual Machines
           ↓
       Containers

---

# 3. Physical Servers

## What is the traditional approach?

One application may run directly on one physical machine.

    Physical Server
          ↓
      Application

### Problems

- Slow to provision
- Expensive
- Hardware can remain idle
- Scaling takes time

The presentation describes physical server provisioning as potentially taking **weeks**.

---

# 4. Virtual Machines

## What is a Virtual Machine?

A hypervisor runs multiple isolated operating systems on one physical host.

    Physical Infrastructure
             ↓
         Hypervisor
        /     |     \
       ↓      ↓      ↓
      VM 1   VM 2   VM 3
       ↓      ↓      ↓
      OS     OS     OS
      App    App    App

Each VM contains:

- Application
- Libraries
- Guest Operating System

### VM Architecture

    VM 1
     ├── Application
     ├── Libraries
     └── Guest OS

    VM 2
     ├── Application
     ├── Libraries
     └── Guest OS

    VM 3
     ├── Application
     ├── Libraries
     └── Guest OS
            ↓
        Hypervisor
            ↓
    Host Operating System
            ↓
       Infrastructure

### Advantages

- Better hardware utilization
- Strong isolation
- Different operating systems can run on the same physical infrastructure

### Disadvantages

Every VM carries a complete guest operating system.

Therefore:

- More memory usage
- More disk usage
- More overhead
- Slower startup

The presentation describes VM startup as typically taking **minutes**.

---

# 5. Containers

Containers allow applications to share the host kernel while remaining isolated from each other.

    Container 1
     ├── Application
     └── Libraries

    Container 2
     ├── Application
     └── Libraries

    Container 3
     ├── Application
     └── Libraries
            ↓
        Docker Engine
            ↓
    Host Operating System
            ↓
       Infrastructure

### Important Point

Containers do not carry a separate guest operating system for every application.

The applications share the host kernel.

The presentation describes container startup as **seconds or less**.

---

# 6. Virtual Machines vs Containers

| Feature | Virtual Machine | Container |
|---|---|---|
| What it virtualizes | Hardware | Operating system |
| Includes | Full guest OS, libraries, application | Application and libraries |
| Typical size | Gigabytes | Megabytes |
| Startup time | Minutes | Seconds or less |
| Isolation | Strong — separate kernels | Process-level — shared kernel |
| Density per host | Tens | Hundreds |
| Best for | Different OSes, strong isolation | Packaging and shipping applications |

---

## Rule of Thumb

Use a VM when:

- You need a different kernel.
- You need a hard security boundary.
- You need a different operating system.

Use a container for application packaging and shipping when that level of isolation is sufficient.

---

# 7. Important Container Architecture Point

Containers do not replace virtual machines completely.

The presentation specifically explains:

> Containers usually run inside VMs in cloud environments.

The important difference is:

    Traditional approach:

    One Application
          ↓
    One Operating System
          ↓
    VM

    Container approach:

    Multiple Applications
          ↓
       Containers
          ↓
    Host Operating System
          ↓
         VM
          ↓
    Cloud Infrastructure

Containers replace the habit of giving every application its own operating system.

---

# 8. What Is a Container Actually?

A container is essentially an ordinary Linux process that the Linux kernel has been instructed to isolate.

There is no separate "container object" in the Linux kernel.

Docker combines Linux kernel features to create the container experience.

The three important concepts introduced in the presentation are:

1. Namespaces
2. Control Groups — cgroups
3. Union Filesystem

---

# 9. Namespaces

## What are Namespaces?

Namespaces control **what a process can see**.

A process inside a container can be isolated from the host and other containers.

Namespaces can isolate things such as:

- Process tree
- Network interfaces
- Hostname
- Mount points
- Users

Inside the container, the application can appear as though it has its own machine.

### Simple Example

    Host
     ├── Process A
     ├── Process B
     └── Container
            └── Process

The container can have its own view of processes.

### Key Idea

> Namespaces control what a process can see.

---

# 10. Control Groups — cgroups

## What are cgroups?

Control groups control **what resources a process can use**.

They can control resources such as:

- CPU
- Memory
- Disk I/O

For example:

    Container
        ↓
    CPU Limit
        ↓
       1 CPU

or:

    Container
        ↓
    Memory Limit
        ↓
      512 MB

Docker options such as:

    --memory

and:

    --cpus

are enforced using cgroups.

### Key Idea

> Namespaces control what a process can see, while cgroups control what a process can use.

---

# 11. Union Filesystem

## What is a Union Filesystem?

Docker images are built from multiple read-only layers.

A thin writable layer is added when a container runs.

Conceptually:

    Writable Container Layer
    ------------------------
    Changes made by container

    Image Layer
    ------------------------
    Application files

    Image Layer
    ------------------------
    Packages

    Image Layer
    ------------------------
    Base Image

The layers are combined to create the filesystem visible to the container.

### Why is this useful?

Many containers can share the same read-only image layers.

For example:

    Nginx Image
          ↓
    ┌─────┼─────┐
    ↓     ↓     ↓
    C1    C2    C3

The image layers are shared.

Each container gets its own writable layer.

---

# 12. Container Is a Process

The presentation demonstrates that a container is fundamentally a process running on the host.

Example command:

    docker run -d --name web nginx

Then on the host:

    ps aux | grep nginx

The Nginx process can be observed from the host.

### Key Idea

> A container is an isolated Linux process with filesystem, network, user, and resource boundaries.

---

# 13. Docker Architecture

Docker consists of several important components:

    Docker Client
          ↓
       REST API
          ↓
    Docker Daemon
          ↓
    ┌─────┼──────────────┐
    ↓     ↓              ↓
Images Containers   Networks/Volumes
↓
Registry
↓
Docker Hub

---

# 14. Docker Client

## What is Docker Client?

The Docker Client is the command-line interface that we interact with.

Example:

    docker build
    docker pull
    docker run

When we type these commands, the Docker Client communicates with the Docker Daemon through the Docker API.

### Important Point

The Docker Client does not perform all Docker operations itself.

It sends commands to the Docker Daemon.

---

# 15. Docker Daemon — dockerd

## What is Docker Daemon?

The Docker Daemon is the component that performs the actual Docker work.

The daemon:

- Builds images
- Runs containers
- Manages networks
- Manages volumes
- Manages Docker resources

The daemon process is commonly called:

    dockerd

### Architecture

    Docker CLI
        ↓
    REST API
        ↓
    dockerd
        ↓
    Docker Resources

---

# 16. Docker Client Can Talk to Another Machine

The Docker Client can communicate with a Docker Daemon running on another machine.

Conceptually:

    Developer Machine
          ↓
      Docker CLI
          ↓
       Network
          ↓
    Remote Docker Daemon

This is useful for remote Docker management.

---

# 17. Docker Images

## What is a Docker Image?

A Docker image is a **read-only template** used to create containers.

Example:

    Nginx Image
         ↓
    docker run
         ↓
    Nginx Container

An image contains what is required to create the container environment.

### Simple Mental Model

    Image
      ↓
    Blueprint / Template

    Container
      ↓
    Instance created from the image

---

# 18. Docker Containers

A container is created from an image.

Example:

    nginx Image
         ↓
    Container 1
    Container 2
    Container 3

One image can be used to create many containers.

Each container can have its own state.

---

# 19. Docker Registry

## What is a Registry?

A registry stores and distributes Docker images.

Examples mentioned in the presentation:

- Docker Hub
- Amazon ECR
- Azure Container Registry
- Google Container Registry
- Harbor
- Nexus

### Basic Flow

    Developer
        ↓
    Docker Image
        ↓
    docker push
        ↓
    Registry
        ↓
    docker pull
        ↓
    Server

---

# 20. Docker Hub

Docker Hub is the public default registry commonly used with Docker.

It contains many publicly available images.

Examples:

- nginx
- postgres
- python
- redis
- eclipse-temurin

---

# 21. Docker Hub Image Categories

The presentation describes four important categories.

## 21.1 Official Images

Official images are curated and maintained by Docker and upstream projects.

Examples:

    nginx
    postgres
    python
    eclipse-temurin

These are a good starting point.

---

## 21.2 Verified Publishers

These are images published by commercial vendors whose identity Docker has verified.

---

## 21.3 Community Images

Anyone can publish community images.

Before trusting a community image, check:

- Source
- Update history
- Pull count
- Reputation
- Image contents

---

## 21.4 Private Repositories

Organizations can store their own private images.

These images are visible only to authorized users.

Example:

    Company
       ↓
    Private Registry
       ↓
    Private Application Image

---

# 22. Docker Image Reference

An image can be referenced using a structure like:

    docker.io/library/nginx:1.27-alpine

Breakdown:

    docker.io
        ↓
    Registry

    library
        ↓
    Namespace

    nginx
        ↓
    Repository

    1.27-alpine
        ↓
    Tag

### General Format

    registry / namespace / repository : tag

---

# 23. Searching Docker Hub

Command:

    docker search nginx

This searches Docker Hub for images related to Nginx.

---

# 24. Pulling a Specific Image Version

Command:

    docker pull nginx:1.27

This downloads the specified Nginx image tag.

Instead of using an unspecified tag, specify the version when reproducibility matters.

---

# 25. Docker Login

Command:

    docker login

This authenticates with a registry.

It is required when accessing private repositories or pushing images to a registry.

---

# 26. Why Avoid `:latest` for Important Builds?

A tag such as:

    :latest

is just a tag.

It does not necessarily mean "the newest version" in the way people may assume.

A tag can move.

For example:

    nginx:latest
          ↓
       Image A

Later:

    nginx:latest
          ↓
       Image B

Therefore, the same Dockerfile using `:latest` may pull different image content at different times.

### Better Practice

Use an explicit version when reproducibility matters.

Example:

    nginx:1.27

This helps ensure that today's build and next month's build use the same tagged base version, assuming the tag is not changed.

---

# 27. Image Tags and Digests

## Tag

Example:

    nginx:1.27

A tag is a human-friendly reference.

Tags can move.

---

## Digest

Example:

    sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

A digest identifies the exact image content.

### Important Difference

    Tag
      ↓
    Human-friendly reference
      ↓
    Can move

    Digest
      ↓
    Content identity
      ↓
    Exact image bytes

### Key Idea

> A tag such as `:latest` can move, while a SHA-256 digest identifies exactly the same image content.

---

# 28. Images, Layers and Containers

A Docker image is a stack of read-only layers.

Example:

    FROM eclipse-temurin:17-jre
            ↓
       Base image layer

    RUN apt-get install ...
            ↓
       Package layer

    COPY app.jar /app/
            ↓
       Application layer

    Container
            ↓
    Writable layer

---

# 29. Docker Image Layers

Each instruction in a Dockerfile can add a layer.

Layers are:

- Read-only in the image
- Cached
- Reused between builds
- Shared between containers created from the same image

### Example

    Base Image
        ↓
    Package Installation
        ↓
    Application Files
        ↓
    Image

When multiple containers use the same image:

    Image Layers
       ↙ ↓ ↘
      C1 C2 C3

The image layers can be shared.

---

# 30. Container Writable Layer

When a container runs, Docker adds a writable layer on top of the read-only image layers.

Conceptually:

    Writable Container Layer
    ------------------------
    Changes made by running container

    Read-only Image Layer
    ------------------------
    Application

    Read-only Image Layer
    ------------------------
    Dependencies

    Read-only Image Layer
    ------------------------
    Base Image

If the container is deleted, its writable layer is also deleted.

### Important

This is why data persistence is important.

The presentation points forward to Session 2, where volumes and persistent data are covered.

---

# 31. Image vs Container Mental Model

A useful mental model is:

    Image
      ↓
    Class / Blueprint

    Container
      ↓
    Object / Instance

One image can create many container instances.

Each container has its own state.

---

# 32. Viewing Image Layers

To view the history/layers of an image:

    docker image history nginx

To inspect detailed image metadata:

    docker image inspect nginx

---

# 33. Installing Docker

## Linux

Install Docker Engine from Docker's own package repository.

The presentation notes that distribution packages can sometimes be outdated.

Docker runs natively on Linux without requiring a separate VM for Docker itself.

---

## macOS

Install Docker Desktop.

Docker Desktop runs a lightweight Linux VM behind the scenes.

Choose the appropriate build:

- Apple Silicon
- Intel

---

## Windows

Install Docker Desktop with the WSL 2 backend.

Linux containers are the default.

Virtualization may need to be enabled in the system BIOS.

---

# 34. Linux Docker Post-Installation

Enable and start Docker:

    sudo systemctl enable --now docker

This:

- Enables Docker to start automatically.
- Starts Docker immediately.

---

## Add Current User to Docker Group

Command:

    sudo usermod -aG docker $USER

After adding the user to the group, log in again for the group membership to take effect.

---

# 35. Important Docker Group Security Warning

The Docker group is effectively **root-equivalent**.

Anyone who can communicate with the Docker socket may be able to mount the host filesystem into a container and gain root-level access to the host.

Therefore:

- Add only trusted users to the Docker group.
- Protect access to the Docker socket.
- Do not expose the Docker daemon on a network port without proper TLS/security controls.

---

# 36. Verify Docker Installation

Check Docker version:

    docker version

Run the test container:

    docker run hello-world

### Why `hello-world`?

It verifies that Docker can:

1. Run the Docker command.
2. Communicate with the Docker daemon.
3. Pull the required image.
4. Create a container.
5. Run the container.

---

# 37. Docker Container Lifecycle

A Docker container moves through different states.

    Created
       ↓
    Running
       ↓
    Paused
       ↓
    Running
       ↓
    Stopped
       ↓
    Removed

Not every container must pass through every state.

---

# 38. Create a Container

Command:

    docker create nginx

This creates a container but does not start it.

Flow:

    Nginx Image
         ↓
    docker create
         ↓
    Created Container
         ↓
    Not Running

---

# 39. Start a Container

Command:

    docker start nginx

This starts an existing container.

Flow:

    Created / Stopped
          ↓
    docker start
          ↓
       Running

---

# 40. `docker run`

The command:

    docker run nginx

performs the equivalent of:

    docker create
          +
    docker start

Therefore:

    docker run
        ↓
    Create + Start

This is the command used most frequently when starting a new container.

---

# 41. Stop a Container

Command:

    docker stop web

Docker stop sends:

    SIGTERM

It waits for the application to shut down cleanly.

The presentation specifies a default wait of **10 seconds**, after which Docker sends:

    SIGKILL

if the process has not stopped.

### Purpose

`docker stop` gives the application an opportunity to clean up gracefully.

---

# 42. Kill a Container

Command:

    docker kill web

Docker kill sends:

    SIGKILL

immediately.

It does not provide the same graceful shutdown opportunity as `docker stop`.

### Use

`docker kill` is generally a last resort when an immediate termination is required.

---

# 43. Restart a Container

Command:

    docker restart web

Conceptually:

    docker stop
         ↓
    docker start

The writable layer survives the restart.

---

# 44. Pause a Container

Command:

    docker pause web

This pauses the processes inside the container.

---

# 45. Unpause a Container

Command:

    docker unpause web

This resumes the paused container.

The container continues from where it was paused.

---

# 46. Remove a Container

Command:

    docker rm web

This removes a stopped container.

The container's writable layer is removed.

---

# 47. Force Remove a Container

Command:

    docker rm -f web

This can stop and remove the container in one operation.

The writable layer is lost.

---

# 48. Container Lifecycle Summary

    docker create
          ↓
       Created
          ↓
    docker start
          ↓
       Running
          ↓
    docker pause
          ↓
       Paused
          ↓
    docker unpause
          ↓
       Running
          ↓
    docker stop
          ↓
       Stopped
          ↓
     docker rm
          ↓
      Removed

Another common path:

    docker run
       ↓
    Created + Started
       ↓
    Running

---

# 49. Working with Docker Images

## List Local Images

Command:

    docker images

This lists local Docker images and their sizes.

---

## Pull an Image

Command:

    docker pull redis:7

This downloads the Redis version 7 image.

---

## Inspect an Image

Command:

    docker image inspect redis

This displays detailed image metadata in JSON format.

---

## View Image History

Command:

    docker image history redis

This shows:

- Image layers
- Commands that created layers
- Layer history

---

## Tag an Image

Command:

    docker tag redis myrepo/redis:7

This gives an existing image another name/tag.

It does not necessarily create another full copy of the image.

---

## Push an Image

Command:

    docker push myrepo/redis:7

This uploads the image to a registry.

---

## Remove an Image

Command:

    docker rmi redis:7

This removes the local image if it is not required by existing containers.

---

## Remove Unused Images

Command:

    docker image prune -a

This removes unused local images.

Use cleanup commands carefully because unused images may still be useful for development or rollback.

---

# 50. Image Size Matters

Example from the presentation:

    nginx:1.27
        ↓
      192 MB

    nginx:alpine
        ↓
       47 MB

The same application can have different image sizes depending on the base image.

### Why Smaller Images Matter

Smaller images can:

- Pull faster
- Start faster
- Use less storage
- Contain fewer packages
- Potentially have fewer packages that could contain vulnerabilities

### Important Point

The base image is one of the biggest size decisions when building a container image.

---

# 51. Running and Managing Containers

## Run a Container

Command:

    docker run nginx

Creates and starts an Nginx container.

---

## Run in Detached Mode

Command:

    docker run -d --name web nginx

Explanation:

    -d
    ↓
    Detached mode

    --name web
    ↓
    Give the container the name "web"

### Why Name Containers?

Without a name, Docker can generate a random name.

Example:

    quirky_hopper

A meaningful name such as:

    web

is easier to:

- Remember
- Type
- Script
- Troubleshoot

### Best Practice

Always name long-running containers.

---

# 52. List Running Containers

Command:

    docker ps

Shows currently running containers.

---

# 53. List All Containers

Command:

    docker ps -a

Shows:

- Running containers
- Stopped containers
- Exited containers
- Other existing containers

---

# 54. Stop and Start by Name

Stop:

    docker stop web

Start:

    docker start web

Using a meaningful name makes management easier.

---

# 55. Remove a Stopped Container

Command:

    docker rm web

The container should normally be stopped before removal.

---

# 56. Automatically Remove a Container

Command:

    docker run --rm alpine echo hi

The `--rm` option automatically removes the container after it exits.

This is useful for one-off commands.

Example:

    docker run --rm alpine echo hi

Flow:

    Alpine Image
         ↓
    Temporary Container
         ↓
       echo hi
         ↓
       Exits
         ↓
    Container Removed

---

# 57. Environment Variables

Environment variables can be passed to a container using `-e`.

Example:

    docker run -e DB_HOST=db app

This passes:

    DB_HOST=db

into the container.

The application can then read the environment variable.

---

# 58. Resource Limits

Docker can limit resources available to a container.

Example:

    docker run --memory 512m --cpus 1 app

This means:

    Memory → 512 MB

    CPU → 1 CPU

These limits are enforced using cgroups.

### Why Resource Limits Matter

Without appropriate resource controls, one application may consume excessive resources and affect other workloads.

---

# 59. Remove All Stopped Containers

Command:

    docker container prune

This removes every stopped container.

Use carefully because removed containers cannot be recovered through Docker.

---

# 60. Looking Inside a Running Container

Docker provides several commands for troubleshooting.

---

## 60.1 View Container Logs

Command:

    docker logs web

This displays everything the container writes to:

- stdout
- stderr

---

## 60.2 Follow Logs Live

Command:

    docker logs -f --tail 50 web

Explanation:

    --tail 50
    ↓
    Show the last 50 lines

    -f
    ↓
    Follow new log output live

This is useful while testing an application.

---

## 60.3 Execute a Shell Inside a Container

Command:

    docker exec -it web sh

This opens a shell inside the running container.

The `-it` options provide an interactive terminal.

---

## 60.4 Execute One Command Inside a Container

Command:

    docker exec web ls /etc

This executes:

    ls /etc

inside the container.

It does not require opening a persistent shell.

---

# 61. Inspect a Container

Command:

    docker inspect web

This provides detailed information about the container.

It can show:

- Container ID
- Image
- Configuration
- Environment variables
- Network information
- IP address
- Mounts
- State
- Restart policy
- Port configuration

---

# 62. Extract One Value from `docker inspect`

Instead of reading the complete JSON output, we can extract a specific value.

### Container IP

    docker inspect -f '{{.NetworkSettings.IPAddress}}' web

### Container State

    docker inspect -f '{{.State.Status}}' web

This is useful for scripting and troubleshooting.

---

# 63. Docker Stats

Command:

    docker stats

This displays live resource usage for containers.

It can show:

- CPU
- Memory
- Network
- Other runtime statistics

Example:

    docker stats

This is useful for quickly checking whether a container is consuming excessive resources.

---

# 64. Docker Top

Command:

    docker top web

Shows processes currently running inside the container.

Example:

    docker top web

This helps understand what processes are actually running.

---

# 65. Docker Copy

Command:

    docker cp web:/etc/nginx ./ 

This copies files from the container to the host.

Docker `cp` can also be used to copy files into a container.

Conceptually:

    Host
      ↕
    docker cp
      ↕
    Container

---

# 66. Docker Debugging Sequence

When a container is not working correctly, use a systematic troubleshooting process.

## Step 1 — Check Container State

    docker ps -a

Question:

> Is the container running, or did it exit?

---

## Step 2 — Check Logs

    docker logs web

Question:

> What did the application say before it stopped?

---

## Step 3 — Inspect the Container

    docker inspect web

Check:

- Exit code
- Configuration
- Environment variables
- Network
- Mounts
- State

---

## Step 4 — Enter the Container

    docker exec -it web sh

Look around from inside the container.

### Debugging Flow

    docker ps -a
          ↓
    docker logs
          ↓
    docker inspect
          ↓
    docker exec

This should become a common Docker troubleshooting habit.

---

# 67. Attached Mode

## What is Attached Mode?

Attached mode is the default behavior when running a foreground container.

Example:

    docker run nginx

The container's output streams to your terminal.

Conceptually:

    Terminal
        ↓
    Docker Container
        ↓
    Container Output
        ↓
    Terminal

For a quick test, this can be useful.

---

## Ctrl+C in Attached Mode

When attached to a foreground container:

    Ctrl+C

can stop the container.

---

# 68. Detached Mode

## What is Detached Mode?

Detached mode runs the container in the background.

Command:

    docker run -d nginx

The command returns the container ID and the container continues running in the background.

Conceptually:

    Terminal
        ↓
    docker run -d
        ↓
    Container
        ↓
    Runs in background

This is the typical mode for long-running services.

Example:

    docker run -d --name web nginx

---

# 69. Interactive Mode

Interactive mode allows the user to interact directly with a process inside a container.

Command:

    docker run -it ubuntu bash

Here:

    -i
    ↓
    Keeps standard input open

    -t
    ↓
    Allocates a terminal

Together:

    -it
    ↓
    Interactive terminal

The result is an interactive Bash shell inside the Ubuntu container.

---

# 70. Attached vs Detached vs Interactive

## Attached

    docker run nginx

Purpose:

- Quick testing
- Observe output directly

Behavior:

    Container output
          ↓
    Your terminal

---

## Detached

    docker run -d nginx

Purpose:

- Run services in the background
- Long-running applications

Behavior:

    Terminal
       ↓
    Container runs in background

---

## Interactive

    docker run -it ubuntu bash

Purpose:

- Open a shell
- Troubleshooting
- Manually execute commands

Behavior:

    Terminal
       ↓
    Container shell
       ↓
    User interacts directly

---

# 71. Restart Policies

Docker can automatically restart a container when it exits.

---

## 71.1 No Restart

Command:

    --restart no

This is the default.

The container is not automatically restarted.

---

## 71.2 Restart on Failure

Command:

    --restart on-failure:3

This means:

- Restart when the container exits with a non-zero exit code.
- Try up to three times.

Example:

    docker run --restart on-failure:3 app

---

## 71.3 Unless Stopped

Command:

    --restart unless-stopped

The container is automatically restarted unless the user explicitly stopped it.

The presentation notes that this also supports restarting after a reboot unless the container had been explicitly stopped.

---

# 72. Detaching Without Stopping

When attached to a container, we can detach without stopping it using:

    Ctrl + P
    then
    Ctrl + Q

The container continues running.

To attach again:

    docker attach web

---

# 73. `docker attach` vs `docker exec`

## `docker attach`

Command:

    docker attach web

Attaches the terminal to the container's main process/output.

---

## `docker exec`

Command:

    docker exec -it web sh

Starts another command/process inside an already running container.

### Important Difference

    docker attach
        ↓
    Attach to the existing main process

    docker exec
        ↓
    Start another process inside the running container

For troubleshooting, `docker exec -it` is often more useful because it allows us to open a separate shell without taking over the main process.

---

# 74. Port Binding — Reaching a Container From Outside

Containers are private by default.

Suppose Nginx listens on:

    Container Port 80

Without publishing the port, something outside the host cannot normally reach that container port.

---

# 75. Docker Port Binding

The syntax is:

    -p HOST_PORT:CONTAINER_PORT

Example:

    docker run -d -p 8080:80 nginx

Meaning:

    Host Port 8080
          ↓
    Container Port 80
          ↓
        Nginx

---

# 76. Port Binding Example

Run:

    docker run -d --name web -p 8080:80 nginx

Then access:

    http://localhost:8080

Flow:

    Browser
        ↓
    localhost:8080
        ↓
    Host Port 8080
        ↓
    Docker Port Mapping
        ↓
    Container Port 80
        ↓
    Nginx

---

# 77. Host Port Comes First

This is extremely important.

The syntax is:

    -p HOST:CONTAINER

Therefore:

    -p 8080:80

means:

    8080 → Host Port
    80   → Container Port

Remember:

> **HOST FIRST, CONTAINER SECOND**

---

# 78. Bind to a Specific Host IP

Command:

    docker run -d -p 127.0.0.1:8080:80 nginx

This means the container port is reachable only through the host's loopback address.

Flow:

    127.0.0.1:8080
          ↓
    Container Port 80

This is different from:

    -p 8080:80

which normally publishes the port on all host interfaces.

---

# 79. Random Port Publishing with `-P`

Command:

    docker run -d -P nginx

`-P` maps every port declared with `EXPOSE` to a random host port.

Example:

    Container
       ↓
    EXPOSE 80
       ↓
    Docker
       ↓
    Random Host Port

Check the actual mapping using:

    docker port web

---

# 80. Check Current Port Mappings

Command:

    docker port web

Example:

    80/tcp -> 0.0.0.0:8080

Meaning:

    Host Port 8080
          ↓
    Container Port 80

---

# 81. EXPOSE vs Port Binding

These two concepts are different.

## EXPOSE

Example:

    EXPOSE 80

`EXPOSE` records/document the port used by the application.

It does **not** publish the port by itself.

---

## `-p`

Example:

    docker run -d -p 8080:80 nginx

`-p` actually publishes/maps the host port to the container port.

### Remember

    EXPOSE
       ↓
    Documentation / Image Metadata

    -p
       ↓
    Actual Port Publishing

---

# 82. One Host Port — One Container Binding

Two containers cannot normally bind the same host port.

For example, this is not possible:

    Container 1 → Host Port 8080
    Container 2 → Host Port 8080

Instead use different host ports:

    Container 1 → Host 8080 → Container 80

    Container 2 → Host 8081 → Container 80

Commands:

    docker run -d --name web1 -p 8080:80 nginx

    docker run -d --name web2 -p 8081:80 nginx

Access:

    http://localhost:8080

    http://localhost:8081

Both containers can use port 80 internally.

---

# 83. Port Binding Mental Model

    Browser
       ↓
    Host Port
       ↓
    Docker Port Mapping
       ↓
    Container Port
       ↓
    Application

Example:

    Browser
       ↓
    localhost:8080
       ↓
    Host Port 8080
       ↓
    Container Port 80
       ↓
    Nginx

---

# 84. Hands-on Lab 1 — Your First Containers

The first lab focuses on creating, running, inspecting, and understanding containers.

---

## 84.1 Verify Docker

Run:

    docker version

Check both parts of the output.

---

## 84.2 Run Hello World

Run:

    docker run hello-world

Read and understand the output.

This verifies the Docker installation and basic image/container workflow.

---

## 84.3 Pull Nginx

Run:

    docker pull nginx

---

## 84.4 Pull Nginx Alpine

Run:

    docker pull nginx:alpine

Compare the image sizes.

The presentation demonstrates that the Alpine-based image is significantly smaller.

---

## 84.5 Start Nginx in Detached Mode

Run:

    docker run -d --name web -p 8080:80 nginx

Explanation:

    -d
    → Detached mode

    --name web
    → Container name

    -p 8080:80
    → Host port 8080 → Container port 80

    nginx
    → Image

---

## 84.6 Open Nginx in Browser

Open:

    http://localhost:8080

The request flow is:

    Browser
       ↓
    localhost:8080
       ↓
    Docker Port Binding
       ↓
    Nginx Container:80
       ↓
    Nginx Web Server

---

## 84.7 Start a Second Nginx Container

Run:

    docker run -d --name web2 -p 8081:80 nginx

Now:

    http://localhost:8080
        ↓
    web container

    http://localhost:8081
        ↓
    web2 container

---

## 84.8 Follow Nginx Logs

Run:

    docker logs -f --tail 50 web

Refresh the browser.

Observe the requests appearing in the logs.

This demonstrates the relationship between:

    Browser Request
          ↓
    Nginx Container
          ↓
    Container Logs

---

## 84.9 Enter the Container

Run:

    docker exec -it web sh

Now you are inside the running Nginx container.

You can inspect the container filesystem.

---

## 84.10 Edit `index.html`

From inside the container, locate the Nginx web content and modify the page.

Then refresh the browser and observe the change.

This demonstrates that the running container has its own writable layer.

---

## 84.11 Find the Container IP

Run:

    docker inspect -f '{{.NetworkSettings.IPAddress}}' web

This extracts the container IP from the inspection data.

---

## 84.12 Stop the Container

Run:

    docker stop web

Then check:

    docker ps -a

The container should now be stopped.

---

## 84.13 Start the Same Container Again

Run:

    docker start web

Refresh the browser.

The edit made to the container's writable layer should still exist because the container itself was stopped and started rather than removed.

---

## 84.14 Remove the Container

Run:

    docker stop web

    docker rm web

The container and its writable layer are removed.

---

## 84.15 Create a New Container

Run:

    docker run -d --name web -p 8080:80 nginx

The previous modification is no longer present because the old container and its writable layer were removed.

---

# 85. Important Observation From Lab 1

The most important point of the lab is understanding the difference between:

    STOPPING a container

and:

    REMOVING a container

When the container is stopped:

    Container
       ↓
    Writable Layer
       ↓
    Still Exists

Therefore, restarting the same container can preserve changes made to its writable layer.

When the container is removed:

    Container
       ↓
    Writable Layer
       ↓
    Deleted

Therefore, starting a new container from the same image does not contain changes that existed only in the old container's writable layer.

This is the problem that persistent storage mechanisms such as volumes solve in the next session.

---

# 86. Important Docker Commands — Session 1 Quick Reference

## Docker Installation / Verification

    docker version

    docker run hello-world

---

## Image Commands

    docker images

    docker pull nginx

    docker pull redis:7

    docker image inspect nginx

    docker image history nginx

    docker tag redis myrepo/redis:7

    docker push myrepo/redis:7

    docker rmi redis:7

    docker image prune -a

---

## Container Creation and Lifecycle

    docker create nginx

    docker run nginx

    docker run -d --name web nginx

    docker start web

    docker stop web

    docker restart web

    docker pause web

    docker unpause web

    docker kill web

    docker rm web

    docker rm -f web

---

## Container Listing

    docker ps

    docker ps -a

---

## Container Inspection

    docker inspect web

    docker inspect -f '{{.State.Status}}' web

    docker inspect -f '{{.NetworkSettings.IPAddress}}' web

---

## Container Debugging

    docker logs web

    docker logs -f --tail 50 web

    docker exec -it web sh

    docker exec web ls /etc

    docker stats

    docker top web

    docker cp web:/etc/nginx ./

---

## Container Modes

    docker run nginx

    docker run -d nginx

    docker run -it ubuntu bash

---

## Restart Policies

    docker run --restart no nginx

    docker run --restart on-failure:3 app

    docker run --restart unless-stopped app

---

## Port Binding

    docker run -d -p 8080:80 nginx

    docker run -d -p 127.0.0.1:8080:80 nginx

    docker run -d -P nginx

    docker port web

---

# 87. Complete Docker Mental Model

    Docker Client
          ↓
       REST API
          ↓
    Docker Daemon
          ↓
    ┌───────────────┐
    │               │
    ↓               ↓
    Images       Containers
    │               │
    ↓               ↓
    Layers       Writable Layer
                    │
                    ↓
                Application
                    │
                    ↓
                Port Binding
                    │
                    ↓
                  Users

---

# 88. Complete Container Flow

    Docker Image
          ↓
    docker run
          ↓
    Container Created
          ↓
    Container Started
          ↓
    Application Running
          ↓
    Port Binding
          ↓
    User Accesses Application

Example:

    nginx Image
         ↓
    docker run -d --name web -p 8080:80 nginx
         ↓
    Nginx Container
         ↓
    Container Port 80
         ↓
    Host Port 8080
         ↓
    Browser
         ↓
    http://localhost:8080

---

# 89. Complete Container Lifecycle

    Docker Image
         ↓
    docker create
         ↓
      CREATED
         ↓
    docker start
         ↓
      RUNNING
       ↙     ↘
      ↓       ↓
PAUSED   docker stop
↓       ↓
unpause   STOPPED
↓       ↓
RUNNING   ↓
↓
docker start
↓
RUNNING

    STOPPED
       ↓
    docker rm
       ↓
    REMOVED

Shortcut:

    docker run
       ↓
    CREATE + START
       ↓
    RUNNING

---

# 90. Most Important Concepts From Session 1

## Container

A container is an isolated Linux process with its own view of processes, networking, filesystem, users, and resource limits.

## Image

An image is a read-only template used to create containers.

## Dockerfile

A Dockerfile is used to define how an image is built.

## Docker Client

The CLI used to send commands to Docker.

## Docker Daemon

The component that actually builds images, runs containers, and manages Docker resources.

## Registry

A system that stores and distributes Docker images.

## Docker Hub

The public default registry used by Docker.

## Namespace

Controls what a process can see.

## cgroups

Control what resources a process can use.

## Union Filesystem

Combines read-only image layers with a writable container layer.

## Port Binding

Maps a host port to a container port.

    -p HOST_PORT:CONTAINER_PORT

## Attached Mode

    docker run nginx

Runs in the foreground with output connected to the terminal.

## Detached Mode

    docker run -d nginx

Runs the container in the background.

## Interactive Mode

    docker run -it ubuntu bash

Provides an interactive terminal.

---

# 91. Interview Questions — Session 1

## Q1. Why do we need containers?

Containers package applications together with their dependencies and environment, helping reduce environment differences and the "it works on my machine" problem.

---

## Q2. What is the difference between a VM and a container?

A VM includes a complete guest operating system, while a container shares the host kernel and packages the application with its libraries and dependencies.

---

## Q3. Are containers a replacement for VMs?

Not completely.

Containers usually run inside VMs in cloud environments.

Containers replace the need to give every application its own operating system, but VMs are still useful for isolation, different kernels, and infrastructure boundaries.

---

## Q4. What is a Docker image?

A Docker image is a read-only template from which containers are created.

---

## Q5. What is a Docker container?

A container is a running or stopped instance created from a Docker image.

---

## Q6. What are namespaces?

Namespaces isolate what a process can see, such as processes, network interfaces, hostname, mount points, and users.

---

## Q7. What are cgroups?

Control groups limit and account for resources such as CPU, memory, and disk I/O.

---

## Q8. What is a Docker registry?

A registry stores and distributes Docker images.

Examples include:

    Docker Hub
    Amazon ECR
    Azure Container Registry
    Google Container Registry
    Harbor
    Nexus

---

## Q9. What is the difference between an image and a container?

    Image
    → Read-only template

    Container
    → Instance created from the image with its own writable layer and runtime state

---

## Q10. What does `docker run` do?

`docker run` creates and starts a container from an image.

Conceptually:

    docker run
        =
    docker create
        +
    docker start

---

## Q11. What is the difference between `docker stop` and `docker kill`?

    docker stop
    → Graceful shutdown
    → Sends SIGTERM
    → Waits before forcing termination

    docker kill
    → Immediate termination
    → Sends SIGKILL

---

## Q12. What is the difference between `docker stop` and `docker rm`?

    docker stop
    → Stops the container

    docker rm
    → Removes the container

Stopping does not delete the container.

---

## Q13. What is the difference between `docker logs` and `docker inspect`?

    docker logs
    → Application/container output

    docker inspect
    → Detailed Docker configuration and metadata

---

## Q14. What does `-d` mean?

`-d` means detached mode.

The container runs in the background.

---

## Q15. What does `-it` mean?

    -i
    → Keep standard input open

    -t
    → Allocate a terminal

Together they provide an interactive terminal.

---

## Q16. What does `-p 8080:80` mean?

    Host Port 8080
          ↓
    Container Port 80

The host port is written first.

---

## Q17. Does `EXPOSE 80` publish port 80?

No.

`EXPOSE` documents the port used by the application.

Actual port publishing requires something such as:

    -p 8080:80

---

## Q18. Why can two containers both use container port 80?

Because the container ports belong to separate network namespaces.

However, they cannot normally both bind the same host port.

Example:

    Container 1 → Host 8080 → Container 80

    Container 2 → Host 8081 → Container 80

---

## Q19. Why should long-running containers be named?

A meaningful name is easier to:

- Remember
- Type
- Script
- Troubleshoot

Example:

    web

is easier to use than a randomly generated container name.

---

## Q20. Why should we care about image size?

Smaller images can:

- Pull faster
- Start faster
- Use less storage
- Contain fewer packages that could contain vulnerabilities

The base image is an important factor in image size.

---

# 92. Final Revision — Session 1

    Physical Server
          ↓
    Virtual Machine
          ↓
      Container
          ↓
    Docker Image
          ↓
    Docker Container
          ↓
    Application
          ↓
    Port Binding
          ↓
       User

### Docker Architecture

    Docker CLI
       ↓
    REST API
       ↓
    dockerd
       ↓
    Images
    Containers
    Networks
    Volumes

### Container Internals

    Namespace
       ↓
    What the process can see

    cgroups
       ↓
    What the process can use

    Union Filesystem
       ↓
    How image layers + writable layer work

### Container Lifecycle

    Created
       ↓
    Running
       ↓
    Paused / Stopped
       ↓
    Running
       ↓
    Removed

### Port Binding

    -p HOST:CONTAINER

    Example:

    -p 8080:80

    Host 8080
       ↓
    Container 80
       ↓
    Nginx

### Session 1 Core Understanding

    Docker Image
         ↓
    docker run
         ↓
    Docker Container
         ↓
    Application
         ↓
    Port Binding
         ↓
    External Access

The main purpose of Docker is to package and run applications consistently by shipping the application together with its required environment.