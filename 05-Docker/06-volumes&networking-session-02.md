# Module 07 — Containerization with Docker

## 📚 Course / Module

**Module:** Containerization with Docker  
**Session:** Session 2 — Volumes and Networking  
**Duration:** 2 Hours  
**Slides Covered:** 20–30

---

# 🎯 Learning Objectives

By the end of Session 2, I should be able to:

- Understand why data disappears when a container is removed.
- Understand why containers should be treated as disposable.
- Understand the limitations of the container writable layer.
- Understand why persistent data should live outside the container.
- Understand Docker volumes.
- Understand bind mounts.
- Understand tmpfs mounts.
- Create and manage named volumes.
- Understand named volumes vs anonymous volumes.
- Understand how data survives container removal.
- Understand volume backup and restore.
- Understand sharing data between containers.
- Understand read-only mounts.
- Understand why databases require consistent backups.
- Understand Docker container networking.
- Understand the default bridge network.
- Understand user-defined bridge networks.
- Understand Docker network drivers.
- Understand Docker's embedded DNS and service discovery.
- Understand how containers communicate using service names.
- Understand network segmentation.
- Build a multi-container application network.
- Understand the practical differences between the default bridge and user-defined networks.

---

# Session 2 — Volumes and Networking

## 📌 Topics Covered

1. Why Container Data Disappears
2. Containers Are Disposable by Design
3. Problems With the Writable Container Layer
4. The Stateless Container Principle
5. Three Ways to Store Data Outside a Container
6. Docker Volumes
7. Bind Mounts
8. tmpfs Mounts
9. Working With Volumes
10. Named Volumes
11. Anonymous Volumes
12. Pre-Populated Volumes
13. Proving Data Survives Container Removal
14. Volumes vs Bind Mounts
15. Backing Up Volumes
16. Restoring Volumes
17. Sharing Volumes Between Containers
18. Read-Only Consumers
19. Database Backup Considerations
20. How Containers Communicate
21. Network Namespaces
22. Default Bridge Network
23. Problems With the Default Bridge
24. User-Defined Networks
25. Docker Network Drivers
26. Bridge Network
27. Host Network
28. None Network
29. Overlay Network
30. Macvlan Network
31. User-Defined Networks and Service Discovery
32. Docker Embedded DNS
33. Connecting Containers to Networks
34. Network Segmentation
35. Lab 2 — Data That Survives, Apps That Connect

---

# 1. Why Container Data Disappears

## What is the problem?

Everything a container writes goes into its writable layer.

That writable layer belongs to the container.

Therefore, when the container is removed, the writable layer is removed with it.

### Basic Flow

    Docker Image
         ↓
    Container
         ↓
    Writable Container Layer
         ↓
    Application Data

If the container is removed:

    Container
         ↓
    Removed
         ↓
    Writable Layer Removed
         ↓
    Data Gone

---

## Example

Suppose an application writes:

    /app/data/file.txt

inside the container.

That file belongs to the container's writable layer unless the path is backed by external storage.

If the container is deleted:

    docker rm container

the data stored only in that writable layer disappears.

---

# 2. Containers Are Disposable by Design

Containers are intended to be:

    Created
       ↓
    Used
       ↓
    Replaced

They are not normally treated like traditional servers that are manually repaired for years.

### Example

Suppose version 1 of an application is running:

    Application v1
         ↓
    Container v1

When upgrading:

    Remove old container
          ↓
    Start new container
          ↓
    Application v2

The presentation describes the principle as:

> Containers are meant to be replaced, not repaired.

---

# 3. Why the Writable Layer Is Not Suitable for Important Data

The container writable layer has several limitations.

## 3.1 It Belongs to the Container

The writable layer is tied to the lifetime of the container.

Remove the container:

    Container removed
         ↓
    Writable layer removed
         ↓
    Data lost

---

## 3.2 It Is Slow for Heavy Data Writes

Writes go through the union filesystem's copy-on-write mechanism.

This is poorly suited to workloads such as databases.

### Example

    PostgreSQL
         ↓
    Frequent database writes
         ↓
    Container writable layer
         ↓
    Poor choice for persistent database data

