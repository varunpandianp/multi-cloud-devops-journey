# Docker Networking, Container Communication, Network Types, Application Segmentation, Stateful vs Stateless Applications, Docker Limitations & Docker Swarm

---

# 1. What is Docker Networking?

Docker networking allows containers to communicate with:

1. Other containers
2. The Docker host
3. External networks
4. The internet
5. Services running on other Docker hosts

Docker provides networking so that containers can communicate while still maintaining network isolation.

Basic architecture:

    Internet
       |
       v
    Host Machine
       |
    Docker Network
       |
    +-----------+       +-----------+
    | Container | <---> | Container |
    |   Web     |       |    API    |
    +-----------+       +-----------+
                              |
                              v
                         +----------+
                         | Database |
                         +----------+

---

# 2. Why Do We Need Docker Networking?

Without networking, containers would be isolated processes that could not easily communicate with each other.

Example application:

    User
      |
      v
    Nginx
      |
      v
    Backend API
      |
      v
    Database

Each component can run inside a separate container.

Docker networking allows:

    nginx-container
          |
          v
    backend-container
          |
          v
    database-container

This gives us:

- Isolation
- Service-to-service communication
- Network segmentation
- Service discovery
- Security
- Multi-container application architecture

---

# 3. How Docker Containers Communicate

Containers communicate through Docker networks.

Example:

    docker network create my-app-network

Create containers:

    docker run -d --name backend --network my-app-network nginx

    docker run -d --name frontend --network my-app-network nginx

Because both containers are connected to the same user-defined network, they can communicate with each other.

The important concept is:

    Same Docker Network
          |
          +---- frontend
          |
          +---- backend
          |
          +---- database

---

# 4. Container-to-Container Communication

Suppose we have:

    frontend
       |
       v
    backend
       |
       v
    database

All three containers can be attached to the same Docker network.

    docker network create app-network

    docker run -d --name frontend --network app-network nginx

    docker run -d --name backend --network app-network nginx

    docker run -d --name database --network app-network mysql

The containers can communicate using container/service names.

For example:

    backend -> database

The backend can connect using:

    database

instead of trying to find the database container's IP address.

This is possible because Docker's user-defined networks provide DNS-based service discovery.

---

# 5. Docker DNS

Docker provides internal DNS functionality for containers connected to user-defined networks.

Example:

    docker network create app-network

    docker run -d --name db --network app-network mysql

    docker run -d --name backend --network app-network my-backend

The backend can use:

    db

as the hostname for the database.

Architecture:

    backend
       |
       | DNS lookup: db
       v
    Docker DNS
       |
       v
    database container

This is better than hardcoding container IP addresses.

Why?

Because container IP addresses can change when containers are recreated.

Bad approach:

    DATABASE_HOST=172.18.0.5

Better approach:

    DATABASE_HOST=db

---

# 6. User-Defined Bridge Network

A user-defined bridge network is commonly used when multiple containers need to communicate on the same Docker host.

Create:

    docker network create app-network

Run containers:

    docker run -d --name frontend --network app-network nginx

    docker run -d --name backend --network app-network nginx

    docker run -d --name db --network app-network mysql

Architecture:

    app-network
    |
    +--- frontend
    |
    +--- backend
    |
    +--- db

Advantages:

- Container-to-container communication
- Automatic DNS service discovery
- Better isolation
- Application segmentation
- Easy management

For normal multi-container applications on one Docker host, user-defined bridge networks are commonly used.

---

# 7. Important Docker Network Commands

List networks:

    docker network ls

Inspect a network:

    docker network inspect app-network

Create a network:

    docker network create app-network

Create a bridge network:

    docker network create --driver bridge app-network

Remove a network:

    docker network rm app-network

Remove unused networks:

    docker network prune

Connect a running container to a network:

    docker network connect app-network container-name

Disconnect a container from a network:

    docker network disconnect app-network container-name

Run a container on a specific network:

    docker run -d --name backend --network app-network nginx

---

# 8. Docker Network Drivers

Important Docker network types/drivers:

1. bridge
2. host
3. none
4. overlay
5. macvlan

Each network type solves a different problem.

---

# 9. Bridge Network

Bridge is the common networking model for containers running on a single Docker host.

Architecture:

    Docker Host
    |
    +--- Bridge Network
         |
         +--- Container A
         |
         +--- Container B
         |
         +--- Container C

Create:

    docker network create --driver bridge app-network

Run:

    docker run -d --name app1 --network app-network nginx

    docker run -d --name app2 --network app-network nginx

