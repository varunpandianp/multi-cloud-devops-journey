# Module 07 — Containerization with Docker

## 📚 Course / Module

**Module:** Containerization with Docker  
**Session:** Session 3 — Images, Compose, Swarm and Jenkins  
**Duration:** 2 Hours  
**Slides Covered:** 31–49

---

# 🎯 Learning Objectives

By the end of Session 3, I should be able to:

- Understand what a Dockerfile is.
- Understand important Dockerfile instructions.
- Understand `FROM`, `RUN`, `COPY`, `ADD`, `WORKDIR`, `ENV`, `ARG`, `HEALTHCHECK`, `EXPOSE`, `USER`, `VOLUME`, `CMD`, `ENTRYPOINT`, and `LABEL`.
- Understand the difference between `CMD` and `ENTRYPOINT`.
- Build a Docker image from a Dockerfile.
- Understand Docker image layers and layer caching.
- Use `.dockerignore`.
- Optimize Dockerfile instruction order.
- Understand multi-stage Docker builds.
- Build smaller production images.
- Run containers as non-root users.
- Understand production-style container execution.
- Configure resource limits.
- Configure health checks.
- Configure restart policies.
- Use environment files.
- Tag and push Docker images to a registry.
- Understand immutable image tags.
- Understand Docker Compose.
- Define a complete multi-container application using `compose.yaml`.
- Understand Compose services, networks, volumes and environment variables.
- Understand `depends_on`.
- Manage applications using Docker Compose commands.
- Understand Docker Swarm.
- Understand nodes, services, tasks and stacks.
- Scale services in Swarm.
- Understand self-healing and rolling updates.
- Understand Jenkins and Docker integration.
- Build, push and deploy Docker images through Jenkins.
- Understand the complete CI/CD flow from commit to deployment.
- Troubleshoot common Docker problems.
- Follow Docker production best practices.

---

# Session 3 — Images, Compose, Swarm and Jenkins

## 📌 Topics Covered

1. Dockerfiles and Custom Images
2. Dockerfile Line by Line
3. Important Dockerfile Instructions
4. CMD vs ENTRYPOINT
5. Building Docker Images
6. Docker Layer Cache
7. `.dockerignore`
8. Multi-Stage Builds
9. Production Image Best Practices
10. Running Containers in Production
11. Resource Limits
12. Health Checks
13. Environment Files
14. Restart Policies
15. Docker Image Tagging
16. Docker Registry
17. Pushing and Pulling Images
18. Docker Compose
19. `compose.yaml`
20. Compose Services
21. Compose Volumes and Networks
22. Compose Environment Variables
23. `depends_on`
24. Docker Compose Commands
25. Docker Swarm
26. Swarm Architecture
27. Nodes, Services, Tasks and Stacks
28. Swarm Self-Healing
29. Rolling Updates
30. Swarm Load Balancing
31. Swarm Commands
32. Jenkins and Docker Integration
33. Jenkins as Docker Build Environment
34. Jenkins as Docker Image Pipeline
35. Build → Test → Image → Registry → Deploy
36. Docker Troubleshooting
37. Production Docker Practices
38. Lab 3 — Build It, Compose It, Ship It
39. Module Recap

---

# 1. Dockerfile

## What is a Dockerfile?

A Dockerfile is a recipe used to build a Docker image.

It contains instructions that describe:

- Which base image to use.
- Which files to copy.
- Which commands to execute.
- Which environment variables to define.
- Which user should run the application.
- Which port the application uses.
- Which command should start the application.

### Basic Flow

    Dockerfile
         ↓
    docker build
         ↓
    Docker Image
         ↓
    docker run
         ↓
    Docker Container

A Dockerfile makes the application environment reproducible.

Instead of manually installing everything inside a container, we define the environment in the Dockerfile.

---

# 2. Dockerfile Example

The presentation provides the following example:

    FROM eclipse-temurin:17-jre-alpine
    LABEL maintainer="platform@meridian.example"
    WORKDIR /app
    COPY target/invoice-service.jar app.jar
    ENV JAVA_OPTS="-Xmx512m"
    EXPOSE 8080
    USER 1000
    ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

Each instruction has a specific purpose.

---

# 3. Dockerfile — `FROM`

## What is `FROM`?

`FROM` specifies the base image for the Docker image.

Example:

    FROM eclipse-temurin:17-jre-alpine

This means the application image starts from the Eclipse Temurin Java 17 JRE Alpine image.

### Why is it important?

Every Docker image needs a starting point.

The base image provides the basic environment required by the application.

### Mental Model

    Base Image
         ↓
    Your Dockerfile Instructions
         ↓
    Custom Docker Image

---

# 4. Dockerfile — `LABEL`

## What is `LABEL`?

`LABEL` adds metadata to the image.

Example:

    LABEL maintainer="platform@meridian.example"

Labels can contain information such as:

- Maintainer
- Owner
- Version
- Project information

### Example

    LABEL maintainer="platform@meridian.example"

The label does not run the application. It provides metadata about the image.

---

# 5. Dockerfile — `WORKDIR`

## What is `WORKDIR`?

`WORKDIR` sets the working directory inside the image/container.

Example:

    WORKDIR /app

After this instruction, subsequent instructions operate relative to:

    /app

### Example

    WORKDIR /app
    COPY target/invoice-service.jar app.jar

The file is copied to:

    /app/app.jar

---

# 6. Dockerfile — `COPY`

## What is `COPY`?

