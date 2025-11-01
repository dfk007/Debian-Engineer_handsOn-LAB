## **COMPREHENSIVE DOCKER SCENARIO-BASED Q&A (BEGINNER TO ADVANCED)**

### **PHASE 1: BASICS & FOUNDATION (Beginner)**

#### **Scenario 1.1: Docker Installation & Verification**

**Problem:** Need to install Docker and verify it works correctly.

**Solution:**
```bash
# 1. Install Docker on Debian
sudo apt update
sudo apt install -y docker.io docker-compose

# 2. Start Docker service
sudo systemctl enable docker
sudo systemctl start docker

# 3. Add user to docker group (optional, for non-sudo usage)
sudo usermod -aG docker $USER
newgrp docker

# 4. Verify installation
docker --version
docker info
```

**Command explanations:**
- `docker --version`: Shows Docker client and server version numbers
- `docker info`: Displays detailed information about Docker daemon configuration, storage driver, logging driver, etc.
- `usermod -aG docker $USER`: Adds current user to docker group so they can run Docker commands without sudo
- `newgrp docker`: Refreshes group membership without logging out

---

#### **Scenario 1.2: Hello World - First Container**

**Problem:** Run your first container to verify Docker works.

**Solution:**
```bash
docker run --rm hello-world
```

**Command explanations:**
- `docker run`: Creates and starts a new container
- `--rm`: Automatically removes the container when it exits (cleanup)

**Verify:**
- You should see a welcome message explaining Docker concepts

---

#### **Scenario 1.3: List Containers and Images**

**Problem:** Check what containers and images are on your system.

**Solution:**
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List images
docker images

# List with more details
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Names}}"
```

**Command explanations:**
- `docker ps`: Shows running containers
- `docker ps -a`: Shows all containers including stopped ones
- `docker images`: Lists all downloaded images
- `--format`: Custom output format (shows ID, Image, Status, Names)

---

#### **Scenario 1.4: Inspect Container/Image Metadata**

**Problem:** Get detailed information about a container or image.

**Solution:**
```bash
# Inspect a running container
docker inspect <container-id-or-name>

# Inspect an image
docker inspect <image-name:tag>

# Get specific field (JSON path)
docker inspect -f '{{.Config.Cmd}}' <container-id>

# Get IP address of container
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-id>
```

**Command explanations:**
- `docker inspect`: Returns detailed JSON metadata about container/image
- `-f '{{...}}'`: Format output using Go template syntax

---

### **PHASE 2: BASIC CONTAINER OPERATIONS (Beginner)**

#### **Scenario 2.1: Run a Web Server Container**

**Problem:** Run an Nginx web server accessible on host port 8080.

**Solution:**
```bash
docker run -d --name demo-nginx -p 8080:80 nginx:alpine
```

**Command explanations:**
- `docker run`: Creates and starts a new container
- `-d`: Runs container in detached mode (background)
- `--name demo-nginx`: Assigns a name to the container
- `-p 8080:80`: Maps host port 8080 to container port 80
- `nginx:alpine`: Uses lightweight Alpine-based Nginx image

**Verify:**
```bash
curl -I http://localhost:8080
# Should return HTTP 200 OK
```

**Cleanup:**
```bash
docker stop demo-nginx
docker rm demo-nginx
```

---

#### **Scenario 2.2: View Container Logs**

**Problem:** Check logs from a running or stopped container.

**Solution:**
```bash
# View logs
docker logs <container-id-or-name>

# Follow logs (like tail -f)
docker logs -f <container-id-or-name>

# View last N lines
docker logs --tail 100 <container-id-or-name>

# View logs with timestamps
docker logs -t <container-id-or-name>

# View logs since timestamp
docker logs --since "2024-01-01T00:00:00" <container-id-or-name>
```

**Command explanations:**
- `docker logs`: Shows logs from container's stdout/stderr
- `-f`: Follow log output (streams logs continuously)
- `--tail N`: Shows last N lines
- `-t`: Shows timestamps
- `--since`: Shows logs since timestamp or duration (e.g., "30m", "1h")

---

#### **Scenario 2.3: Execute Commands in Running Container**

**Problem:** Debug or run commands inside a running container.

**Solution:**
```bash
# Execute single command
docker exec <container-id> ls -la /var/www/html