---

## 3.3 It Cannot Be Shared Easily

Another container cannot simply read another container's writable layer.

Example:

    Container A
       ↓
    Writable Layer A

    Container B
       ↓
    Writable Layer B

Container B cannot directly use Container A's writable layer as shared persistent storage.

---

## 3.4 It Cannot Be Backed Up Cleanly

The writable layer is stored inside Docker's internal storage and is tied to the container's lifetime.

This makes it a poor place for important data that needs reliable backup and recovery.

---

# 4. The Stateless Container Principle

The important rule from this session is:

> **Containers are stateless. Data lives outside them.**

Anything that would be painful to lose should be stored outside the container.

Examples:

- Database files
- User uploads
- Application state
- Important configuration
- Persistent application data

### Mental Model

    Container
       ↓
    Application
       ↓
    Temporary / Replaceable

    Persistent Data
       ↓
    External Storage
       ↓
    Volume / Bind Mount / Other Storage

---

# 5. Three Ways to Store Data Outside a Container

Docker provides three important mechanisms discussed in this session:

1. Volumes
2. Bind mounts
3. tmpfs mounts

---

# 6. Docker Volumes

## What is a Volume?

A Docker volume is storage managed by Docker outside the container's writable layer.

Example:

    Volume
      ↓
    pgdata

    Container
      ↓
    /var/lib/postgresql/data

The volume can survive even when the container is removed.

---

## Example

    docker run -d \
      -v pgdata:/var/lib/postgresql/data \
      postgres:16

The volume:

    pgdata

is mounted into:

    /var/lib/postgresql/data

inside the PostgreSQL container.

---

## Why Use Volumes?

Volumes are:

- Managed by Docker
- Portable between hosts compared with host-directory-specific bind mounts
- Easy to back up using Docker-based workflows
- Recommended for persistent application data

### Typical Use

Volumes are especially useful for:

- Databases
- Application state
- Persistent data

---

# 7. Bind Mounts

## What is a Bind Mount?

A bind mount maps a specific directory from the host into a container.

Example:

    -v $(pwd)/src:/app/src

This means:

    Host Directory
    $(pwd)/src
          ↓
    Container Directory
    /app/src

Changes made on the host are immediately visible inside the container, and changes made inside the mounted directory are visible on the host.

---

## Typical Use

Bind mounts are useful for:

- Local development
- Live code editing
- Configuration files

### Example

    Host
    ./src
       ↓
    Container
    /app/src

Edit the source code on the host and the container sees the change.

---

# 8. tmpfs Mounts

## What is tmpfs?

A tmpfs mount stores data in the host's memory instead of writing it to disk.

Example:

    --tmpfs /app/cache

The data exists in memory.

When the container stops, the tmpfs data disappears.

### Typical Use

tmpfs can be useful for:

- Secrets
- Scratch space
- Temporary caches

### Important Characteristic

    tmpfs
       ↓
    Host memory
       ↓
    Not written to disk
       ↓
    Gone when container stops

---

# 9. Volume vs Bind Mount vs tmpfs

| Feature | Volume | Bind Mount | tmpfs |
|---|---|---|---|
| Managed by | Docker | User/Host | Docker/Host memory |
| Storage location | Docker-managed storage | Any host path | Host memory |
| Survives container removal | Yes | Yes | No |
| Typical use | Production data | Development/configuration | Secrets/cache/scratch |
| Host path required | No | Yes | No |
| Persistent on disk | Yes | Yes | No |

---

# 10. The `-v` Syntax

The general format is:

    -v SOURCE:TARGET

For a named volume:

    -v pgdata:/var/lib/postgresql/data

Here:

    pgdata
       ↓
    Docker Volume

    /var/lib/postgresql/data
       ↓
    Path inside container

For a bind mount:

    -v $(pwd)/src:/app/src

Here:

    $(pwd)/src
       ↓
    Host directory

    /app/src
       ↓
    Container directory

---

# 11. The `--mount` Syntax

Docker also provides the more explicit `--mount` syntax.

Example:

    --mount type=volume,source=pgdata,target=/var/lib/postgresql/data

