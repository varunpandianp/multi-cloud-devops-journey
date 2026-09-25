# Docker Learning Notes

## 1. What is Docker?

Docker is a **containerization platform** used to package an application together with everything it needs to run.

This can include:

- Application code
- Runtime
- Libraries
- Dependencies
- Configuration

The package is called a **Docker Image**.

When the image is executed, it becomes a **Docker Container**.

```text
Application
     +
Dependencies
     +
Runtime
     +
Configuration
     ↓
Docker Image
     ↓
Docker Container
     ↓
Running Application
```

---

# 2. Why Do We Need Docker?

A common problem in software development is:

> "It works on my machine."

For example, a developer may have:

```text
Python 3.12
Flask 3.x
Required Libraries
Required System Packages
```

But the production server may have:

```text
Python 3.10
Different Libraries
Different System Packages
```

The application may work on the developer's machine but fail on the server.

Docker solves this problem by packaging the application and its required environment into an image.

```text
Developer Machine
       ↓
Docker Image
       ↓
Test Environment
       ↓
Production Environment
```

The same image can be used across environments.

---

# 3. Docker Image

A **Docker Image** is a read-only template or blueprint used to create containers.

Examples:

```text
nginx
ubuntu
python
mysql
redis
```

Example:

```bash
docker pull nginx
```

This downloads the Nginx image.

---

# 4. Docker Container

A **Docker Container** is a running instance of a Docker image.

```text
Docker Image
     ↓
docker run
     ↓
Docker Container
```

Simple analogy:

```text
Image     = Recipe
Container = Actual Dish
```

Another analogy:

```text
Image     = Class
Container = Object
```

One image can be used to create multiple containers.

```text
              Docker Image
                   |
        ┌──────────┼──────────┐
        ↓          ↓          ↓
   Container 1  Container 2  Container 3
```

---

# 5. Docker Architecture

Basic Docker architecture:

```text
                 Docker Client
                       |
                       | Docker Commands
                       ↓
                 Docker Daemon
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Containers     Images      Networks
                       |
                       ↓
                Docker Registry
                       |
                       ↓
                  Docker Hub
```

## Docker Client

The Docker Client is the interface through which we interact with Docker.

Examples:

```bash
docker run
docker build
docker pull
docker push
docker ps
```

## Docker Daemon

The Docker Daemon manages Docker resources such as:

- Containers
- Images
- Networks
- Volumes

## Docker Registry

A Docker Registry stores Docker images.

Examples:

- Docker Hub
- Amazon ECR
- GitHub Container Registry

---

# 6. Important Docker Commands

## Check Docker Version

```bash
docker --version
```

## Get Docker Information

```bash
docker info
```

## Download an Image

```bash
docker pull nginx
```

## List Docker Images

```bash
docker images
```

## Run a Container

```bash
docker run nginx
```

## List Running Containers

```bash
docker ps
```

## List All Containers

```bash
docker ps -a
```

## Stop a Container

```bash
docker stop <container_id>
```

## Start a Stopped Container

```bash
docker start <container_id>
```

## Restart a Container

```bash
docker restart <container_id>
```

## Remove a Container

```bash
docker rm <container_id>
```

## Remove an Image

```bash
docker rmi <image_id>
```

---

# 7. First Docker Example

Run Nginx in the background:

```bash
docker run -d -p 8080:80 nginx
```

Explanation:

```text
docker run
```

Creates and starts a container.

```text
-d
```

Runs the container in detached/background mode.

```text
-p 8080:80
```

Maps the host port to the container port.

```text
nginx
```

Specifies the Docker image to use.

The complete flow is:

```text
Your Laptop
    |
    | Port 8080
    ↓
Docker Port Mapping
    |
    | Port 80
    ↓
Nginx Container
    |
    ↓
Nginx Application
```

Open:

```text
http://localhost:8080
```

to access Nginx.

---

# 8. Docker Port Mapping

Port mapping connects a port on the host machine to a port inside the container.

Syntax:

```bash
-p HOST_PORT:CONTAINER_PORT
```

Example:

```bash
-p 8080:80
```

Means:

```text
Host Port       Container Port
    8080   →         80
```

Flow:

```text
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
```

---

