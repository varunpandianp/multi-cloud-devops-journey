# Docker Container Management, Lifecycle, Inspect and Port Binding

# 1. Docker Container

A **Docker container** is a running or stopped instance created from a Docker image.

```text
Docker Image
     ↓
docker run
     ↓
Docker Container
     ↓
Application runs inside container
```

Example:

```bash
docker run -d --name nginx nginx
```

Here:

```text
nginx image
    ↓
nginx container
    ↓
Nginx application
```

An image is the **template**, while a container is the **instance created from that image**.

---

# 2. Docker Container Management Commands

## List Running Containers

```bash
docker ps
```

Shows only currently running containers.

---

## List All Containers

```bash
docker ps -a
```

Shows:

- Running containers
- Stopped containers
- Exited containers

---

## Create a Container Without Starting It

```bash
docker create --name nginx nginx
```

This creates the container but does not start it.

Flow:

```text
Nginx Image
    ↓
docker create
    ↓
Created Container
    ↓
Not Running
```

---

## Start a Container

```bash
docker start nginx
```

Starts an existing stopped container.

---

## Stop a Container

```bash
docker stop nginx
```

Stops a running container gracefully.

---

## Restart a Container

```bash
docker restart nginx
```

Restarts the container.

Flow:

```text
Running
   ↓
Stop
   ↓
Start
   ↓
Running
```

---

## Pause a Container

```bash
docker pause nginx
```

Pauses the processes inside the container.

The container still exists, but its processes are paused.

---

## Unpause a Container

```bash
docker unpause nginx
```

Resumes the paused processes.

---

## Kill a Container

```bash
docker kill nginx
```

Immediately terminates the main process of the container.

### Difference

```text
docker stop
→ Graceful stop

docker kill
→ Immediate termination
```

Normally prefer `docker stop` when you want a graceful shutdown.

---

## Remove a Container

```bash
docker rm nginx
```

Removes a stopped container.

If the container is running, Docker normally requires it to be stopped first.

You can force removal with:

```bash
docker rm -f nginx
```

---

# 3. Container Lifecycle / States

A Docker container can move through different states during its lifecycle.

Basic lifecycle:

```text
                docker create
                     ↓
                  CREATED
                     ↓
                docker start
                     ↓
                  RUNNING
                  ↙       ↘
          docker pause    Application exits
               ↓                ↓
             PAUSED          EXITED
               ↓                ↓
        docker unpause       docker start
               ↓                ↓
             RUNNING ←─────────┘

RUNNING
   ↓
docker stop
   ↓
STOPPED / EXITED
   ↓
docker start
   ↓
RUNNING

STOPPED / EXITED
   ↓
docker rm
   ↓
REMOVED
```

---

# 4. Important Container States

## Created

The container has been created but has not started.

Command:

```bash
docker create --name nginx nginx
```

State:

```text
CREATED
```

---

## Running

The container is currently running.

Command:

```bash
docker start nginx
```

or:

```bash
docker run nginx
```

State:

```text
RUNNING
```

---

## Paused

The container is still present, but its processes are temporarily paused.

Command:

```bash
docker pause nginx
```

State:

```text
PAUSED
```

Resume:

```bash
docker unpause nginx
```

---

## Exited

The container's main process has stopped.

This can happen because:

```text
docker stop
```

or because the application/process inside the container finished or crashed.

Example:

```text
RUNNING
   ↓
Application exits
   ↓
EXITED
```

---

## Removed

The container no longer exists.

Command:

```bash
docker rm nginx
```

Flow:

```text
EXITED
   ↓
docker rm
   ↓
REMOVED
```

---

# 5. Important Concept: Container Lifetime

A container normally exists as long as its **main process** exists.

For example:

```bash
docker run ubuntu
```

Ubuntu may start and then immediately exit because there is no long-running foreground process.

But:

```bash
docker run nginx
```

keeps running because the Nginx process stays active.

Therefore:

```text
Container Running
       ↓
Main Process Running
       ↓
Container stays Running
```

When the main process exits:

```text
Main Process Exits
       ↓
Container Exits
```

---

# 6. Container Management Command Summary

```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Create container
docker create --name nginx nginx

# Start container
docker start nginx

# Stop container
docker stop nginx

# Restart container
docker restart nginx

# Pause container
docker pause nginx

# Resume container
docker unpause nginx

# Kill container
docker kill nginx

# Remove container
docker rm nginx

# Force remove container
docker rm -f nginx
```

---

# 7. Docker Image Inspect

`docker inspect` provides detailed information about Docker objects.

To inspect an image:

```bash
docker image inspect nginx
```

You can also use:

```bash
docker inspect nginx
```

when Docker identifies `nginx` as an image.

The output is in **JSON format** and contains detailed metadata.

It can include information such as:

- Image ID
- Architecture
- OS
- Created time
- Image layers
- Environment variables
- Entrypoint
- CMD
- Working directory
- Configuration
- Exposed ports

---

# 8. Why Inspect a Docker Image?

Suppose you want to understand how the Nginx image is configured.

Run:

```bash
docker image inspect nginx
```

You can inspect:

```text
Image
 ├── ID
 ├── OS
 ├── Architecture
 ├── Layers
 ├── Environment
 ├── CMD
 ├── Entrypoint
 ├── Working Directory
 └── Exposed Ports
```

This is useful when troubleshooting or understanding an unfamiliar image.

---

# 9. Docker Container Inspect

To inspect a container:

```bash
docker inspect nginx
```

This gives detailed information about the container.

It can include:

- Container ID
- Container name
- Image used
- Container state
- IP address
- Network configuration
- Port bindings
- Mounts
- Environment variables
- Restart policy
- Command
- Entrypoint

---

# 10. Inspect Container State

You can specifically inspect the state:

```bash
docker inspect --format='{{.State.Status}}' nginx
```

Possible output:

```text
running
```

or:

```text
exited
```

or:

```text
paused
```

This is useful when troubleshooting.

---

# 11. Inspect Container IP Address

You can find the container IP:

```bash
docker inspect --format='{{.NetworkSettings.IPAddress}}' nginx
```

Example:

```text
172.17.0.2
```

However, don't assume this IP is the address users should access from the Internet.

For external access, we normally use **host/EC2 port binding**.

---

# 12. Inspect Port Binding

To inspect port information:

```bash
docker port nginx
```

Example:

```text
80/tcp -> 0.0.0.0:8080
```

This means:

```text
Host Port 8080
      ↓
Container Port 80
```

You can also inspect the full configuration:

```bash
docker inspect nginx
```

and look under the network/port configuration.

---

# 13. Docker Port Binding

A container has its own network namespace.

Suppose Nginx inside the container listens on:

```text
Port 80
```

That does not automatically mean that users can access it through the EC2 host's port 80.

We need **port binding**.

Example:

```bash
docker run -d --name nginx -p 8080:80 nginx
```

The syntax is:

```text
-p HOST_PORT:CONTAINER_PORT
```

Therefore:

```text
-p 8080:80
```

means:

```text
EC2 Host Port 8080
        ↓
Container Port 80
        ↓
Nginx
```

---

# 14. Port Binding Example

Run:

```bash
docker run -d --name nginx -p 8080:80 nginx
```

Now access:

```text
http://<EC2-PUBLIC-IP>:8080
```

Flow:

```text
Browser
   ↓
EC2 Public IP :8080
   ↓
EC2 Host Port 8080
   ↓
Docker Port Binding
   ↓
Container Port 80
   ↓
Nginx
```

---

# 15. Port Binding With Port 80

If you want users to access Nginx without specifying `:8080`:

```bash
docker run -d --name nginx -p 80:80 nginx
```

Now:

```text
EC2 Port 80
    ↓
Container Port 80
    ↓
Nginx
```

Browser:

```text
http://<EC2-PUBLIC-IP>
```

The browser uses HTTP port 80 by default.

---

# 16. Host Port vs Container Port

This is very important.

Example:

```bash
docker run -d -p 8080:80 nginx
```

Here:

```text
8080 = Host Port
80   = Container Port
```

Remember:

```text
-p HOST:CONTAINER
```

So:

```text
-p 8080:80
```

means:

```text
HOST       CONTAINER
8080   →      80
```

---

# 17. Multiple Containers and Port Binding

Suppose you want to run two Nginx containers.

You cannot normally bind both containers to the same host port.

This will cause a conflict:

```text
Container 1 → Host Port 8080
Container 2 → Host Port 8080
```

Instead:

```text
Container 1 → Host 8080 → Container 80
Container 2 → Host 8081 → Container 80
```

Commands:

```bash
docker run -d --name nginx1 -p 8080:80 nginx
```

```bash
docker run -d --name nginx2 -p 8081:80 nginx
```

Now:

```text
EC2:8080 → Nginx Container 1:80

EC2:8081 → Nginx Container 2:80
```

---

# 18. Bind to a Specific Host IP

You can specify the host IP as well.

Example:

```bash
docker run -d -p 127.0.0.1:8080:80 nginx
```

This means:

```text
127.0.0.1:8080
       ↓
Container:80
```

The port is bound only to the host's loopback interface.

This is different from:

```bash
-p 8080:80
```

which normally publishes the port on all host interfaces.

---

# 19. Port Binding vs EXPOSE

These two concepts are different.

## EXPOSE

Dockerfile:

```dockerfile
EXPOSE 80
```

This documents that the application uses port 80.

It does **not** publish the port to the host.

## Port Binding