The presentation recommends the newer `--mount` syntax in scripts because it is more explicit.

### Comparison

Short syntax:

    -v pgdata:/var/lib/postgresql/data

Explicit syntax:

    --mount type=volume,source=pgdata,target=/var/lib/postgresql/data

---

# 12. Working With Volumes

## Create a Named Volume

    docker volume create pgdata

This creates a Docker-managed volume named:

    pgdata

---

## List Volumes

    docker volume ls

This lists available Docker volumes.

---

## Inspect a Volume

    docker volume inspect pgdata

This shows information such as:

- Mountpoint
- Driver
- Labels
- Volume configuration

---

## Remove a Volume

    docker volume rm pgdata

The volume can be removed only when it is no longer being used by a container.

---

## Remove Unused Volumes

    docker volume prune

This removes unused volumes.

Use carefully because the deleted volumes may contain data you still need.

---

# 13. Named Volumes

## What is a Named Volume?

A named volume is a volume whose name you choose.

Example:

    pgdata

Then you can refer to it again:

    -v pgdata:/var/lib/postgresql/data

### Why Use Named Volumes?

Named volumes are easier to:

- Identify
- Reuse
- Backup
- Restore
- Manage

### Best Practice

> Always name volumes for data you care about.

---

# 14. Anonymous Volumes

An anonymous volume does not have a name chosen by you.

Docker generates an identifier for it.

### Problem

Anonymous volumes are:

- Easy to lose track of
- Harder to identify
- Easier to delete accidentally

For important application data, named volumes are preferred.

---

# 15. Pre-Populated Volumes

When an empty volume is mounted into a container at a path where the image already has files, Docker can copy the existing files from that image path into the volume on first use.

Conceptually:

    Image
      ↓
    Existing Files
      ↓
    Empty Volume Mounted
      ↓
    Files Copied Into Volume

This can be useful when an image provides default files at the mount location.

---

# 16. Prove That Volume Data Survives Container Removal

A PostgreSQL example:

    docker run -d --name db \
      -e POSTGRES_PASSWORD=secret \
      -v pgdata:/var/lib/postgresql/data \
      postgres:16

The important part is:

    -v pgdata:/var/lib/postgresql/data

The PostgreSQL data is stored in the named volume.

---

## Create Data

Use `docker exec` to enter PostgreSQL and create a table.

Conceptually:

    PostgreSQL Container
          ↓
    docker exec
          ↓
    psql
          ↓
    Create Table

---

## Remove the Container

    docker rm -f db

The container is removed.

But:

    pgdata

still exists.

---

## Start PostgreSQL Again With the Same Volume

Run PostgreSQL again using:

    -v pgdata:/var/lib/postgresql/data

The existing database data is still available.

### Important Concept

    Container Removed
          ↓
    Volume Remains
          ↓
    New Container
          ↓
    Same Volume
          ↓
    Data Still Exists

This demonstrates why persistent data should be stored outside the container.

---

# 17. Volume vs Bind Mount

The presentation compares them across several areas.

| Feature | Volume | Bind Mount |
|---|---|---|
| Location | Managed by Docker | Any host path |
| Created by | Docker, on demand | User |
| Host path required | No | Yes |
| Portability | Works consistently across hosts | Depends on host directory layout |
| Mac/Windows performance | Faster | Can be slower because of VM boundary |
| Backup | Docker commands or volume drivers | Normal host file tools |
| Risk | Lower | Container can modify host files |
| Typical use | Production data | Live development |

---

# 18. Bind Mount Example With Nginx

Run:

    docker run -d \
      -p 8080:80 \
      -v "$(pwd)/site":/usr/share/nginx/html:ro \
      nginx

Here:

    $(pwd)/site
       ↓
    Host directory

    /usr/share/nginx/html
       ↓
    Nginx document root

    :ro
       ↓
    Read-only

---

# 19. Read-Only Bind Mount

The `:ro` option means read-only.

Example:

    -v "$(pwd)/site":/usr/share/nginx/html:ro

The container can read the files but cannot modify them through that mount.

### Why Use Read-Only?