# Interactive shell (bash)
docker exec -it <container-id> bash

# Interactive shell (sh for Alpine)
docker exec -it <container-id> sh

# Execute with environment variable
docker exec -e DEBUG=true <container-id> env
```

**Command explanations:**
- `docker exec`: Executes command in running container
- `-it`: Allocates interactive TTY (terminal) for interactive commands
- `-e KEY=value`: Sets environment variable for the command

---

#### **Scenario 2.4: Stop, Start, Restart Containers**

**Problem:** Manage container lifecycle.

**Solution:**
```bash
# Stop container gracefully (SIGTERM, then SIGKILL)
docker stop <container-id-or-name>

# Start stopped container
docker start <container-id-or-name>

# Restart container
docker restart <container-id-or-name>

# Kill container immediately (SIGKILL)
docker kill <container-id-or-name>

# Remove stopped container
docker rm <container-id-or-name>

# Force remove running container
docker rm -f <container-id-or-name>
```

**Command explanations:**
- `docker stop`: Stops container gracefully (sends SIGTERM, waits, then SIGKILL)
- `docker start`: Starts a stopped container
- `docker restart`: Stops and starts container
- `docker kill`: Immediately kills container (SIGKILL)
- `docker rm`: Removes container (must be stopped unless using `-f`)

---

### **PHASE 3: IMAGE BUILDING (Beginner to Intermediate)**

#### **Scenario 3.1: Create Simple Dockerfile**

**Problem:** Build a custom image with a simple web page.

**Solution:**
```bash
# Create directory
mkdir -p ~/docker-practice-here/basic-web
cd ~/docker-practice-here/basic-web

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Create index.html
echo "<h1>Hello from Docker!</h1>" > index.html

# Build image
docker build -t my-nginx:1.0 .

# Run container
docker run -d --name my-web -p 8081:80 my-nginx:1.0

# Test
curl http://localhost:8081
```

**Command explanations:**
- `docker build -t name:tag .`: Builds image from Dockerfile in current directory
- `-t name:tag`: Tags image with name and version
- `.`: Build context (current directory)

**Dockerfile explanation:**
- `FROM nginx:alpine`: Base image (lightweight Alpine Linux with Nginx)
- `COPY index.html ...`: Copies file from host to container
- `EXPOSE 80`: Documents that container listens on port 80
- `CMD ["nginx", "-g", "daemon off;"]`: Default command when container starts

---

#### **Scenario 3.2: COPY vs ADD - When to Use What**

**Problem:** Understand difference between COPY and ADD instructions.

**Solution - Use COPY (Recommended):**
```dockerfile
FROM alpine:3.20
WORKDIR /app
COPY app.py .
COPY config.json /etc/
```

**Solution - Use ADD (Special Cases Only):**
```dockerfile
FROM alpine:3.20
WORKDIR /app
# ADD can auto-extract archives and download from URLs
ADD app.tar.gz .
ADD https://example.com/data.json .
```

**Why COPY over ADD:**
- `COPY`: Simple file copy, more predictable, recommended for most cases
- `ADD`: Has extra features (auto-extract tar, download from URL) but less predictable and potential security risks

**Best Practice:** Use COPY unless you specifically need ADD's archive extraction or URL download features.

---

#### **Scenario 3.3: Multi-Stage Build - Optimize Image Size**

**Problem:** Build a Go application but final image is 1.2GB. Need it under 100MB.

**Solution:**
```dockerfile
# Build stage
FROM golang:1.22 AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags "-s -w" -o server

