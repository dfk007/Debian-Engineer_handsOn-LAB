## **COMPREHENSIVE DOCKER SCENARIO-BASED Q&A GUIDE FOR TURING TEST**

### **PHASE 1: FOUNDATION & BASICS**

#### **MODULE 1.1: Docker Installation & Verification**

**Exercise 1.1.1: Verify Docker Installation**
```bash
# 1. Check Docker version
docker version
```
- **What it does**: Shows Docker client and server versions, API version, and build info.
- **Flags**: None (default command).

```bash
# 2. Check Docker daemon info
docker info
```
- **What it does**: Displays detailed system-wide Docker information including storage driver, network settings, and container stats.

```bash
# 3. Test Docker with hello-world
docker run --rm hello-world
```
- **What it does**: Pulls and runs the hello-world image to verify Docker is working.
- **Flags**: `--rm` automatically removes the container after it exits.

```bash
# 4. List running containers
docker ps
```

```bash
# 5. List all containers (including stopped)
docker ps -a
```

```bash
# 6. List Docker images
docker images
```

---

#### **MODULE 1.2: Basic Container Operations**

**Exercise 1.2.1: Run, Stop, Remove Containers**
```bash
# 1. Run a container in detached mode
docker run -d --name my-nginx -p 8080:80 nginx:alpine
```
- **What it does**: Runs Nginx in the background, maps host port 8080 to container port 80.
- **Flags**: `-d` runs container in detached mode (background); `--name` assigns a name; `-p host:container` maps ports.

```bash
# 2. Check container logs
docker logs my-nginx
```

```bash
# 3. Follow logs in real-time
docker logs -f my-nginx
```
- **Flags**: `-f` follows log output (like `tail -f`).

```bash
# 4. Stop a running container
docker stop my-nginx
```

```bash
# 5. Start a stopped container
docker start my-nginx
```

```bash
# 6. Restart a container
docker restart my-nginx
```

```bash
# 7. Remove a stopped container
docker rm my-nginx
```

```bash
# 8. Force remove a running container
docker rm -f my-nginx
```
- **Flags**: `-f` forces removal of running container (sends SIGKILL).

---

#### **MODULE 1.3: Inspect and Debug Containers**

**Exercise 1.3.1: Inspect Container Metadata**
```bash
# 1. Inspect container details
docker inspect my-nginx
```
- **What it does**: Returns JSON metadata about container configuration, network, volumes, etc.

```bash
# 2. Inspect specific field
docker inspect --format='{{.NetworkSettings.IPAddress}}' my-nginx
```
- **Flags**: `--format` uses Go template to extract specific fields.

```bash
# 3. View container processes
docker top my-nginx
```
- **What it does**: Shows running processes inside the container.

```bash
# 4. View container resource usage
docker stats my-nginx
```
- **What it does**: Displays live CPU, memory, network, and disk I/O stats.

```bash
# 5. View one-shot stats
docker stats --no-stream my-nginx
```
- **Flags**: `--no-stream` shows stats once instead of continuous stream.

```bash
# 6. Execute command in running container
docker exec -it my-nginx sh
```
- **Flags**: `-it` allocates interactive TTY; `exec` runs command in running container.

---

### **PHASE 2: IMAGE BUILDING & DOCKERFILES**

#### **MODULE 2.1: Basic Dockerfile**

**Exercise 2.1.1: Create Simple Dockerfile**
```bash
# Create practice directory
mkdir -p ~/docker-practice/app1
cd ~/docker-practice/app1

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Create index.html
echo "<h1>Hello from Docker!</h1>" > index.html
```

**Exercise 2.1.2: Build and Run Image**
```bash
# 1. Build image
docker build -t my-app:1.0 .
```
- **What it does**: Builds Docker image from Dockerfile in current directory.
- **Flags**: `-t` tags the image with name:tag; `.` is build context (current directory).

```bash
# 2. Run the image
docker run -d --name my-app -p 8081:80 my-app:1.0
```

```bash
# 3. Test the application
curl http://localhost:8081
```

```bash
# 4. Check image layers
docker history my-app:1.0
```
- **What it does**: Shows image build history with layer sizes and commands.

---

#### **MODULE 2.2: COPY vs ADD**

**Exercise 2.2.1: Understand COPY and ADD**
```bash
# Create example demonstrating COPY
cat > Dockerfile.copy << 'EOF'
FROM alpine:3.20
WORKDIR /app
COPY file.txt .
CMD ["cat", "file.txt"]
EOF
```
- **Why COPY over ADD**: COPY is preferred for deterministic builds; ADD can extract archives and download from URLs (generally avoid URL downloads in Dockerfiles).