Read-only mounts reduce the ability of a container to modify important host data.

---

# 20. Backing Up a Volume

The presentation demonstrates backing up a volume into a tarball using a temporary container.

Command:

    docker run --rm \
      -v pgdata:/data:ro \
      -v "$(pwd)":/backup \
      alpine tar czf /backup/pgdata.tgz -C /data .

### What Happens?

    pgdata Volume
         ↓
    Mounted at /data
         ↓
    Read-only

    Current Host Directory
         ↓
    Mounted at /backup

    Alpine Temporary Container
         ↓
    tar
         ↓
    /backup/pgdata.tgz

The `--rm` option removes the temporary container after the command finishes.

---

# 21. Restore a Volume

First create a new volume:

    docker volume create pgdata2

Then restore the backup:

    docker run --rm \
      -v pgdata2:/data \
      -v "$(pwd)":/backup \
      alpine tar xzf /backup/pgdata.tgz -C /data

### Flow

    Backup Tarball
         ↓
    Temporary Container
         ↓
    /backup
         ↓
    Extract
         ↓
    /data
         ↓
    pgdata2 Volume

---

# 22. The Temporary Container Backup Pattern

The presentation uses a useful pattern:

    Existing Volume
          ↓
    Temporary Container
          ↓
    Host Backup Directory
          ↓
    Copy Data
          ↓
    Temporary Container Removed

This allows Docker volumes to be backed up without directly manipulating Docker's internal storage.

---

# 23. Sharing Data Between Containers

The same named volume can be mounted into multiple containers.

Example:

    Named Volume
        ↓
    ┌─────┴─────┐
    ↓           ↓
    Web       Log Shipper

Both containers can see the same directory.

### Example Use Case

A web server writes logs:

    Web Server
        ↓
    Shared Volume
        ↓
    Log Shipper

The log shipper reads the same log files.

---

# 24. Read-Only Consumers

When multiple containers use the same data, it is safer to allow only the writer to modify it.

Example:

    Writer Container
        ↓
    Read-Write Volume

    Consumer Container
        ↓
    Read-Only Volume

Use:

    :ro

for consumers that should not modify the data.

### Principle

> Give the writer read-write access and other containers read-only access when possible.

---

# 25. Database Backup Considerations

A critical point from the presentation:

> Copying live database files can produce a corrupt backup.

Why?

A database may be writing data while the files are being copied.

Therefore, file-level copying of an active database does not necessarily produce a consistent backup.

### Safer Approaches

Use the database's own dump/backup tool.

For PostgreSQL:

    pg_dump

Or stop the database container before copying its files.

### Important Production Principle

> Database backups should normally use the database's native backup mechanism or another method that guarantees a consistent snapshot.

---

# 26. Container Networking

Now we move from storage to networking.

## How Do Containers Talk to Each Other?

Every container gets its own network namespace.

This means a container has its own:

- Network interfaces
- IP address
- Ports

Docker connects these isolated network namespaces using network drivers.

---

# 27. Container Network Namespace

Conceptually:

    Container 1
       ↓
    Network Namespace 1
       ↓
    Interface + IP + Ports

    Container 2
       ↓
    Network Namespace 2
       ↓
    Interface + IP + Ports

Each container has its own network environment.

Docker networking connects them when they share a network.

---

# 28. Default Bridge Network

Docker provides a default bridge network.

The default bridge is commonly associated with:

    docker0

Example network:

    172.17.0.0/16

Example containers:

    web
    172.17.0.2

    app
    172.17.0.3

    db
    172.17.0.4

---

# 29. Default Bridge Behavior

Every container joins the default bridge unless another network is specified.

Containers on the default bridge can reach one another by IP address.

However:

> Container names do not resolve automatically on the default bridge in the same way they do on user-defined networks.

Example:

    web
      ↓
    172.17.0.2

    app
      ↓
    172.17.0.3

The application can communicate using the IP address.

---

# 30. Problem With Using Container IP Addresses

Container IP addresses can change.

Example:

    db
      ↓
    172.17.0.4

Container is restarted.

Now:

    db
      ↓
    172.17.0.7