Use bridge when:

- Containers run on the same host
- Containers need to communicate with each other
- You want network isolation
- You are building a normal multi-container application

---

# 10. Default Bridge vs User-Defined Bridge

Docker has a default bridge network.

You can see it with:

    docker network ls

Example:

    bridge
    host
    none

However, for application workloads, a user-defined bridge network is generally preferred.

Example:

    docker network create my-network

Why?

User-defined bridge networks provide better:

- Isolation
- DNS-based container discovery
- Network management
- Application segmentation

Example:

    frontend
       |
       +---- app-network
               |
               +---- backend
               |
               +---- database

---

# 11. Host Network

Host networking removes the normal network isolation between the container and the Docker host's networking stack.

Example:

    docker run --network host nginx

Conceptually:

    Host Network
        |
        +--- Container
        |
        +--- Host Processes

The container uses the host's network namespace/network stack rather than having its own isolated network interface in the normal way.

Use cases:

- Network-intensive workloads
- Situations where network performance is important
- Applications that need direct access to host networking

Important:

Host networking reduces network isolation, so it should be used deliberately.

---

# 12. None Network

The `none` network provides a container with no external network connectivity.

Example:

    docker run --network none nginx

Concept:

    Container
       |
       X
       |
    No external network

The container still has its own network namespace and loopback interface, but it does not have normal external networking.

Use cases:

- Highly isolated workloads
- Testing
- Workloads that do not require network access

---

# 13. Overlay Network

An overlay network is designed for communication across multiple Docker hosts.

Example:

    Docker Host 1                  Docker Host 2

    +-----------+                  +-----------+
    | Container |                  | Container |
    |     A     |                  |     B     |
    +-----------+                  +-----------+
          \                              /
           \                            /
            +--------------------------+
                  Overlay Network

This is important when containers are distributed across multiple hosts.

Overlay networking is strongly associated with Docker Swarm and multi-host container communication.

Example architecture:

    Node 1                         Node 2
    +---------+                    +---------+
    | Web     |                    | API     |
    +---------+                    +---------+
          \                          /
           \                        /
            +----------------------+
                 Overlay Network

Use overlay networks when:

- Containers/services run on multiple Docker hosts
- Docker Swarm is being used
- Services need multi-host communication

---

# 14. Macvlan Network

Macvlan allows containers to appear directly on the physical network with their own MAC addresses.

Conceptually:

    Physical Network
          |
      Network Switch
       /           \
      /             \
    Host          Container
                  MAC Address
                  IP Address

The container can appear more like a physical machine on the network.

Use cases:

- Legacy applications
- Applications that require direct Layer 2 network presence
- Situations where containers need their own MAC addresses/IP identities

Important:

Macvlan has additional networking considerations, including host-to-container communication limitations depending on configuration.

---

# 15. Network Type Comparison

    bridge
    |
    +-- Single Docker host
    +-- Container-to-container communication
    +-- Common application networking

    host
    |
    +-- Uses host networking
    +-- Less network isolation
    +-- Useful for specific performance/network requirements

    none
    |
    +-- No external network connectivity
    +-- Highly isolated container

    overlay
    |
    +-- Multi-host networking
    +-- Docker Swarm
    +-- Distributed services

    macvlan
    |
    +-- Container gets its own MAC/IP presence
    +-- Direct physical network integration

---

# 16. Docker Port Publishing

Docker networking and port publishing are different concepts.

Example:

    docker run -d -p 8080:80 nginx

Here:

    8080 = Host port
    80   = Container port

Architecture:

    Browser
       |
       | http://host:8080
       v
    Host Port 8080
       |
       v
    Container Port 80
       |
       v
    Nginx

Remember:

    Network
    = How containers communicate

    Port publishing
    = How external clients reach a container through the host

---

# 17. Docker Application Segmentation

Large applications should not put every container on one common network unnecessarily.

Example application:

    Internet
       |
       v
    Frontend
       |
       v
    Backend
       |
       v
    Database

We can create separate networks.

    frontend-network
          |
       Frontend
          |
          v
    backend-network
          |
       Backend
          |
          v
    database-network
          |
       Database

A container can belong to multiple networks.

Example:

    frontend
       |
       | frontend-network
       |
    backend
       |
       | backend-network
       |
    database

Backend can connect to both networks if required.

Database does not need to be directly exposed to the frontend.

---

# 18. Network Segmentation Example

Suppose we have:

    Frontend
    Backend
    Database

Create:

    docker network create frontend-network

    docker network create backend-network

