# Copilot Instructions for k8s-debug-pods

## Repository Purpose

This repository maintains purpose-built container images and Kubernetes pod manifests for debugging applications in Kubernetes clusters. Each image is optimized for size and focused on specific debugging tasks.

## Architecture & Design Principles

### Image Structure
- **Base image**: Always use `debian:bookworm-slim` for consistency and small footprint
- **Organization**: Each debugging tool set lives in `images/<purpose>/Dockerfile`
- **Registry**: All images push to `ghcr.io/c-gerke/k8s-debug-pods/<purpose>:latest`
- **Platform**: Build for `linux/amd64` only (no arm64)

### Dockerfile Best Practices
```dockerfile
FROM debian:bookworm-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    <package1> \
    <package2> && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
           /tmp/* \
           /var/tmp/* \
           /var/cache/debconf/* \
           /var/lib/dpkg/status-old \
           /var/cache/apt/*.bin

CMD ["/bin/bash"]
```

Key points:
- Use `--no-install-recommends` to minimize installed packages
- Chain all RUN commands to reduce layers
- Clean up apt cache, temp files, and dpkg cache in the same layer
- Target 98%+ efficiency when analyzed with `dive`

### Testing Image Efficiency
Always test new images with dive:
```bash
docker build -t test-image:local images/<purpose>/
dive --ci test-image:local
```

Aim for:
- Efficiency: >98%
- Wasted space: <5MB

## CI/CD Pipeline

### GitHub Actions Workflow
- **Trigger**: Only builds when `images/**/Dockerfile` files change
- **Path filtering**: Automatically detects which images changed and builds only those
- **Authentication**: Uses built-in `GITHUB_TOKEN` for ghcr.io
- **Manual trigger**: `gh workflow run build-images.yml` builds all images
- **Tagging strategy**:
  - Main branch: `latest`, `main`, `sha-<commit>`
  - PRs: `pr-<number>`, `sha-<commit>`

### Renovate Configuration
- **Pinning**: All dependencies use SHA digests for reproducibility
- **Auto-merge**: Minor and patch updates auto-merge
- **Manual approval**: Major version updates require review
- **Monitoring**:
  - Dockerfile base images
  - GitHub Actions versions (grouped as "GitHub Actions")

## Adding New Debug Images

1. Create directory structure:
```bash
mkdir -p images/new-debugger
```

2. Add Dockerfile following best practices above

3. Update README.md:
   - Add section under "Available Images"
   - Document installed tools
   - Provide usage example
   - Include image reference

4. Optionally add pod manifest in `pods/`

5. Commit and push - GitHub Actions handles the rest

## Pod Manifests

Pod manifests in `pods/` directory serve as templates for the deployment script. Each manifest should:

1. Be named `<purpose>.yml` (e.g., `network-debug.yml`)
2. Include standard labels: `app: debug-pod` and `type: <purpose>`
3. Set reasonable default resources (128Mi memory/ephemeral-storage)
4. Include any volumes, environment variables, or special configurations

Example structure:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: descriptive-debug-pod
  labels:
    app: debug-pod
    type: network-debug
spec:
  containers:
  - name: debug-container
    image: ghcr.io/c-gerke/k8s-debug-pods/<purpose>:latest
    command: ['sleep', '3600']
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
        ephemeral-storage: "128Mi"
      limits:
        memory: "128Mi"
        cpu: "500m"
        ephemeral-storage: "128Mi"
  restartPolicy: Never
```

The deployment script uses `yq` to dynamically modify:
- Pod name
- Resource requests/limits (memory and ephemeral-storage)

All other configurations (volumes, commands, env vars, etc.) are preserved from the template.

## Code Style & Documentation

### Comments
- Keep minimal and descriptive
- No AI-style filler language
- Use plain English
- Only add when necessary for clarity

### Documentation
- Update README.md when adding new images
- Document all installed tools and their purpose
- Provide clear usage examples
- Keep documentation concise and actionable

### README Maintenance (CRITICAL)
**ALWAYS review and update README.md when making changes.** The README must remain accurate and copy-pasteable at all times.

Update README.md when:
- Adding new images (add to "Available Images" section)
- Changing image names or registry paths
- Modifying repository name or structure
- Updating tooling or packages in existing images
- Changing workflow behavior or CI/CD process

Ensure all code blocks and commands:
- Are tested and working
- Use current/correct image names and paths
- Can be copied and pasted directly without modification
- Include full commands with all necessary flags/options

Before completing any change:
1. Review entire README.md for affected sections
2. Update all references to changed names/paths
3. Verify code examples match actual implementation
4. Test that commands are copy-pasteable

### Git Workflow
- Descriptive commit messages
- One logical change per commit
- Test builds locally before pushing when possible

## Repository-Specific Commands

### Deployment Scripts

```bash
# Deploy debug pod with auto-resource calculation
./bin/deploy-debug-pod -c <context> -n <namespace> <pod-type>