If the application hard-codes:

    172.17.0.4

the connection may break.

### Why?

Container IP addresses are not stable identifiers.

---

# 31. User-Defined Networks

User-defined networks solve important problems with the default bridge.

Create a network:

    docker network create invoice-net

Then connect containers to it.

Containers on the user-defined network can discover each other by name through Docker's built-in DNS.

---

# 32. Service Discovery

Suppose we have:

    app
       ↓
    connects to
       ↓
    db:5432

The application does not need to know the database's current IP address.

It can use:

    db

Docker's embedded DNS resolves:

    db
      ↓
    Current IP address of database container

### Important Principle

> Use service/container names instead of hard-coding container IP addresses.

---

# 33. User-Defined Network Example

Create the network:

    docker network create invoice-net

Start the database:

    docker run -d --name db \
      --network invoice-net \
      -e POSTGRES_PASSWORD=secret \
      postgres:16

Start the application:

    docker run -d --name app \
      --network invoice-net \
      -p 8080:8080 \
      -e DB_HOST=db \
      invoice-service

The application uses:

    DB_HOST=db

instead of a hard-coded IP address.

---

# 34. How the Application Finds the Database

The flow is:

    app container
         ↓
    DB_HOST=db
         ↓
    Docker Embedded DNS
         ↓
    Resolves "db"
         ↓
    Current IP of db container
         ↓
    PostgreSQL :5432

This allows the database container to be replaced or restarted without requiring the application to know its IP address.

---

# 35. Docker Network Commands

## List Networks

    docker network ls

The default networks include:

    bridge
    host
    none

---

## Create a Network

    docker network create invoice-net

---

## Connect a Running Container

    docker network connect invoice-net web

This attaches the running container to the network.

---

## Inspect a Network

    docker network inspect invoice-net

This shows information such as:

- Network configuration
- Connected containers
- Container IP addresses

---

## Remove a Network

    docker network rm invoice-net

The network must not have attached containers.

---

# 36. Network Drivers

Docker provides different network drivers for different requirements.

The session covers:

1. Bridge
2. Host
3. None
4. Overlay
5. Macvlan

---

# 37. Bridge Network

## What does it do?

A bridge network provides a private network for containers on one Docker host.

Containers can communicate with each other.

Traffic going outside the host can use NAT.

### Typical Use

Bridge networking is the default choice for single-host container applications.

Example:

    Container A
       ↓
    Bridge Network
       ↓
    Container B

---

# 38. Host Network

With host networking, the container uses the host's network stack directly.

Command concept:

    --network host

### Characteristics

- No separate network isolation
- No Docker port mapping required
- Container uses host networking directly
- Can provide maximum network performance on Linux

### Important

With:

    --network host

`-p` is ignored because the container's ports are already the host's ports.

The presentation notes that this behavior applies this way on Linux.

---

# 39. None Network

The `none` network provides no normal container networking.

The container gets only a loopback interface.

Conceptually:

    Container
       ↓
    No external network
       ↓
    Loopback only

### Typical Use

Batch jobs that must not reach anything.

---

# 40. Overlay Network

An overlay network can span multiple Docker hosts.

Conceptually:

    Docker Host 1
       ↓
    Overlay Network
       ↓
    Docker Host 2
       ↓
    Overlay Network

### Typical Use

Docker Swarm services running across multiple nodes.

---

# 41. Macvlan Network

With macvlan, the container gets its own MAC address and can appear as a device on the physical LAN.

Conceptually:

    Physical LAN
        ↓
    Container
        ↓
    Own MAC Address

### Typical Use

Legacy applications that expect to appear as a real device on the physical network.

---

# 42. Network Driver Comparison

| Driver | What It Does | Typical Use |
|---|---|---|
| bridge | Private network on one host, external access through NAT | Single-host applications |
| host | Uses host network stack directly | Maximum network performance on Linux |
| none | No network except loopback | Isolated batch jobs |
| overlay | Network across multiple Docker hosts | Docker Swarm services |
| macvlan | Container appears as a physical LAN device | Legacy applications requiring LAN identity |

---

# 43. User-Defined Network and Service Discovery