Connect:

    Frontend -> frontend-network

    Backend -> frontend-network
               backend-network

    Database -> backend-network

Architecture:

                    frontend-network
                   /                 \
              Frontend             Backend
                                      |
                                      |
                               backend-network
                                      |
                                      v
                                  Database

This provides better isolation.

Frontend cannot directly communicate with the database if it is not connected to the database's network.

---

# 19. Why Network Segmentation Is Important

Network segmentation helps with:

- Security
- Isolation
- Reduced attack surface
- Application architecture
- Separation of responsibilities
- Easier troubleshooting
- Controlling which services can communicate

Production principle:

    Do not allow every service to communicate with every other service unless required.

Example:

    Internet
       |
       v
    Load Balancer
       |
       v
    Frontend
       |
       v
    Backend
       |
       v
    Database

The database should normally not be directly exposed to the internet.

---

# 20. Stateful vs Stateless Applications

Docker can run both:

1. Stateless applications
2. Stateful applications

But they require different storage and operational designs.

---

# 21. Stateless Application

A stateless application does not depend on data stored inside the container's writable filesystem.

Example:

    Nginx
    API server
    Microservice
    Web application

Architecture:

    Request
       |
       v
    Container
       |
       v
    Response

If the container is deleted:

    Old Container X
          |
        DELETE
          |
          X

New container:

    New Container Y
          |
       Application
          |
       Continue

The application can continue because important state is stored outside the container.

Examples of external state:

- Database
- Redis
- Object storage
- External persistent volume
- External service

---

# 22. Why Stateless Applications Are Good for Scaling

Suppose:

    Load Balancer
        |
        +--- App Container 1
        |
        +--- App Container 2
        |
        +--- App Container 3

If the application is stateless, we can easily add more replicas.

    Load Balancer
        |
        +--- App 1
        +--- App 2
        +--- App 3
        +--- App 4

This is one of the major reasons cloud-native applications prefer stateless application design where possible.

---

# 23. Stateful Application

A stateful application needs persistent data.

Examples:

- MySQL
- PostgreSQL
- MongoDB
- Elasticsearch
- Stateful services

Example:

    Application
        |
        v
    Database Container
        |
        v
    Persistent Volume
        |
        v
    Actual Data

If the database container is deleted, the data should remain in persistent storage.

---

# 24. Docker Container Writable Layer

Every container has a writable layer.

If data is stored only inside that writable layer:

    Container
       |
       v
    Writable Layer
       |
       v
    Application Data

When the container is removed:

    Container
       |
      DELETE
       |
       X
    Writable Layer
       |
       X
    Data

Therefore:

    Container = Disposable
    Important Data = Persistent Storage

Important rule:

    Containers should be treated as disposable.

    Persistent data should live outside the container's writable layer.

---

# 25. Three Main Ways to Store Docker Data

Docker commonly provides three approaches:

1. Volumes
2. Bind mounts
3. tmpfs mounts

---

# 26. Docker Volumes

Volumes are Docker-managed persistent storage.

Create:

    docker volume create app-data

Use:

    docker run -d \
      --name database \
      --mount source=app-data,target=/var/lib/mysql \
      mysql

Concept:

    Docker
       |
       +--- Volume
              |
              +--- Database Container

Advantages:

- Persistent
- Managed by Docker
- Easy to reuse
- Good for production persistent data
- Can be backed up
- Can be shared with multiple containers when appropriate

General recommendation:

    Use Docker volumes when application data must persist independently of the container lifecycle.

---

# 27. Bind Mounts

A bind mount maps a specific host filesystem path into a container.

Concept:

    Host Directory
          |
          | Bind Mount
          v
    Container Directory

Example:

    docker run -d \
      --mount type=bind,source=/home/user/app,target=/app \
      nginx

Use cases:

- Local development
- Source code sharing
- Configuration files
- Host-managed files
- Development workflows

Example:

    Host:
    /home/user/project

          |
          v

    Container:
    /app

---

# 28. tmpfs Mount

tmpfs stores data in memory rather than persistent disk storage.

Concept:

    Container
       |
       v
    RAM
       |
       X
    Not persistent after container stops/removes

Use cases:

- Temporary files
- Cache
- Sensitive temporary data
- Scratch data

Example:

    docker run -d \
      --tmpfs /tmp \
      nginx

Important:

    tmpfs = temporary memory-backed storage

Do not use tmpfs when you need persistent application data.

---