```bash
# ADD example (use sparingly)
cat > Dockerfile.add << 'EOF'
FROM alpine:3.20
WORKDIR /app
ADD archive.tar.gz .
CMD ["ls", "-la"]
EOF
```

---

#### **MODULE 2.3: Multi-Stage Builds**

**Exercise 2.3.1: Build Small Production Image**
```bash
# Create multi-stage Dockerfile
cat > Dockerfile.multistage << 'EOF'
FROM golang:1.22 AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags "-s -w" -o server

FROM gcr.io/distroless/base-debian12
WORKDIR /srv
COPY --from=build /app/server ./server
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/srv/server"]
EOF
```
- **Why**: Multi-stage removes build tools from final image; distroless drops shell/package manager for smaller, safer runtime.
- **Flags**: `--from=build` copies artifacts from named stage.

```bash
# Build the image
docker build -f Dockerfile.multistage -t go-app:distroless .
```

```bash
# Check image size
docker images go-app:distroless
```

---

#### **MODULE 2.4: Layer Caching Optimization**

**Exercise 2.4.1: Optimize Build Cache**
```bash
# BAD: Cache busting Dockerfile
cat > Dockerfile.bad << 'EOF'
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]
EOF
```
- **Problem**: `COPY . .` busts cache on every code change, forcing npm install to rerun.

```bash
# GOOD: Cache-friendly Dockerfile
cat > Dockerfile.good << 'EOF'
FROM node:20-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]
EOF
```
- **Why**: Copy dependency manifests first; dependency layer caches unless manifests change.
- **Flags**: `--only=production` installs production dependencies only.

---

#### **MODULE 2.5: Non-Root User**

**Exercise 2.5.1: Run Container as Non-Root**
```bash
# Create Dockerfile with non-root user
cat > Dockerfile.nonroot << 'EOF'
FROM alpine:3.20
RUN addgroup -S app && adduser -S app -G app
USER app
WORKDIR /app
CMD ["sh", "-c", "id && whoami"]
EOF
```
- **Why**: Dropping privileges reduces security risk.

```bash
# Build and run
docker build -f Dockerfile.nonroot -t nonroot:1.0 .
docker run --rm nonroot:1.0
```

---

### **PHASE 3: CONTAINER CONFIGURATION**

#### **MODULE 3.1: Resource Limits**

**Exercise 3.1.1: Set Memory and CPU Limits**
```bash
# Run container with resource limits
docker run -d --name limited \
  --memory=512m \
  --cpus=1 \
  alpine:3.20 sh -c "yes > /dev/null"
```
- **What it does**: Runs container with memory and CPU constraints.
- **Flags**: `--memory=512m` caps RAM at 512MB; `--cpus=1` limits to 1 CPU core.

```bash
# Check resource usage
docker stats limited
```

```bash
# Clean up
docker rm -f limited
```

---

#### **MODULE 3.2: Environment Variables**

**Exercise 3.2.1: Pass Environment Variables**
```bash
# Run with environment variables
docker run -d --name env-test \
  -e DATABASE_URL=postgres://localhost/db \
  -e NODE_ENV=production \
  alpine:3.20 sh -c "env | grep -E 'DATABASE|NODE'"
```
- **Flags**: `-e` sets environment variable.

```bash
# Check environment in container
docker exec env-test env

# Clean up
docker rm -f env-test
```

---

#### **MODULE 3.3: Restart Policies**

**Exercise 3.3.1: Configure Auto-Restart**
```bash
# Run container with restart policy
docker run -d --name auto-restart \
  --restart unless-stopped \
  alpine:3.20 sh -c "sleep 10 && exit 1"
```
- **Flags**: `--restart unless-stopped` restarts container always unless explicitly stopped.
- **Other options**: `no`, `on-failure`, `always`, `unless-stopped`.

```bash
# Wait and check if container restarted
sleep 15
docker ps -a | grep auto-restart

# Clean up
docker rm -f auto-restart
```

---

#### **MODULE 3.4: Healthchecks**

**Exercise 3.4.1: Configure Healthcheck**
```bash
# Create Dockerfile with healthcheck
cat > Dockerfile.healthcheck << 'EOF'
FROM nginx:alpine
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:80 || exit 1
EOF
```
- **What it does**: Defines periodic health probe; non-zero exit marks container unhealthy.
- **Flags**: `--interval` time between checks; `--timeout` max time per check; `--retries` failures before unhealthy.

