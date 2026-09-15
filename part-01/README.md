# 🐳 Part 1: Docker Fundamentals

Welcome to **Part 1** of the Docker Mastery Notes. In this section, we will cover the absolute basics of Docker, including what it is, how it works, and how to create your first Docker image using a Dockerfile.

---

## 📑 Table of Contents
1. [What is Docker?](#1-what-is-docker)
2. [Docker Architecture](#2-docker-architecture)
3. [Dockerfile & Workflow](#3-dockerfile--workflow)
4. [Dockerfile Instructions (Deep Dive)](#4-dockerfile-instructions-deep-dive)
5. [Docker Image Layers & Caching](#5-docker-image-layers--caching)
6. [Building & Running Containers](#6-building--running-containers)
7. [Essential Dockerfile Best Practices](#7-essential-dockerfile-best-practices)

---

## 1. What is Docker?

Docker is an open-source platform that enables developers to build, ship, and run applications in **containers**. 

*   **Container:** A lightweight, standalone, executable package of software that includes everything needed to run an application (code, runtime, system tools, libraries, and settings).
*   **Why Docker?** It solves the "It works on my machine" problem by ensuring the application runs the same way on any environment (Dev, Test, Prod).

---

## 2. Docker Architecture

Understanding the core components of Docker is crucial for interviews.

1.  **Docker Client:** The Command Line Interface (CLI) where you type commands (`docker build`, `docker run`). It sends instructions to the Docker Daemon.
2.  **Docker Daemon (`dockerd`):** The background service that listens to Docker API requests and manages Docker objects (Images, Containers, Networks, Volumes).
3.  **Docker Registry:** A storage and distribution system for Docker images (e.g., Docker Hub, AWS ECR). 
4.  **Docker Objects:** Images, Containers, Networks, Volumes, Plugins.

**Flow:** `Docker Client` ➔ `REST API` ➔ `Docker Daemon` ➔ `Docker Registry` (if pulling/pushing images)

---

## 3. Dockerfile & Workflow

A **Dockerfile** is a text document that contains all the commands a user could call on the command line to assemble an image. 

*   **Primary Purpose:** It is **only used to create a Docker Image**.
*   **Workflow:** 
    `Dockerfile` ➔ `Build Image` ➔ `Run Container` ➔ `Container Terminal`

---

## 4. Dockerfile Instructions (Deep Dive)

Here is a breakdown of the most important Dockerfile instructions:

| Instruction | Purpose | Example |
| :--- | :--- | :--- |
| **`FROM`** | Sets the Base Image. Must be the first instruction. | `FROM openjdk:17-jdk-alpine` <br> *(openjdk = image name, 17-jdk-alpine = tag)* |
| **`WORKDIR`** | Sets the working directory inside the container. | `WORKDIR /app` |
| **`COPY`** | Copies files from the Host machine to the Container. | `COPY . /app` <br> *(Host path ➔ Container path)* |
| **`RUN`** | Executes commands during the **image build** process (e.g., installing packages, compiling code). | `RUN javac Main.java` |
| **`CMD`** | Provides default commands for the container. **Can be overridden** by `docker run`. | `CMD ["java", "Main"]` |
| **`ENTRYPOINT`** | Configures the container to run as an executable. **Cannot be overridden** easily. | `ENTRYPOINT ["java", "Main"]` |
| **`EXPOSE`** | Informs Docker that the container listens on specific network ports at runtime. (Documentation only). | `EXPOSE 8080` |
| **`ENV`** | Sets environment variables (available at build & runtime). | `ENV APP_HOME=/app` |
| **`ARG`** | Build-time variables (only available during build, not at runtime). | `ARG VERSION=1.0` |
| **`USER`** | Sets the user for running the container (Security best practice). | `USER appuser` |
| **`HEALTHCHECK`** | Tells Docker how to test a container to check if it's still working. | `HEALTHCHECK CMD curl -f http://localhost/ || exit 1` |

**💡 CMD vs ENTRYPOINT:**
*   `CMD`: Sets default command and/or parameters, which can be overwritten from the command line when docker run.
*   `ENTRYPOINT`: Configures a container that will run as an executable. You can use `CMD` to provide default arguments to `ENTRYPOINT`.

---

## 5. Docker Image Layers & Caching

*   **Layers:** Each instruction in a Dockerfile creates a new layer in the image. Layers are **read-only**.
*   **Caching:** When you rebuild an image, Docker checks if the instruction has changed. 
    *   If an instruction hasn't changed, Docker uses the **cached layer**.
    *   If an instruction changes, **all subsequent layers** are rebuilt from scratch.
*   **Best Practice:** Place instructions that change frequently (like `COPY . .`) at the **bottom** of the Dockerfile. Place instructions that rarely change (like `FROM`, `RUN apt-get install`) at the **top**.

---

## 6. Building & Running Containers

### Building an Image
```bash
docker build -t <image_name> <location_of_Dockerfile>
# Example: docker build -t my-app .
```
### Running a Container
```bash
docker run -p <host_port>:<container_port> <image_name>
# Example: docker run -p 80:80 my-app
```

### Updating an Image

If you update your source code and run `docker build` again:
- Docker will use the cache for unchanged layers.
- Only the `COPY` and `RUN` instructions (and anything after them) will execute again.
- You must create a new container to use the updated image (old containers won't update automatically).

### Checking Logs

```bash
docker logs <container_id>
```

---

## 7. Essential Dockerfile Best Practices

1. Use `.dockerignore`: Always create a `.dockerignore` file to exclude unnecessary files (`node_modules`, `.git`, `*.log`) from the build context. This speeds up builds and reduces image size.

2. Minimal Base Images: Use `alpine` or `distroless` images to reduce attack surface and image size.

3. Non-Root User: Never run containers as `root`. Use the `USER` instruction.

4. Multi-Stage Builds: Use multi-stage builds to separate build dependencies from runtime dependencies (covered in Part 6).

5. Combine RUN Commands: Combine multiple RUN commands using && to reduce the number of layers.

    - Bad: `RUN apt-get update` then `RUN apt-get install -y nginx`

    - Good: `RUN apt-get update && apt-get install -y nginx`

6. Use Specific Tags: Avoid using latest tag. Use specific versions (e.g., `node:18-alpine` instead of `node:latest`).


---

## 🎯 What's Next?
Now that you understand Docker Fundamentals, move on to Part 2: Docker Storage & Volumes to learn how to persist your container data.

---

Happy Learning! Keep Dockerizing! 🐳

---

[< Previous ](../README.md) ---- [ Next >](../part-02/README.md)