Example architecture:

    Network: invoice-net

    ┌─────────────────────────────┐
    │                             │
    │       invoice-net           │
    │                             │
    │    app  ───────→  db        │
    │               db:5432       │
    │                             │
    └─────────────────────────────┘

The application connects to:

    db:5432

Docker DNS resolves:

    db

to the database container's current IP.

---

# 44. Why User-Defined Networks Are Important

The presentation gives a practical rule:

> Never rely on the default bridge for multi-container applications.

Instead:

    Create a network for each application.

Example:

    docker network create invoice-net

Then place the application's containers on that network.

This provides:

- Name-based service discovery
- Better application isolation
- Stable service names
- Easier multi-container configuration

---

# 45. Network Segmentation

A larger application can be divided into multiple networks.

Example:

    Frontend Network
    ┌───────────────────────┐
    │                       │
    │  web ←→ app           │
    │                       │
    └───────────────────────┘

    Backend Network
    ┌───────────────────────┐
    │                       │
    │  app ←→ db            │
    │                       │
    └───────────────────────┘

The application container can belong to both networks.

---

# 46. Network Segmentation Example

Architecture:

    frontend network

    web
      ↕
    app


    backend network

    app
      ↕
    db

Therefore:

    web → app
       ↓
    Allowed

    app → db
       ↓
    Allowed

    web → db
       ↓
    Not directly connected

---

# 47. Why Network Segmentation Is Useful

Network segmentation limits which services can communicate directly.

In the example:

    web
       ↓
    frontend
       ↓
    app
       ↓
    backend
       ↓
    db

The web container does not share a network directly with the database.

The presentation explains that even if the web container were compromised, the lack of a shared network prevents it from directly seeing the database network.

This provides an additional isolation boundary.

---

# 48. Implement Network Segmentation

Create two networks:

    docker network create frontend

    docker network create backend

Start the database:

    docker run -d \
      --name db \
      --network backend \
      postgres:16

Start the application:

    docker run -d \
      --name app \
      --network backend \
      invoice-service

Connect the application to the frontend network:

    docker network connect frontend app

Start the web container:

    docker run -d \
      --name web \
      --network frontend \
      -p 80:80 \
      nginx

Architecture:

    frontend
       │
       ├── web
       │
       └── app
             │
             │
           backend
             │
             └── db

---

# 49. Important Networking Rule

A container can communicate directly with another container when they share an appropriate Docker network.

Example:

    web ───── app

They share:

    frontend

Therefore they can communicate.

But:

    web ───── db

If they do not share a network, they do not have the same direct network relationship.

---

# 50. Lab 2 — Data That Survives, Apps That Connect

The second hands-on lab has four parts:

1. Persist
2. Bind
3. Back up
4. Connect

---

# 51. Lab 2 — Persist

## Objective

Prove that data survives container removal when stored in a named volume.

### Tasks

- Run PostgreSQL with a named volume.
- Create a table using `docker exec` and `psql`.
- Remove the PostgreSQL container.
- Recreate the container using the same volume.
- Verify that the table still exists.

### Expected Concept

    Container Removed
          ↓
    Volume Remains
          ↓
    New Container
          ↓
    Same Data

---

# 52. Lab 2 — Bind

## Objective

Understand bind mounts using Nginx.

### Tasks

- Serve a local host folder using Nginx.
- Edit a file on the host.
- Refresh the browser.
- Observe the change inside the container.
- Remount the directory as read-only.
- Try writing from inside the container.

### Expected Concept

    Host File
       ↓
    Bind Mount
       ↓
    Container
       ↓
    Nginx

Changes on the host appear inside the container.

With:

    :ro

the mounted directory becomes read-only from the container's perspective.

---

# 53. Lab 2 — Back Up

## Objective

Understand how to back up and restore Docker volume data.

### Tasks

- Back up the PostgreSQL volume to a tarball.
- Create a second volume.
- Restore the tarball into the second volume.
- Explain why `pg_dump` would be safer for a live PostgreSQL database.

### Important Lesson

A file-level volume copy is not automatically a consistent database backup.