```bash
# Build and run
docker build -f Dockerfile.healthcheck -t hc-nginx:1.0 .
docker run -d --name hc-test hc-nginx:1.0

# Check health status
docker inspect --format='{{json .State.Health}}' hc-test | jq .

# Clean up
docker rm -f hc-test
```

---

### **PHASE 4: VOLUMES & DATA PERSISTENCE**

#### **MODULE 4.1: Named Volumes**

**Exercise 4.1.1: Create and Use Named Volume**
```bash
# Create named volume
docker volume create dbdata
```
- **What it does**: Creates a named volume for data persistence.

```bash
# Run PostgreSQL with named volume
docker run -d --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -v dbdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16
```
- **Flags**: `-v src:dest` mounts volume; `-e` sets environment variable.

```bash
# Verify volume is mounted
docker inspect postgres-db | grep -A 10 Mounts

# List volumes
docker volume ls
```

---

#### **MODULE 4.2: Bind Mounts**

**Exercise 4.2.1: Mount Host Directory**
```bash
# Create host directory
mkdir -p ~/docker-practice/html

# Create test file
echo "<h1>Bind Mount Test</h1>" > ~/docker-practice/html/index.html

# Run container with bind mount
docker run -d --name bind-test \
  -v ~/docker-practice/html:/usr/share/nginx/html:ro \
  -p 8082:80 \
  nginx:alpine
```
- **Flags**: `-v host:container:ro` mounts host directory read-only; `:ro` makes mount read-only.

```bash
# Test
curl http://localhost:8082

# Modify host file and verify
echo "<h1>Updated!</h1>" > ~/docker-practice/html/index.html
curl http://localhost:8082

# Clean up
docker rm -f bind-test
```

---

#### **MODULE 4.3: Backup and Restore**

**Exercise 4.3.1: Backup Database Volume**
```bash
# Create backup
docker exec -t postgres-db pg_dumpall -U postgres > backup.sql
```
- **Flags**: `-t` allocates pseudo-TTY for clean output.

```bash
# Stop and remove container (data persists in volume)
docker stop postgres-db
docker rm postgres-db

# Create new container with same volume
docker run -d --name postgres-restored \
  -e POSTGRES_PASSWORD=secret \
  -v dbdata:/var/lib/postgresql/data \
  -p 5433:5432 \
  postgres:16

# Restore backup
cat backup.sql | docker exec -i postgres-restored psql -U postgres

# Clean up
docker rm -f postgres-restored
docker volume rm dbdata
```

---

### **PHASE 5: NETWORKING**

#### **MODULE 5.1: Default Networks**

**Exercise 5.1.1: Understand Docker Networks**
```bash
# List networks
docker network ls
```

```bash
# Inspect default bridge network
docker network inspect bridge
```

```bash
# Check container network settings
docker inspect my-nginx | grep -A 20 NetworkSettings
```

---

#### **MODULE 5.2: Custom Networks**

**Exercise 5.2.1: Create Custom Bridge Network**
```bash
# Create custom network
docker network create appnet
```
- **What it does**: Creates a new bridge network for container communication.

```bash
# Run containers on custom network
docker run -d --name db --network appnet \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

docker run -d --name api --network appnet \
  -p 8083:8080 \
  alpine:3.20 sh -c "apk add --no-cache curl && sleep infinity"
```
- **Flags**: `--network` attaches container to network.

```bash
# Test DNS-based service discovery
docker exec api sh -c "apk add --no-cache drill && drill db"
```
- **What it does**: Resolves container name to IP via Docker's embedded DNS.

```bash
# Clean up
docker rm -f db api
docker network rm appnet
```

---

#### **MODULE 5.3: Network Troubleshooting**

**Exercise 5.3.1: Debug Network Issues**
```bash
# Install network tools in container
docker exec -it api sh -c "apk add --no-cache curl && curl -v http://db:5432"
```
- **Flags**: `-it` interactive TTY; `-v` verbose curl output.

```bash
# Check container network namespace
docker exec api ip addr show

# Check routing
docker exec api ip route show
```

---

### **PHASE 6: DOCKER COMPOSE**

#### **MODULE 6.1: Basic Compose**

