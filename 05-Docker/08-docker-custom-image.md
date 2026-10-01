# Docker Custom Image

## 📚 Topic

**Docker Custom Images — Create, Build, Upload, Pull and Run**

---

# 1. What is a Docker Image?

A Docker image is a packaged template that contains everything required to create and run a container.

An image can contain:

- Application code
- Application dependencies
- Required libraries
- Runtime
- Configuration
- Required files
- Startup instructions

Example:

    Docker Image
         ↓
    docker run
         ↓
    Docker Container
         ↓
    Application Running

---

# 2. What is a Custom Docker Image?

A **custom Docker image** is an image that we create for our own application instead of directly using a pre-built image from Docker Hub or another registry.

For example, Docker Hub may already provide:

    nginx
    ubuntu
    node
    python
    java
    mysql
    postgres

But our application may require:

    Java 17
    Application JAR
    Application configuration
    Required dependencies
    Specific startup command

So we create our own image.

Example:

    Base Image
        ↓
    Java 17
        ↓
    Application JAR
        ↓
    Configuration
        ↓
    Startup Command
        ↓
    Custom Image
        ↓
    Container

---

# 3. Why Do We Need a Custom Image?

## Problem

Suppose we have our own application:

    invoice-service.jar

Every time we create a server/container, we would otherwise need to:

    Install Java
        ↓
    Install dependencies
        ↓
    Copy application
        ↓
    Configure application
        ↓
    Configure startup command
        ↓
    Start application

Doing this manually is:

- Time-consuming
- Error-prone
- Difficult to reproduce
- Difficult to automate

---

## Solution — Custom Image

We package everything required by the application into an image.

Then:

    Custom Image
         ↓
    docker run
         ↓
    Container
         ↓
    Application starts

The same image can be used in:

- Development
- Testing
- Staging
- Production
- CI/CD pipelines
- Multiple servers

---

# 4. Real-World Example

Suppose we have:

    invoice-service.jar

The application requires:

    Java 17

We can create:

    invoice-service:1.0

Then the image contains:

    Java Runtime
         +
    invoice-service.jar
         +
    Startup Configuration

Now we can run the same image anywhere Docker is available.

Example:

    Developer Machine
          ↓
    invoice-service:1.0

    Testing Server
          ↓
    invoice-service:1.0

    Production Server
          ↓
    invoice-service:1.0

This provides consistency between environments.

---

# 5. How Do We Create a Custom Image?

The normal process is:

    Application
         ↓
    Dockerfile
         ↓
    docker build
         ↓
    Custom Docker Image
         ↓
    docker run
         ↓
    Container

If the image needs to be shared with another server:

    Custom Image
         ↓
    docker tag
         ↓
    docker push
         ↓
    Docker Registry
         ↓
    docker pull
         ↓
    Another Server
         ↓
    docker run

---

# 6. Step 1 — Create Application

For example:

    invoice-service.jar

Our application is ready.

Project directory:

    invoice-service/
    ├── Dockerfile
    └── target/
        └── invoice-service.jar

---

# 7. Step 2 — Create a Dockerfile

Create a file named:

    Dockerfile

Example:

    FROM eclipse-temurin:17-jre-alpine

    WORKDIR /app

    COPY target/invoice-service.jar app.jar

    EXPOSE 8080

    ENTRYPOINT ["java", "-jar", "app.jar"]

---

# 8. Understanding the Dockerfile

## FROM

    FROM eclipse-temurin:17-jre-alpine

Specifies the base image.

The application gets a Java 17 runtime environment from the base image.

---

## WORKDIR

    WORKDIR /app

Creates/sets the working directory:

    /app

---

## COPY

    COPY target/invoice-service.jar app.jar

Copies the application JAR into the image.

The result is:

    /app/app.jar

---

## EXPOSE

    EXPOSE 8080

Documents that the application listens on port:

    8080

Important:

`EXPOSE` does not publish the port to the host.

---

## ENTRYPOINT

    ENTRYPOINT ["java", "-jar", "app.jar"]

Defines the command that starts the application.

When the container starts:

    java -jar app.jar

is executed.

---

# 9. Step 3 — Build the Custom Image

Run:

    docker build -t invoice-service:1.0 .

### Meaning

    docker build
        ↓
    Build an image

    -t invoice-service:1.0
        ↓
    Name and tag the image

    .
        ↓
    Use the current directory as build context

---

# 10. Verify the Image

Run:

    docker images

Or:

    docker images invoice-service

You should see something similar to:

    REPOSITORY          TAG       IMAGE ID
    invoice-service     1.0       abc123

Now the custom image exists locally.

---

# 11. Step 4 — Run the Custom Image

Run:

    docker run -d \
      --name invoice-service \
      -p 8080:8080 \
      invoice-service:1.0

