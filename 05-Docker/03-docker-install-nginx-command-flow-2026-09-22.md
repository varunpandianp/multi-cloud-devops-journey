# Docker Basic Nginx Commands and Flow

## 1. Goal

Run an **Nginx web server inside a Docker container** on an EC2 instance.

The basic flow is:

```text
EC2 Instance
     ↓
Docker
     ↓
Pull Nginx Image
     ↓
Run Container
     ↓
Nginx Starts
     ↓
EC2 Port 80
     ↓
Browser
     ↓
Nginx Welcome Page
```

---

# 2. Check Docker Installation

First, check whether Docker is installed.

```bash
docker --version
```

Example output:

```text
Docker version 28.x.x
```

If Docker is installed, we can continue.

---

# 3. Download the Nginx Docker Image

Use:

```bash
docker pull nginx
```

`docker pull` downloads the Docker image from a Docker registry.

By default, Docker pulls from **Docker Hub**.

Flow:

```text
Docker Hub
    ↓
Nginx Image
    ↓
EC2 Local Docker Storage
```

Check the downloaded image:

```bash
docker images
```

You should see something similar to:

```text
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    xxxxxxxx       ...           ...
```

---

# 4. What Is the Nginx Image?

The Nginx image is a packaged template containing the Nginx application and the required files/configuration needed to create an Nginx container.

Important:

```text
Image = Template / Blueprint
Container = Running Instance of the Image
```

The image itself is not the running Nginx server.

---

# 5. Run Nginx Container

Run Nginx using:

```bash
docker run -d --name nginx -p 80:80 nginx
```

Explanation:

```text
docker run
→ Create and start a container

-d
→ Run in detached/background mode

--name nginx
→ Give the container the name "nginx"

-p 80:80
→ Map EC2 host port 80 to container port 80

nginx
→ Use the Nginx Docker image
```

---

# 6. What Happens When We Run the Command?

When we execute:

```bash
docker run -d --name nginx -p 80:80 nginx
```

Docker does the following:

```text
Nginx Image
     ↓
Create Container
     ↓
Start Container
     ↓
Nginx Process Starts
     ↓
Nginx Listens on Container Port 80
     ↓
EC2 Port 80 → Container Port 80
```

---

# 7. Check Running Containers

Use:

```bash
docker ps
```

Example:

```text
CONTAINER ID   IMAGE   COMMAND                  PORTS
xxxxxxxx       nginx   "/docker-entrypoint..."  0.0.0.0:80->80/tcp
```

The important part is:

```text
0.0.0.0:80->80/tcp
```

This means:

```text
EC2 Host Port 80
       ↓
Container Port 80
       ↓
Nginx
```

---

# 8. Check All Containers

To see both running and stopped containers:

```bash
docker ps -a
```

Difference:

```text
docker ps
→ Shows running containers

docker ps -a
→ Shows all containers
```

---

# 9. Access Nginx from Browser

The EC2 Security Group must allow inbound HTTP traffic on port 80.

Security Group rule:

```text
Type:   HTTP
Port:   80
Source: 0.0.0.0/0
```

Then use the EC2 public IP:

```text
http://<EC2-PUBLIC-IP>
```

Example:

```text
http://13.234.xx.xx
```

Flow:

```text
Browser
   ↓
EC2 Public IP :80
   ↓
EC2 Security Group
   ↓
EC2 Port 80
   ↓
Docker Port Mapping
   ↓
Nginx Container Port 80
   ↓
Nginx
   ↓
Nginx Welcome Page
```

---

# 10. Stop the Nginx Container

To stop the running container:

```bash
docker stop nginx
```

Flow:

```text
Running Nginx Container
          ↓
     docker stop
          ↓
Stopped Container
```

The container is stopped but still exists.

Check:

```bash
docker ps -a
```

You should see its status as:

```text
Exited
```

---

# 11. Start the Stopped Nginx Container

If you want to start the same container again:

```bash
docker start nginx
```

Check:

```bash
docker ps
```

Nginx should be running again.

Important:

```text
docker stop
→ Stops the container

docker start
→ Starts the existing stopped container
```

You don't need to create a new container every time.

---

# 12. Restart the Nginx Container

You can restart the existing container using:

```bash
docker restart nginx
```

Flow:

```text
Running Container
       ↓
    Restart
       ↓
Stopped
       ↓
Started Again
```

---

# 13. Remove the Nginx Container

First stop it:

```bash
docker stop nginx
```

Then remove it:

```bash
docker rm nginx
```

Check:

```bash
docker ps -a
```

The container should no longer appear.

Important:

```text
docker stop
→ Stop container

docker rm
→ Remove container
```

---

# 14. Remove the Nginx Image

If you also want to remove the downloaded Nginx image:

```bash
docker rmi nginx
```

If a container still exists using that image, Docker may prevent the image from being removed.

So the normal cleanup flow is:

```bash
docker stop nginx
docker rm nginx
docker rmi nginx
```

---

# 15. Complete Nginx Docker Practice

## Step 1 — Check Docker

```bash
docker --version
```

## Step 2 — Download Nginx Image

```bash
docker pull nginx
```

## Step 3 — Check Image

```bash
docker images
```

## Step 4 — Run Nginx

```bash
docker run -d --name nginx -p 80:80 nginx
```

## Step 5 — Check Container

```bash
docker ps
```

## Step 6 — Open Browser

```text
http://<EC2-PUBLIC-IP>
```

## Step 7 — Stop Container

```bash
docker stop nginx
```

## Step 8 — Check Container

```bash
docker ps -a
```

## Step 9 — Start Again

```bash
docker start nginx
```

## Step 10 — Stop Again

```bash
docker stop nginx
```

## Step 11 — Remove Container

```bash
docker rm nginx
```

## Step 12 — Remove Image

```bash
docker rmi nginx
```

---

# 16. Complete Docker Nginx Flow

```text
                  Docker Hub
                      |
                      | docker pull nginx
                      ↓
                Nginx Image
                      |
                      | docker run
                      ↓
                Nginx Container
                      |
                      ↓
                  Nginx :80
                      |
                      ↓
                EC2 Port :80
                      |
                      ↓
               EC2 Public IP
                      |
                      ↓
                   Browser
                      |
                      ↓
              Nginx Welcome Page
```

---

# 17. Image vs Container

Remember this clearly:

```text
docker pull nginx
        ↓
Downloads IMAGE
```

```text
docker run nginx
        ↓
Uses IMAGE
        ↓
Creates CONTAINER
        ↓
Starts Nginx
```

```text
docker stop nginx
        ↓
Stops CONTAINER
```

```text
docker start nginx
        ↓
Starts existing CONTAINER
```

```text
docker rm nginx
        ↓
Removes CONTAINER
```

```text
docker rmi nginx
        ↓
Removes IMAGE
```

---

# 18. Important Commands to Remember

```bash
# Check Docker
docker --version

# Download image
docker pull nginx

# List images
docker images

# Run Nginx container
docker run -d --name nginx -p 80:80 nginx

# Show running containers
docker ps

# Show all containers
docker ps -a

# Stop container
docker stop nginx

# Start stopped container
docker start nginx

# Restart container
docker restart nginx

# Remove container
docker rm nginx

# Remove image
docker rmi nginx
```

---

# 19. Simple Mental Model

```text
docker pull
     ↓
Download Image

docker run
     ↓
Image → Container
     ↓
Application Running

docker ps
     ↓
Check Running Container

docker stop
     ↓
Stop Container

docker start
     ↓
Start Existing Container

docker rm
     ↓
Remove Container

docker rmi
     ↓
Remove Image
```

## One-Line Summary

> **`docker pull` downloads the image, `docker run` creates and starts a container from that image, `docker stop` stops the container, `docker start` starts it again, `docker rm` removes the container, and `docker rmi` removes the image.**