# Docker - Complete Oneshot Guide

## 📋 Video Overview
- **Topic**: Complete Docker Course (One-shot video)
- **Goal**: Working knowledge of Docker for real-world applications
- **Why Important**: Microservices containerization, cloud deployment, open-source contributions

---

## 🚨 The Problem: "It Works on My Machine"

### Scenario: Development Environment Mismatch

| Person | OS | Java Version | MySQL Version | Other |
|--------|-----|--------------|---------------|-------|
| **Bob** (Developer) | Windows | Java 25 | MySQL 8 | Version 6 |
| **Alice** (QA) | Linux | Java 17 | MySQL 7 | Version 5 |
| **Charlie** (New Dev) | MacBook | Java 21 | MySQL 7 | Version 9 |

### Problems:
- ❌ Different operating systems
- ❌ Version mismatches
- ❌ Dependency conflicts
- ❌ Deployment failures
- ❌ Heavy documentation required

**Traditional Solution**: Write extensive README files (ineffective)

---

## 💡 What is Docker?

**Docker** = A tool that packages your application + entire environment into a single unit

### The Docker Unit Contains:
- Application code
- Java runtime environment
- Libraries & dependencies
- OS packages
- Configuration files

### Analogy: Building Architecture

| Concept | Analogy |
|---------|---------|
| **Docker Image** | Building blueprint/architecture (template) |
| **Docker Container** | Actual building (running instance) |
| **Multiple Containers** | Multiple buildings from same blueprint |

### Java Comparison:
- **Class** = Docker Image (template)
- **Object** = Docker Container (instance)

---

## 🏗️ Docker Architecture

### Components:

```
┌─────────────────────────────────────────────────────────┐
│                    USER (Bob)                           │
│         ┌──────────────┐    ┌──────────────┐           │
│         │ Docker CLI   │    │ Docker       │           │
│         │ (Commands)   │    │ Desktop (UI) │           │
│         └──────┬───────┘    └──────────────┘           │
│                │ REST API                               │
│         ┌──────▼────────────────────────────────────┐   │
│         │           DOCKER ENGINE                   │   │
│         │  ┌──────────────────────────────────┐    │   │
│         │  │      DOCKER DAEMON               │    │   │
│         │  │  (Brain - executes commands)     │    │   │
│         │  └──────────────────────────────────┘    │   │
│         └──────────────────────────────────────────┘   │
│                    │                                    │
│         ┌──────────▼──────────┐                        │
│         │   DOCKER REGISTRY   │                        │
│         │   (Docker Hub)      │                        │
│         └─────────────────────┘                        │
└─────────────────────────────────────────────────────────┘
```

### Key Terms:

| Component | Description |
|-----------|-------------|
| **Docker CLI** | Command-line interface to interact with Docker |
| **Docker Daemon** | Background service that executes commands (brain) |
| **Docker Engine** | CLI + Daemon together |
| **Docker Desktop** | UI option for local machines (not available in cloud) |
| **Docker Registry** | Storage for images (Docker Hub is the public registry) |

---

## 📥 Docker Installation