# 29. Volumes vs Bind Mounts vs tmpfs

    Volume
    |
    +-- Docker-managed
    +-- Persistent
    +-- Production application data
    +-- Database data

    Bind Mount
    |
    +-- Host-managed path
    +-- Development
    +-- Source code
    +-- Configuration

    tmpfs
    |
    +-- Memory
    +-- Temporary
    +-- Cache
    +-- Scratch/sensitive temporary data

Memory trick:

    Volume   = Persistent application data

    Bind     = Host directory sharing

    tmpfs    = Temporary memory storage

---

# 30. Docker Single-Host Limitation

A major limitation of basic Docker host management is the single-host boundary.

Suppose:

    Docker Host 1
    +----------------+
    | Container A    |
    | Container B    |
    | Container C    |
    +----------------+

If the host fails:

    Docker Host 1
          |
        FAIL
          X

All containers running on that host can become unavailable.

This creates problems with:

- High availability
- Scaling
- Scheduling
- Failover
- Self-healing
- Multi-host networking
- Rolling deployments
- Managing many containers

This is why container orchestration becomes important.

---

# 31. Why Orchestration Is Needed

Suppose an application has:

    100 containers
    10 hosts
    Multiple replicas
    Multiple services

Manually managing everything becomes difficult.

We need a system that can:

- Schedule containers
- Maintain desired number of replicas
- Restart failed containers
- Distribute workloads
- Perform rolling updates
- Provide service discovery
- Provide networking between hosts
- Handle scaling

This is the role of a container orchestrator.

Examples:

- Docker Swarm
- Kubernetes

---

# 32. What is Docker Swarm?

Docker Swarm is Docker's native container orchestration technology.

It allows multiple Docker hosts to operate together as a cluster.

Instead of managing:

    Host 1
    Host 2
    Host 3

individually, we can manage them as:

    Docker Swarm Cluster

Architecture:

                    Swarm Cluster
                         |
          +--------------+--------------+
          |              |              |
       Manager         Worker         Worker
          |              |              |
       Services       Containers      Containers

---

# 33. Docker Swarm Cluster

A Swarm cluster consists primarily of:

1. Manager nodes
2. Worker nodes

Architecture:

    +----------------------+
    |     Manager Node     |
    | Cluster Management  |
    +----------+-----------+
               |
       -------------------
       |                 |
       v                 v
    +------+          +------+
    |Worker|          |Worker|
    +------+          +------+

---

# 34. Manager Node

The manager is responsible for cluster management.

Responsibilities include:

- Maintaining cluster state
- Scheduling services
- Managing nodes
- Maintaining desired state
- Managing Swarm configuration

Example desired state:

    Desired replicas = 3

Swarm attempts to maintain:

    App 1
    App 2
    App 3

---

# 35. Worker Node

Worker nodes run the workloads assigned by the manager.

Example:

    Manager
       |
       +---- Worker 1
       |       |
       |     App
       |
       +---- Worker 2
               |
             App

Workers execute tasks assigned by the Swarm manager.

---

# 36. Desired State

One important orchestration concept is desired state.

Suppose we create:

    replicas = 3

Desired state:

    3 application replicas

Current state:

    3 replicas

Everything is healthy.

If one container fails:

    Desired = 3
    Current = 2

Swarm detects the difference.

It can create another task/container:

    Desired = 3
    Current = 3

This is called reconciliation toward the desired state.

---

# 37. Docker Swarm Service

In Swarm, we generally deploy applications as services.

Example:

    docker service create --name web nginx

List services:

    docker service ls

Inspect service:

    docker service inspect web

List service tasks:

    docker service ps web

Remove service:

    docker service rm web

---

# 38. Scaling a Swarm Service

Suppose:

    docker service create \
      --name web \
      --replicas 3 \
      nginx

Swarm attempts to maintain:

    web
      |
      +--- Replica 1
      +--- Replica 2
      +--- Replica 3

Scale to 5:

    docker service scale web=5

Now the desired state becomes:

    5 replicas

Swarm schedules additional tasks.

---

# 39. Self-Healing in Docker Swarm

Suppose:

    web replicas = 3

One task fails:

    Replica 1 -> FAILED
    Replica 2 -> RUNNING
    Replica 3 -> RUNNING

Swarm detects:

    Desired = 3
    Current = 2

Swarm creates another task:

    Replica 1 -> NEW
    Replica 2 -> RUNNING
    Replica 3 -> RUNNING

Now:

    Desired = 3
    Current = 3

This is one of the major benefits of orchestration.

---

# 40. Swarm Overlay Networking

