## Docker Scenario-Based Q&A (Concise, Ready-to-Practice)

### 1) Build once, run many: multi-stage, small image
Problem: Your CI builds a Go app. Images are 1.2GB and slow to pull. Make it <100MB and reproducible.

Answer:
```Dockerfile
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
Why: multi-stage removes toolchain; distroless drops shell/package manager for small, safer runtime.

---

### 2) Pin base images and verify provenance
Problem: Security asks you to stop using floating tags like `node:latest` and verify images.

Answer:
- Pin by digest: `node:20.10@sha256:<digest>` ensures identical bytes everywhere.
- Enable content trust (Notary v1) in CI: `DOCKER_CONTENT_TRUST=1`.

Command (explanation included):
```bash
DOCKER_CONTENT_TRUST=1 docker pull node:20.10@sha256:ABC... 
```
- What it does: pulls an image only if signed.
- Flags/env: `DOCKER_CONTENT_TRUST=1` forces signature verification.

---

### 3) Fix "works on my machine" with dev containers
Problem: New devs fail to run the app locally.

Answer: Provide a `docker-compose.yml` dev stack with hot-reload.
```yaml
services:
  api:
    build: .
    command: npm run dev
    volumes:
      - .:/workspace
      - node_modules:/workspace/node_modules
    ports:
      - "3000:3000"
volumes:
  node_modules: {}
```
Why: identical env for everyone; bind-mount for live code; named volume for stable deps.

---

### 4) Container keeps crashing; need logs and quick shell
Problem: App exits instantly in Kubernetes/Compose; you need logs and an ephemeral shell.

Answer (local container):
```bash
docker logs <cid> --tail 200 -f
```
- What it does: streams last 200 lines and follows.
- Flags: `--tail 200` limit output; `-f` follow.

```bash
docker run --rm -it --entrypoint sh <image>
```
- What it does: runs a throwaway shell to inspect the image.
- Flags: `--rm` auto-remove; `-it` interactive TTY; `--entrypoint sh` override entrypoint.

---

### 5) Secrets: remove hardcoded credentials
Problem: Developers baked `DB_PASSWORD` into the image.

Answer (Compose secrets):
```yaml
services:
  api:
    image: my-api:1.0
    secrets:
      - db_password
secrets:
  db_password:
    file: ./secrets/db_password.txt
```
Why: secret mounted as a file at runtime, not in image layers or env history.

---

### 6) Layer caching is broken in CI; builds are slow
Problem: `COPY . .` busts cache on every change.

Answer: copy dependency manifests first.
```Dockerfile
COPY package.json package-lock.json ./
RUN npm ci --only=production
COPY . .
```
Why: dependency layer is cached unless manifests change.

---

### 7) Non-root user requirement
Problem: Security policy forbids root in containers.

Answer:
```Dockerfile
RUN addgroup --system app && adduser --system --ingroup app app
USER app
```
Why: drops privileges; safer defaults for file/FS access.

---

### 8) Reduce CVEs in image scans
Problem: Scanner flags critical CVEs in base images.

Answer:
- Use slim/distroless images.
- Update frequently and rebuild from base.
```bash
docker build --pull -t myapp:secure .
```
- What it does: pulls newest base before building.
- Flags: `--pull` refreshes base image.

---

### 9) Deterministic multi-arch images (x86_64/arm64)
Problem: Need one tag for both architectures.

Answer (Buildx):
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t org/app:1.0 --push .
```
- What it does: builds and pushes a multi-arch manifest.
- Flags: `--platform` target archs; `-t` tag; `--push` upload to registry.

---

### 10) Limit runaway containers
Problem: A bug consumes all memory and crashes the node.

Answer:
```bash
docker run --memory=512m --cpus=1 -d --name capped org/app:1.0
```
- What it does: runs container with resource limits.
- Flags: `--memory=512m` cap RAM; `--cpus=1` cap CPU; `-d` detached; `--name` name.

---

### 11) Persist data safely with volumes
Problem: DB data lost after container removal.

Answer:
```bash
docker volume create dbdata
docker run -v dbdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=secret -d postgres:16
```
- What it does: creates a named volume and mounts it for Postgres.
- Flags: `-v src:dest` mount; `-e` set env; `-d` detached.

---

### 12) Private registry access in CI
Problem: CI cannot pull/push to a private registry.

Answer:
```bash
docker login registry.example.com -u ci-bot -p "$REGISTRY_TOKEN"
docker build -t registry.example.com/team/app:ci-$GIT_SHA .
docker push registry.example.com/team/app:ci-$GIT_SHA
```
- What it does: authenticates, builds, pushes.
- Flags: `-u` username; `-p` password/token; `-t` tag image.

---

### 13) Zero-downtime reload
Problem: You must reload a service without dropping connections.

Answer (Compose rolling style):
```bash
docker compose up -d --no-deps --scale web=2 --build web
```
- What it does: rebuilds and updates `web` with two replicas.
- Flags: `-d` detached; `--no-deps` avoid touching deps; `--scale` set replica count; `--build` build before up.

---

### 14) Debug networking issues inside a container
Problem: App cannot reach a dependency.

Answer:
```bash
docker exec -it <cid> sh -c "apk add --no-cache curl && curl -v http://svc:8080/health"
```
- What it does: installs curl in a throwaway Alpine-based container and tests connectivity.
- Flags: `exec -it` interactive; `-c` run command; `--no-cache` faster install; `-v` verbose curl.

---

### 15) Clean up disk space quickly
Problem: Runner is out of disk due to old images/volumes.

Answer:
```bash
docker system prune -af --volumes
```
- What it does: removes unused images, containers, networks, and volumes.
- Flags: `-a` include dangling and unused; `-f` no prompt; `--volumes` include volumes.

---

### 16) Promote image from staging to prod without rebuild
Problem: You want the same artifact in prod.

Answer:
```bash
docker pull registry/app:staging@sha256:ABC...
docker tag registry/app:staging@sha256:ABC... registry/app:prod
docker push registry/app:prod
```
Why: tag-by-digest guarantees byte-for-byte promotion.

---

### 17) Healthchecks for self-healing
Problem: Orchestrator should restart unhealthy containers.

Answer (Dockerfile):
```Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 CMD curl -fsS http://localhost:8080/health || exit 1
```
Explanation: periodic health probe; non-zero exit marks container unhealthy.

---

### 18) Timezone and locale drift
Problem: Logs show wrong tz/locale.

Answer:
```bash
docker run -e TZ=UTC -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 ...
```
- Flags: `-e` set environment variables.

---

### 19) Prevent config drift between environments
Problem: Different env files cause surprises.

Answer: parameterize builds and lock compose profiles.
```bash
docker build --build-arg APP_ENV=staging -t org/app:staging .
docker compose --profile api up -d
```
- Flags: `--build-arg` pass build-time var; `--profile` enable selected services.

---

### 20) Quick local registry for air‑gapped tests
Problem: Need a registry on localhost.

Answer:
```bash
docker run -d -p 5000:5000 --name registry registry:2
```
- What it does: runs a private registry locally.
- Flags: `-d` detached; `-p host:container` port map; `--name` container name.

---

Use these scenarios to practice fast: read the problem, state the principle, run the command, and explain the flags briefly.


