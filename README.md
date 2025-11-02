# k8s-debugging

A collection of purpose-built container images and Kubernetes manifests for debugging and troubleshooting applications running in Kubernetes clusters.

## Repository Structure

```
k8s-debugging/
├── images/              # Container image definitions
│   └── network-debug/   # Network debugging tools
│       └── Dockerfile
├── pods/                # Example Kubernetes pod manifests
│   └── debug-pod.yml
├── .github/
│   └── workflows/       # CI/CD automation
└── renovate.json        # Automated dependency updates
```

## Available Images

### network-debug

Network troubleshooting tools for diagnosing connectivity issues.

**Image:** `ghcr.io/c-gerke/k8s-debugging/network-debug:latest`

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
  --image=ghcr.io/c-gerke/k8s-debugging/network-debug:latest \
  --restart=Never \
  -- /bin/bash
```

Or apply the example pod manifest:
```bash
kubectl apply -f pods/debug-pod.yml
kubectl exec -it ubuntu-debug-pod -- /bin/bash
```

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
   ghcr.io/c-gerke/k8s-debugging/my-new-debugger:latest
   ```

## CI/CD Pipeline

### Automated Image Builds

When a Dockerfile is modified in the `images/` directory:
1. GitHub Actions detects the change
2. Builds the affected image(s) using Docker Buildx
3. Tags with multiple formats (latest, branch name, git sha)
4. Pushes to GitHub Container Registry (ghcr.io)
5. Supports multi-architecture builds (amd64, arm64)

### Dependency Updates

Renovate automatically:
- Monitors base images in all Dockerfiles
- Creates PRs for updates
- Auto-merges minor and patch version updates
- Requires manual approval for major version updates

## Requirements

- Kubernetes cluster with appropriate RBAC permissions
- Access to pull from GitHub Container Registry (public by default)

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