**Exercise 6.1.1: Create Multi-Service Application**
```bash
# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8084:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  db_data:
EOF

# Create html directory
mkdir -p html
echo "<h1>Docker Compose Works!</h1>" > html/index.html
```

**Exercise 6.1.2: Manage Compose Services**
```bash
# Start services
docker compose up -d
```
- **Flags**: `-d` runs in detached mode.

```bash
# View running services
docker compose ps
```

```bash
# View logs
docker compose logs -f
```
- **Flags**: `-f` follows log output.

```bash
# Stop services
docker compose stop

# Start services
docker compose start

# Remove services and volumes
docker compose down -v
```
- **Flags**: `-v` removes volumes as well.

---

#### **MODULE 6.2: Compose with Build**

**Exercise 6.2.1: Build Images in Compose**
```bash
# Create docker-compose.build.yml
cat > docker-compose.build.yml << 'EOF'
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8085:80"
    restart: unless-stopped
EOF

# Build and start
docker compose -f docker-compose.build.yml up -d --build
```
- **Flags**: `--build` forces rebuild of images.

---

#### **MODULE 6.3: Compose Secrets**

**Exercise 6.3.1: Use Secrets in Compose**
```bash
# Create docker-compose.secrets.yml
cat > docker-compose.secrets.yml << 'EOF'
version: '3.8'

services:
  api:
    image: my-api:1.0
    secrets:
      - db_password
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
EOF

# Create secrets directory and file
mkdir -p secrets
echo "supersecret123" > secrets/db_password.txt
chmod 600 secrets/db_password.txt
```
- **Why**: Secrets are mounted as files at runtime, not in image layers or environment history.

```bash
# Run with secrets
docker compose -f docker-compose.secrets.yml up -d
```

---

### **PHASE 7: ADVANCED TOPICS**

#### **MODULE 7.1: Content Trust & Image Provenance**

**Exercise 7.1.1: Enable Content Trust**
```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1
```
- **What it does**: Enables Docker Content Trust for image signature verification.

```bash
# Pull image with content trust
docker pull nginx:alpine
```
- **Note**: Pulls only if image is signed.

```bash
# Pull by digest for reproducibility
docker pull nginx:alpine@sha256:$(docker manifest inspect nginx:alpine | jq -r '.config.digest')
```
- **Why**: Digest ensures identical bytes everywhere.

---

#### **MODULE 7.2: Multi-Architecture Builds**

**Exercise 7.2.1: Build Multi-Arch Images**
```bash
# Create buildx builder
docker buildx create --name multiarch --use
```

```bash
# Build for multiple architectures
docker buildx build --platform linux/amd64,linux/arm64 \
  -t myorg/app:1.0 \
  --push .
```
- **What it does**: Builds and pushes multi-arch manifest.
- **Flags**: `--platform` specifies target architectures; `--push` uploads to registry.

```bash
# Inspect manifest
docker manifest inspect myorg/app:1.0
```

---

#### **MODULE 7.3: Private Registry**

**Exercise 7.3.1: Use Private Registry**
```bash
# Login to private registry
docker login registry.example.com -u ci-bot -p "$REGISTRY_TOKEN"
```
- **Flags**: `-u` username; `-p` password/token.

```bash
# Tag and push image
docker tag my-app:1.0 registry.example.com/team/app:ci-1
docker push registry.example.com/team/app:ci-1
```

```bash
# Pull from private registry
docker pull registry.example.com/team/app:ci-1
```

---

#### **MODULE 7.4: Local Registry for Testing**

**Exercise 7.4.1: Run Local Registry**
```bash
# Run local registry
docker run -d -p 5000:5000 --name registry registry:2
```
- **Flags**: `-d` detached; `-p` port mapping; `--name` container name.

```bash
# Tag and push to local registry
docker tag my-app:1.0 localhost:5000/my-app:1.0
docker push localhost:5000/my-app:1.0

# Pull from local registry
docker pull localhost:5000/my-app:1.0
```

---

#### **MODULE 7.5: Image Promotion**

**Exercise 7.5.1: Promote Image Without Rebuild**
```bash
# Pull staging image by digest
docker pull registry/app:staging@sha256:ABC...

# Tag for production
docker tag registry/app:staging@sha256:ABC... registry/app:prod

# Push production tag
docker push registry/app:prod
```
- **Why**: Tag-by-digest guarantees byte-for-byte promotion.

---

### **PHASE 8: DOCKER SWARM**

#### **MODULE 8.1: Swarm Basics**

