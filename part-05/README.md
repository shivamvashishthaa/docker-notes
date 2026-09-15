# 🐳 Part 5: Docker Registry & Image Management

Welcome to **Part 5** of the Docker Mastery Notes. In this section, we will learn how to store, share, and manage Docker images using **Docker Registry**. We will also cover essential image management commands to keep your system clean and optimized.

---

## 📑 Table of Contents
1. [What is a Docker Registry?](#1-what-is-a-docker-registry)
2. [Types of Docker Registries](#2-types-of-docker-registries)
3. [Docker Hub: The Default Registry](#3-docker-hub-the-default-registry)
4. [Tagging & Pushing Images](#4-tagging--pushing-images)
5. [Pulling Images from a Registry](#5-pulling-images-from-a-registry)
6. [Image Management Commands](#6-image-management-commands)
7. [Cleaning Up: Prune Commands](#7-cleaning-up-prune-commands)
8. [Real-World Example: Push an Image to Docker Hub](#8-real-world-example-push-an-image-to-docker-hub)

---

## 1. What is a Docker Registry?

A **Docker Registry** is a storage and distribution system for Docker images. 

*   It allows you to **push** (upload) your custom images and **pull** (download) images created by others.
*   It acts as a central repository for sharing images across teams, servers, and CI/CD pipelines.

**Analogy:** Just like GitHub stores your code, Docker Registry stores your Docker images.

---

## 2. Types of Docker Registries

There are two main types of registries:

1.  **Public Registry:** 
    *   Open to everyone.
    *   Example: **Docker Hub** (default), GitHub Container Registry (GHCR), Quay.io.
    *   Anyone can pull public images.

2.  **Private Registry:**
    *   Restricted access. Only authorized users can pull/push images.
    *   Examples: **AWS ECR** (Elastic Container Registry), Google Artifact Registry, Azure Container Registry (ACR).
    *   Used for proprietary/company-specific images.

---

## 3. Docker Hub: The Default Registry

Docker Hub is the world's largest library and community for container images.

*   **Free Tier:** Unlimited public repositories, 1 private repository.
*   **Official Images:** Verified images maintained by Docker (e.g., `nginx`, `mysql`, `ubuntu`).
*   **URL:** [hub.docker.com](https://hub.docker.com)

**Searching for Images:**
```bash
docker search <image_name>
# Example: docker search nginx
```

---

## 4. Tagging & Pushing Images
Before you can push an image to a registry, you need to tag it properly. The tag must follow this format:

```text
<registry_url>/<username>/<image_name>:<tag>
```

- For Docker Hub: `<username>/<image_name>:<tag>`

- For AWS ECR: `<aws_account_id>.dkr.ecr.<region>.amazonaws.com/<image_name>:<tag>`

### Step 1: Build or Identify Your Image
```bash
docker build -t my-app .
```

### Step 2: Tag the Image
```bash
docker image tag <old_image_name>:<tag> <username>/<new_image_name>:<tag>
# Example: docker image tag my-app:latest john/my-app:v1
```

### Step 3: Login to the Registry
```bash
docker login
# Enter your Docker Hub username and password
```
For AWS ECR:

```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

### Step 4: Push the Image
```bash
docker push <username>/<image_name>:<tag>
# Example: docker push john/my-app:v1
```

---

## 5. Pulling Images from a Registry
To download an image from a registry:

```bash
docker pull <image_name>:<tag>
# Example: docker pull nginx:latest
# Example: docker pull john/my-app:v1
```

If you don't specify a tag, Docker defaults to `latest`.

---

## 6. Image Management Commands
Here are the essential commands to manage your local images:

```bash
# List all local images
docker images

# View the history (layers) of an image
docker history <image_name>
# Example: docker history nginx

# Inspect an image (see metadata, env vars, etc.)
docker inspect <image_name>

# Remove a specific image
docker rmi <image_name>
# Example: docker rmi my-app:latest

# Remove an image forcefully (even if used by a container)
docker rmi -f <image_name>

# Save an image to a tar file (for offline transfer)
docker save -o my-app.tar my-app:latest

# Load an image from a tar file
docker load -i my-app.tar
```
---

## 7. Cleaning Up: Prune Commands
Over time, your system will accumulate unused images, containers, and volumes. Use these commands to clean up:

```bash
# Remove all unused images (dangling + unused)
docker image prune -a

# Remove all stopped containers
docker container prune

# Remove all unused networks
docker network prune

# Remove all unused volumes (DANGER: Data loss!)
docker volume prune

# Remove EVERYTHING unused (containers, images, networks, build cache)
docker system prune -a
```

⚠️ Warning: `docker system prune -a` is very powerful. It will delete all stopped containers, all networks not used by at least one container, all images without at least one container associated with them, and all build cache. Use it with caution.


---

## 8. Real-World Example: Push an Image to Docker Hub
Let's push a custom image to Docker Hub.

### Step 1: Build Your Image
```bash
docker build -t my-flask-app .
```

### Step 2: Tag the Image
```bash
docker image tag my-flask-app:latest yourusername/my-flask-app:v1
```

### Step 3: Login to Docker Hub
```bash
docker login
```
### Step 4: Push the Image
```bash
docker push yourusername/my-flask-app:v1
```
### Step 5: Verify on Docker Hub
Go to `https://hub.docker.com/r/yourusername/my-flask-app` and you will see your image.

### Step 6: Pull the Image on Another Machine
```bash
docker pull yourusername/my-flask-app:v1
docker run -p 5000:5000 yourusername/my-flask-app:v1
```
---

## 🎯 What's Next?
Now that you understand Docker Registry, move on to Part 6: Advanced Docker Concepts to learn about Multi-Stage Builds, Security Best Practices, and AWS Deployment.

Happy Learning! Keep Dockerizing! 🐳

---

[< Previous ](../part-04/README.md) ---- [ Next >](../part-06/README.md)