### Meaning

    -d
        ↓
    Detached mode

    --name invoice-service
        ↓
    Container name

    -p 8080:8080
        ↓
    Host port 8080 → Container port 8080

    invoice-service:1.0
        ↓
    Image to use

---

# 12. Check the Container

Run:

    docker ps

You should see:

    invoice-service

running.

---

# 13. Access the Application

If the Docker container is running on an EC2 instance:

    Internet
       ↓
    EC2 Public IP
       ↓
    Port 8080
       ↓
    Docker Host
       ↓
    Container Port 8080
       ↓
    Application

Example:

    http://<EC2-PUBLIC-IP>:8080

The EC2 Security Group must allow inbound traffic to port `8080`.

---

# 14. Check Application Logs

Run:

    docker logs invoice-service

To continuously watch logs:

    docker logs -f invoice-service

This is one of the first commands to use when troubleshooting an application container.

---

# 15. Stop the Container

Run:

    docker stop invoice-service

The container stops, but the image still exists.

Important:

    docker stop
        ↓
    Stops container

It does NOT delete the image.

---

# 16. Start the Existing Container Again

Run:

    docker start invoice-service

The existing container starts again using the same image.

---

# 17. Remove the Container

Run:

    docker rm invoice-service

The container is removed.

The image remains.

---

# 18. Run the Same Image Again

Even after removing the container, we can create another container from the same image:

    docker run -d \
      --name invoice-service \
      -p 8080:8080 \
      invoice-service:1.0

This demonstrates an important Docker concept:

> An image is a reusable template from which containers are created.

---

# 19. Why Upload the Custom Image?

A custom image created on our laptop exists only on our laptop.

Suppose we want to run the same application on:

    EC2
    Jenkins
    Kubernetes
    Another Developer Machine
    Production Server

We need a way to distribute the image.

The solution is a:

# Docker Registry

A registry stores Docker images.

Examples include:

- Docker Hub
- Amazon ECR
- GitHub Container Registry
- Private container registries

---

# 20. Image Upload Flow

The complete flow is:

    Local Machine
         ↓
    Custom Image
         ↓
    docker tag
         ↓
    docker push
         ↓
    Docker Registry
         ↓
    docker pull
         ↓
    EC2 / Server
         ↓
    docker run
         ↓
    Container

---

# 21. Step 5 — Login to Docker Registry

For Docker Hub:

    docker login

Enter the required authentication details.

For production/CI systems, use an appropriate access token or credential mechanism instead of exposing passwords.

---

# 22. Step 6 — Tag the Image

Suppose the Docker Hub username is:

    myusername

Local image:

    invoice-service:1.0

Tag it:

    docker tag invoice-service:1.0 myusername/invoice-service:1.0

Now the image has a registry-compatible name.

Format:

    <username>/<repository>:<tag>

Example:

    myusername/invoice-service:1.0

---

# 23. Why Do We Need `docker tag`?

The local image:

    invoice-service:1.0

does not contain the Docker Hub repository name.

The tag tells Docker where the image belongs.

Example:

    invoice-service:1.0

becomes:

    myusername/invoice-service:1.0

---

# 24. Step 7 — Push the Image

Run:

    docker push myusername/invoice-service:1.0

Docker uploads the image layers to the registry.

Flow:

    Local Image
         ↓
    Docker Push
         ↓
    Registry
         ↓
    Image Stored

---

# 25. Verify the Image in the Registry

After pushing, the image will be available in the registry repository.

Example:

    myusername/invoice-service:1.0

The registry now stores the image.

---

# 26. Step 8 — Pull the Image on Another Server

Suppose we have an EC2 instance.

First:

    docker login

Then:

    docker pull myusername/invoice-service:1.0

The image is downloaded from the registry to the EC2 instance.

---

# 27. Step 9 — Run the Pulled Image

Run:

    docker run -d \
      --name invoice-service \
      -p 8080:8080 \
      myusername/invoice-service:1.0

Now the EC2 server runs the same image that was created on the development machine.

---

# 28. Complete Custom Image Lifecycle

The complete process is:

    1. Write Application
            ↓
    2. Create Dockerfile
            ↓
    3. docker build
            ↓
    4. Custom Docker Image
            ↓
    5. Test Locally
            ↓
    6. docker tag
            ↓
    7. docker push
            ↓
    8. Docker Registry
            ↓
    9. docker pull
            ↓
    10. Production Server
            ↓
    11. docker run
            ↓
    12. Application Running

---

# 29. Important Difference — Image vs Container

## Image

An image is the packaged template.

Example:

    invoice-service:1.0

## Container

A container is a running instance created from that image.

Example:

    invoice-container

Mental model:

    Docker Image
         ↓
    docker run
         ↓
    Container

One image can create multiple containers.

Example:

    invoice-service:1.0
          ↓
    ┌─────┼─────┐
    ↓     ↓     ↓
    C1    C2    C3

