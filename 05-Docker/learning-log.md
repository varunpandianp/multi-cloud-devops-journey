# Learning Log — 25-09-2026

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