`COPY` copies files from the Docker build context into the image.

Example:

    COPY target/invoice-service.jar app.jar

This copies:

    target/invoice-service.jar

into:

    /app/app.jar

because the previous instruction set:

    WORKDIR /app

### Why is it important?

It allows us to put our application artifact into the image.

---

# 7. Dockerfile — `ENV`

## What is `ENV`?

`ENV` defines an environment variable available at runtime.

Example:

    ENV JAVA_OPTS="-Xmx512m"

The application can use:

    JAVA_OPTS

with the value:

    -Xmx512m

### Important Concept

Environment variables allow configuration to be supplied without hard-coding every value directly into the application.

---

# 8. Dockerfile — `EXPOSE`

## What is `EXPOSE`?

`EXPOSE` documents the port that the application listens on.

Example:

    EXPOSE 8080

This tells users:

> The application is expected to listen on port 8080.

### Important

`EXPOSE` does **not** publish the port to the host by itself.

To make the container reachable from outside, port publishing is still required.

Example:

    docker run -p 8080:8080 image

---

# 9. Dockerfile — `USER`

## What is `USER`?

`USER` specifies the user that runs the application inside the container.

Example:

    USER 1000

This means the application process runs as user ID 1000 instead of the default root user.

### Why is this important?

Running as a non-root user reduces the impact of a compromise.

### Production Principle

> Run application containers as a non-root user whenever possible.

---

# 10. Dockerfile — `ENTRYPOINT`

## What is `ENTRYPOINT`?

`ENTRYPOINT` specifies the main executable that the container runs.

Example:

    ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

When the container starts, this command starts the Java application.

### Mental Model

    docker run
         ↓
    Container starts
         ↓
    ENTRYPOINT executes
         ↓
    Application starts

---

# 11. Dockerfile Instructions You Need to Know

The presentation covers the following instructions:

| Instruction | Purpose |
|---|---|
| `FROM` | Base image / starts a build stage |
| `RUN` | Executes a command during image build |
| `COPY` | Copies files from build context |
| `ADD` | Similar to COPY, with additional behavior |
| `WORKDIR` | Sets working directory |
| `ENV` | Environment variable available at build/runtime |
| `ARG` | Build-time variable |
| `HEALTHCHECK` | Defines how Docker checks application health |
| `EXPOSE` | Documents application listening port |
| `USER` | Runs the process as a specified user |
| `VOLUME` | Declares a mount point |
| `CMD` | Provides default arguments/command |
| `ENTRYPOINT` | Defines the main executable |
| `LABEL` | Adds metadata |

---

# 12. Dockerfile — `RUN`

## What is `RUN`?

`RUN` executes a command while building the image.

Example:

    RUN apt-get update

The command executes during:

    docker build

It does not execute every time the container starts.

### Important Difference

    RUN
      ↓
    Image build time

    ENTRYPOINT / CMD
      ↓
    Container runtime

---

# 13. Dockerfile — `ADD`

## What is `ADD`?

`ADD` is similar to `COPY`, but it provides additional behavior such as:

- Fetching URLs
- Unpacking tar archives

Example:

    ADD application.tar.gz /app/

### Best Practice

The presentation recommends:

> Prefer `COPY` over `ADD`.

Why?

`COPY` has simpler and more predictable behavior.

The extra behavior of `ADD` is rarely required and can make Dockerfiles harder to reason about.

---

# 14. Dockerfile — `ARG`

## What is `ARG`?

`ARG` defines a variable available during image build time.

Example concept:

    ARG VERSION=1.0

The value is available during:

    docker build

Unlike `ENV`, `ARG` is primarily a build-time variable.

---

# 15. Dockerfile — `HEALTHCHECK`

## What is `HEALTHCHECK`?

`HEALTHCHECK` defines how Docker checks whether an application is healthy.

Conceptually:

    Container
         ↓
    Health Check
         ↓
    Healthy / Unhealthy

The presentation later demonstrates a runtime health check using:

    --health-cmd

Health checks are important because a container can be running while the application inside it is not actually healthy.

---

# 16. Dockerfile — `VOLUME`

## What is `VOLUME`?

`VOLUME` declares a mount point for data.

Example concept:

    VOLUME /data

It identifies a location intended for persistent data.

Session 2 covered the actual use of volumes for persistent storage.

---

# 17. CMD vs ENTRYPOINT

This is an important Docker interview topic.

## ENTRYPOINT

`ENTRYPOINT` defines the program that the container runs.

## CMD

`CMD` supplies default arguments or a default command.

### Example

    ENTRYPOINT ["ping"]
    CMD ["localhost"]

Running:

    docker run image

results conceptually in:

    ping localhost

If we run:

    docker run image 8.8.8.8

the argument changes.

The result becomes:

    ping 8.8.8.8

### Key Concept

> `ENTRYPOINT` is the program; `CMD` supplies its default arguments.

---

# 18. Building a Docker Image

The presentation uses:

    docker build -t invoice-service:1.0 .

### Meaning

    docker build
        ↓
    Build an image

    -t invoice-service:1.0
        ↓
    Give the image a name and tag

    .
        ↓
    Use the current directory as the build context

---

# 19. List the Image

After building:

    docker images invoice-service

This displays the image information.

---

# 20. Run the Custom Image

Example:

    docker run -d -p 8080:8080 invoice-service:1.0

Flow:

    Dockerfile
         ↓
    docker build
         ↓
    invoice-service:1.0
         ↓
    docker run
         ↓
    Running Container
         ↓
    Port 8080