# Deploy and exec in one command
./bin/deploy-debug-pod --auto <pod-type>

# Override resources
./bin/deploy-debug-pod -m 512Mi -e 512Mi <pod-type>

# List available pod types
./bin/deploy-debug-pod --list-images

# Cleanup debug pods
./bin/cleanup-debug-pods -n <namespace> --all
```

### Local Development
```bash
# Build image locally
cd images/<purpose>
docker build -t <purpose>:local .

# Test the image
docker run --rm -it <purpose>:local

# Analyze efficiency
dive <purpose>:local
dive --ci <purpose>:local

# Pull from registry (force amd64 on ARM Macs)
docker pull --platform linux/amd64 ghcr.io/c-gerke/k8s-debug-pods/<purpose>:latest
```

### GitHub Actions
```bash
# View recent runs
gh run list --limit 5

# View specific run logs
gh run view <run-id> --log

# Manually trigger build of all images
gh workflow run build-images.yml

# Watch workflow run
gh run watch
```

### Kubernetes Usage
```bash
# Quick ephemeral pod
kubectl run <name> --rm -it \
  --image=ghcr.io/c-gerke/k8s-debug-pods/<purpose>:latest \
  --restart=Never \
  -- /bin/bash

# Apply manifest
kubectl apply -f pods/<manifest>.yml
kubectl exec -it <pod-name> -- /bin/bash

# Cleanup
kubectl delete pod <pod-name>
```

## Common Debugging Packages

### Network Tools
- `curl`, `wget` - HTTP/download
- `dnsutils` - dig, nslookup
- `iputils-ping` - ping
- `net-tools` - netstat, ifconfig
- `iproute2` - ip, ss
- `telnet` - TCP testing
- `netcat-traditional` - nc utility

### System Tools
- `procps` - ps, top
- `htop` - interactive process viewer
- `strace` - system call tracer
- `lsof` - list open files

### File Tools
- `vim` or `nano` - text editor
- `jq` - JSON processor
- `file` - file type identifier

## Image Naming Convention

Format: `ghcr.io/c-gerke/k8s-debug-pods/<purpose>:latest`

Examples:
- `ghcr.io/c-gerke/k8s-debug-pods/network-debug:latest`
- `ghcr.io/c-gerke/k8s-debug-pods/system-debug:latest`
- `ghcr.io/c-gerke/k8s-debug-pods/db-debug:latest`

Keep names:
- Lowercase with hyphens
- Descriptive of primary purpose
- Short and memorable

## Known Configurations

- **Platform**: amd64 only (no multi-arch builds)
- **Registry**: GitHub Container Registry (ghcr.io)
- **Base OS**: Debian Bookworm Slim
- **Shell**: Bash (default CMD)
- **Package manager**: apt-get

## Troubleshooting

### Build fails with "invalid tag" error
Check metadata-action tag configuration - ensure no empty prefixes like `:-<sha>`

### Image not found on ARM Mac
Use `--platform linux/amd64` when pulling

### Renovate not running
Ensure Renovate GitHub App is installed on the repository

### Workflow doesn't trigger
Check if Dockerfile path matches `images/**/Dockerfile` pattern

### Build happens but image size is large
Review cleanup steps in Dockerfile - ensure cache removal happens in same RUN layer

## Future Expansion Ideas

Consider adding specialized images for:
- Database debugging (mysql-client, psql, redis-cli, influx)
- Application debugging (language-specific tools)
- Storage debugging (filesystem tools)
- Security scanning (vulnerability scanners)
- Performance profiling (perf, flamegraphs)

Each should follow the same patterns established here.

