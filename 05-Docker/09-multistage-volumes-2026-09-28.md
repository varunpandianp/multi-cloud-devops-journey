# Docker Study Notes
## Session Date: 28-09-2026

## Topics Covered

1. Multi-Stage Docker Build
2. Why We Need Multi-Stage Build
3. Multi-Stage Build Architecture
4. Alpine Linux
5. ENTRYPOINT
6. CMD
7. ENTRYPOINT vs CMD
8. Docker Storage
9. Volumes
10. Bind Mounts
11. tmpfs Mounts
12. When to Use Each Storage Type
13. Production Best Practices
14. Interview Questions
15. Learning Log

---

# 1. Multi-Stage Docker Build

## What is a Multi-Stage Build?

A multi-stage Docker build means using multiple `FROM` instructions in the same Dockerfile.

Each `FROM` starts a separate build stage.

The main idea is:

> Use one stage to BUILD the application and another stage to RUN the application.

Example:

    FROM maven:3.9-eclipse-temurin-17 AS build

    # Build application
    ...

    FROM eclipse-temurin:17-jre-alpine

    # Run application
    ...

The build stage contains the tools required to build the application.

The runtime stage contains only the tools required to run the application.

---

# 2. Why Do We Need Multi-Stage Builds?

Suppose we have a Java application.

To BUILD the application, we may need:

- Maven
- JDK
- Source code
- Maven dependencies
- Build tools
- Testing tools

But after the application has been built, the production container may only need:

- JRE
- Application JAR

We do not need Maven or the full JDK to run the already-built application.

Therefore, we separate:

    BUILD ENVIRONMENT

from:

    RUNTIME ENVIRONMENT

---

# 3. Single-Stage Build Problem

In a single-stage build, the final image may contain:

    Maven
    JDK
    Source Code
    Dependencies
    Build Tools
    Application

Architecture:

    Single Production Image
            |
            +-- Maven
            +-- JDK
            +-- Source Code
            +-- Dependencies
            +-- Build Tools
            +-- Application

The problem is that production does not need all of these components.

---

# 4. Problems With Single-Stage Builds

## 4.1 Larger Image

More software means a larger Docker image.

Larger images take more time to:

- Build
- Push
- Pull
- Transfer
- Deploy

---

## 4.2 More Security Vulnerabilities

Every additional package or tool can potentially contain vulnerabilities.

If unnecessary tools are removed from the production image:

    Fewer packages
          |
          v
    Smaller attack surface
          |
          v
    Fewer vulnerabilities to manage

---

## 4.3 Larger Attack Surface

Production does not need:

- Compilers
- Maven
- Build tools
- Source code

Keeping unnecessary tools inside the production container increases the attack surface.

---

## 4.4 Difficult to Maintain

The final image contains tools that are not required to run the application.

A production image should contain only what is necessary to run the application.

---

# 5. Multi-Stage Build Architecture

The architecture is:

    Dockerfile
        |
        +-----------------------+
        |                       |
        v                       v
    BUILD STAGE             RUNTIME STAGE
    -----------             -------------
    Maven                   JRE
    JDK                     Application JAR
    Source Code             Runtime files
    Dependencies
    Build Tools
        |
        |
        v
    Build Application
        |
        v
    Application Artifact
        |
        |
        +-----------------------+
                                |
                                v
                         COPY --from=build
                                |
                                v
                         FINAL PRODUCTION
                              IMAGE

The important concept is:

    BUILD STAGE
         |
         | Build application
         v
    Application Artifact
         |
         | COPY --from=build
         v
    RUNTIME STAGE
         |
         v
    FINAL IMAGE

Only the required application artifact is copied into the final stage.

---