# Runtime stage
FROM gcr.io/distroless/base-debian12
WORKDIR /srv
COPY --from=build /app/server ./server
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/srv/server"]
```

**Build and verify:**
```bash
docker build -t go-app:slim .
docker images | grep go-app
# Should show much smaller size
```

**Why multi-stage:**
- Removes build tools from final image
- Distroless base has no shell/package manager (smaller, more secure)
- Final image only contains the compiled binary

**Command explanations:**
- `FROM ... AS build`: Named build stage
- `COPY --from=build ...`: Copies from previous stage
- `-trimpath -ldflags "-s -w"`: Strips debug info from binary (smaller)
- `USER nonroot:nonroot`: Runs as non-root user (security)

---

#### **Scenario 3.4: Optimize Layer Caching - Dependency Management**

**Problem:** Every code change invalidates entire Docker build cache, making builds slow.

**Solution:**
```dockerfile
FROM node:20-alpine
WORKDIR /app

# Copy dependency files first (changes rarely)
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Copy application code last (changes frequently)
COPY . .
CMD ["node", "server.js"]
```

**Why this works:**
- Dependencies change less frequently than code
- Copying dependency manifests first allows Docker to cache the `npm install` layer
- Only when package.json changes does npm install re-run

**Verify caching:**
```bash
# First build
docker build -t myapp:1.0 .

# Change only source code, rebuild
# Edit app.js, then:
docker build -t myapp:1.0 .
# Should show "CACHED" for npm install step
```

**Command explanations:**
- `npm ci --only=production`: Installs only production dependencies (faster, reproducible)
- Docker uses layer caching: if COPY files didn't change, previous RUN layers are reused

---

### **PHASE 4: SECURITY & BEST PRACTICES (Intermediate)**

#### **Scenario 4.1: Run Container as Non-Root User**

**Problem:** Security policy requires containers to not run as root.

**Solution:**
```dockerfile
FROM alpine:3.20
RUN addgroup -S app && adduser -S app -G app
USER app
WORKDIR /app
COPY app.bin .
CMD ["./app.bin"]
```

**Verify:**
```bash
docker build -t nonroot-app:1.0 .
docker run --rm nonroot-app:1.0 id
# Should show uid=1000(app) gid=1000(app)
```

**Command explanations:**
- `addgroup -S app`: Creates system group "app" (-S = system group)
- `adduser -S app -G app`: Creates system user "app" in group "app"
- `USER app`: Switches to non-root user for subsequent commands

---

#### **Scenario 4.2: Pin Base Images by Digest for Reproducibility**

**Problem:** Security team wants reproducible builds - same base image bytes everywhere.

**Solution:**
```bash
# Get digest of base image
docker pull node:20-alpine
docker images --digests | grep node:20-alpine

# Use digest in Dockerfile
FROM node:20-alpine@sha256:abc123def456...
```

**Enable Content Trust:**
```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Pull with verification
docker pull node:20-alpine@sha256:abc123def456...
```

**Command explanations:**
- `docker images --digests`: Shows SHA256 digest for each image
- `@sha256:...`: Pins image to exact digest (immutable)
- `DOCKER_CONTENT_TRUST=1`: Enables signature verification (requires Notary setup)

---

#### **Scenario 4.3: Reduce CVEs in Base Images**

**Problem:** Security scanner flags critical CVEs in base image.

**Solution:**
```bash
# Use slim/distroless images
FROM gcr.io/distroless/base-debian12

# Or Alpine (smaller attack surface)
FROM alpine:3.20

# Rebuild with latest base (pulls fresh base)
docker build --pull -t myapp:secure .

# Scan image for vulnerabilities
docker scan myapp:secure
```

**Command explanations:**
- `--pull`: Forces Docker to pull latest base image before building
- `docker scan`: Uses Snyk to scan image for vulnerabilities (requires Docker Desktop or Snyk CLI)

---

### **PHASE 5: VOLUMES & DATA PERSISTENCE (Intermediate)**

#### **Scenario 5.1: Create Named Volume for Database**

**Problem:** Database data is lost when container is removed.

**Solution:**
```bash
# Create named volume
docker volume create pgdata