**Exercise 8.1.1: Initialize Swarm**
```bash
# Initialize swarm (single node for practice)
docker swarm init
```
- **What it does**: Turns current Docker host into Swarm manager.

```bash
# Check swarm status
docker info | grep -i swarm
```

```bash
# List nodes
docker node ls
```

---

#### **MODULE 8.2: Swarm Services**

**Exercise 8.2.1: Deploy Service**
```bash
# Create service with replicas
docker service create --name web \
  --replicas 3 \
  -p 8086:80 \
  nginx:alpine
```
- **Flags**: `--replicas` sets desired number of tasks; `-p` publishes port.

```bash
# List services
docker service ls
```

```bash
# List service tasks
docker service ps web
```

```bash
# Scale service
docker service scale web=5
```

---

#### **MODULE 8.3: Rolling Updates**

**Exercise 8.3.1: Update Service with Rolling Strategy**
```bash
# Update service with rolling update
docker service update --image nginx:1.25-alpine \
  --update-parallelism 1 \
  --update-delay 10s \
  web
```
- **Flags**: `--update-parallelism` controls how many tasks update simultaneously; `--update-delay` sets delay between updates.

```bash
# Monitor update progress
docker service ps web
```

---

#### **MODULE 8.4: Swarm Secrets**

**Exercise 8.4.1: Create and Use Secrets**
```bash
# Create secret
echo "supersecret" | docker secret create api_key -
```
- **Flags**: `-` reads secret from stdin.

```bash
# Create service with secret
docker service create --name app \
  --secret api_key \
  alpine:3.20 sh -c "cat /run/secrets/api_key && sleep 3600"
```
- **What it does**: Mounts secret at `/run/secrets/api_key`.

```bash
# View secret in service
docker service logs app

# Remove service
docker service rm app

# Remove secret
docker secret rm api_key
```

---

#### **MODULE 8.5: Swarm Configs**

**Exercise 8.5.1: Create and Use Configs**
```bash
# Create config
echo "app_config=true" | docker config create app_cfg -

# Create service with config
docker service create --name app \
  --config app_cfg \
  alpine:3.20 sh -c "cat /app_cfg && sleep 3600"
```

```bash
# Update config (creates new version)
echo "app_config=false" | docker config create app_cfg_v2 -

# Update service to use new config
docker service update --config-rm app_cfg --config-add app_cfg_v2 app
```

---

#### **MODULE 8.6: Overlay Networks**

**Exercise 8.6.1: Create Overlay Network**
```bash
# Create overlay network
docker network create -d overlay appnet
```
- **Flags**: `-d overlay` creates overlay network for multi-host.

```bash
# Deploy services on overlay network
docker service create --name db --network appnet \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

docker service create --name api --network appnet \
  alpine:3.20 sleep 3600
```

```bash
# Test service discovery
docker service logs api
```

---

### **PHASE 9: SECURITY & BEST PRACTICES**

#### **MODULE 9.1: Image Security**

**Exercise 9.1.1: Use Minimal Base Images**
```bash
# Use slim/distroless images
docker build --pull -t myapp:secure .
```
- **Flags**: `--pull` ensures latest base image before building.

```bash
# Scan image for vulnerabilities (if scanner available)
# docker scan myapp:secure
```

---

#### **MODULE 9.2: Least Privilege**

**Exercise 9.2.1: Run Non-Root Containers**
```bash
# Dockerfile already includes USER nonroot:nonroot
# Verify in running container
docker run --rm nonroot:1.0 id
```

---

#### **MODULE 9.3: Resource Constraints**

**Exercise 9.3.1: Limit Container Resources**
```bash
# Set comprehensive resource limits
docker run -d --name constrained \
  --memory=512m \
  --cpus=1 \
  --pids-limit=100 \
  alpine:3.20 sleep 3600
```
- **Flags**: `--pids-limit` limits number of processes.

```bash
# Monitor resource usage
docker stats constrained
```

---

### **PHASE 10: TROUBLESHOOTING & DEBUGGING**

#### **MODULE 10.1: Container Logs**

**Exercise 10.1.1: Debug Using Logs**
```bash
# View last 50 lines
docker logs --tail 50 <container>
```
- **Flags**: `--tail` shows last N lines.

```bash
# Follow logs
docker logs -f <container>
```

```bash
# View logs since timestamp
docker logs --since 2024-01-01T00:00:00 <container>
```

---

#### **MODULE 10.2: Container Inspection**

