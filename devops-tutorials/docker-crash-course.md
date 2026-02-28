# Docker Crash Course for Beginners

Master Docker containerization with this hands-on guide. Perfect for beginners starting their DevOps journey.

## What is Docker?

Docker is a platform for developing, shipping, and running applications in containers. Containers are lightweight, standalone packages that include everything needed to run software.

## Why Docker?

- **Consistency**: Runs same way on any machine
- **Isolation**: Apps don't interfere with each other
- **Portability**: Move between environments easily
- **Efficiency**: Lightweight compared to VMs
- **Scalability**: Easy to scale applications

## Installation

### Windows/macOS

1. Download Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop)
2. Install following setup wizard
3. Start Docker Desktop

### Linux

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Verify installation
docker --version
```

## First Docker Container

### Pull and Run

```bash
# Pull official nginx image
docker pull nginx:latest

# Run container
docker run -d -p 8080:80 nginx
```

This starts nginx web server, mapping port 8080 on host to 80 in container. Visit `http://localhost:8080` to see it running!

### Check Running Containers

```bash
# List all running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs
docker logs <container_id>

# Stop container
docker stop <container_id>

# Remove container
docker rm <container_id>
```

## Docker Images

### Common Commands

```bash
# List local images
docker images

# Search Docker Hub
docker search nginx

# Pull specific version
docker pull nginx:1.21

# Tag image
docker tag my-image:latest user/my-image:tag

# Push to registry
docker push user/my-image:latest

# Remove image
docker rmi image_id
```

## Docker Containers

### Create Container from Image

```bash
# Named container with volume mount
docker run -d \
  --name my-web-server \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx:latest
```

### Container Lifecycle

```bash
# Start stopped container
docker start <container_id>

# Restart container
docker restart <container_id>

# Pause/resume
docker pause <container_id>
docker unpause <container_id>

# Delete container
docker rm -f <container_id>
```

## Docker Compose

### What is Docker Compose?

Docker Compose defines multi-container applications in YAML files. Perfect for development environments with multiple services.

### Basic Compose File

`docker-compose.yml`:
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:80"
    volumes:
      - ./src:/usr/share/nginx/html
    depends_on:
      - api

  api:
    image: node:16-alpine
    working_dir: /app
    command: npm start
    volumes:
      - ./api:/app
    environment:
      - NODE_ENV=development

  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Docker Compose Commands

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Rebuild and restart
docker-compose build --no-cache
docker-compose up -d

# Execute command in container
docker-compose exec web bash
docker-compose exec api npm start
```

## Docker Networking

### Create Network

```bash
# Create bridge network (default)
docker network create my-network

# Run container on network
docker run -d --network my-network --name container1 nginx

# Run another container on same network
docker run -d --network my-network --name container2 nginx

# Connect container to network
docker network connect my-network container1

# Inspect network
docker network inspect my-network

# Remove network
docker network rm my-network
```

## Docker Volumes

### Persist Data

```bash
# Named volume
docker run -d -v my-data-volume:/data nginx

# Volume from existing container
docker run -d --volumes-from container1 nginx

# Volume mount from host
docker run -d -v $(pwd)/data:/data nginx
```

### Volume Commands

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Create volume
docker volume create my-volume

# Remove volume
docker volume rm my-volume
```

## Dockerfile Best Practices

### Example Dockerfile

```dockerfile
# Use official Node runtime as base image
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Set environment
ENV NODE_ENV=production

# Run application
CMD ["node", "server.js"]
```

### Multi-Stage Build

```dockerfile
# Build stage
FROM node:16-alpine as builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production
COPY . .

RUN npm run build

# Production stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Docker Security

### Security Best Practices

1. **Use official images**: Start with trusted base images
2. **Regular updates**: Keep images up to date
3. **Least privilege**: Run containers with minimal permissions
4. **Scan images**: Use vulnerability scanners
5. **Avoid secrets in images**: Use environment variables/secrets

```bash
# Scan image for vulnerabilities
docker scan nginx:latest

# Run as non-root user
USER 1000
```

## Troubleshooting

### Common Issues

```bash
# Container won't start
docker logs <container>
docker inspect <container>

# Port already in use
fuser 8080/tcp  # Find process
docker port inspect  # Check binding

# Can't connect to container
docker network inspect
docker port <container>

# Disk space low
docker system prune -a
```

## Real-World Example

### Full Application Stack

```bash
# Project structure
my-app/
├── docker-compose.yml
├── web/
│   ├── Dockerfile
│   └── src/
├── api/
│   ├── Dockerfile
│   └── src/
└── nginx/
    └── Dockerfile
```

`docker-compose.yml`:
```yaml
version: '3.8'

services:
  nginx:
    build: ./nginx
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - web
      - api

  web:
    build: ./web
    environment:
      - API_URL=http://api:3000

  api:
    build: ./api
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: secretpassword
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Getting Started

1. **Install Docker Desktop**
2. **Run first container**: `docker run hello-world`
3. **Build image**: Create Dockerfile, build with `docker build -t myapp .`
4. **Run container**: `docker run -d -p 3000:3000 myapp`
5. **Compose services**: Create docker-compose.yml for multi-service apps

---

*Generated by AI • Updated February 2026*