### Steps:
1. Go to [docker.com](https://docker.com)
2. Download Docker Desktop for your OS (Windows/Mac/Linux)
3. Install the application
4. Start Docker Desktop (starts daemon automatically)

### Verify Installation:
```bash
# Check version
docker --version

# List running containers (should show none, no error)
docker ps

# If error about daemon not running → start Docker Desktop
```

### Docker Desktop UI:
- **Containers**: Running and stopped containers
- **Images**: Downloaded images
- **Volumes**: Persistent data storage
- **Builds**: Build history

---

## 🖼️ Docker Hub (Registry)

Public repository where you can:
- Pull existing images
- Push your own images
- Search for images

### Example: Pulling and Running Images

```bash
# Search for Java images on Docker Hub
# (Visit hub.docker.com)

# Pull an image
docker pull openjdk:17

# Run Ubuntu directly (pulls if not present)
docker run -it ubuntu bash

# List downloaded images
docker images
```

---

## 📦 Docker Images vs Containers

### Docker Image:
- **Template/Blueprint** for creating containers
- Contains everything needed to run application
- Read-only

### Docker Container:
- **Running instance** of an image
- Isolated environment
- Can be started, stopped, moved, deleted

### Basic Commands:

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Run an image (creates new container)
docker run <image-name>

# Run in detached mode (background)
docker run -d <image-name>

# Run with interactive terminal
docker run -it <image-name> bash

# Stop a container
docker stop <container-id>

# Start an existing container
docker start <container-id>

# Remove all stopped containers
docker container prune

# Remove specific container
docker rm <container-id>
```

### Example: Running Nginx
```bash
# Run Nginx in background
docker run -d nginx

# See running containers
docker ps

# Stop the container
docker stop <container-id>
```

---

## 🐳 Containerizing a Spring Boot Application

### Step 1: Create Dockerfile

```dockerfile
# 1. Base image (with Java)
FROM openjdk:17-jdk-alpine

# 2. Working directory inside container
WORKDIR /app

# 3. Copy jar file (from target folder)
COPY target/*.jar app.jar

# 4. Expose port
EXPOSE 8080

# 5. Startup command
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Dockerfile Instructions Summary:

| Instruction | Purpose | Example |
|-------------|---------|---------|
| **FROM** | Base image | `FROM openjdk:17` |
| **WORKDIR** | Working directory | `WORKDIR /app` |
| **COPY** | Copy files to container | `COPY target/*.jar app.jar` |
| **EXPOSE** | Document exposed port | `EXPOSE 8080` |
| **ENTRYPOINT** | Startup command | `ENTRYPOINT ["java", "-jar", "app.jar"]` |

---

## 🔧 Building and Running Images

### Build Image:
```bash
# Build image with tag
docker build -t my-app:1.0 .

# Verify image created
docker images
```

### Run Container with Port Mapping:
```bash
# Map host port 8080 to container port 8080
docker run -p 8080:8080 my-app:1.0

# Run in detached mode
docker run -d -p 8080:8080 my-app:1.0
```

### Port Mapping Explanation:
```
┌─────────────────┐     ┌─────────────────┐
│   HOST/MacBook  │     │    CONTAINER    │
│                 │     │                 │
│  Port 8080 ─────┼────►│  Port 8080     │
│                 │     │  (Spring Boot)  │
└─────────────────┘     └─────────────────┘

Command: docker run -p 8080:8080 my-app:1.0
                     ↑      ↑
                  Host    Container
                  Port    Port
```

### Run with Environment Variable:
```bash
# Override server port to 9090
docker run -d -p 9090:9090 -e SERVER_PORT=9090 my-app:1.0
```

---

## 📁 Docker Volumes

### Problem: Container data is ephemeral
- Each new container has its own data
- When container is deleted, data is lost
- Different containers cannot share data

### Solution: Volumes (persistent storage)

### Types of Volumes:

#### 1. Named Volume
```bash
# Create volume and mount to container
docker run -d -p 8080:8080 -v my-volume:/app my-app:1.0

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume
```

#### 2. Bind Mount (Host directory)
```bash
# Mount local file/directory into container
docker run -d -p 8080:8080 -v /path/on/host:/app/data my-app:1.0
```

### Volume Comparison:

| Type | Data Location | Use Case |
|------|---------------|----------|
| **Named Volume** | Docker-managed | Shared data between containers |
| **Bind Mount** | Host filesystem | Development, config files |

---

## 🎯 Docker Compose

### Problem:
Long commands with multiple flags:
```bash
docker run -d -p 8080:8080 -e SERVER_PORT=8080 -v my-volume:/app my-app:1.0
```

### Solution: Docker Compose (YAML configuration)

### docker-compose.yml:

```yaml
version: '3.8'

services:
  app:
    image: my-app:1.0
    ports:
      - "8080:8080"
    environment:
      - SERVER_PORT=8080
    volumes:
      - my-volume:/app
    depends_on:
      - mysql

  mysql:
    image: mysql:8
    container_name: mysql-db
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=mydb
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  my-volume:
  mysql-data:
```

### Docker Compose Commands:

```bash
# Start all services (foreground)
docker compose up

# Start in background
docker compose up -d

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v

# View logs
docker compose logs

# Rebuild and start
docker compose up -d --build
```

---

## 🌐 Docker Networking

### Problem:
- Spring Boot container needs to communicate with MySQL container
- Each container has its own network namespace

### Solution: Custom Networks

### How Networking Works:

```
┌─────────────────────────────────────────────────────┐
│              DOCKER NETWORK: app-network            │
│                                                      │
│  ┌──────────────────┐      ┌──────────────────┐    │
│  │  Spring Boot App │      │     MySQL        │    │
│  │  (app)           │      │  (mysql)         │    │
│  │                  │      │                  │    │
│  │  Connects to:    │◄────►│  Port: 3306      │    │
│  │  mysql:3306      │      │                  │    │
│  └──────────────────┘      └──────────────────┘    │
└─────────────────────────────────────────────────────┘
```

### Key Points:
- Containers on same network can communicate using **service names**
- Docker Compose creates a network automatically
- Service name = container name = hostname for communication

### Application Properties (Spring Boot):
```properties
# Use container name as hostname
spring.datasource.url=jdbc:mysql://mysql:3306/mydb
spring.datasource.username=root
spring.datasource.password=root
```

### Network Commands:
```bash
# List networks
docker network ls

# Inspect network
docker network inspect <network-name>

# Create custom network
docker network create my-network

# Connect container to network
docker network connect my-network <container-id>
```

---

## 📊 Complete Flow Summary

### Building and Running Multi-Container App:

```bash
# Step 1: Build application
mvn clean install

# Step 2: Build Docker image
docker build -t my-app:latest .

# Step 3: Run with Docker Compose
docker compose up -d

# Step 4: Verify running containers
docker ps

# Step 5: View logs
docker compose logs -f

# Step 6: Stop everything
docker compose down
```

---

## 🔄 Docker Commands Cheat Sheet

### Images:
```bash
docker images                  # List images
docker build -t name:tag .     # Build image
docker rmi <image-id>          # Remove image
docker pull <image>            # Pull from registry
docker push <image>            # Push to registry
```

### Containers:
```bash
docker ps                      # Running containers
docker ps -a                   # All containers
docker run <image>             # Create and run
docker start <container>       # Start existing
docker stop <container>        # Stop running
docker rm <container>          # Remove container
docker exec -it <container> bash  # Enter container
docker logs <container>        # View logs
docker container prune         # Remove stopped containers
```

### Volumes:
```bash
docker volume ls               # List volumes
docker volume create <name>    # Create volume
docker volume inspect <name>   # View details
docker volume rm <name>        # Remove volume
docker volume prune            # Remove unused volumes
```

### Networks:
```bash
docker network ls              # List networks
docker network create <name>   # Create network
docker network inspect <name>  # View details
docker network connect <net> <container>  # Connect
```

### Docker Compose:
```bash
docker compose up -d           # Start services
docker compose down            # Stop services
docker compose logs -f         # View logs
docker compose ps              # List services
docker compose exec <service> bash  # Enter container
docker compose build           # Rebuild images
```

---

## ✅ Key Takeaways

1. **Docker solves environment mismatch** - "It works on my machine" problem
2. **Image = Template** (like class), **Container = Instance** (like object)
3. **Dockerfile** defines how to build an image
4. **Port mapping** connects host ports to container ports
5. **Volumes** provide persistent storage across container restarts
6. **Docker Compose** simplifies running multi-container applications
7. **Networks** enable container-to-container communication
8. **Always use Docker Desktop** for local development

---

## 🚀 Quick Start Checklist

- [ ] Install Docker Desktop
- [ ] Verify with `docker --version`
- [ ] Create Dockerfile in project root
- [ ] Build image: `docker build -t my-app .`
- [ ] Test run: `docker run -p 8080:8080 my-app`
- [ ] Create docker-compose.yml for multi-service apps
- [ ] Use volumes for persistent data
- [ ] Use networks for inter-container communication

---

## 📝 Sample Project Structure

```
my-app/
├── Dockerfile
├── docker-compose.yml
├── src/
│   └── main/
│       ├── java/
│       └── resources/
│           └── application.properties
├── target/
│   └── *.jar
└── README.md
```

---

## 🎯 Final Notes

- Docker is **essential** for modern development
- **One image, run anywhere** regardless of host OS
- **Docker Compose** is the standard for local development
- **Always tag your images** with versions (avoid `latest` in production)
- **Clean up** unused containers, images, and volumes regularly
