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