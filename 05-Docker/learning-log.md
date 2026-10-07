# Learning Log — 22-09-2026

## Docker

- Learned Docker fundamentals and why containerization is used.
- Understood **Docker Image vs Docker Container**.
- Learned `docker pull` to download images.
- Practiced running an **Nginx container** using `docker run`.
- Learned Docker port mapping using `-p 80:80`.
- Understood EC2 → Docker → Nginx → Browser flow.
- Practiced `docker ps` and `docker ps -a`.
- Learned `docker stop`, `docker start`, `docker restart`, and `docker rm`.
- Learned `docker rmi` to remove Docker images.
- Understood that `docker run` creates a container from an image.
- Understood the basic Docker image → container → application flow.
- Learned how Docker can provide application and dependency isolation.
- Understood the basic difference between **Virtual Machines and Containers**.
- Learned how a Dockerfile can be used to create a customized image from an existing image.

## Practical Flow

```text
docker pull nginx
      ↓
Nginx Image
      ↓
docker run
      ↓
Nginx Container
      ↓
Port 80
      ↓
EC2 Public IP
      ↓
Browser
      ↓
Nginx Welcome Page
```

# Learning Log — 24-09-2026

## Docker

- Learned Docker container management and important container commands.
- Practiced `docker ps`, `docker ps -a`, `docker create`, `docker start`, `docker stop`, `docker restart`, `docker pause`, `docker unpause`, `docker kill`, and `docker rm`.
- Understood the **Docker container lifecycle**: Created → Running → Paused/Exited → Running → Removed.
- Learned that a container's lifecycle is closely related to its **main process**.
- Learned `docker inspect` for inspecting containers and `docker image inspect` for inspecting images.
- Learned how to inspect container state, IP address, configuration, and port bindings.
- Learned **Docker port binding** using `-p HOST_PORT:CONTAINER_PORT`.
- Understood the difference between **container port and host/EC2 port**.
- Learned Docker **attached, detached, and interactive modes**.
- Practiced `-d` for detached mode and `-it` for interactive terminal access.
- Learned the difference between `docker attach` and `docker exec -it`.
- Understood why **custom Docker images** are required for packaging application code, dependencies, runtime, and configuration.
- Learned the purpose of a **Dockerfile**.
- Studied Dockerfile instructions: `FROM`, `LABEL`, `WORKDIR`, `COPY`, `RUN`, `ENV`, `EXPOSE`, `USER`, `ENTRYPOINT`, and `CMD`.
- Understood the difference between `ENTRYPOINT` and `CMD`.
- Learned why Docker becomes difficult to manage at large scale.
- Understood why **Kubernetes** is used for container orchestration.
- Learned Kubernetes concepts such as **scaling, self-healing, service discovery, load balancing, rolling updates, and desired state**.

## Practical Flow

```text
Dockerfile
    ↓
docker build
    ↓
Custom Docker Image
    ↓
docker run
    ↓
Docker Container
    ↓
Port Binding
    ↓
Application
```

## Kubernetes Understanding

```text
Many Containers
      ↓
Operational Complexity
      ↓
Kubernetes
      ↓
Orchestration
      ↓
Scaling + Self-Healing + Networking + Deployments
```


## Session Learning Log 28-9-2026
 
## study notes update date Date: 06-10-2026

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

# Learning Log — Docker Networking & Docker Swarm
## session date -29-09-2026 
**study Date:** 07-10-2026

## Topics Learned

- Learned the fundamentals of Docker networking and how containers communicate with each other.
- Learned how containers connected to the same user-defined network can communicate using container/service names.
- Learned about Docker's internal DNS and service discovery.
- Practiced important Docker network commands such as `docker network ls`, `docker network create`, `docker network inspect`, `docker network connect`, and `docker network disconnect`.
- Learned the major Docker network types/drivers: Bridge, Host, None, Overlay, and Macvlan.
- Learned when to use Bridge networking for single-host container communication.
- Learned how Host networking uses the host's networking stack.
- Learned how the None network provides isolated networking.
- Learned how Overlay networking enables communication across multiple Docker hosts.
- Learned the purpose of Macvlan networking and how containers can have their own MAC/IP identity on the physical network.
- Learned the difference between Docker networking and port publishing.
- Learned how to segment an application using multiple Docker networks.
- Learned why network segmentation improves isolation, security, and control over service-to-service communication.
- Learned the difference between stateless and stateful applications.
- Learned why stateless applications are easier to scale and replace.
- Learned how stateful applications require persistent storage and careful backup/recovery planning.
- Learned the importance of Docker volumes, bind mounts, and tmpfs for different storage requirements.
- Learned that data stored only in a container's writable layer can be lost when the container is removed.
- Learned the concept that containers should be treated as disposable and persistent data should be stored outside the container lifecycle.
- Learned that a major limitation of basic Docker host management is the single-host boundary.
- Learned why container orchestration is required when managing containers across multiple hosts.
- Learned what Docker Swarm is and why it is used for container orchestration.
- Learned the difference between Swarm Manager and Worker nodes.
- Learned the concept of desired state in Docker Swarm.
- Learned how Swarm maintains the required number of service replicas.
- Learned about Swarm services and scaling replicas.
- Learned how Swarm provides self-healing by replacing failed tasks.
- Learned how Swarm uses overlay networking for multi-host service communication.
- Learned about Swarm service discovery.
- Learned the concept of rolling updates in Docker Swarm.
- Understood the overall relationship between Docker containers, networking, storage, application state, multi-host deployment, and orchestration.

## Key Takeaway

Docker provides the foundation for running containers, networking provides communication and isolation, volumes provide persistent data, and Docker Swarm adds orchestration capabilities such as scaling, scheduling, self-healing, service discovery, and multi-host management.