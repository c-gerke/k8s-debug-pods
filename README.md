# Kubernetes debugging pods

A collection of purpose-built container images and Kubernetes manifests for debugging and troubleshooting applications running in Kubernetes clusters.

## Repository Structure

```
k8s-debug-pods/
├── images/                  # Container image definitions
│   ├── network-debug/       # Network debugging tools
│   │   └── Dockerfile
│   └── postgresql-debug/    # PostgreSQL debugging tools
│       └── Dockerfile
├── pods/                    # Kubernetes pod manifests
│   ├── network-debug.yml
│   └── postgresql-debug.yml
├── bin/                     # Deployment helper scripts
│   ├── deploy-debug-pod
│   └── cleanup-debug-pods
├── .github/
│   └── workflows/           # CI/CD automation
└── renovate.json            # Automated dependency updates
```

## Available Images

### network-debug

Network troubleshooting tools for diagnosing connectivity issues.

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/network-debug:latest`

**Installed Tools:**
- `curl` - HTTP client
- `wget` - File downloader
- `dig` / `nslookup` - DNS lookup utilities
- `ping` - ICMP connectivity testing
- `netstat` / `ss` - Network statistics
- `ip` / `ifconfig` - Network interface configuration
- `telnet` - TCP connection testing
- `netcat` - Network utility

**Usage:**
```bash
kubectl run network-debug --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/network-debug:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/network-debug.yml
kubectl exec -it network-debug-pod -- /bin/bash
```

### postgresql-debug

PostgreSQL database debugging and development tools for database administration and troubleshooting.

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/postgresql-debug:latest`

**Installed Tools:**
- `psql` - PostgreSQL interactive terminal
- `pg_dump` - PostgreSQL database backup utility
- `pg_restore` - PostgreSQL database restoration utility
- `pg_isready` - Check PostgreSQL server availability
- `createdb` - Create a PostgreSQL database
- `dropdb` - Remove a PostgreSQL database
- `curl` - HTTP client
- `wget` - File downloader

**Usage:**
```bash
kubectl run postgresql-debug --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/postgresql-debug:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/postgresql-debug.yml
kubectl exec -it postgresql-debug-pod -- /bin/bash
```

Example connection to a PostgreSQL database:
```bash
# Inside the debug pod
psql -h postgres-service.default.svc.cluster.local -U myuser -d mydb

# Check if PostgreSQL is ready
pg_isready -h postgres-service.default.svc.cluster.local -p 5432

# Dump a database
pg_dump -h postgres-service.default.svc.cluster.local -U myuser mydb > backup.sql

# Restore a database
pg_restore -h postgres-service.default.svc.cluster.local -U myuser -d mydb backup.sql
```

### Deployment Scripts

For easier deployment with intelligent resource allocation, use the provided scripts:

```bash
# Deploy to specific context and namespace
./bin/deploy-debug-pod -c my-context -n my-namespace network-debug

# Deploy and automatically exec into the pod
./bin/deploy-debug-pod --auto network-debug

# Override resources (useful for constrained environments)
./bin/deploy-debug-pod -m 256Mi -e 256Mi network-debug

# List all debug pods in a namespace
./bin/cleanup-debug-pods -n my-namespace

# Clean up all debug pods
./bin/cleanup-debug-pods -n my-namespace --all
```

The deployment script:
- Automatically calculates appropriate resource allocations based on namespace quotas
- Uses pod manifests as templates (maintains volumes, commands, etc.)
- Steps up in 64Mi increments from 128Mi default
- Leaves 20% buffer for other pods in the namespace
- Caps at 1Gi maximum regardless of quota
- Supports manual resource overrides
- Safe by default: Won't recreate existing pods (use --force to recreate)

See [bin/README.md](bin/README.md) for detailed usage and examples.

## Adding New Debug Images

1. Create a new directory under `images/` with a descriptive name:
   ```bash
   mkdir -p images/my-new-debugger
   ```

2. Add a `Dockerfile` in that directory:
   ```bash
   touch images/my-new-debugger/Dockerfile
   ```

3. Build your Dockerfile following these guidelines:
   - Use `debian:bookworm-slim` as base for consistency
   - Install only necessary tools
   - Clean up apt cache to minimize image size
   - Set appropriate CMD or ENTRYPOINT

4. Commit and push the Dockerfile:
   ```bash
   git add images/my-new-debugger/Dockerfile
   git commit -m "Add my-new-debugger image"
   git push
   ```

5. GitHub Actions will automatically build and push the image to:
   ```
   ghcr.io/c-gerke/k8s-debug-pods/my-new-debugger:latest
   ```

## CI/CD Pipeline

### Automated Image Builds

When a Dockerfile is modified in the `images/` directory:
1. GitHub Actions detects the change
2. Builds the affected image(s) using Docker Buildx
3. Tags with multiple formats (latest, branch name, full git commit hash)
4. Pushes to GitHub Container Registry (ghcr.io)
5. Builds for linux/amd64 platform

### Dependency Updates

Renovate automatically:
- Monitors base images in all Dockerfiles
- Creates PRs for updates
- Auto-merges minor and patch version updates
- Requires manual approval for major version updates

## Requirements

### For Using Images
- Kubernetes cluster with appropriate RBAC permissions
- Access to pull from GitHub Container Registry (public by default)

### For Deployment Scripts
- `kubectl` configured with cluster access
- `yq` YAML processor (install with `brew install yq` on macOS)

## Local Development

To build and test images locally:

```bash
cd images/network-debug
docker build -t network-debug:local .
docker run --rm -it network-debug:local
```

## Contributing

When adding new images:
- Keep images focused on specific debugging tasks
- Document installed tools in this README
- Follow Debian bookworm-slim base image convention
- Optimize for size by cleaning up package manager caches