# 9. Dockerfile

A **Dockerfile** is a text file containing instructions used to build a Docker image.

Example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

Build the image:

```bash
docker build -t my-python-app .
```

Run the image:

```bash
docker run my-python-app
```

---

# 10. Important Dockerfile Instructions

## FROM

Defines the base image.

```dockerfile
FROM python:3.12
```

Example:

```dockerfile
FROM ubuntu:24.04
```

---

## WORKDIR

Sets the working directory inside the image/container.

```dockerfile
WORKDIR /app
```

After this instruction, commands operate from:

```text
/app
```

---

## COPY

Copies files from the build context into the image.

```dockerfile
COPY . .
```

Example:

```dockerfile
COPY requirements.txt .
```

---

## RUN

Executes a command during image build.

Example:

```dockerfile
RUN pip install -r requirements.txt
```

Important:

```text
RUN → Build Time
```

---

## CMD

Defines the default command executed when the container starts.

Example:

```dockerfile
CMD ["python", "app.py"]
```

Important:

```text
CMD → Container Startup
```

---

## EXPOSE

Documents the port that the application uses.

Example:

```dockerfile
EXPOSE 8080
```

Important:

`EXPOSE` does **not** publish the port.

You still need port mapping:

```bash
docker run -p 8080:8080 myapp
```

---

# 11. RUN vs CMD

This is an important Docker interview concept.

## RUN

Runs during image creation.

```dockerfile
RUN pip install flask
```

Flow:

```text
Dockerfile
    ↓
docker build
    ↓
RUN executes
    ↓
Docker Image created
```

## CMD

Runs when the container starts.

```dockerfile
CMD ["python", "app.py"]
```

Flow:

```text
Docker Image
    ↓
docker run
    ↓
CMD executes
    ↓
Application starts
```

Remember:

```text
RUN → Build Time

CMD → Container Runtime
```

---

# 12. Docker Image Lifecycle

Typical Docker workflow:

```text
Application Code
       ↓
Dockerfile
       ↓
docker build
       ↓
Docker Image
       ↓
docker run
       ↓
Docker Container
       ↓
Running Application
```

---

# 13. Docker Image Tags

Docker images can have versions using tags.

Example:

```bash
docker build -t myapp:1.0 .
```

Here:

```text
Repository = myapp
Tag        = 1.0
```

Another version:

```bash
docker build -t myapp:2.0 .
```

Now we can have:

```text
myapp:1.0
myapp:2.0
```

Production applications commonly use explicit versioning or immutable identifiers such as:

```text
myapp:1.0.0
myapp:1.0.1
myapp:2026-09-25
myapp:<git-commit-sha>
```

Avoid relying blindly on:

```text
latest
```

because `latest` does not necessarily tell you exactly which application version is running.

---

# 14. Docker Registry

A Docker Registry is a place where Docker images are stored.

Examples:

```text
Docker Hub
Amazon ECR
GitHub Container Registry
```

Typical workflow:

```text
Developer
    ↓
docker build
    ↓
Docker Image
    ↓
docker push
    ↓
Docker Registry
    ↓
Production Server
    ↓
docker pull
    ↓
Docker Container
```

For AWS DevOps, **Amazon ECR (Elastic Container Registry)** is an important service to learn.

---

# 15. Docker Volumes

Containers are generally treated as **ephemeral**.

If important data exists only inside the container's writable layer, removing the container can cause that data to be lost.

Docker Volumes provide persistent storage.

```text
Container
    ↓
Docker Volume
    ↓
Persistent Data
```

Create a volume:

```bash
docker volume create mydata
```

Run a container using the volume:

```bash
docker run -v mydata:/data myimage
```

Here:

```text
Docker Volume → /data inside Container
```

Volumes are commonly useful for persistent application data.

---

# 16. Docker Networking

Containers often need to communicate with each other.

Example:

```text
Frontend Container
        ↓
Backend Container
        ↓
Database Container
```

Docker provides networking capabilities for container communication.

Create a network:

```bash
docker network create mynetwork
```

Containers can then be attached to the network.

Example architecture:

```text
Nginx
  ↓
Backend API
  ↓
PostgreSQL
```

Each component can run in its own container.

---