---

# 30. Important Difference — Build vs Run

## Build

    docker build

Creates the image.

    Dockerfile
         ↓
    docker build
         ↓
    Image

## Run

    docker run

Creates and starts a container from the image.

    Image
      ↓
    docker run
      ↓
    Container

---

# 31. Important Difference — Push vs Pull

## Push

    docker push

Uploads an image:

    Local Machine
         ↓
    Registry

## Pull

    docker pull

Downloads an image:

    Registry
         ↓
    Local Machine / EC2

---

# 32. Custom Image in a Real DevOps Pipeline

In a real CI/CD environment:

    Developer
        ↓
    Git Push
        ↓
    Jenkins / GitHub Actions
        ↓
    Build Application
        ↓
    Run Tests
        ↓
    docker build
        ↓
    Custom Docker Image
        ↓
    Security Scan
        ↓
    docker push
        ↓
    Container Registry
        ↓
    Deployment
        ↓
    EC2 / Kubernetes / Swarm
        ↓
    Container
        ↓
    Application

---

# 33. Why Custom Images Are Important in DevOps

Custom images provide:

- Consistent environments
- Reproducible deployments
- Faster application deployment
- Easy distribution
- Easier CI/CD automation
- Versioned application releases
- Reduced manual configuration
- Easier rollback

Example:

    invoice-service:1.0
    invoice-service:1.1
    invoice-service:1.2

If version `1.2` has a problem, we can deploy a previously tested version such as:

    invoice-service:1.1

This is one reason image versioning is important.

---

# 34. Production Best Practices

## Use Versioned Tags

Prefer:

    invoice-service:1.0

or:

    invoice-service:2026.10.01

or a Git commit-based tag.

Avoid relying on:

    invoice-service:latest

for production deployments.

---

## Keep Images Small

Use:

- Small base images
- Multi-stage builds
- `.dockerignore`
- Only required runtime dependencies

---

## Do Not Put Secrets in Images

Do not put:

- Passwords
- API keys
- Tokens
- Private keys

inside the Dockerfile or image.

Inject sensitive configuration at runtime using appropriate secret-management mechanisms.

---

## Run as Non-Root

Use:

    USER

in the Dockerfile whenever possible.

---

# 35. Quick Command Cheat Sheet

## Build

    docker build -t invoice-service:1.0 .

## List Images

    docker images

## Run

    docker run -d --name invoice-service -p 8080:8080 invoice-service:1.0

## Check Containers

    docker ps

## Logs

    docker logs invoice-service

## Stop

    docker stop invoice-service

## Start

    docker start invoice-service

## Remove Container

    docker rm invoice-service

## Login

    docker login

## Tag

    docker tag invoice-service:1.0 myusername/invoice-service:1.0

## Push

    docker push myusername/invoice-service:1.0

## Pull

    docker pull myusername/invoice-service:1.0

## Run Pulled Image

    docker run -d --name invoice-service -p 8080:8080 myusername/invoice-service:1.0

---

# 36. Interview Questions

## Q1. What is a custom Docker image?

A custom Docker image is an image built specifically for an application using a Dockerfile and usually based on an existing base image.

## Q2. Why do we need custom images?

To package the application, runtime and required configuration into a reproducible unit that can be used consistently across environments.

## Q3. How do you create a custom image?

    Dockerfile
        ↓
    docker build
        ↓
    Custom Image

## Q4. How do you run a custom image?

    docker run <image>

Example:

    docker run -d -p 8080:8080 invoice-service:1.0

## Q5. How do you upload an image?

    docker tag
        ↓
    docker push

## Q6. Where do you upload it?

To a Docker registry such as:

- Docker Hub
- Amazon ECR
- GitHub Container Registry
- Private Registry

## Q7. How do you run the image on another server?

    docker pull <image>
        ↓
    docker run <image>

## Q8. What is the difference between Dockerfile and Docker image?

**Dockerfile:** Instructions/recipe used to build the image.

**Docker image:** The packaged result produced from those instructions.

## Q9. What is the difference between image and container?

**Image:** Immutable template/package.

**Container:** Running instance created from the image.

---

# 37. Final Mental Model

Remember this simple flow:

    Dockerfile
        ↓
    docker build
        ↓
    Custom Image
        ↓
    docker run
        ↓
    Container
        ↓
    Application

When another server needs the same application:

    Custom Image
        ↓
    docker tag
        ↓
    docker push
        ↓
    Container Registry
        ↓
    docker pull
        ↓
    Another Server
        ↓
    docker run
        ↓
    Container
        ↓
    Application

## One-Line Summary

> **A custom Docker image packages our application and its required runtime into a reusable, versioned image that can be built once, stored in a registry, pulled onto different environments, and run consistently as a container.** 
> 