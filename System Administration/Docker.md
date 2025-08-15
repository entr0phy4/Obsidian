# Docker

## Overview
Docker is a platform that enables developers to package applications into containers—lightweight, portable, and self-sufficient units that include everything needed to run an application: code, runtime, system tools, libraries, and settings.

## Architecture

### Core Components
- **Docker Engine**: The runtime that manages containers
- **Docker Images**: Read-only templates used to create containers
- **Docker Containers**: Running instances of Docker images
- **Docker Registry**: Storage for Docker images (Docker Hub, private registries)
- **Dockerfile**: Text file with instructions to build Docker images

## Installation

### Ubuntu/Debian
```bash
# Update package index
sudo apt update

# Install required packages
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Add user to docker group (logout/login required)
sudo usermod -aG docker $USER
```

### CentOS/RHEL
```bash
# Install required packages
sudo yum install -y yum-utils

# Add Docker repository
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo yum install docker-ce docker-ce-cli containerd.io

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

## Basic Commands

### Container Management
```bash
# Run a container
docker run hello-world
docker run -it ubuntu:20.04 /bin/bash
docker run -d --name webserver -p 80:80 nginx

# List containers
docker ps                    # Running containers
docker ps -a                 # All containers
docker ps -q                 # Only container IDs

# Start/stop containers
docker start container_name
docker stop container_name
docker restart container_name

# Remove containers
docker rm container_name
docker rm -f container_name  # Force remove
docker container prune       # Remove all stopped containers
```

### Image Management
```bash
# List images
docker images
docker image ls

# Pull images
docker pull ubuntu:20.04
docker pull nginx:latest

# Build images
docker build -t my-app:1.0 .
docker build -f Dockerfile.prod -t my-app:prod .

# Remove images
docker rmi image_name:tag
docker image prune          # Remove unused images
docker image prune -a       # Remove all unused images
```

### Information and Logs
```bash
# Container information
docker inspect container_name
docker stats                 # Live resource usage
docker logs container_name
docker logs -f container_name # Follow logs

# System information
docker info
docker version
docker system df            # Disk usage
```

## Dockerfile

### Basic Structure
```dockerfile
# Use official base image
FROM ubuntu:20.04

# Set maintainer
LABEL maintainer="admin@example.com"

# Set environment variables
ENV APP_DIR=/app
ENV NODE_ENV=production

# Set working directory
WORKDIR $APP_DIR

# Update package list and install packages
RUN apt-get update && \
    apt-get install -y \
        curl \
        nodejs \
        npm && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy application files
COPY package*.json ./
COPY . .

# Install dependencies
RUN npm install --production

# Expose port
EXPOSE 3000

# Create non-root user
RUN useradd -m -u 1001 appuser
USER appuser

# Define startup command
CMD ["npm", "start"]
```

### Multi-stage Build
```dockerfile
# Build stage
FROM node:16-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:16-alpine AS production

WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

USER node
EXPOSE 3000
CMD ["npm", "start"]
```

## Docker Compose

### docker-compose.yml Example
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - database
    networks:
      - app-network

  database:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"
    networks:
      - app-network

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

### Compose Commands
```bash
# Start services
docker-compose up
docker-compose up -d        # Detached mode
docker-compose up --scale web=3  # Scale service

# Stop services
docker-compose down
docker-compose stop

# View logs
docker-compose logs
docker-compose logs -f web  # Follow web service logs

# Execute commands
docker-compose exec web bash
docker-compose run web npm test
```

## Networking

### Network Types
```bash
# List networks
docker network ls

# Create custom network
docker network create my-network
docker network create --driver bridge --subnet=192.168.100.0/24 custom-net

# Connect container to network
docker network connect my-network container_name

# Inspect network
docker network inspect my-network
```

### Network Examples
```bash
# Run container on custom network
docker run -d --name web --network my-network nginx

# Container communication
docker run -d --name db --network my-network postgres
docker run -d --name app --network my-network -e DB_HOST=db my-app
```

## Volume Management

### Volume Types
```bash
# Named volumes
docker volume create my-volume
docker volume ls
docker volume inspect my-volume

# Bind mounts
docker run -v /host/path:/container/path nginx

# tmpfs mounts (memory)
docker run --tmpfs /tmp nginx
```

### Volume Examples
```bash
# Persistent database storage
docker run -d \
  --name postgres-db \
  -v postgres_data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=password \
  postgres:13

# Development with bind mounts
docker run -d \
  --name dev-server \
  -v $(pwd):/app \
  -p 3000:3000 \
  node:16 \
  sh -c "cd /app && npm start"
```

## Security Best Practices

### Image Security
```bash
# Use official base images
FROM node:16-alpine

# Don't run as root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Use specific tags, not 'latest'
FROM nginx:1.21-alpine

# Scan images for vulnerabilities
docker scan my-image:latest
```

### Runtime Security
```bash
# Run with read-only filesystem
docker run --read-only --tmpfs /tmp nginx

# Limit resources
docker run -m 512m --cpus="1.5" nginx

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# Use security options
docker run --security-opt=no-new-privileges nginx
```

## Monitoring and Debugging

### Container Inspection
```bash
# Enter running container
docker exec -it container_name bash

# Copy files
docker cp file.txt container_name:/path/
docker cp container_name:/path/file.txt ./

# Monitor resources
docker stats container_name

# Process information
docker top container_name
```

### Troubleshooting
```bash
# Debug container startup
docker run --rm -it --entrypoint=/bin/bash image_name

# Check container exit codes
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.ExitCode}}"

# Inspect layers
docker history image_name
```

## Production Considerations

### Resource Management
```bash
# Set memory limits
docker run -m 1g my-app

# Set CPU limits
docker run --cpus="1.5" my-app

# Set restart policy
docker run --restart=unless-stopped my-app
```

### Health Checks
```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# Command line
docker run --health-cmd="curl -f http://localhost/" nginx
```

### Registry Operations
```bash
# Tag image
docker tag my-app:latest registry.example.com/my-app:latest

# Push to registry
docker push registry.example.com/my-app:latest

# Login to registry
docker login registry.example.com
```

## Common Use Cases

### Development Environment
```bash
# Quick database for development
docker run -d \
  --name dev-postgres \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=dev123 \
  postgres:13
```

### CI/CD Pipeline
```yaml
# .github/workflows/docker.yml
- name: Build and push Docker image
  run: |
    docker build -t ${{ secrets.REGISTRY_URL }}/my-app:${{ github.sha }} .
    docker push ${{ secrets.REGISTRY_URL }}/my-app:${{ github.sha }}
```

### Microservices
```yaml
# docker-compose.yml for microservices
services:
  api-gateway:
    image: nginx:alpine
    ports: ["80:80"]
  
  user-service:
    build: ./user-service
    
  payment-service:
    build: ./payment-service
```

## Related Tools
- [[Docker Compose]] - Multi-container applications
- [[Kubernetes]] - Container orchestration
- [[Podman]] - Alternative container runtime
- [[Jenkins]] - CI/CD with Docker
- [[Portainer]] - Docker management UI

## Related Topics
- [[Containerization]]
- [[Microservices architecture]]
- [[DevOps]]
- [[Container security]]
- [[Orchestration]]