# Run PostgreSQL with volume
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16
```

**Verify persistence:**
```bash
# Create test data
docker exec -it postgres-db psql -U postgres -c "CREATE TABLE test (id INT); INSERT INTO test VALUES (1);"

# Remove container
docker rm -f postgres-db

# Recreate container with same volume
docker run -d --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16

# Data should still be there
docker exec -it postgres-db psql -U postgres -c "SELECT * FROM test;"
```

**Command explanations:**
- `docker volume create pgdata`: Creates named volume that persists independently of containers
- `-v pgdata:/var/lib/postgresql/data`: Mounts named volume to container path
- Data in `/var/lib/postgresql/data` persists even if container is removed

---

#### **Scenario 5.2: Backup and Restore Volume Data**

**Problem:** Need to backup database volume before update.

**Solution:**
```bash
# Backup volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine:3.20 \
  tar czf /backup/pgdata-backup-$(date +%Y%m%d).tar.gz -C /data .

# Restore volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine:3.20 \
  tar xzf /backup/pgdata-backup-YYYYMMDD.tar.gz -C /data
```

**Command explanations:**
- `docker run --rm`: Runs temporary container, removes when done
- `-v pgdata:/data`: Mounts volume to container
- `-v $(pwd):/backup`: Mounts current directory for backup location
- `tar czf`: Creates compressed archive

---

#### **Scenario 5.3: Bind Mount for Development**

**Problem:** Need to edit code on host and see changes in container immediately.

**Solution:**
```bash
docker run -d \
  --name dev-app \
  -v $(pwd)/src:/app/src:ro \
  -v $(pwd)/config:/app/config:rw \
  -p 3000:3000 \
  node:20-alpine \
  sh -c "cd /app && npm start"
```

**Command explanations:**
- `-v /host/path:/container/path`: Bind mount (mounts host directory into container)
- `:ro`: Read-only mount (container can't modify files)
- `:rw`: Read-write mount (default)

---

### **PHASE 6: NETWORKING (Intermediate)**

#### **Scenario 6.1: Create Custom Bridge Network**

**Problem:** Containers need to communicate using service names instead of IPs.

**Solution:**
```bash
# Create custom network
docker network create appnet

# Run containers on same network
docker run -d --name db --network appnet \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

docker run -d --name api --network appnet \
  -p 8082:8080 \
  alpine:3.20 \
  sh -c "apk add --no-cache curl && sleep infinity"

# Test DNS resolution
docker exec -it api ping db
# Should resolve "db" to database container IP
```

**Command explanations:**
- `docker network create appnet`: Creates isolated bridge network
- `--network appnet`: Attaches container to network
- Containers on same network can reach each other by container name (DNS)

---

#### **Scenario 6.2: Expose Ports and Port Mapping**

**Problem:** Container app listens on port 8080, need to access from host on port 3000.

**Solution:**
```bash
# Map host:container ports
docker run -d --name app \
  -p 3000:8080 \
  my-app:1.0

# Map multiple ports
docker run -d --name app \
  -p 3000:8080 \
  -p 3001:8081 \
  my-app:1.0

# Map all ports to random host ports
docker run -d --name app -P my-app:1.0
docker port app
```

**Command explanations:**
- `-p host:container`: Maps host port to container port
- `-p 3000:8080`: Host port 3000 → container port 8080
- `-P`: Maps all EXPOSE ports to random host ports
- `docker port`: Shows port mappings for container

---

#### **Scenario 6.3: Debug Network Connectivity**

**Problem:** App in container cannot reach database.

**Solution:**
```bash
# Check container network
docker network inspect bridge

# Inspect container network config
docker inspect <container-id> | grep -A 20 NetworkSettings

# Test connectivity from container
docker exec -it <container-id> sh -c "apk add --no-cache curl && curl -v http://db:5432"