**Exercise 10.2.1: Inspect Container State**
```bash
# Inspect container configuration
docker inspect <container>

# Inspect specific field
docker inspect --format='{{.State.Status}}' <container>
```

```bash
# View container processes
docker top <container>
```

```bash
# Check container events
docker events
```

---

#### **MODULE 10.3: Network Debugging**

**Exercise 10.3.1: Troubleshoot Network Issues**
```bash
# Test connectivity from container
docker exec <container> sh -c "apk add --no-cache curl && curl -v http://service:port"

# Check DNS resolution
docker exec <container> nslookup service

# Inspect network
docker network inspect <network>
```

---

#### **MODULE 10.4: Cleanup**

**Exercise 10.4.1: System Cleanup**
```bash
# Remove unused resources
docker system prune -af --volumes
```
- **What it does**: Removes unused containers, images, networks, and volumes.
- **Flags**: `-a` includes dangling and unused; `-f` no prompt; `--volumes` includes volumes.

```bash
# Prune specific resource types
docker container prune -f
docker image prune -af
docker volume prune -f
docker network prune -f
```

---

### **PRACTICE SCENARIOS**

#### **Scenario 1: Multi-Stage Build for Production**
**Problem**: Go application produces 1.2GB images; need <100MB production image.

**Solution**:
```dockerfile
FROM golang:1.22 AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags "-s -w" -o server

FROM gcr.io/distroless/base-debian12
WORKDIR /srv
COPY --from=build /app/server ./server
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/srv/server"]
```

---

#### **Scenario 2: Fix Broken Cache in CI**
**Problem**: `COPY . .` busts cache on every change, making builds slow.

**Solution**:
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]
```

---

#### **Scenario 3: Zero-Downtime Update**
**Problem**: Update service without dropping connections.

**Solution (Compose)**:
```bash
docker compose up -d --no-deps --scale web=2 --build web
```

**Solution (Swarm)**:
```bash
docker service update --image new-version --update-parallelism 1 --update-delay 10s web
```

---

#### **Scenario 4: Debug Crashing Container**
**Problem**: Container exits immediately; need logs and shell access.

**Solution**:
```bash
# View logs
docker logs <cid> --tail 200 -f

# Run ephemeral shell
docker run --rm -it --entrypoint sh <image>
```

---

#### **Scenario 5: Secure Secret Management**
**Problem**: Developers baked `DB_PASSWORD` into image.

**Solution (Compose)**:
```yaml
services:
  api:
    secrets:
      - db_password
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**Solution (Swarm)**:
```bash
echo "password" | docker secret create db_password -
docker service create --secret db_password my-service
```

---

### **DAILY PRACTICE SCHEDULE**

**Day 1-2: Basics & Containers**
- Module 1.1, 1.2, 1.3 (Installation, Basic Operations, Inspection)

**Day 3-4: Images & Dockerfiles**
- Module 2.1, 2.2, 2.3, 2.4, 2.5 (Dockerfiles, Multi-stage, Caching)

**Day 5-6: Configuration & Volumes**
- Module 3.1, 3.2, 3.3, 3.4, 4.1, 4.2, 4.3 (Resources, Restart, Health, Volumes)

**Day 7-8: Networking & Compose**
- Module 5.1, 5.2, 5.3, 6.1, 6.2, 6.3 (Networking, Compose)

**Day 9-10: Advanced Topics & Swarm**
- Module 7.1-7.5, 8.1-8.6 (Content Trust, Multi-arch, Swarm)

**Day 11-12: Security & Troubleshooting**
- Module 9.1-9.3, 10.1-10.4 (Security, Debugging)

**Day 13-14: Practice Scenarios & Review**
- Work through all practice scenarios
- Review weak areas

---

### **TEST DAY CHECKLIST**

Before the test:
- [ ] Docker installed and verified (`docker version`, `docker info`)
- [ ] Practice directory created and organized
- [ ] Basic commands memorized (run, build, exec, logs, inspect)
- [ ] Dockerfile syntax understood
- [ ] Docker Compose basics practiced
- [ ] Swarm basics understood (if applicable)
- [ ] Troubleshooting commands ready

During the test:
- [ ] Read questions carefully
- [ ] Show your thought process
- [ ] Use proper commands with correct flags
- [ ] Handle edge cases (non-root, resource limits, etc.)
- [ ] Document your work
- [ ] Verify solutions work

---

Use these scenarios to practice: read the problem, state the principle, run the command, and explain the flags briefly.

