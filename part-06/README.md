# 🐳 Part 6: Advanced Docker Concepts

Welcome to **Part 6** (the final part) of the Docker Mastery Notes. In this section, we will cover advanced topics like **Multi-Stage Builds**, **Docker Security**, **AWS Deployment**, **Troubleshooting**, and a **Complete Commands Cheatsheet** for quick reference.

---

## 📑 Table of Contents
1. [Multi-Stage Docker Builds](#1-multi-stage-docker-builds)
2. [Docker Security Best Practices](#2-docker-security-best-practices)
3. [AWS Deployment & Troubleshooting](#3-aws-deployment--troubleshooting)
4. [Cloud Native Buildpacks (Bonus)](#4-cloud-native-buildpacks-bonus)
5. [Complete Docker Commands Cheatsheet](#5-complete-docker-commands-cheatsheet)
6. [Interview Questions & Answers](#6-interview-questions--answers)

---

## 1. Multi-Stage Docker Builds

A **Multi-Stage Build** allows you to use multiple `FROM` instructions in a single Dockerfile. Each `FROM` instruction starts a new stage. You can copy artifacts from one stage to another.

### Why Use Multi-Stage Builds?
*   **Smaller Image Size:** You can discard build-time dependencies (like compilers, SDKs) and keep only the runtime dependencies.
*   **Better Security:** Less number of tools in the final image means a smaller attack surface.
*   **Faster Deployment:** Smaller images are quicker to push and pull.

### Real-World Example (Java Application)

**Problem:** Compiling Java requires JDK (~300MB), but running it only requires JRE (~50MB). If we use a single-stage build, our final image will be 300MB+.

**Solution:** Use Multi-Stage Build.

```dockerfile
# ----------------------------
# Stage 1: Build Stage
# ----------------------------
FROM maven:3.8.4-openjdk-17 AS build
WORKDIR /app
COPY . .
RUN mvn clean package

# ----------------------------
# Stage 2: Runtime Stage
# ----------------------------
FROM openjdk:17-jdk-alpine
WORKDIR /app
# Copy only the compiled JAR from the build stage
COPY --from=build /app/target/my-app.jar app.jar
CMD ["java", "-jar", "app.jar"]
```

### Explanation:

- Stage 1 (`build`): Uses Maven image to compile the code. Produces `my-app.jar`.

- Stage 2: Uses a minimal Alpine image. It only copies the `my-app.jar` from Stage 1. All Maven dependencies and source code are discarded.

- Result: Final image size is drastically reduced (from ~350MB to ~60MB).

---

## 2. Docker Security Best Practices
Security is critical in production. Here are the top best practices:

### 1. Never Run as Root
By default, containers run as `root`. If an attacker compromises the container, they get root access to the host.

Solution: Create a non-root user in the Dockerfile and use the `USER` instruction.

``` dockerfile
RUN adduser -D appuser
USER appuser
```

### 2. Use Minimal Base Images
- Avoid `ubuntu` or `debian` base images for production.

- Use `alpine`, `distroless`, or `scratch` images. They have fewer packages, reducing vulnerabilities.

### 3. Don't Hardcode Secrets
- Never put passwords, API keys, or tokens in the Dockerfile or `docker-compose.yml`.

- Solution: Use Docker Secrets, AWS Secrets Manager, or environment variables injected at runtime.

### 4. Use Read-Only Filesystem
- Prevent attackers from modifying files inside the container.

```bash
docker run --read-only my-app
```

### 5. Scan Images for Vulnerabilities
- Use `docker scan` (Docker Scout) to check for known CVEs.

```bash
docker scan my-app:latest
```

### 6. Limit Resources
Prevent a single container from consuming all host resources.

```bash
docker run --memory="512m" --cpus="1.0" my-app
```

### 7. Keep Docker Updated
Always use the latest stable version of Docker to get security patches.

---

## 3. AWS Deployment & Troubleshooting
When running Docker containers on AWS EC2, you need to configure security groups to allow traffic.

### Step-by-Step: Allow Application Port in AWS
1. Go to AWS Console ➔ EC2 ➔ Instances.
2. Select your instance.
3. Click on the Security tab ➔ Security Groups.
4. Click on the Security Group ID.
5. Go to Inbound Rules ➔ Edit Inbound Rules.
6. Add Rule:
    - Type: Custom TCP
    - Port Range: `80` (or your application port)
    - Source: `0.0.0.0/0` (Anywhere)
7. Click Save Rules.

Now your application running inside the Docker container will be accessible from the internet.

### Debugging on AWS
Check Logs:

```bash
docker logs <container_id>
```

Attach Terminal:

```bash
docker attach <container_id>
```
Check Running Containers:

```bash
docker ps -a
```
Check Port Mapping:

```bash
docker port <container_id>
```

---

## 4. Cloud Native Buildpacks (Bonus)
Cloud Native Buildpacks are an alternative to Dockerfiles. They automatically detect your application type (Node.js, Java, Python, etc.) and build an optimized container image without you writing a Dockerfile.

- Tool: `pack` CLI

- Command: `pack build my-app --builder paketobuildpacks/builder:base`

- Advantage: No need to write Dockerfiles. Best practices are built-in.

---

## 5. Complete Docker Commands Cheatsheet

### Images
| Command  | Description  |
| ----------  | -------  |
| `docker search <image>`   | Search Docker Hub for an image. |
| `docker pull <image>`   | Pull an image from a registry. |
| `docker images`   | List all local images. |
| `docker rmi <image>`   | Remove an image. |
| `docker history <image>`   | View image layers. |
| `docker image prune -a`   | SRemove all unused images. |

### Containers

### Images
| Command  | Description  |
| ----------  | -------  |
| `docker run -it -d <image>`   | Run container in interactive & detached mode. |
| `docker run -p 80:80 <image>`   | Map host port to container port. |
| `docker run --name <name> <image>`   | Run with a custom name. |
| `docker ps`   | List running containers. |
| `docker ps -a`   | List all containers (running + stopped). |
| `docker start <name>`   | Start a stopped container. |
| `docker stop <name>`    | Stop a running container. |
| `docker rm <name>`   | Remove a container. |
| `docker exec -it <id> bash`   | Open bash inside a running container. |
| `docker logs -f <id>`   | Follow logs in real-time. |
| `docker stats`   | Live CPU/Memory usage. |
| `docker cp <id>:/path ./local`   | Copy files from container to host. |
| `docker inspect <id>`   | View JSON details of a container. |
| `docker attach <id>`   | Attach to a running container. |

### Networks

 Command  | Description  |
| ----------  | -------  |
| `docker network ls`   | List networks. |
| `docker network create <name> `  | Create a network. |
| `docker network inspect <name>`   | Inspect a network. |
| `docker network rm <name>`   | Remove a network. |
| `docker network prune`   | Remove unused networks. |

### Volumes

 Command  | Description  |
| ----------  | -------  |
| `docker volume create <name>`   | Create a volume. |
| `docker volume ls `  | List volumes. |
| `docker volume inspect <name>`   | Inspect a volume. |
| `docker volume rm <name>`   | Remove a volume. |
| `docker volume prune`   | Remove unused volumes. |

### Docker Compose

 Command  | Description  |
| ----------  | -------  |
| `docker compose up -d`   | Start all services in detached mode. |
| `docker compose up -d --build`  | Build and start services. |
| `docker compose down`   | Stop and remove containers. |
| `docker compose down -v`   | Stop and remove containers + volumes. |
| `docker compose logs -f`   | Follow logs of all services. |
| `docker compose ps`   | Check status of services. |

### System Cleanup

 Command  | Description  |
| ----------  | -------  |
| `docker system prune`   | Remove unused data. |
| `docker system prune -a`  | Remove all unused data (aggressive). |
| `docker compose down`   | Show Docker disk usage. |

---

## 6. Interview Questions & Answers
**Q1: What is the difference between a Docker Image and a Container?**
**A:** A Docker Image is a read-only template with instructions for creating a container. A Container is a runnable instance of an image.

**Q2: What is the difference between CMD and ENTRYPOINT?**
**A:** CMD provides default arguments that can be overridden. ENTRYPOINT configures the container to run as an executable and cannot be easily overridden.

**Q3: What is a Multi-Stage Build?**  
**A:** A technique to use multiple FROM instructions in a Dockerfile to optimize image size by discarding build-time dependencies.

**Q4: What is the difference between a Volume and a Bind Mount?**  
**A:** Volumes are managed by Docker (/var/lib/docker/volumes/), while Bind Mounts map any host directory to a container directory.

**Q5: How do containers communicate with each other?**  
**A:** Through Docker Networks. Custom bridge networks provide automatic DNS resolution, allowing containers to communicate by name.

**Q6: What is Docker Compose?**  
**A:** A tool for defining and running multi-container Docker applications using a YAML file.

**Q7: How do you persist data in Docker?**  
**A:** Using Volumes or Bind Mounts.

**Q8: What is the default network driver in Docker?**  
**A:** Bridge.

---

## 🎉 Congratulations!
You have completed all 6 parts of the Docker Mastery Notes. You now have a solid understanding of Docker from basics to advanced concepts.

---

## 📚 Complete Series:

[Part 1: Docker Fundamentals](../part-01/README.md)

[Part 2: Docker Storage & Volumes](../part-02/README.md)

[Part 3: Docker Networking](../part-03/README.md)

[Part 4: Docker Compose](../part-04/README.md)

[Part 5: Docker Registry & Image Management](../part-05/README.md)

[Part 6: Advanced Docker Concepts](../part-06/README.md)

Happy Learning! Keep Dockerizing! 🐳