---

# 21. Docker Image Layers

Each Dockerfile instruction can create a layer.

Conceptually:

    FROM
      ↓
    Layer

    RUN
      ↓
    Layer

    COPY
      ↓
    Layer

    ENV
      ↓
    Layer

    Final Image
      ↓
    Multiple Layers

Docker caches layers and can reuse them between builds.

---

# 22. Docker Layer Cache

Layer caching can make Docker builds much faster.

Consider this:

    COPY . .
    RUN mvn dependency:go-offline
    RUN mvn package

If source code changes, the `COPY . .` layer changes.

That can invalidate the following cache layers.

This can cause dependencies to be downloaded again.

---

# 23. Better Dockerfile Instruction Order

The presentation recommends ordering instructions from:

    Least Frequently Changed
             ↓
    Most Frequently Changed

Instead of:

    COPY . .
    RUN mvn dependency:go-offline
    RUN mvn package

Use:

    COPY pom.xml .
    RUN mvn dependency:go-offline
    COPY src ./src
    RUN mvn package

### Why?

The dependency layer can remain cached as long as:

    pom.xml

has not changed.

Only the application source changes frequently.

Therefore:

    pom.xml
       ↓
    Dependencies
       ↓
    Cached

    src/
       ↓
    Changes frequently

This makes builds faster.

---

# 24. `.dockerignore`

A `.dockerignore` file tells Docker which files/directories should not be sent as part of the build context.

