# 🐳 Part 3: Docker Networking

Welcome to **Part 3** of the Docker Mastery Notes. In this section, we will learn how containers communicate with each other and with the outside world. By default, containers are isolated, but Docker Networking allows them to talk to each other securely.

---

## 📑 Table of Contents
1. [Why Do We Need Docker Networking?](#1-why-do-we-need-docker-networking)
2. [Types of Docker Networks (Drivers)](#2-types-of-docker-networks-drivers)
3. [Default Bridge vs Custom Bridge Network](#3-default-bridge-vs-custom-bridge-network)
4. [Port Publishing vs Exposing](#4-port-publishing-vs-exposing)
5. [DNS Resolution in Docker Networks](#5-dns-resolution-in-docker-networks)
6. [Docker Network Commands](#6-docker-network-commands)
7. [Real-World Example: Two Containers Communicating](#7-real-world-example-two-containers-communicating)

---

## 1. Why Do We Need Docker Networking?

Containers are designed to be **isolated**. However, in real-world applications, they need to communicate:
*   A **Backend** container needs to talk to a **Database** container.
*   A **Frontend** container needs to talk to a **Backend** container.
*   Containers need to be accessed from the **Host machine** or the **Internet**.

Docker Networking provides a way for containers to communicate securely and efficiently.

---

## 2. Types of Docker Networks (Drivers)

Docker has **7 types of network drivers**. Here are the most important ones:

| # | Network Driver | Description | Use Case |
| :--- | :--- | :--- | :--- |
| 1 | **Host** | Removes network isolation. Container uses the host's network directly. | High-performance apps (no port mapping needed). |
| 2 | **Bridge (Default)** | Default network driver. Containers get their own IP. | Basic container communication. |
| 3 | **Custom User-defined Bridge** | User-created bridge network. Provides automatic DNS resolution. | **Best practice for multi-container apps.** |
| 4 | **None** | Completely isolated network. No network access. | Security-sensitive batch jobs. |
| 5 | **MACVLAN** | Assigns a MAC address to a container, making it appear as a physical device on the network. | Legacy apps that need direct physical network access. |
| 6 | **IPVLAN** | Similar to MACVLAN but uses IPv4/IPv6. | High-performance networking. |
| 7 | **Overlay** | Connects multiple Docker daemons across different hosts. | Docker Swarm / Kubernetes (Multi-host networking). |

**💡 Note:** 
*   **Docker Swarm** (which uses Overlay networks) is now largely replaced by **Kubernetes** for orchestration.
*   However, Overlay networks are still used in Kubernetes and Docker Swarm environments.

---

## 3. Default Bridge vs Custom Bridge Network

This is a **very important interview question**.

| Feature | Default Bridge (`docker0`) | Custom Bridge |
| :--- | :--- | :--- |
| **Creation** | Automatically created by Docker. | Created manually by user. |
| **DNS Resolution** | ❌ No (must use IP addresses). | ✅ Yes (can use container names). |
| **Isolation** | All containers share the same bridge. | Containers are isolated per network. |
| **Security** | Less secure (all containers can talk). | More secure (only connected containers talk). |
| **Best Practice** | Not recommended for production. | **Recommended for production.** |

**💡 Key Takeaway:** Always create a **Custom Bridge Network** for your applications. It allows containers to communicate using their **names** instead of IP addresses.

---

## 4. Port Publishing vs Exposing

Many people confuse these two terms. Here's the difference:

*   **EXPOSE (in Dockerfile):** 
    *   It is just **documentation**. 
    *   It tells the user which port the application listens on.
    *   It does **NOT** actually publish the port to the host machine.

*   **`-p` (in `docker run`):** 
    *   It actually **publishes** the container port to the host machine.
    *   Syntax: `-p <host_port>:<container_port>`
    *   Example: `docker run -p 8080:80 nginx` (Host port 8080 ➔ Container port 80).

*   **`-P` (in `docker run`):**
    *   Publishes all exposed ports to random ports on the host.

---

## 5. DNS Resolution in Docker Networks

*   **Default Bridge:** Containers can only communicate via **IP addresses**. If a container restarts, its IP might change, breaking communication.
*   **Custom Bridge:** Docker provides an **embedded DNS server**. Containers can communicate using their **Container Names** or **Service Names** (in Docker Compose).
    *   Example: A Flask container can connect to MySQL using `mysql` as the hostname, instead of an IP address.

---

## 6. Docker Network Commands

```bash
# List all networks
docker network ls

# Create a new network
docker network create <network_name> -d <driver>
# Example: docker network create my-network -d bridge

# Inspect a network (see connected containers, IPs, etc.)
docker network inspect <network_name>

# Connect a running container to a network
docker network connect <network_name> <container_name>

# Disconnect a container from a network
docker network disconnect <network_name> <container_name>

# Remove a network
docker network rm <network_name>

# Remove all unused networks
docker network prune
```
---

## 7. Real-World Example: Two Containers Communicating
Let's create a custom network and connect two containers.

### Step 1: Create a Custom Network

```bash
docker network create my-app-network
```
### Step 2: Run a MySQL Container

```bash
docker run -d \
  --name mysql-db \
  --network my-app-network \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:8.0
```

### Step 3: Run an App Container (e.g., Alpine Linux)
```bash
docker run -it --name my-app --network my-app-network alpine sh
```

## Step 4: Test Communication
Inside the `my-app` container, try to ping the MySQL container using its name:

```bash
ping mysql-db
```
*You will see it resolves to an IP address and responds successfully!*

💡 This works because both containers are on the same custom bridge network, which provides automatic DNS resolution.

---
### 🎯 What's Next?
Now that you understand Docker Networking, move on to Part 4: Docker Compose to learn how to manage multi-container applications easily.

Happy Learning! Keep Dockerizing! 🐳

---
[< Previous ](../part-02/README.md) ---- [ Next >](../part-04/README.md)