# 17. Docker Compose

Real applications often contain multiple services.

Example:

```text
Frontend
Backend
Database
Redis
Nginx
```

Managing all these containers manually can become difficult.

**Docker Compose** allows multiple services to be defined in a YAML file.

Example:

```yaml
services:

  web:
    image: nginx

  backend:
    image: my-backend

  database:
    image: postgres
```

Start the services:

```bash
docker compose up
```

Stop the services:

```bash
docker compose down
```

Conceptually:

```text
docker-compose.yml
        ↓
   Docker Compose
        ↓
 ┌──────┼──────┐
 ↓      ↓      ↓
Web  Backend  Database
```

---

# 18. Docker vs Virtual Machine

## Virtual Machine

```text
Physical Server
      ↓
Hypervisor
      ↓
Virtual Machine
 ┌───────────────────┐
 │ Guest OS           │
 │ Libraries          │
 │ Application        │
 └───────────────────┘

Virtual Machine
 ┌───────────────────┐
 │ Guest OS           │
 │ Libraries          │
 │ Application        │
 └───────────────────┘
```

Each VM has its own guest operating system.

## Docker Container

```text
Physical Server
      ↓
Host OS
      ↓
Docker Engine
      ↓
Container
 ┌───────────────────┐
 │ Application        │
 │ Dependencies       │
 └───────────────────┘

Container
 ┌───────────────────┐
 │ Application        │
 │ Dependencies       │
 └───────────────────┘
```

Containers share the host OS kernel.

Therefore, containers are generally more lightweight and faster to start than full virtual machines.

---

# 19. Docker in DevOps

Docker becomes especially important when integrated with CI/CD.

A typical DevOps pipeline can look like:

```text
Developer
    ↓
Git / GitHub
    ↓
CI Pipeline
    ↓
Build Application
    ↓
Run Tests
    ↓
Docker Build
    ↓
Docker Image
    ↓
Security Scan
    ↓
Container Registry
    ↓
Deployment
    ↓
AWS / Kubernetes
    ↓
Production
```

Example AWS flow:

```text
GitHub
   ↓
Jenkins / GitHub Actions
   ↓
Build & Test
   ↓
Docker Build
   ↓
Docker Image
   ↓
Amazon ECR
   ↓
ECS / EKS / EC2
   ↓
Load Balancer
   ↓
Users
```

---

# 20. Docker Learning Roadmap

Learn Docker in this order:

```text
1. Docker Fundamentals
2. Docker Images
3. Docker Containers
4. Docker Commands
5. Dockerfile
6. Dockerfile Instructions
7. Image Layers
8. Container Lifecycle
9. Port Mapping
10. Environment Variables
11. Docker Volumes
12. Docker Networking
13. Docker Compose
14. Multi-Stage Builds
15. Docker Image Optimization
16. Docker Security
17. Docker Registry
18. Docker Hub
19. Amazon ECR
20. Docker + Jenkins
21. Docker + GitHub Actions
22. Docker + AWS
23. Docker + Kubernetes
```

---

# 21. Important Docker Concepts to Remember

```text
Docker
→ Containerization Platform

Image
→ Blueprint used to create containers

Container
→ Running instance of an image

Dockerfile
→ Instructions used to build an image

Docker Registry
→ Stores Docker images

Docker Hub
→ Public Docker registry

Amazon ECR
→ AWS container image registry

Volume
→ Persistent storage

Network
→ Enables container communication

Docker Compose
→ Manages multiple containers/services

Port Mapping
→ Connects host port to container port
```

---

# 22. Most Important Docker Commands — Quick Reference

```bash
# Docker version
docker --version

# Docker information
docker info

# Download image
docker pull nginx

# List images
docker images

# Run container
docker run nginx

# Run container in background
docker run -d nginx

# Run with port mapping
docker run -d -p 8080:80 nginx

# Running containers
docker ps

# All containers
docker ps -a

# Stop container
docker stop <container_id>

# Start container
docker start <container_id>

# Restart container
docker restart <container_id>

# Remove container
docker rm <container_id>

# Remove image
docker rmi <image_id>

# Build image
docker build -t myapp:1.0 .

# Create volume
docker volume create mydata

# Create