Swarm can use overlay networks for communication between services running on different nodes.

Architecture:

    Node 1                         Node 2

    Web Service                    API Service
       |                              |
       +--------------+---------------+
                      |
               Overlay Network
                      |
                 Docker Swarm

This allows distributed services to communicate across Docker hosts.

---

# 41. Swarm Service Discovery

Swarm provides service discovery so services can communicate without manually tracking container IP addresses.

Concept:

    Service A
       |
       | request
       v
    Service B

Swarm handles the underlying service/task addressing.

This makes applications easier to scale because tasks can move or be recreated.

---

# 42. Docker Swarm Rolling Updates

Suppose we have:

    Version 1
    replicas = 5

We want:

    Version 2

Instead of stopping everything at once, Swarm can update tasks progressively.

Concept:

    V1 V1 V1 V1 V1

       |
       v

    V2 V1 V1 V1 V1

       |
       v

    V2 V2 V1 V1 V1

       |
       v

    V2 V2 V2 V2 V2

This reduces downtime during deployments.

---

# 43. Docker Swarm High-Level Architecture

                    Users
                   |
                   v
             Load Balancer
                   |
                   v
          +-------------------+
          |   Swarm Cluster   |
          +-------------------+
             /      |       \
            /       |        \
           v        v         v
       Manager    Worker    Worker
          |          |         |
          |       Service   Service
          |
       Cluster
        State

The manager controls the desired state and scheduling while worker nodes execute workloads.

---

# 44. Docker vs Docker Swarm

    Docker
      |
      +-- Container runtime
      +-- Images
      +-- Container networking
      +-- Volumes
      +-- Container lifecycle

    Docker Swarm
      |
      +-- Orchestration
      +-- Cluster management
      +-- Scheduling
      +-- Replicas
      +-- Self-healing
      +-- Rolling updates
      +-- Multi-host networking

Simple memory:

    Docker = Run containers

    Swarm = Manage containers across a cluster

---

# 45. Important Docker Limitations

Docker itself is powerful, but running containers manually at scale introduces challenges.

Important limitations/challenges include:

1. Single-host boundary
2. Manual scaling
3. Manual scheduling
4. Limited self-healing when using standalone Docker
5. High-availability challenges
6. Multi-host networking complexity
7. Container lifecycle management
8. Persistent storage management
9. Monitoring and logging requirements
10. Security management
11. Configuration and secret management
12. Rolling deployment complexity
13. Large-scale cluster management

These are major reasons orchestration platforms are used.

---

# 46. Docker and Stateful Applications

Docker CAN run stateful applications.

Example:

    Docker Container
          |
          v
       MySQL
          |
          v
    Persistent Volume

But production stateful applications require careful planning for:

- Persistent storage
- Backup
- Restore
- Replication
- High availability
- Disaster recovery
- Data consistency
- Storage performance

Therefore:

    Docker can run stateful applications.

    But running a stateful application safely in production
    requires more than simply starting a database container.

---

# 47. Docker and Stateless Applications

Stateless applications are usually easier to scale.

Example:

    Load Balancer
          |
      +---+---+---+
      |   |   |   |
     App App App App

If one container fails:

    App 1 -> Failed

Traffic can continue to other replicas.

Orchestration can create a replacement.

This makes stateless applications a strong fit for containerized and cloud-native architectures.

---

# 48. Production Architecture Example

A typical architecture could look like:

    Internet
       |
       v
    Load Balancer
       |
       v
    Frontend / API
       |
       v
    Backend Services
       |
       +----------------+
       |                |
       v                v
    Database          Cache
       |
       v
    Persistent Storage

Docker networking can provide:

- Network isolation
- Service communication
- DNS discovery
- Segmentation
- Multi-container connectivity

Docker Swarm can additionally provide:

- Replicas
- Scheduling
- Self-healing
- Cluster management
- Rolling updates
- Overlay networking

---

# 49. Important Production Best Practices

1. Do not store important application data only inside the container writable layer.

2. Use volumes for persistent application data where appropriate.

3. Use bind mounts mainly when host filesystem sharing is actually required.

4. Use tmpfs for temporary memory-backed data.

5. Prefer user-defined bridge networks for multi-container applications on one host.

6. Use network segmentation to restrict unnecessary communication.

7. Do not expose databases directly to the public internet.

8. Use service/container names instead of hardcoding dynamic container IP addresses.

9. Treat containers as disposable.

10. Design applications to be stateless where practical.

11. For stateful services, plan storage, backup, recovery and availability carefully.