# Check DNS resolution
docker exec -it <container-id> nslookup db
docker exec -it <container-id> ping db
```

**Command explanations:**
- `docker network inspect bridge`: Shows network details and connected containers
- `curl -v`: Verbose output showing connection details
- `nslookup`: Tests DNS resolution

---

### **PHASE 7: HEALTHCHECKS & MONITORING (Intermediate)**

#### **Scenario 7.1: Add Healthcheck to Dockerfile**

**Problem:** Orchestrator needs to know if container is healthy.

**Solution:**
```dockerfile
FROM nginx:alpine
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost:80 || exit 1
```

**Verify:**
```bash
docker build -t healthy-nginx:1.0 .
docker run -d --name nginx-healthy healthy-nginx:1.0

# Check health status
docker inspect --format='{{.State.Health.Status}}' nginx-healthy
docker inspect --format='{{json .State.Health}}' nginx-healthy | jq .
```

**Command explanations:**
- `HEALTHCHECK`: Defines periodic health check command
- `--interval=30s`: Runs check every 30 seconds
- `--timeout=3s`: Check must complete within 3 seconds
- `--retries=3`: Container marked unhealthy after 3 consecutive failures
- Exit code 0 = healthy, non-zero = unhealthy

---

#### **Scenario 7.2: Monitor Container Resources**

**Problem:** Need to monitor CPU, memory, network usage of containers.

**Solution:**
```bash
# Real-time stats for all containers
docker stats

# Stats for specific container
docker stats <container-id>

# One-time snapshot (no continuous updates)
docker stats --no-stream

# Custom format
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

**Command explanations:**
- `docker stats`: Shows real-time resource usage (CPU%, memory, network I/O, block I/O)
- `--no-stream`: Shows one-time snapshot instead of continuous updates
- `--format`: Custom output format using Go templates

---

### **PHASE 8: DOCKER COMPOSE (Intermediate)**

#### **Scenario 8.1: Multi-Service Application with Compose**

**Problem:** Need to run web app + database + cache together.

**Solution:**
```bash
mkdir -p ~/docker-practice-here/compose-app
cd ~/docker-practice-here/compose-app

cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - db
    restart: unless-stopped
    networks:
      - appnet

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - appnet

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - appnet
    restart: unless-stopped

volumes:
  db_data:
  redis_data:

networks:
  appnet:
    driver: bridge
EOF

# Start all services
docker compose up -d

# View logs
docker compose logs -f

# Check status
docker compose ps

# Stop and remove
docker compose down -v
```

**Command explanations:**
- `docker compose up -d`: Builds/creates and starts services in detached mode
- `docker compose logs -f`: Shows and follows logs from all services
- `docker compose ps`: Shows status of all services
- `docker compose down -v`: Stops and removes containers, networks, and volumes
- `depends_on`: Ensures db starts before web
- `restart: unless-stopped`: Restarts container unless explicitly stopped

---

#### **Scenario 8.2: Rolling Update with Compose**

**Problem:** Update service without downtime.

**Solution:**
```bash
# Update service with rolling strategy
docker compose up -d --no-deps --scale web=2 --build web

# Or update specific service
docker compose pull web
docker compose up -d --no-deps web
```

**Command explanations:**
- `--no-deps`: Don't start dependencies (only update specified service)
- `--scale web=2`: Run 2 replicas of web service
- `--build`: Rebuild image before starting

---

#### **Scenario 8.3: Environment Variables and Secrets in Compose**

**Problem:** Configure services with environment variables and secrets.

**Solution:**
```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  api:
    image: my-api:1.0
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    env_file:
      - .env
    secrets:
      - db_password
      - api_key
    configs:
      - app_config

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true

configs:
  app_config:
    file: ./config/app.conf
EOF

# Create secrets file
echo "s3cr3t-p4ss" > secrets/db_password.txt
chmod 600 secrets/db_password.txt

# Create external secret (for Swarm)
echo "secret-key" | docker secret create api_key -

# Start services
docker compose up -d
```

**Command explanations:**
- `environment`: Sets environment variables directly
- `env_file`: Loads environment variables from file
- `secrets`: Mounts secrets as files in container (Swarm mode) or as environment variables (Compose)
- `configs`: Mounts configuration files (Swarm mode feature)

---

### **PHASE 9: