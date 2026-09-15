# 🐳 Part 4: Docker Compose (Multi-Container Apps)

Welcome to **Part 4** of the Docker Mastery Notes. In this section, we will learn how to manage multi-container applications using **Docker Compose**. Instead of running multiple `docker run` commands manually, we define everything in a single YAML file and run it with one command.

---

## 📑 Table of Contents
1. [What is Docker Compose?](#1-what-is-docker-compose)
2. [Why Use Docker Compose?](#2-why-use-docker-compose)
3. [Docker Compose File Structure](#3-docker-compose-file-structure)
4. [Key Concepts (Services, Networks, Volumes)](#4-key-concepts-services-networks-volumes)
5. [Real-World Example: Flask + MySQL (2-Tier App)](#5-real-world-example-flask--mysql-2-tier-app)
6. [Healthchecks in Docker Compose](#6-healthchecks-in-docker-compose)
7. [Environment Variables & env_file](#7-environment-variables--env_file)
8. [Docker Compose Commands](#8-docker-compose-commands)

---

## 1. What is Docker Compose?

Docker Compose is a tool for defining and running **multi-container Docker applications**. 

*   It uses a **YAML file** (`docker-compose.yml`) to configure your application's services.
*   With a single command (`docker compose up`), you can create and start all the services from your configuration.
*   It automates the process of building images, creating networks, and starting containers.

---

## 2. Why Use Docker Compose?

*   **Automation:** No need to run multiple `docker run` commands manually.
*   **Simplicity:** Everything is defined in one file.
*   **Reproducibility:** Anyone can clone your repo and run `docker compose up` to get the exact same environment.
*   **Networking:** Compose automatically creates a custom bridge network for your services.
*   **Dependency Management:** You can define which service starts first using `depends_on`.

---

## 3. Docker Compose File Structure

A typical `docker-compose.yml` file has three main sections:

```yaml
version: '3.8'   # Docker Compose version

services:        # Define your containers here
  web:
    image: nginx
    ports:
      - "80:80"

networks:        # Define custom networks
  my-network:

volumes:         # Define persistent volumes
  my-data:
```

- `version`: The Docker Compose file format version.

- `services`: Defines the containers (e.g., `web`, `db`, `app`).

- `networks`: Defines custom networks.

- `volumes`: Defines persistent volumes.

---

## 4. Key Concepts (Services, Networks, Volumes)

### Services
A service is a container definition. You can define:
- image: The Docker image to use.
- build: The path to the Dockerfile (if you want to build a custom image).
- ports: Port mapping (host:container).
- environment: Environment variables.
- volumes: Volume mounts.
- networks: Which network to connect to.
- depends_on: Which services this service depends on.
- healthcheck: How to test if the service is healthy.

### Networks
Compose automatically creates a network for your app. You can also define custom networks:

```yaml
networks:
  two-tier:
    driver: bridge
```

All services in the same network can communicate using their service names.

### Volumes
Used to persist data:

```yaml
volumes:
  mysql-data:
```

---

## 5. Real-World Example: Flask + MySQL (2-Tier App)
Let's create a 2-tier application with a Flask backend and a MySQL database.

```yaml
# docker-compose.yml

version: '3.8'

services:
  # MySQL Database Service
  mysql:
    container_name: mysql
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: "devops"
      MYSQL_ROOT_PASSWORD: "root"
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - two-tier
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 60s

  # Flask Backend Service
  flask:
    build:
      context: .
    container_name: two-tier-backend
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: root
      MYSQL_DB: devops
    networks:
      - two-tier
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:5000/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
    depends_on:
      - mysql

networks:
  two-tier:

volumes:
  mysql-data:
```

### Explanation:
- `mysql` Service: Uses the official MySQL image. It has a volume (`mysql-data`) to persist data and a healthcheck to ensure the database is ready before the Flask app starts.

- `flask Service`: Builds from the current directory (`context: .`). It connects to the MySQL container using `MYSQL_HOST: mysql` (thanks to Docker's DNS resolution).

- `depends_on`: Ensures that the Flask container starts only after the MySQL container is up.

- `networks`: Both services are on the `two-tier` network, allowing them to communicate by name.

---

## 6. Healthchecks in Docker Compose
A healthcheck is used to determine if a container is healthy and ready to accept traffic.

- `test`: The command to run.
    - `["CMD", "..."]` runs the command directly.
    - `["CMD-SHELL", "..."]` runs the command inside a shell.
- `interval`: How often to run the test.
- `timeout`: How long to wait for a response.
- `retries`: How many times to retry before marking as unhealthy.
- `start_period`: Grace period for the container to start.

#### Why use healthchecks?

- To ensure that the database is ready before the app connects to it.
- To automatically restart unhealthy containers.

---

## 7. Environment Variables & env_file
Instead of hardcoding environment variables in the YAML file, you can use a `.env` file.

```.env
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=devops
```

```yaml
# docker-compose.yml
services:
  mysql:
    image: mysql:8.0
    env_file:
      - .env
```

💡 Pro Tip: Never commit your `.env` file to GitHub. Add it to `.gitignore`.

---

## 8. Docker Compose Commands

```bash
# Install Docker Compose V2 (Ubuntu)
sudo apt-get install docker-compose-v2

# Start all services in detached mode
docker compose up -d

# Build images and start services
docker compose up -d --build

# Stop all services
docker compose down

# Stop services and remove volumes (DANGER: Data loss!)
docker compose down -v

# View logs of all services
docker compose logs

# Follow logs in real-time
docker compose logs -f

# Check status of services
docker compose ps

# Restart a specific service
docker compose restart <service_name>

# Scale a service (run multiple instances)
docker compose up -d --scale web=3
```

---

## 🎯 What's Next?
Now that you understand Docker Compose, move on to Part 5: Docker Registry & Image Management to learn how to share your images with the world.

Happy Learning! Keep Dockerizing! 🐳

---

[< Previous ](../part-03/README.md) ---- [ Next >](../part-05/README.md)