12. For multi-host workloads, use an orchestration platform and appropriate networking.

13. Monitor containers, networks, storage and application health.

14. Do not assume that simply containerizing an application automatically makes it highly available.

---

# 50. Interview Questions

## Q1. What is Docker networking?

Docker networking provides communication between containers, the host, external systems and other networks while providing isolation.

## Q2. How do two Docker containers communicate?

If both containers are attached to the same user-defined network, they can communicate using the network and Docker's internal DNS/service discovery.

## Q3. Why use container names instead of IP addresses?

Container IP addresses can change when containers are recreated. Service/container names provide a stable logical way to discover the service.

## Q4. What is a bridge network?

A bridge network connects containers running on the same Docker host.

## Q5. What is an overlay network?

An overlay network enables communication across multiple Docker hosts and is commonly used with Docker Swarm.

## Q6. What is host networking?

Host networking allows a container to use the host's network stack rather than normal isolated container networking.

## Q7. What is the none network?

It provides a container with no normal external network connectivity.

## Q8. What is macvlan?

Macvlan allows containers to appear on the physical network with their own MAC/IP identity.

## Q9. Can Docker run stateful applications?

Yes. Docker can run stateful applications, but persistent storage, backup, recovery and availability must be designed properly.

## Q10. Why are stateless applications easier to scale?

Because replicas do not depend on local container state, containers can be added, removed or recreated more easily.

## Q11. What happens to data stored only in a container's writable layer?

It is lost when the container is removed.

## Q12. What are the three main Docker storage mechanisms?

    1. Volumes
    2. Bind mounts
    3. tmpfs mounts

## Q13. What is Docker Swarm?

Docker Swarm is Docker's native container orchestration technology for managing services and containers across a cluster.

## Q14. What is a Swarm manager?

The manager maintains cluster state, schedules services and manages the desired state of the cluster.

## Q15. What is a Swarm worker?

A worker node executes tasks assigned by the Swarm manager.

## Q16. What is desired state?

Desired state is the condition the orchestrator is instructed to maintain.

Example:

    Desired replicas = 3

If one replica fails, Swarm attempts to create another to return to three.

## Q17. What problem does Docker Swarm solve?

It helps solve the challenges of managing containers across multiple hosts, including scheduling, scaling, service discovery, self-healing, rolling updates and multi-host networking.

## Q18. Why is a single Docker host a limitation?

If the host fails, workloads running only on that host can become unavailable. Multi-host orchestration helps provide better availability and workload distribution.

---

# 51. Most Important Memory Map

    Docker Networking
          |
          +-- bridge
          |     |
          |     +-- Single host
          |
          +-- host
          |     |
          |     +-- Host network
          |
          +-- none
          |     |
          |     +-- Isolated networking
          |
          +-- overlay
          |     |
          |     +-- Multi-host
          |     +-- Swarm
          |
          +-- macvlan
                |
                +-- Physical network identity


    Docker Storage
          |
          +-- Volume
          |     |
          |     +-- Persistent
          |
          +-- Bind Mount
          |     |
          |     +-- Host directory
          |
          +-- tmpfs
                |
                +-- Temporary RAM storage


    Container Architecture
          |
          +-- Stateless
          |     |
          |     +-- Easy to scale
          |
          +-- Stateful
                |
                +-- Persistent storage required


    Docker
       |
       +-- Containers
       +-- Images
       +-- Networks
       +-- Volumes


    Docker Swarm
       |
       +-- Cluster
       +-- Managers
       +-- Workers
       +-- Services
       +-- Replicas
       +-- Desired State
       +-- Scheduling
       +-- Self-Healing
       +-- Rolling Updates
       +-- Overlay Network


# 52. Final Mental Model

Remember Docker networking and orchestration in this order:

    1. Container
       |
       v
    2. Network
       |
       v
    3. Container-to-container communication
       |
       v
    4. DNS / service discovery
       |
       v
    5. Network segmentation
       |
       v
    6. Persistent storage
       |
       v
    7. Stateful vs Stateless
       |
       v
    8. Multiple Docker hosts
       |
       v
    9. Orchestration
       |
       v
    10. Docker Swarm

The key DevOps mental model is:

    Containers are disposable.
    Networks provide communication and isolation.
    Volumes provide persistent data.
    Stateless applications are easier to scale.
    Stateful applications require persistent storage and careful operations.
    A single Docker host creates an availability and scaling boundary.
    Docker Swarm provides native orchestration across multiple Docker hosts.