# 6. Multi-Stage Dockerfile Example

    # Stage 1 - Build
    FROM maven:3.9-eclipse-temurin-17 AS build

    WORKDIR /src

    COPY pom.xml .

    RUN mvn -B dependency:go-offline

    COPY src ./src

    RUN mvn -B package -DskipTests


    # Stage 2 - Runtime
    FROM eclipse-temurin:17-jre-alpine

    WORKDIR /app

    COPY --from=build /src/target/*.jar app.jar

    USER 1000

    ENTRYPOINT ["java", "-jar", "app.jar"]

---

# 7. Understanding Stage 1

    FROM maven:3.9-eclipse-temurin-17 AS build

This creates the first stage.

The stage is named:

    build

This stage contains the tools required to build the application.

For example:

- Maven
- JDK
- Source code
- Dependencies
- Build tools

Its purpose is:

    BUILD THE APPLICATION

---

# 8. Build the Application

    RUN mvn -B package -DskipTests

Maven builds the application and creates the JAR file.

For example:

    /src/target/invoice-service.jar

The JAR is the application artifact that we need in the final runtime image.

---

# 9. Understanding Stage 2

    FROM eclipse-temurin:17-jre-alpine

This starts a completely new stage.

This is the runtime stage.

It only needs the environment required to RUN the application.

It does not need:

- Maven
- Full JDK
- Source code
- Build dependencies

The final image contains:

    JRE
      +
    Application JAR

---

# 10. What is Alpine?

Alpine is a lightweight Linux distribution.

Example:

    FROM eclipse-temurin:17-jre-alpine

This means that the Java runtime image is based on Alpine Linux.

Alpine is commonly used when we want a lightweight runtime environment.

The idea is:

    BUILD STAGE
    Maven + JDK + Source Code
            |
            | Build
            v
    Application JAR
            |
            | Copy only required artifact
            v
    RUNTIME STAGE
    JRE + Application JAR

---

# 11. COPY --from

    COPY --from=build /src/target/*.jar app.jar

This is one of the most important commands in a multi-stage Dockerfile.

It means:

> Copy the JAR file from the `build` stage into the current runtime stage.

Architecture:

    BUILD STAGE

    /src/target/invoice-service.jar
                  |
                  |
                  | COPY --from=build
                  v
    RUNTIME STAGE

    /app/app.jar

We do not copy the entire build environment.

We copy only the artifact required by the production container.

---

# 12. Advantages of Multi-Stage Builds

## 12.1 Smaller Images

Only runtime components are included.

---

## 12.2 Better Security

Build tools and unnecessary packages are not included in production.

---

## 12.3 Faster Deployment

Smaller images are faster to:

- Push
- Pull
- Transfer
- Deploy

---

## 12.4 Reproducible Builds

The build process can happen inside Docker.

Developers do not need to install the exact Maven/JDK environment locally.

---

## 12.5 Cleaner Production Environment

Production contains only what the application needs to run.

---

# 13. Multi-Stage Build Mental Model

    SOURCE CODE
         |
         v
    BUILD STAGE
         |
         | Maven / JDK / Compiler
         |
         v
    COMPILE
         |
         v
    APPLICATION ARTIFACT
         |
         | COPY --from
         v
    RUNTIME STAGE
         |
         | JRE / Runtime
         |
         v
    SMALL PRODUCTION IMAGE

---

# 14. Interview Answer: Multi-Stage Build

> A multi-stage Docker build uses multiple FROM instructions to separate the build environment from the runtime environment. We can use heavy tools such as Maven, JDK, or compilers in the build stage and copy only the final application artifact into a lightweight runtime stage. This reduces image size, attack surface, vulnerabilities, and deployment time.

---

# 15. ENTRYPOINT in Dockerfile

## What is ENTRYPOINT?

`ENTRYPOINT` defines the main executable or program that the container is intended to run.

Example:

    ENTRYPOINT ["java", "-jar", "app.jar"]

This tells Docker:

> When the container starts, run the Java application.

Architecture:

    Container Starts
          |
          v
    ENTRYPOINT
          |
          v
    java -jar app.jar

---

# 16. CMD in Dockerfile

## What is CMD?

`CMD` defines the default command or default arguments for a container.

Example:

    CMD ["localhost"]

CMD can easily be overridden when running the container.

---

# 17. ENTRYPOINT vs CMD

The easiest way to remember:

    ENTRYPOINT = MAIN PROGRAM / FIXED EXECUTABLE

    CMD = DEFAULT COMMAND / DEFAULT ARGUMENTS

Think:

    ENTRYPOINT = What program should run?

    CMD = What should be the default input/arguments?

---

# 18. ENTRYPOINT + CMD Example

Dockerfile:

    FROM alpine:3.20

    ENTRYPOINT ["ping"]

    CMD ["localhost"]

When we run:

    docker run myimage

Docker runs:

    ping localhost

Because:

    ENTRYPOINT = ping
    CMD        = localhost

---

# 19. Overriding CMD

If we run:

    docker run myimage 8.8.8.8

Docker runs:

    ping 8.8.8.8

Why?

Because:

    ENTRYPOINT = ping
    CMD        = localhost

The supplied `8.8.8.8` replaces the default CMD argument.

Architecture:

    ENTRYPOINT ["ping"]
    CMD ["localhost"]

    Default:

    ping localhost

    Override:

    docker run image 8.8.8.8

    Result:

    ping 8.8.8.8

---

# 20. When Should We Use CMD?

Use `CMD` when:

- You want to provide a default command.
- You expect the command to be overridden.
- You want to provide default arguments.
- The container may be used with different commands.

Example:

    CMD ["sleep", "10"]

The user can override it:

    docker run image sleep 30

---

# 21. When Should We Use ENTRYPOINT?

Use `ENTRYPOINT` when:

- The container has one main purpose.
- You want a fixed executable.
- The container should behave like a specific application.
- You want arguments to be passed to that application.

Example:

    ENTRYPOINT ["java", "-jar", "app.jar"]

The container's main purpose is:

    Run this Java application

---

# 22. ENTRYPOINT + CMD Best Practice

A common pattern is:

    ENTRYPOINT ["java", "-jar", "app.jar"]

CMD can be used for default arguments when appropriate.

Conceptually:

    ENTRYPOINT
         |
         v
    Fixed application

    CMD
         |
         v
    Default arguments

---

# 23. CMD vs ENTRYPOINT Comparison

| Feature | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default command/arguments | Main executable |
| Easily overridden | Yes | Less easily overridden |
| Fixed application | Not ideal | Good |
| Default arguments | Good | Often combined with CMD |
| Example | `CMD ["localhost"]` | `ENTRYPOINT ["ping"]` |

---

# 24. Easy Memory Trick

    ENTRYPOINT = "WHO ARE YOU?"

    CMD = "WHAT ARE YOUR DEFAULT ARGUMENTS?"

---

# 25. Docker Storage

## Why Do We Need Docker Storage?

Containers are designed to be disposable.

Anything written only to the container's writable layer belongs to that container.

When the container is removed, that data is lost.

Therefore:

    Container
        |
        | Application Data
        v
    Do NOT depend only on writable layer

Instead:

    Container
        |
        | Mount
        v
    External Storage

---

# 26. Important Docker Storage Rule

> Containers are disposable. Persistent data should live outside the container.

Examples of data that should usually survive container replacement:

- Database data
- User uploads
- Application state
- Persistent files

---

# 27. Three Ways to Store Data in Docker

The three important Docker storage mechanisms are:

    1. Volumes
    2. Bind Mounts
    3. tmpfs Mounts

---

# 28. Docker Volumes

## What is a Docker Volume?

A Docker volume is persistent storage managed by Docker.

Create a volume:

    docker volume create pgdata

Use it:

    docker run -d \
      --mount type=volume,source=pgdata,target=/var/lib/postgresql/data \
      postgres

Short `-v` syntax:

    docker run -d \
      -v pgdata:/var/lib/postgresql/data \
      postgres

---

# 29. Volume Architecture

    Docker Host
         |
         v
    Docker-managed Storage
         |
         v
       pgdata
         |
       +---+---+
       |       |
       v       v
    Container A  Container B
       |       |
       +---data-+

Multiple containers can mount the same volume.

---

# 30. When Should We Use Volumes?

Use Docker volumes when you need:

- Persistent application data
- Database storage
- Data that survives container replacement
- Data shared between containers
- Docker-managed storage

Typical examples:

- PostgreSQL
- MySQL
- MongoDB
- Application persistent state
- Uploaded files

---

# 31. Why Volumes Are Usually the Default for Production Data

Advantages:

- Managed by Docker
- Portable
- Easy to back up
- Designed for persistent data
- Less dependent on host directory structure

For production persistent data, named volumes are generally preferred over storing data only in the container writable layer.

---

# 32. Bind Mounts

## What is a Bind Mount?

A bind mount maps a specific file or directory from the Docker host into the container.

Example:

    docker run -d \
      -v "$(pwd)/src":/app/src \
      nginx

Architecture:

    Docker Host

    /home/user/project/src
              |
              | Bind Mount
              v
    Container

    /app/src

Changes on the host can immediately appear inside the container and vice versa.

---

# 33. When Should We Use Bind Mounts?

Use bind mounts mainly when you need direct access to host files.

Typical use cases:

## Local Development

You edit source code on your host:

    Host:
    src/app.java
          |
          | Bind Mount
          v
    Container:
    /app/src/app.java

---

## Configuration Files

Example:

    Host:
    ./nginx.conf

    Container:
    /etc/nginx/nginx.conf

---

## Development Tools

Bind mounts are useful when the container needs to work directly with files from the local machine.

---

# 34. Bind Mount Disadvantages

Bind mounts depend on the host filesystem.

For example:

    Host A:
    /home/user/project

    Host B:
    /opt/project

The directory layout can differ between hosts.

Therefore:

    Bind Mount
        |
        v
    More dependent on host filesystem

A container can also modify host files if the mount is read-write.

---

# 35. tmpfs Mounts

## What is tmpfs?

`tmpfs` stores data in memory instead of persistent disk storage.

Example:

    docker run -d \
      --tmpfs /app/cache \
      nginx

Architecture:

    Container
        |
        v
    /app/cache
        |
        v
       RAM

The data is temporary.

When the container stops or is removed, the tmpfs data does not persist.

---

# 36. When Should We Use tmpfs?

Use tmpfs for temporary data such as:

- Temporary files
- Scratch space
- Caches
- Memory-backed temporary data
- Data that should not be persisted to disk

Example:

    Application
         |
         v
    Temporary Cache
         |
         v
       tmpfs
         |
         v
        RAM

Important:

> tmpfs is not a complete secrets-management solution. For production secrets, use an appropriate secret-management mechanism.

---

# 37. tmpfs Advantages

    RAM
     |
     v
    Fast
     |
     v
    No persistent disk storage
     |
     v
    Data disappears with container lifecycle

Use tmpfs when persistence is NOT required.

---

# 38. Volumes vs Bind Mounts vs tmpfs

| Feature | Volume | Bind Mount | tmpfs |
|---|---|---|---|
| Managed by Docker | Yes | No | Docker manages mount |
| Stored on disk | Yes | Yes | No, memory |
| Persistent | Yes | Yes | No |
| Host path required | No | Yes | No |
| Best for | Production data | Development/files | Temporary data |
| Database | Excellent | Usually avoid | No |
| Local development | Possible | Excellent | Rare |
| Cache | Possible | Possible | Excellent |
| Survives container removal | Yes | Yes | No |

---

# 39. Easy Way to Remember Docker Storage

    VOLUME
        |
        v
    "I want Docker-managed persistent storage."


    BIND MOUNT
        |
        v
    "I want to use a specific folder/file from my host."


    TMPFS
        |
        v
    "I want temporary data in memory."

---

# 40. Real-World Storage Architecture

Suppose we have:

    Application
    Database
    Source Code
    Temporary Cache

We can choose:

    Application
        |
        +---- Database Data
        |          |
        |          v
        |       VOLUME
        |
        +---- Source Code
        |          |
        |          v
        |       BIND MOUNT
        |
        +---- Temporary Cache
                   |
                   v
                 TMPFS

---

# 41. Production Database Example

For PostgreSQL:

    PostgreSQL Container
            |
            v
         pgdata
          Volume
            |
            v
    Persistent Database Data

If the container is removed:

    Old PostgreSQL Container
            |
            v
         Removed

    pgdata Volume
            |
            v
         Remains

Then a new PostgreSQL container can mount the same volume:

    New PostgreSQL Container
            |
            v
       Mount pgdata
            |
            v
      Old Database Data

This is why database data should not be stored only in the container writable layer.

---

# 42. Read-Only Mounts

A mount can be made read-only.

Example:

    docker run -d \
      -v "$(pwd)/site":/usr/share/nginx/html:ro \
      nginx

Here:

    ro = read-only

The container can read the files but cannot modify them.

This is useful when the container only needs to consume data.

---

# 43. Modern --mount Syntax

Docker supports the short `-v` syntax:

    -v pgdata:/var/lib/postgresql/data

Docker also supports the more explicit `--mount` syntax:

    --mount type=volume,source=pgdata,target=/var/lib/postgresql/data

The `--mount` syntax is more explicit and useful in scripts.

---

# 44. Docker Storage Commands

## Create Volume

    docker volume create pgdata

## List Volumes

    docker volume ls

## Inspect Volume

    docker volume inspect pgdata

## Use Volume

    docker run -d \
      --mount type=volume,source=pgdata,target=/var/lib/postgresql/data \
      postgres

## Bind Mount

    docker run -d \
      -v "$(pwd)/src":/app/src \
      nginx

## tmpfs

    docker run -d \
      --tmpfs /app/cache \
      nginx

---

# 45. Volume Backup Concept

A Docker volume can be backed up by mounting the volume into a temporary container and creating an archive.

Example:

    docker run --rm \
      -v pgdata:/data:ro \
      -v "$(pwd)":/backup \
      alpine tar czf /backup/pgdata.tgz -C /data .

Basic pattern:

    Volume
       |
       v
    Temporary Container
       |
       v
    Backup Archive

For databases, be careful when copying live database files.

A consistent database-native backup or snapshot is generally safer.

---

# 46. Volume Sharing

The same named volume can be mounted into multiple containers.

Example:

    pgdata volume
         |
       +---+---+
       |       |
       v       v
    Container A  Container B
       |       |
       +--data-+

If multiple containers consume the same data, use read-only access for consumers when possible.

Example:

    Writer Container
          |
          | Read-Write
          v
       Volume
          |
          | Read-Only
          v
    Reader Container

---

# 47. Docker Storage Production Best Practices

## Use Volumes for Persistent Data

Especially for:

- Database data
- Application state
- Persistent files

---

## Use Bind Mounts Mainly for Development

Useful for:

- Source code
- Local configuration
- Development workflows

---

## Use tmpfs for Temporary Data

Useful for:

- Caches
- Scratch space
- Temporary files

---

## Use Read-Only Mounts When Possible

If a container only needs to read data:

    :ro

This reduces accidental modification.

---

## Do Not Store Important Data Only in the Container Writable Layer

Containers should be disposable.

Persistent data should have a separate lifecycle.

---

# 48. Multi-Stage Build + Storage Together

A typical production architecture could look like:

    SOURCE CODE
         |
         v
    BUILD STAGE
    Maven + JDK
         |
         v
    Application JAR
         |
         v
    RUNTIME STAGE
    JRE + Application
         |
         v
    Docker Container
        / \
       /   \
      v     v
    Application  Data
                   |
                   v
                 Volume

The application image and persistent data have different lifecycles.

    IMAGE
      |
      | Can be replaced
      v
    CONTAINER
      |
      | Can be replaced
      v
    VOLUME
      |
      | Persistent data
      v
    DATABASE / APPLICATION DATA

---

# 49. Important Interview Questions

## Q1. What is a multi-stage Docker build?

A multi-stage Docker build uses multiple `FROM` instructions to separate the build environment from the runtime environment.

---

## Q2. Why do we use multi-stage builds?

Main reasons:

- Reduce image size
- Remove build tools from production
- Reduce attack surface
- Reduce vulnerabilities
- Improve deployment efficiency
- Create reproducible builds

---

## Q3. What does COPY --from do?

It copies files from a previous build stage into the current stage.

Example:

    COPY --from=build /src/target/*.jar app.jar

---

## Q4. What is ENTRYPOINT?

ENTRYPOINT defines the main executable that the container runs.

Example:

    ENTRYPOINT ["java", "-jar", "app.jar"]

---

## Q5. What is CMD?

CMD defines the default command or default arguments.

It can easily be overridden.

---

## Q6. What is the difference between CMD and ENTRYPOINT?

    ENTRYPOINT = Main/fixed executable
    CMD        = Default command/arguments

Example:

    ENTRYPOINT ["ping"]
    CMD ["localhost"]

Default:

    ping localhost

Override:

    docker run image 8.8.8.8

Result:

    ping 8.8.8.8

---

## Q7. What are the three Docker storage mechanisms?

    1. Volumes
    2. Bind Mounts
    3. tmpfs Mounts

---

## Q8. When should you use a Docker volume?

Use a Docker volume for persistent Docker-managed data, especially databases and application state.

---

## Q9. When should you use a bind mount?

Use a bind mount when you need a specific host file or directory inside the container, especially during local development.

---

## Q10. When should you use tmpfs?

Use tmpfs for temporary, non-persistent data that should stay in memory.

---

# 50. Production Best Practices

## Docker Images

- Use multi-stage builds.
- Keep compilers and build tools out of production.
- Start from small official base images.
- Pin image versions.
- Run applications as non-root users.
- Scan images for vulnerabilities.
- Do not put secrets inside the image.
- Avoid unnecessary packages.

---

## Docker Storage

- Keep persistent data outside the container writable layer.
- Use named volumes for production persistent data.
- Use bind mounts mainly for development and host-specific files.
- Use tmpfs for temporary memory-backed data.
- Use read-only mounts where possible.
- Back up important persistent data.
- Use consistent database backup methods.

---

# 51. Complete Mental Model

## Multi-Stage Build

    SOURCE CODE
         |
         v
    BUILD STAGE
         |
         | Maven / JDK / Compiler
         |
         v
    APPLICATION ARTIFACT
         |
         | COPY --from
         v
    RUNTIME STAGE
         |
         | JRE / Runtime
         |
         v
    SMALL PRODUCTION IMAGE

---

## ENTRYPOINT vs CMD

    ENTRYPOINT
         |
         v
    MAIN PROGRAM / EXECUTABLE

    CMD
         |
         v
    DEFAULT ARGUMENTS

---

## Docker Storage

                     Docker Storage
                           |
              +------------+------------+
              |            |            |
              v            v            v
           Volume      Bind Mount     tmpfs
              |            |            |
         Persistent     Host Files      RAM
         Docker         Development    Temporary
         Managed        Config Files   Data

---

# 52. Final Memory Notes

    MULTI-STAGE BUILD
    =================

    BUILD BIG
        |
        v
    Maven / JDK / Compiler
        |
        v
    CREATE ARTIFACT
        |
        v
    COPY ONLY ARTIFACT
        |
        v
    RUN SMALL
        |
        v
    Production Runtime Image


    ENTRYPOINT vs CMD
    =================

    ENTRYPOINT = MAIN PROGRAM

    CMD = DEFAULT ARGUMENTS


    DOCKER STORAGE
    ==============

    VOLUME
        |
        v
    Persistent Docker-managed data

    BIND MOUNT
        |
        v
    Host file/directory

    TMPFS
        |
        v
    Temporary memory storage


    PRODUCTION RULE
    ===============

    Container = Disposable
    Image     = Application Package
    Volume    = Persistent Data

---

# 53. Session Learning Log

## Date: 28-09-2026

Today I learned:

- Multi-stage Docker builds
- Why multi-stage builds are required
- Single-stage vs multi-stage builds
- Build stage and runtime stage
- Multi-stage build architecture
- `FROM ... AS build`
- `COPY --from`
- Alpine Linux
- Smaller production images
- Security benefits of multi-stage builds
- Docker `ENTRYPOINT`
- Docker `CMD`
- Difference between ENTRYPOINT and CMD
- How CMD can be overridden
- Docker container writable layer
- Docker persistent storage
- Docker volumes
- Bind mounts
- tmpfs mounts
- Volume use cases
- Bind mount use cases
- tmpfs use cases
- `-v` syntax
- `--mount` syntax
- Read-only mounts
- Volume sharing
- Volume backup concepts
- Docker storage production best practices

---

# 54. One-Line Revision

    Multi-stage build = Build with a big image, run with a small image.

    ENTRYPOINT = Main executable.

    CMD = Default command or arguments.

    Volume = Docker-managed persistent storage.

    Bind Mount = Host directory/file mounted into container.

    tmpfs = Temporary memory-backed storage.

    Container = Disposable.

    Persistent Data = Store outside the container.

# END OF SESSION 28-09-2026