```bash
docker run -p 8080:80 nginx
```

This actually publishes/binds the host port to the container port.

Remember:

```text
EXPOSE
→ Documentation / image metadata

-p
→ Actual host-to-container port publishing
```

---

# 20. Port Binding Mental Model

```text
                 EC2 Instance
                      |
                Host Port 8080
                      |
                      ↓
              Docker Port Mapping
                      |
                      ↓
              Container Port 80
                      |
                      ↓
                    Nginx
```

Command:

```bash
docker run -d -p 8080:80 nginx
```

Remember:

```text
-p HOST_PORT:CONTAINER_PORT
```

---

# 21. Complete Docker Troubleshooting Flow

Suppose Nginx is not accessible.

Check whether the container is running:

```bash
docker ps
```

If it is not running:

```bash
docker ps -a
```

Check container details:

```bash
docker inspect nginx
```

Check port binding:

```bash
docker port nginx
```

Check container logs:

```bash
docker logs nginx
```

Check the application/process inside the container if needed:

```bash
docker exec -it nginx /bin/bash
```

Then check the AWS EC2 Security Group and make sure the required inbound port is allowed.

For example:

```text
Browser
   ↓
EC2 Security Group
   ↓
EC2 Host Port
   ↓
Docker Port Binding
   ↓
Container Port
   ↓
Application
```

---

# 22. Important Commands — Quick Reference

```bash
# Container management
docker ps
docker ps -a
docker create --name nginx nginx
docker start nginx
docker stop nginx
docker restart nginx
docker pause nginx
docker unpause nginx
docker kill nginx
docker rm nginx
docker rm -f nginx

# Image inspection
docker image inspect nginx
docker inspect nginx

# Container inspection
docker inspect nginx

# Container state
docker inspect --format='{{.State.Status}}' nginx

# Container IP
docker inspect --format='{{.NetworkSettings.IPAddress}}' nginx

# Port information
docker port nginx

# Container logs
docker logs nginx

# Execute command inside container
docker exec -it nginx /bin/bash

# Port binding
docker run -d --name nginx -p 8080:80 nginx

# Bind to port 80
docker run -d --name nginx -p 80:80 nginx
```

---

# 23. Complete Docker Container Lifecycle

```text
                 Docker Image
                      |
                      | docker create
                      ↓
                   CREATED
                      |
                      | docker start
                      ↓
                   RUNNING
                  /         \
                 /           \
        docker pause       Application exits
              ↓                  ↓
            PAUSED             EXITED
              |                   |
       docker unpause             |
              ↓                   |
           RUNNING                |
                                  |
                            docker start
                                  |
                                  ↓
                               RUNNING

RUNNING
   |
   | docker stop
   ↓
EXITED
   |
   | docker rm
   ↓
REMOVED
```

---

# 24. Complete Nginx Example

Pull the image:

```bash
docker pull nginx
```

Run the container:

```bash
docker run -d --name nginx -p 8080:80 nginx
```

Check:

```bash
docker ps
```

Inspect the image:

```bash
docker image inspect nginx
```

Inspect the container:

```bash
docker inspect nginx
```

Check port binding:

```bash
docker port nginx
```

Check logs:

```bash
docker logs nginx
```

Open:

```text
http://<EC2-PUBLIC-IP>:8080
```

Stop:

```bash
docker stop nginx
```

Start again:

```bash
docker start nginx
```

Remove:

```bash
docker stop nginx
docker rm nginx
```

---

# 25. Key Concepts to Remember

```text
Docker Image
→ Template used to create containers

Docker Container
→ Instance created from an image

docker create
→ Creates container without starting it

docker start
→ Starts an existing container

docker stop
→ Gracefully stops a running container

docker restart
→ Restarts a container

docker pause
→ Pauses container processes

docker unpause
→ Resumes container processes

docker kill
→ Immediately terminates container

docker rm
→ Removes a container

docker inspect
→ Shows detailed Docker object information

docker image inspect
→ Shows detailed image information

docker port
→ Shows container port bindings

-p HOST:CONTAINER
→ Publishes/maps host port to container port

EXPOSE
→ Documents the container/application port; does not publish it
```

## Final Mental Model

```text
IMAGE
  ↓
docker create
  ↓
CREATED
  ↓
docker start
  ↓
RUNNING
  ↓
docker stop
  ↓
EXITED
  ↓
docker start
  ↓
RUNNING
  ↓
docker rm
  ↓
REMOVED
```

And for networking:

```text
Browser
   ↓
EC2 Public IP
   ↓
Security Group
   ↓
EC2 Host Port
   ↓
Docker Port Binding
   ↓
Container Port
   ↓
Application
```

**Most important syntax to remember:**

```bash
docker run -d -p 8080:80 nginx
```

```text
8080 = Host/EC2 Port
80   = Container Port
```