Example:

    .git
    target/*.original
    *.log
    .env

### Why is `.dockerignore` Important?

Without `.dockerignore`, unnecessary files can be sent to the Docker daemon.

This can:

- Increase build context size
- Slow down builds
- Include unnecessary files
- Accidentally include sensitive files

---

# 25. Multi-Stage Docker Builds

## What is a Multi-Stage Build?

A multi-stage Dockerfile uses multiple `FROM` instructions to separate building the application from running the application.

Example:

    Stage 1
    Build Application
          ↓
    Maven + JDK
          ↓
    JAR

          ↓

    Stage 2
    Run Application
          ↓
    JRE
          ↓
    JAR

The build tools do not need to remain in the final image.

---

# 26. Multi-Stage Build Example

Stage 1:

    FROM maven:3.9-eclipse-temurin-17 AS build
    WORKDIR /src
    COPY pom.xml .
    RUN mvn -B dependency:go-offline
    COPY src ./src
    RUN mvn -B package -DskipTests

Stage 2:

    FROM eclipse-temurin:17-jre-alpine
    WORKDIR /app
    COPY --from=build /src/target/*.jar app.jar
    USER 1000
    ENTRYPOINT ["java", "-jar", "app.jar"]

---

# 27. Why Use Multi-Stage Builds?

## Smaller Images

The final image contains:

    JRE
    +
    Application JAR

It does not contain:

    Maven
    JDK
    Source Code

---

## Fewer Vulnerabilities

Every unnecessary tool removed from the production image is one less tool that could potentially be exploited.

It also reduces the number of packages that need vulnerability patching.

---

## Reproducible Build

The complete build can run inside Docker.

Developers do not need to install Maven locally just to produce the application image.

---

# 28. Typical Multi-Stage Result

The presentation gives a typical Java-service example:

    Full Maven Image
        ↓
    Roughly 600 MB

    JRE-only Final Image
        ↓
    Under 200 MB

The exact size depends on the application and base images, but the important concept is:

> Build tools do not need to be included in the final production image.

---

# 29. Production Docker Image Practices

## DO

- Start from a small official base image.
- Pin the base image version.
- Run as a non-root user.
- Use multi-stage builds.
- Keep one main process per container.
- Scan images before shipping them.

---

## AVOID

- Secrets inside images.
- Using `:latest` as a base.
- Leaving package caches behind.
- Huge build contexts.
- Unnecessary tools in production images.

---

# 30. Secrets Should Not Be Baked Into Images

Do not put secrets directly into:

    ENV

or:

    COPY

because values can become part of image layers.

Someone with access to the image may be able to inspect those layers.

### Better Approach

Inject secrets/configuration at runtime.

Examples from the presentation include:

- Environment files
- Runtime configuration
- Orchestrator secret stores

---

# 31. Avoid `:latest`

Using:

    :latest

does not guarantee a fixed image version.

The upstream image can change.

Therefore today's build and next month's build may use different underlying content.

### Better Practice

Use an explicit version.

Example:

    nginx:1.27

or another traceable release identifier.

---

# 32. Scan Images Before Shipping

The presentation gives:

    docker scout cves invoice-service:1.0

It also mentions:

    Trivy

can be used in the pipeline.

### Goal

Detect vulnerabilities before the image reaches production.

---

# 33. One Process Per Container

The presentation recommends:

> One process per container.

This allows each component to be:

- Scaled independently
- Restarted independently
- Logged independently

Example:

    Web Container
         ↓
    Web Application

    Database Container
         ↓
    Database

Each component has its own lifecycle.

---

# 34. Production-Style Container Run

The presentation provides this example:

    docker run -d --name invoice \
      --restart unless-stopped \
      --memory 768m --cpus 1.0 \
      --network invoice-net \
      -p 8080:8080 \
      --env-file ./invoice.env \
      --health-cmd 'wget -qO- localhost:8080/health' \
      invoice-service:1.0

This combines many production concepts into one command.

---

# 35. Resource Limits

Example:

    --memory 768m --cpus 1.0

This limits the container's resource usage.

### Why?

Without resource limits, a leaking or badly behaving container could consume excessive resources and affect other workloads on the host.

### Important Principle

Resource limits help prevent one container from starving other containers.

---

# 36. Health Checks

Example:

    --health-cmd 'wget -qO- localhost:8080/health'

The health check tests the application's health endpoint.

The container can then be shown as:

    healthy

or:

    unhealthy

### Important Concept

A container can be running but the application inside it can still be unhealthy.

Health checks help distinguish:

    Process Running

from:

    Application Healthy

---

# 37. Environment Files

Example:

    --env-file ./invoice.env

The environment file contains configuration values.

### Why Use It?

It keeps configuration out of:

- The command line
- Shell history
- The Docker image

The presentation specifically recommends using environment files for configuration.

---

# 38. Restart Policy

Example:

    --restart unless-stopped

This causes the container to restart automatically after a Docker daemon restart or host reboot unless the user intentionally stopped it.

### Useful For

Long-running services.

---

# 39. Docker System Disk Usage

Command:

    docker system df

This shows disk usage from:

- Images
- Containers
- Volumes
- Build cache

Useful when investigating disk usage.

---

# 40. Docker System Prune

Command:

    docker system prune

This removes unused Docker resources such as:

- Stopped containers
- Unused networks
- Dangling images

Use cleanup commands carefully because resources that are no longer used may still be needed later.

---

# 41. Update Container Resources

Example:

    docker update --memory 1g invoice

This changes the memory limit of a running container.

---

# 42. Find Unhealthy Containers

Command:

    docker ps --filter health=unhealthy

This finds containers that are failing their health checks.

---

# 43. Docker Image Tagging

A tag identifies a particular image reference.

Example:

    invoice-service:1.0

A production image should use a meaningful version.

Possible traceable identifiers include:

- Build number
- Semantic version
- Git commit SHA

---

# 44. Push Image to Docker Hub

Login:

    docker login

Tag:

    docker tag invoice-service:1.0 \
      meridian/invoice-service:1.0

Push:

    docker push meridian/invoice-service:1.0

Pull it elsewhere:

    docker pull meridian/invoice-service:1.0

---

# 45. Private or Cloud Registry

The same concept applies to private registries.

Example:

    docker tag invoice-service:1.0 \
      registry.meridian.example/billing/invoice-service:1.0

Then:

    docker push registry.meridian.example/...

The registry stores the image so it can be pulled by other environments.

---

# 46. Image Tagging Best Practices

## Use Real Versions

Use:

    1.0

or:

    Build Number

or:

    Git Commit SHA

The goal is traceability.

---

## Never Overwrite Release Tags

If:

    1.0

can refer to two different images, it becomes difficult to know exactly which image is running.

Release tags should be treated as immutable.

---

## Use Access Tokens

For registry authentication, use a scoped access token instead of an account password, especially in CI systems.

---

# 47. Docker Compose

## What is Docker Compose?

Docker Compose allows an entire multi-container application to be defined in one file.

Usually:

    compose.yaml

Instead of manually typing many Docker commands, the application configuration is declared in the Compose file.

---

# 48. Compose Example

The presentation provides:

    services:
      db:
        image: postgres:16
        environment:
          POSTGRES_PASSWORD: ${DB_PASSWORD}
        volumes:
          - pgdata:/var/lib/postgresql/data
        networks: [backend]

      app:
        build: .
        ports: ["8080:8080"]
        environment:
          DB_HOST: db
        depends_on: [db]
        networks: [backend]

    volumes: { pgdata: {} }

    networks: { backend: {} }

---

# 49. What Compose Defines

The Compose file brings together concepts from Sessions 1 and 2:

    Images
    Ports
    Environment
    Volumes
    Networks
    Services

Instead of typing each configuration manually, it is declared in:

    compose.yaml

---

# 50. Compose Services

Example:

    services:
      db:
        image: postgres:16

      app:
        build: .

Here:

    db
       ↓
    PostgreSQL Service

    app
       ↓
    Application Service

Each service represents a containerized component of the application.

---

# 51. Service Names Become Hostnames

The application uses:

    DB_HOST: db

because:

    db

is the Compose service name.

Compose creates a network for the project, allowing services to communicate using service names.

### Flow

    app
     ↓
    db
     ↓
    Docker DNS
     ↓
    PostgreSQL Container

---

# 52. Compose Volumes

The Compose example defines:

    volumes:
      - pgdata:/var/lib/postgresql/data

and:

    volumes:
      pgdata: {}

This means PostgreSQL data is stored in a named volume.

Therefore:

    Container removed
         ↓
    Volume remains
         ↓
    Data survives

---

# 53. Compose Networks

The example defines:

    networks:
      - backend

and:

    networks:
      backend: {}

The database and application are placed on the same backend network.

This allows:

    app → db

using:

    db

as the hostname.

---

# 54. `depends_on`

Example:

    depends_on:
      - db

`depends_on` controls startup order.

It means the application service starts after the database service is started.

### Important

`depends_on` does **not** mean:

> The database is fully ready to accept connections.

The presentation recommends using a healthcheck condition when readiness needs to be checked.

---

# 55. Environment Variables in Compose

Example:

    POSTGRES_PASSWORD: ${DB_PASSWORD}

The value can come from:

    .env

The presentation emphasizes that the `.env` file should not be committed when it contains secrets.

### Principle

    Source Code
        ↓
    compose.yaml

    Secret Configuration
        ↓
    .env

    Do not commit secret values

---

# 56. Compose Is Version-Controlled

A `compose.yaml` file can live in the Git repository.

A new developer can clone the repository and use one command to start the application.

This makes the application environment easier to reproduce.

---

# 57. Docker Compose Commands

## Start Everything

    docker compose up -d

Creates and starts the application in detached mode.

---

## Show Services

    docker compose ps

Shows the status of services in the Compose project.

---

## Follow Logs

    docker compose logs -f app

Follows logs for the `app` service.

---

## Execute a Command

    docker compose exec db psql

Runs `psql` inside the database service.

---

## Build Images

    docker compose build

Rebuilds services that use:

    build:

---

## Build and Start

    docker compose up -d --build

Rebuilds changed images and starts the services.

---

## Stop Services

    docker compose stop

Stops services without removing them.

---

## Remove Containers and Networks

    docker compose down

Stops and removes containers and networks.

Named volumes are normally kept.

---

# 58. `docker compose down -v`

Command:

    docker compose down -v

This also removes named volumes.

Therefore:

    docker compose down
        ↓
    Containers + Networks removed
        ↓
    Volumes kept

But:

    docker compose down -v
        ↓
    Containers + Networks + Volumes removed
        ↓
    Persistent development data can be lost

### Important Warning

> `down -v` can delete your database data.

Use it only when you intentionally want to remove the volumes.

---

# 59. Scale a Compose Service

Example:

    docker compose up --scale app=3

This runs three copies of the application service.

Conceptually:

    app.1
    app.2
    app.3

---

# 60. Resolve Compose Configuration

Command:

    docker compose config

This shows the merged and resolved Compose configuration.

Useful for checking what Compose will actually use.

---

# 61. Compose Override Files

Example:

    docker compose -f a.yaml -f b.yaml

This allows one Compose file to be layered over another.

The second file acts as an override.

---

# 62. `docker compose` vs `docker-compose`

The presentation notes:

    docker compose

with a space is the current Docker Compose plugin.

The older standalone tool used:

    docker-compose

with a hyphen.

The commands are similar, but the modern Compose command is:

    docker compose

---

# 63. Docker Swarm

## What is Docker Swarm?

Docker Compose is primarily used to run an application on one machine.

Docker Swarm allows multiple Docker hosts to work together as a cluster.

Conceptually:

    Docker Host 1
         ↓
    Docker Host 2
         ↓
    Docker Host 3
         ↓
    Swarm Cluster

Swarm keeps the desired number of containers running across the cluster.

---

# 64. Swarm Architecture

A Swarm contains:

- Managers
- Workers
- Services
- Tasks
- Stacks

Conceptually:

    Swarm Cluster
         │
    ┌────┴────┐
    ↓         ↓
    Manager   Worker
               ↓
             Tasks

---

# 65. Swarm Manager

The manager:

- Schedules workloads
- Stores cluster state
- Uses Raft for state management

Conceptually:

    Manager
       ↓
    Scheduling
       ↓
    Worker Nodes

---

# 66. Swarm Worker

A worker is a Docker host that runs tasks assigned by the Swarm manager.

Example:

    Worker 1
       ↓
    web.1
    web.2

    Worker 2
       ↓
    web.3
    web.4

---

# 67. Swarm Node

A node is a Docker host participating in the Swarm.

A node can be:

- Manager
- Worker

---

# 68. Swarm Service

A service represents the desired state.

A service defines concepts such as:

- Image
- Replica count
- Ports
- Networks

Example:

    Service: web
    Replicas: 3
    Image: nginx

Swarm attempts to maintain:

    3 running replicas

---

# 69. Swarm Task

A task is one container running as part of a service.

Example:

    Service: web
       ↓
    Replicas: 3
       ↓
    Task 1
    Task 2
    Task 3

Each task represents a container instance placed on a node.

---

# 70. Swarm Stack

A stack is a group of services deployed together from a Compose file.

Conceptually:

    compose.yaml
         ↓
    Docker Stack
         ↓
    app + db + other services

---

# 71. Swarm Self-Healing

Suppose the desired state is:

    web = 4 replicas

One container fails:

    web = 3 replicas

Swarm detects that the desired state is no longer satisfied.

It starts a replacement:

    web = 4 replicas

### Flow

    Desired: 4
         ↓
    Failure
         ↓
    Running: 3
         ↓
    Swarm detects difference
         ↓
    Replacement created
         ↓
    Running: 4

This is self-healing.

---

# 72. Rolling Updates

Swarm can replace containers gradually during an update.

Instead of:

    Stop everything
         ↓
    Start everything

Swarm can:

    Update container 1
         ↓
    Update container 2
         ↓
    Update remaining containers

This is called a rolling update.

The presentation also notes automatic rollback on failure.

---

# 73. Built-In Load Balancing

A published port can be reachable on every node and routed to a healthy task.

Conceptually:

    Client
      ↓
    Swarm Node
      ↓
    Routing
      ↓
    Healthy Task

This provides built-in load-balancing behavior for published services.

---

# 74. Swarm Commands

## Initialize Swarm

    docker swarm init

Makes the current host the first manager.

---

## Get Worker Join Token

    docker swarm join-token worker

Prints the command workers can use to join the Swarm.

---

## Join a Swarm

    docker swarm join --token ... ip:2377

Adds a node to the Swarm.

---

## List Nodes

    docker node ls

Shows:

- Nodes
- Roles
- Availability

---

## Drain a Node

    docker node update --availability drain n2

Moves work away from the node for maintenance.

---

## Promote a Worker

    docker node promote n2

Promotes a worker to manager.

---

## Leave Swarm

    docker swarm leave

Removes the current node from the Swarm.

---

# 75. Swarm Service Commands

List services:

    docker service ls

Show tasks for a service:

    docker service ps web

Scale a service:

    docker service scale web=5

Update an image:

    docker service update --image nginx:1.27 web

Rollback:

    docker service rollback web

View service logs:

    docker service logs web

---

# 76. Create a Swarm Service

Example:

    docker service create \
      --replicas 3 \
      --name web \
      -p 80:80 \
      nginx

This creates a service named:

    web

with:

    3 replicas

using:

    nginx

---

# 77. Deploy a Compose File as a Stack

Example:

    docker stack deploy -c compose.yaml invoice

List stack services:

    docker stack services invoice

Remove the stack:

    docker stack rm invoice

---

# 78. Jenkins and Docker

Session 3 connects Docker with Jenkins.

There are two important ways Docker is used with Jenkins:

1. Docker as the build environment
2. Docker as the build output

---

# 79. Docker as the Jenkins Build Environment

A Jenkins pipeline stage can run inside a Docker container.

Example:

    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-17'
        }
    }

This means the pipeline stage gets a controlled Maven environment.

### Benefit

Jenkins agents do not need Maven manually installed for every project.

The required build environment comes from the Docker image.

---

# 80. Docker as the Build Output

Jenkins can also build the application's Docker image.

Flow:

    Source Code
         ↓
    Jenkins
         ↓
    Tests
         ↓
    docker build
         ↓
    Docker Image
         ↓
    Registry
         ↓
    Deployment

---

# 81. What Jenkins Needs

The presentation identifies:

### Docker Installed

The Jenkins agent needs Docker available.

### Docker Permissions

The Jenkins user needs access to Docker.

The presentation warns that membership in the Docker group carries root-equivalent risk.

### Plugins

Relevant plugins include:

    Docker Pipeline

and:

    Docker Commons

### Registry Credentials

Registry credentials should be stored as Jenkins credentials and referenced by ID.

They should not be written directly into the Jenkinsfile.

---

# 82. Jenkins Pipeline

The presentation provides this pipeline structure:

    pipeline {
        agent { label 'docker' }

        environment {
            IMAGE = "meridian/invoice-service"
        }

        stages {

            stage('Test') {
                agent {
                    docker {
                        image 'maven:3.9-eclipse-temurin-17'
                    }
                }

                steps {
                    sh 'mvn -B test'
                }
            }

            stage('Build image') {
                steps {
                    sh "docker build -t ${IMAGE}:${BUILD_NUMBER} ."
                }
            }

            stage('Push') {
                steps {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub',
                            usernameVariable: 'U',
                            passwordVariable: 'P'
                        )
                    ]) {
                        sh 'echo $P | docker login -u $U --password-stdin'
                        sh "docker push ${IMAGE}:${BUILD_NUMBER}"
                    }
                }
            }

            stage('Deploy') {
                when {
                    branch 'main'
                }

                steps {
                    sh "docker service update --image ${IMAGE}:${BUILD_NUMBER} invoice_app"
                }
            }
        }
    }

---

# 83. Jenkins Pipeline — Test Stage

The Test stage uses a Maven Docker image:

    maven:3.9-eclipse-temurin-17

Then:

    mvn -B test

runs the tests.

### Benefit

The Jenkins agent does not need Maven installed directly.

---

# 84. Jenkins Pipeline — Build Image Stage

Command:

    docker build -t ${IMAGE}:${BUILD_NUMBER} .

The image receives the Jenkins build number as its tag.

Example:

    invoice-service:105

This makes the image traceable to a specific Jenkins build.

---

# 85. Jenkins Pipeline — Push Stage

Jenkins retrieves credentials using:

    withCredentials

Then authenticates to the registry.

The presentation uses:

    --password-stdin

Example:

    echo $P | docker login -u $U --password-stdin

### Why?

It keeps the password out of the command-line argument list and reduces the chance of exposing it in logs.

Then:

    docker push

uploads the image to the registry.

---

# 86. Jenkins Pipeline — Deploy Stage

The deployment stage contains:

    when {
        branch 'main'
    }

This means the deployment stage executes when the pipeline is running for the `main` branch.

Then:

    docker service update \
      --image ${IMAGE}:${BUILD_NUMBER} \
      invoice_app

updates the Swarm service to the new image.

---

# 87. Complete CI/CD Flow

The central flow of Session 3 is:

    Developer
         ↓
    Git Commit
         ↓
    Jenkins
         ↓
    Test
         ↓
    Build Docker Image
         ↓
    Tag Image
         ↓
    Push to Registry
         ↓
    Docker Swarm
         ↓
    Rolling Deployment
         ↓
    Production

The presentation summarizes this as:

    Commit
       ↓
    Test
       ↓
    Image
       ↓
    Registry
       ↓
    Rolling Deploy

The important principle is:

> The artifact tested in CI is the artifact that runs in production.

---

# 88. Docker Troubleshooting

The presentation provides common problems and solutions.

| Symptom | Likely Cause | What to Do |
|---|---|---|
| Container exits immediately | Main process finished or crashed | `docker logs`, then `docker inspect` |
| Port already allocated | Another process/container owns the host port | Choose another host port or find the existing mapping |
| Permission denied on `docker.sock` | User lacks Docker access | Add user to Docker group and log in again |
| Cannot reach DB by name | Containers are on default bridge | Put both containers on a user-defined network |
| Data disappeared after upgrade | Data was stored in writable layer | Mount a named volume |
| `toomanyrequests` from Docker Hub | Anonymous pull rate limit | Login or use a private registry |
| `no space left on device` | Old Docker resources/build cache | Use `docker system df`, then carefully prune |

---

# 89. First Three Commands for Container Problems

The presentation recommends this troubleshooting sequence:

    docker ps -a

Then:

    docker logs <container>

Then:

    docker inspect <container>

### Sequence

    1. docker ps -a
           ↓
    Is the container running or stopped?

    2. docker logs
           ↓
    What did the application say?

    3. docker inspect
           ↓
    What configuration and exit information exists?

---

# 90. Production Docker Practices

## DO

### Pin Versions

Pin:

- Base images
- Compose images
- Plugins

Goal:

    Reproducible Input
          ↓
    Reproducible Output

---

### Keep Data in Named Volumes

Treat containers as disposable.

Make sure important data is stored separately.

---

### Publish Only the Entry Point

Databases and internal services should stay on private networks when possible.

Do not publish database ports unnecessarily.

Example architecture:

    Internet
       ↓
    Web
       ↓
    App
       ↓
    Private Network
       ↓
    Database

---

### Scan Images

Scan images in the pipeline.

The presentation recommends failing the build for critical vulnerabilities, similar to how a failing test prevents a successful build.

---

# 91. Things to Avoid

## Avoid `--privileged`

Do not use:

    --privileged

casually.

It can give the container extensive access to the host.

---

## Avoid Mounting `docker.sock` Casually

Mounting the Docker socket can give a container significant control over Docker and potentially the host.

Use it only when the requirement is understood and the security implications are accepted.

---

## Avoid Running as Root

Use:

    USER

to run applications as a non-root user whenever possible.

---

## Avoid Manually Modifying Running Containers

For example:

    docker exec

can be useful for troubleshooting.

But manually changing files inside a running container is not a reliable deployment method.

Those changes disappear when the container is replaced.

### Better Principle

Change:

    Dockerfile

instead of manually modifying:

    Running Container

---

# 92. Dockerfile and Compose as Source of Truth

The presentation gives an important principle:

> The Dockerfile and Compose file are the source of truth.

If a configuration change exists only inside a running container and is not represented in the Dockerfile or Compose configuration, it will not reliably survive the next deployment.

### Desired Model

    Git
     ↓
    Dockerfile
     ↓
    Image
     ↓
    Compose / Swarm
     ↓
    Container

Everything important should be reproducible from source-controlled configuration.

---

# 93. Credentials and Secrets

Avoid placing credentials inside:

- Docker images
- Dockerfiles
- Compose files

Use appropriate runtime mechanisms such as:

- Environment files
- Secrets
- Orchestrator secret stores

The goal is:

    Source Code
       ↓
    No hard-coded credentials

    Runtime
       ↓
    Secure Configuration

---

# 94. Lab 3 — Build It, Compose It, Ship It

The final hands-on lab combines the concepts from the entire Docker module.

It contains four sections:

1. Dockerfile
2. Compose
3. Swarm
4. Jenkins

---

# 95. Lab 3 — Dockerfile

### Tasks

- Write a multi-stage Dockerfile for `invoice-service`.
- Run the application as a non-root user.
- Compare the final image size with a single-stage build.

### Expected Learning

    Build Stage
        ↓
    Maven + JDK
        ↓
    JAR

    Runtime Stage
        ↓
    JRE + JAR
        ↓
    Smaller Production Image

---

# 96. Lab 3 — Compose

### Tasks

Write:

    compose.yaml

containing:

    app
    db
    volume
    network

Then run:

    docker compose up -d

Open the application.

Then:

    docker compose down

and:

    docker compose up -d

Confirm that database data survived because the data is stored in the volume.

---

# 97. Lab 3 — Swarm

### Tasks

Initialize:

    docker swarm init

Deploy the stack.

Scale the application to:

    3 replicas

Then kill a container and observe Swarm replace it.

### Expected Learning

    Desired Replicas = 3
            ↓
    Container Failure
            ↓
    Swarm Detects Difference
            ↓
    Replacement Container
            ↓
    Desired Replicas = 3

---

# 98. Lab 3 — Jenkins

### Tasks

- Add a `Build image` stage to the Jenkins pipeline.
- Push the image using registry credentials.
- Deploy the new tag using `service update`.

### Complete Flow

    Git Commit
         ↓
    Jenkins
         ↓
    Test
         ↓
    Build Image
         ↓
    Push Image
         ↓
    Registry
         ↓
    Swarm
         ↓
    Service Update
         ↓
    Rolling Deployment

---

# 99. Module 7 — Complete Docker Learning Flow

The entire module can now be viewed as one flow:

    Application Source Code
             ↓
    Dockerfile
             ↓
    Docker Image
             ↓
    Docker Registry
             ↓
    Docker Container
             ↓
    Volumes
             ↓
    Persistent Data
             ↓
    Docker Networks
             ↓
    Multi-Container Application
             ↓
    Docker Compose
             ↓
    Docker Swarm
             ↓
    Jenkins CI/CD
             ↓
    Production Deployment

---

# 100. Module Recap

After completing the Docker module, I should be able to:

## 1. Explain Containers

Understand:

- Containers vs VMs
- Namespaces
- Cgroups
- Image layers

---

## 2. Run and Manage Containers

Understand:

- Docker images
- Container lifecycle
- Container modes
- Restart policies
- Port binding
- Container management commands

---

## 3. Persist and Connect

Understand:

- Volumes
- Bind mounts
- Persistent data
- Docker networks
- Service discovery
- Network segmentation

---

## 4. Build and Compose

Understand:

- Dockerfiles
- Custom images
- Multi-stage builds
- Docker Compose
- Multi-container applications

---

## 5. Orchestrate and Automate

Understand:

- Docker Swarm
- Services
- Replicas
- Rolling updates
- Self-healing
- Jenkins CI/CD
- Image registry
- Automated deployment

---

# 101. Important Interview Questions — Session 3

## Q1. What is a Dockerfile?

A Dockerfile is a recipe containing instructions used to build a Docker image.

---

## Q2. What is `FROM`?

`FROM` defines the base image and starts a Docker build stage.

---

## Q3. Difference between `RUN` and `CMD`?

    RUN
      ↓
    Executes during image build

    CMD
      ↓
    Provides default runtime command/arguments

---

## Q4. Difference between `CMD` and `ENTRYPOINT`?

`ENTRYPOINT` defines the main executable, while `CMD` provides default arguments or a default command.

---

## Q5. Why prefer `COPY` over `ADD`?

`COPY` has simpler and more predictable behavior. The extra behavior of `ADD` is rarely required.

---

## Q6. Why use `.dockerignore`?

To prevent unnecessary or sensitive files from being included in the Docker build context.

---

## Q7. Why optimize Dockerfile layer order?

To maximize Docker layer caching and reduce build time.

---

## Q8. Why use multi-stage builds?

To keep build tools such as Maven and the JDK out of the final production image.

---

## Q9. Why run containers as non-root?

To reduce the security impact if the application or container is compromised.

---

## Q10. What does `EXPOSE` do?

It documents the port that the application listens on. It does not publish the port by itself.

---

## Q11. What is Docker Compose?

A tool for defining and running multi-container applications using a Compose file.

---

## Q12. What does `depends_on` do?

It controls service startup order but does not by itself guarantee that the dependency is ready to accept connections.

---

## Q13. What does `docker compose down -v` do?

It removes the Compose containers and networks and also removes the named volumes.

---

## Q14. What is Docker Swarm?

Docker's native orchestration system for running services across multiple Docker hosts.

---

## Q15. What is a Swarm service?

A desired state consisting of an image, replica count, ports, networks and other service configuration.

---

## Q16. What is a Swarm task?

A single container running as part of a Swarm service.

---

## Q17. What is a Swarm stack?

A group of services deployed together, commonly from a Compose file.

---

## Q18. What is self-healing in Swarm?

If a task fails, Swarm creates a replacement to return the service to its desired replica count.

---

## Q19. What is a rolling update?

Updating service containers gradually rather than replacing all containers simultaneously.

---

## Q20. How does Jenkins integrate with Docker?

Jenkins can:

    Test
      ↓
    Build Docker Image
      ↓
    Push Image
      ↓
    Deploy Image

Docker can also provide the build environment for Jenkins pipeline stages.

---

# 102. Final Production Mental Model

    Developer
        ↓
    Git Commit
        ↓
    Jenkins
        ↓
    Automated Tests
        ↓
    Docker Build
        ↓
    Multi-Stage Image
        ↓
    Security Scan
        ↓
    Versioned Image
        ↓
    Docker Registry
        ↓
    Docker Swarm
        ↓
    Service
        ↓
    Replicas
        ↓
    Rolling Update
        ↓
    Production

Persistent data is kept separately:

    Application
        ↓
    Container
        ↓
    Docker Network
        ↓
    Database Container
        ↓
    Named Volume
        ↓
    Persistent Data

---

# 103. Final Session 3 Understanding

The most important idea from Session 3 is:

> **Build the application as a reproducible Docker image, define the complete application using Compose, orchestrate containers with Swarm, and automate the complete build-to-deployment process through Jenkins.**

The complete DevOps flow is:

    Code
      ↓
    Test
      ↓
    Build Image
      ↓
    Scan
      ↓
    Tag
      ↓
    Push to Registry
      ↓
    Deploy
      ↓
    Rolling Update
      ↓
    Production

The Dockerfile defines **how the image is built**.

The Compose file defines **how the application is assembled**.

Swarm defines **how services run across multiple hosts**.

Jenkins automates **how code becomes a tested, versioned and deployed container image**.

---

# 104. Keep Practising

The presentation recommends containerizing something that is already being run.

Suggested practice:

    Existing Script / Tool / Service
              ↓
          Dockerfile
              ↓
          Docker Image
              ↓
         compose.yaml
              ↓
        Docker Compose
              ↓
        Docker Swarm
              ↓
           Jenkins
              ↓
        CI/CD Pipeline

A practical project exposes problems that simple laboratory exercises may not reveal.

The presentation also recommends rebuilding the Lab 3 `compose.yaml` from memory, including:

- Volume
- Network
- Application service
- Database service

---

# 105. Next Step

The presentation identifies the next step as:

> **Kubernetes**

The same ideas continue at a larger scale:

- Services
- Replicas
- Rolling updates
- Orchestration

The Docker concepts learned in this module provide the foundation for understanding those larger-scale container orchestration concepts.