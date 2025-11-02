# Kubernetes debugging pods

A collection of purpose-built container images and Kubernetes manifests for debugging and troubleshooting applications running in Kubernetes clusters.

## Repository Structure

```
k8s-debug-pods/
├── images/                     # Container image definitions
│   ├── mysql/                  # MySQL client tools
│   │   ├── 8.0/
│   │   │   └── Dockerfile
│   │   └── 8.4/
│   │       └── Dockerfile
│   ├── network/                # Network debugging tools
│   │   └── debug/
│   │       └── Dockerfile
│   ├── postgresql/             # PostgreSQL client tools
│   │   ├── 13/
│   │   │   └── Dockerfile
│   │   ├── 14/
│   │   │   └── Dockerfile
│   │   └── 15/
│   │       └── Dockerfile
│   └── ruby/                   # Ruby development tools
│       ├── 3.3/
│       │   └── Dockerfile
│       └── 3.4/
│           └── Dockerfile
├── pods/                       # Kubernetes pod manifests
│   ├── mysql/
│   │   ├── 8.0.yml
│   │   └── 8.4.yml
│   ├── network/
│   │   └── debug.yml
│   ├── postgresql/
│   │   ├── 13.yml
│   │   ├── 14.yml
│   │   └── 15.yml
│   └── ruby/
│       ├── 3.3.yml
│       └── 3.4.yml
├── bin/                        # Deployment helper scripts
│   ├── deploy-debug-pod
│   └── cleanup-debug-pods
├── .github/
│   └── workflows/              # CI/CD automation
└── renovate.json               # Automated dependency updates
```

## Available Images

### Network Debug

Network troubleshooting tools for diagnosing connectivity issues.

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/network-debug:latest`

**Source:** `images/network/debug/Dockerfile`

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
kubectl apply -f pods/network/debug.yml
kubectl exec -it network-debug-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto network/debug
```

### MySQL Debug

MySQL database client tools for debugging and troubleshooting MySQL databases. These images use the official MySQL client tools matching specific MySQL server versions.

**Note:** These images are based on official MySQL Docker images and provide the exact MySQL client version specified.

#### MySQL 8.0

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/mysql-8.0:latest`

**Source:** `images/mysql/8.0/Dockerfile`

**Client:** MySQL 8.0.43

**Installed Tools:**
- `mysql` - MySQL client (8.0.43)
- `mysqldump` - Database backup utility
- `mysqladmin` - Server administration utility
- `curl` - HTTP client
- `wget` - File downloader

**Usage:**
```bash
kubectl run mysql-8.0 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/mysql-8.0:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/mysql/8.0.yml
kubectl exec -it mysql-debug-8.0-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto mysql/8.0
```

Example connection to a MySQL database:
```bash
# Inside the debug pod
mysql -h mysql-service.default.svc.cluster.local -u root -p

# Dump a database
mysqldump -h mysql-service.default.svc.cluster.local -u root -p mydb > backup.sql
```

#### MySQL 8.4

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/mysql-8.4:latest`

**Source:** `images/mysql/8.4/Dockerfile`

**Client:** MySQL 8.4.5

**Installed Tools:**
- `mysql` - MySQL client (8.4.5)
- `mysqldump` - Database backup utility
- `mysqladmin` - Server administration utility
- `curl` - HTTP client
- `wget` - File downloader

**Note:** MySQL 8.4 no longer includes `mysqlshow` as it has been deprecated.

**Usage:**
```bash
kubectl run mysql-8.4 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/mysql-8.4:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/mysql/8.4.yml
kubectl exec -it mysql-debug-8.4-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto mysql/8.4
```

### PostgreSQL Debug

PostgreSQL database debugging and development tools for database administration and troubleshooting. Available in multiple PostgreSQL versions.

#### postgresql-13

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/postgresql-13:latest`

**PostgreSQL Version:** 13 (latest patch version)

**Installed Tools:**
- `psql` - PostgreSQL interactive terminal (13.x)
- `pg_dump` - PostgreSQL database backup utility
- `pg_restore` - PostgreSQL database restoration utility
- `pg_isready` - Check PostgreSQL server availability
- `createdb` - Create a PostgreSQL database
- `dropdb` - Remove a PostgreSQL database
- `curl` - HTTP client
- `wget` - File downloader

**Usage:**
```bash
kubectl run postgresql-13 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/postgresql-13:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/postgresql-13.yml
kubectl exec -it postgresql-13-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto postgresql-13
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

#### postgresql-14

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/postgresql-14:latest`

**PostgreSQL Version:** 14 (latest patch version)

**Installed Tools:**
- `psql` - PostgreSQL interactive terminal (14.x)
- `pg_dump` - PostgreSQL database backup utility
- `pg_restore` - PostgreSQL database restoration utility
- `pg_isready` - Check PostgreSQL server availability
- `createdb` - Create a PostgreSQL database
- `dropdb` - Remove a PostgreSQL database
- `curl` - HTTP client
- `wget` - File downloader

**Usage:**
```bash
kubectl run postgresql-14 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/postgresql-14:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/postgresql-14.yml
kubectl exec -it postgresql-14-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto postgresql-14
```

#### postgresql-15

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/postgresql-15:latest`

**PostgreSQL Version:** 15 (latest patch version)

**Installed Tools:**
- `psql` - PostgreSQL interactive terminal (15.x)
- `pg_dump` - PostgreSQL database backup utility
- `pg_restore` - PostgreSQL database restoration utility
- `pg_isready` - Check PostgreSQL server availability
- `createdb` - Create a PostgreSQL database
- `dropdb` - Remove a PostgreSQL database
- `curl` - HTTP client
- `wget` - File downloader