For databases, prefer the database's own backup mechanism when appropriate.

---

# 54. Lab 2 — Connect

## Objective

Understand Docker service discovery and the difference between default and user-defined bridge networks.

### Tasks

- Create a user-defined network.
- Run containers on the network.
- Ping one container from another by name.
- Try the same name-based test on the default bridge.
- Record the result on both networks.

### Expected Learning

User-defined networks provide Docker's embedded DNS-based name resolution.

The default bridge does not provide the same name-resolution behavior.

---

# 55. Important Session 2 Commands

## Volume Commands

    docker volume create pgdata

    docker volume ls

    docker volume inspect pgdata

    docker volume rm pgdata

    docker volume prune

---

## Persistent PostgreSQL Example

    docker run -d --name db \
      -e POSTGRES_PASSWORD=secret \
      -v pgdata:/var/lib/postgresql/data \
      postgres:16

---

## Remove Container

    docker rm -f db

---

## Bind Mount

    docker run -d \
      -p 8080:80 \
      -v "$(pwd)/site":/usr/share/nginx/html:ro \
      nginx

---

## tmpfs

    docker run --tmpfs /app/cache app

---

## Backup Volume

    docker run --rm \
      -v pgdata:/data:ro \
      -v "$(pwd)":/backup \
      alpine tar czf /backup/pgdata.tgz -C /data .

---

## Restore Volume

    docker volume create pgdata2

    docker run --rm \
      -v pgdata2:/data \
      -v "$(pwd)":/backup \
      alpine tar xzf /backup/pgdata.tgz -C /data

---

## Network Commands

    docker network ls

    docker network create invoice-net

    docker network connect invoice-net web

    docker network inspect invoice-net

    docker network rm invoice-net

---

# 56. Important Session 2 Mental Model

## Persistent Data

    Container
        ↓
    Application
        ↓
    Volume
        ↓
    Persistent Data

The container can be replaced while the data remains.

---

## Bind Mount

    Host Directory
        ↓
    Bind Mount
        ↓
    Container Directory

Useful for local development and configuration.

---

## tmpfs

    Container
        ↓
    tmpfs
        ↓
    Host Memory
        ↓
    Data disappears when container stops

---

## Container Networking

    Container A
        ↓
    Docker Network
        ↓
    Container B

---

## Service Discovery

    app
     ↓
    db:5432
     ↓
    Docker Embedded DNS
     ↓
    Database Container IP

The application does not need to hard-code the database IP.

---

# 57. Session 2 — Key Production Principles

## Principle 1

> Containers are disposable.

Do not treat containers like traditional servers that are manually repaired.

---

## Principle 2

> Persistent data should live outside the container writable layer.

Use appropriate storage such as volumes.

---

## Principle 3

> Prefer named volumes for important persistent Docker-managed data.

Example:

    pgdata

---

## Principle 4

> Use bind mounts mainly when the host filesystem itself needs to be directly mapped into the container.

Typical example:

    Local development source code

---

## Principle 5

> Use read-only mounts when a container only needs to read data.

Example:

    :ro

---

## Principle 6

> Do not hard-code container IP addresses.

Container IP addresses can change.

Use service/container names on user-defined networks.

---

## Principle 7

> Use user-defined networks for multi-container applications.

They provide name-based service discovery through Docker's embedded DNS.

---

## Principle 8

> Segment networks according to application communication requirements.

Example:

    frontend
       ↓
    web + app

    backend
       ↓
    app + db

This prevents unnecessary direct connectivity.

---

## Principle 9

> Use database-native backup mechanisms for consistent database backups when appropriate.

For PostgreSQL, the presentation specifically points to:

    pg_dump

as a safer approach than copying live database files.

---

# 58. Interview Questions — Session 2

## Q1. Why does data disappear when a Docker container is removed?

Because data written only to the container's writable layer belongs to that container. Removing the container removes that writable layer.

---

## Q2. Where should persistent application data be stored?

Outside the container's writable layer, using an appropriate persistent storage mechanism such as a Docker volume.

---

## Q3. What is a Docker volume?

A Docker-managed storage location that can be mounted into containers and can survive container removal.

