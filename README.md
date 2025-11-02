# Kubernetes debugging pods

A collection of purpose-built container images and Kubernetes manifests for debugging and troubleshooting applications running in Kubernetes clusters.

## Repository Structure

```
k8s-debug-pods/
├── images/              # Container image definitions
│   ├── network-debug/   # Network debugging tools
│   │   └── Dockerfile
│   ├── ruby-debug-3.3/  # Ruby 3.3.x development tools
│   │   └── Dockerfile
│   └── ruby-debug-3.4/  # Ruby 3.4.x development tools
│       └── Dockerfile
├── pods/                # Kubernetes pod manifests
│   ├── network-debug.yml
│   ├── ruby-debug-3.3.yml
│   └── ruby-debug-3.4.yml
├── bin/                 # Deployment helper scripts
│   ├── deploy-debug-pod
│   └── cleanup-debug-pods
├── .github/
│   └── workflows/       # CI/CD automation
└── renovate.json        # Automated dependency updates
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

### ruby-debug

Ruby development and debugging tools for working with Ruby applications. Available in multiple Ruby versions.

#### ruby-debug-3.3

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/ruby-debug-3.3:latest`

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
kubectl run ruby-debug-3.3 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/ruby-debug-3.3:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/ruby-debug-3.3.yml
kubectl exec -it ruby-debug-3.3-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto ruby-debug-3.3
```

#### ruby-debug-3.4

**Image:** `ghcr.io/c-gerke/k8s-debug-pods/ruby-debug-3.4:latest`

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
kubectl run ruby-debug-3.4 --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/ruby-debug-3.4:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the pod manifest directly:
```bash
kubectl apply -f pods/ruby-debug-3.4.yml
kubectl exec -it ruby-debug-3.4-pod -- /bin/bash
```

Or using the deployment script:
```bash
./bin/deploy-debug-pod --auto ruby-debug-3.4
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