**Usage:**
```bash
kubectl run postgresql-15 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/postgresql-15:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/postgresql-15.yml
kubectl exec -it postgresql-15-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto postgresql-15
```

### Ruby Debug

Ruby development and debugging tools for working with Ruby applications. Available in multiple Ruby versions.

#### ruby-3.3

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/ruby-3.3:latest`

**Ruby Version:** 3.3.10

**Installed Tools:**
- `ruby` - Ruby interpreter (3.3.10)
- `irb` - Interactive Ruby shell (1.13.1)
- `gem` - Ruby package manager (3.5.22)
- `bundler` - Ruby dependency manager (2.5.22)
- `git` - Version control (for fetching dependencies)
- `curl` / `wget` - HTTP clients
- `vim` - Text editor
- `build-essential` - C compiler and build tools (for native gem extensions)
- Development libraries: `libssl-dev`, `libreadline-dev`, `zlib1g-dev`

**Usage:**
```bash
kubectl run ruby-3.3 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/ruby-3.3:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/ruby-3.3.yml
kubectl exec -it ruby-3.3-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto ruby-3.3
```

#### ruby-3.4

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/ruby-3.4:latest`

**Ruby Version:** 3.4.7

**Installed Tools:**
- `ruby` - Ruby interpreter (3.4.7)
- `irb` - Interactive Ruby shell (1.14.3)
- `gem` - Ruby package manager (3.6.9)
- `bundler` - Ruby dependency manager (2.6.9)
- `git` - Version control (for fetching dependencies)
- `curl` / `wget` - HTTP clients
- `vim` - Text editor
- `build-essential` - C compiler and build tools (for native gem extensions)
- Development libraries: `libssl-dev`, `libreadline-dev`, `zlib1g-dev`

**Usage:**
```bash
kubectl run ruby-3.4 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/ruby-3.4:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/ruby-3.4.yml
kubectl exec -it ruby-3.4-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto ruby-3.4
```

### Deployment Scripts

For easier deployment with intelligent resource allocation, use the provided scripts:

```bash
# Deploy to specific context and namespace
./bin/deploy-debug-pod -c my-context -n my-namespace network/debug

# Deploy and automatically exec into the pod
./bin/deploy-debug-pod --auto postgresql/15

# Override resources (useful for constrained environments)
./bin/deploy-debug-pod -m 256Mi -e 256Mi mysql/8.0

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

1. Create a new directory under `images/` organized by category and version:
   ```bash
   mkdir -p images/category/version
   # Example: images/redis/7.0
   ```

2. Add a `Dockerfile` in that directory:
   ```bash
   touch images/category/version/Dockerfile
   # Example: images/redis/7.0/Dockerfile
   ```

3. Build your Dockerfile following these guidelines:
   - Use `debian:bookworm-slim` as base for consistency
   - Install only necessary tools
   - Clean up apt cache to minimize image size
   - Set appropriate CMD or ENTRYPOINT

4. Create a corresponding pod manifest in `pods/`:
   ```bash
   mkdir -p pods/category
   touch pods/category/version.yml
   # Example: pods/redis/7.0.yml
   ```

5. Update the manifest to reference the correct image:
   ```yaml
   image: ghcr.io/c-gerke/k8s-debug-pods/category-version:latest
   # Example: ghcr.io/c-gerke/k8s-debug-pods/redis-7.0:latest
   ```

6. Add tests to `.github/workflows/build-images.yml` for the new image

7. Commit and push the changes:
   ```bash
   git add images/category/ pods/category/
   git commit -m "Add category/version debug image"
   git push
   ```

8. GitHub Actions will automatically build and push the image to:
   ```
   ghcr.io/c-gerke/k8s-debug-pods/category-version:latest
   ```

## CI/CD Pipeline

### Automated Image Builds

When a Dockerfile is modified in the `images/` directory:
1. GitHub Actions detects the change
2. Builds the affected image(s) using Docker Buildx
3. **Runs comprehensive tests** to verify all tools work correctly
4. Tags with multiple formats (latest, branch name, full git commit hash)
5. Pushes to GitHub Container Registry (ghcr.io) only if tests pass
6. Builds for linux/amd64 platform

### Automated Testing

Every image is tested before being pushed to ensure quality:

- **Common tests:** Verify bash and basic functionality
- **Tool-specific tests:** Validate all documented tools work correctly
- **Version verification:** Ensure correct versions are installed (e.g., PostgreSQL 15.x, Ruby 3.4.x, MySQL 8.0.x)
- **Fail-fast:** Build fails immediately if any test fails

Example tests for postgresql-15:
- PostgreSQL 15.x version verification
- psql, pg_dump, pg_restore, pg_isready functionality
- curl and wget availability

Example tests for mysql/8.0:
- MySQL 8.0.x version verification
- mysql, mysqldump, mysqladmin functionality
- curl and wget availability

This testing framework enables safe auto-merging of dependency updates. See [.github/TESTING.md](.github/TESTING.md) for details.

### Dependency Updates

Renovate automatically:
- Monitors base images in all Dockerfiles
- Creates PRs for updates
- **Auto-merges minor and patch version updates** (after tests pass)
- Requires manual approval for major version updates
- Tests ensure no breaking changes reach production

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
cd images/network/debug
docker build -t network-debug:local .
docker run --rm -it network-debug:local
```

## Contributing

When adding new images:
- Keep images focused on specific debugging tasks
- Document installed tools in this README
- Follow Debian bookworm-slim base image convention
- Optimize for size by cleaning up package manager caches