---

## Q4. What is a bind mount?

A specific directory or file on the host mapped into a container.

---

## Q5. What is tmpfs?

A memory-backed mount that is not written to disk and disappears when the container stops.

---

## Q6. Volume vs bind mount?

    Volume
    → Managed by Docker
    → Good for production persistent data

    Bind Mount
    → Uses a specific host path
    → Good for development and configuration

---

## Q7. Why are named volumes preferred for important data?

Because they are easier to identify, reuse, manage, back up, and restore.

---

## Q8. What happens to a volume when its container is removed?

The volume can remain intact and can be mounted into another container.

---

## Q9. What is the default Docker bridge?

It is the default bridge network to which containers are connected when no other network is specified.

---

## Q10. Why should we avoid relying on container IP addresses?

Container IP addresses can change when containers are restarted or recreated.

---

## Q11. How do containers discover each other by name?

Containers connected to a user-defined network can use Docker's embedded DNS to resolve container/service names.

---

## Q12. What is a user-defined bridge network?

A Docker-created bridge network that provides better isolation and name-based service discovery for containers.

---

## Q13. What are Docker network drivers?

Important drivers covered in this session are:

    bridge
    host
    none
    overlay
    macvlan

---

## Q14. What is host networking?

The container uses the host's network stack directly.

There is no separate Docker network isolation and `-p` is ignored in this mode.

---

## Q15. What is an overlay network?

A network that can span multiple Docker hosts and is commonly used by Docker Swarm services across nodes.

---

## Q16. What is network segmentation?

Separating application components into different networks so that only the required services can communicate directly.

---

# 59. Session 2 Final Revision

## Storage

    Container Writable Layer
            ↓
    Temporary Container Data
            ↓
    Lost when container is removed

    Docker Volume
            ↓
    Persistent Data
            ↓
    Survives Container Removal

---

## Three Storage Methods

    Volume
      ↓
    Docker-managed persistent data

    Bind Mount
      ↓
    Host directory mapped into container

    tmpfs
      ↓
    Host memory
      ↓
    Temporary data

---

## Networking

    Container
        ↓
    Network Namespace
        ↓
    Docker Network
        ↓
    Other Containers

---

## Default Bridge

    Container
        ↓
    docker0
        ↓
    IP-based communication

Names do not resolve automatically on the default bridge.

---

## User-Defined Network

    docker network create invoice-net
              ↓
    ┌─────────────────────┐
    │    invoice-net      │
    │                     │
    │  app  ───────→  db  │
    │                     │
    └─────────────────────┘
              ↓
    Docker Embedded DNS
              ↓
          db → IP

---

## Network Segmentation

    frontend network
       ↓
    web ↔ app

    backend network
       ↓
    app ↔ db

    web ✕ db

This limits unnecessary direct communication.

---

# 60. Complete Session 2 Flow

    Container
       ↓
    Writable Layer
       ↓
    Data disappears when container is removed
       ↓
    Persistent Data Requirement
       ↓
    Docker Volume / Bind Mount / tmpfs
       ↓
    Persistent or Temporary External Storage
       ↓
    Multiple Containers
       ↓
    Docker Networking
       ↓
    User-Defined Network
       ↓
    Docker Embedded DNS
       ↓
    Service Discovery
       ↓
    Network Segmentation
       ↓
    Multi-Container Application

---

# 61. Final Session 2 Understanding

The most important idea from Session 2 is:

> **Containers should be replaceable, while important data should survive independently of the container.**

The second major idea is:

> **Containers should communicate through Docker networks using service names rather than hard-coded container IP addresses.**

The overall architecture becomes:

    Persistent Data
          ↓
       Volume
          ↓
    ┌───────────────┐
    │   Container   │
    │   Application │
    └───────┬───────┘
            │
       Docker Network
            │
      ┌─────┴─────┐
      ↓           ↓
    app           db
      │           │
      └─────┬─────┘
            ↓
     Docker Embedded DNS
            ↓
       Service Discovery

This gives us the foundation needed for the next stage of Docker learning: building images, Docker Compose, orchestration, and CI/CD integration.