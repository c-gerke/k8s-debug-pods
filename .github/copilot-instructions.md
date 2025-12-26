# Copilot Instructions for k8s-pods

## Repository Purpose

This repository maintains purpose-built container images and Kubernetes pod manifests for debugging applications in Kubernetes clusters. Each image is optimized for size and focused on specific debugging tasks. Includes deployment scripts for easy pod creation with intelligent resource allocation.

## Architecture & Design Principles

### Image Structure
- **Base image**: Use `debian:bookworm-slim` for consistency and small footprint. Language-specific images (Ruby, MySQL) use official base images (e.g., `ruby:4.0-slim`, `mysql:8.0-debian`)
- **Organization**: Images organized as `images/<category>/<version>/Dockerfile` (e.g., `images/mysql/8.0/Dockerfile`)
- **Registry**: All images push to `ghcr.io/c-gerke/k8s-pods/<category>-<version>:latest`
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
  - Main branch: `latest`, `main`, `sha-<full-commit-hash>`
  - PRs: `pr-<number>`, `sha-<full-commit-hash>`
  - Uses full 40-character commit SHA for precise tracking
- **Testing**: Comprehensive automated tests run before push (see Automated Testing section)

### Automated Testing

Every image is tested before being pushed to the registry to ensure quality and enable safe auto-merging:

**Test execution flow**:
1. Build Docker image
2. Load image locally (`load: true`)
3. Run comprehensive tests
4. Fail immediately if any test fails
5. Push to registry ONLY if all tests pass

**Test categories**:
- **Common tests**: Bash availability (all images)
- **Tool-specific tests**: Verify each documented tool works
- **Version verification**: Ensure correct versions for versioned images

**Image-specific test requirements**:

```bash
# network/debug: Test all network tools
curl --version, wget --version, dig -v, nslookup, ping -V,
netstat --version, ss --version, ip -V, nc, telnet

# postgresql/13, postgresql/14, postgresql/15: Test PostgreSQL tools + version verification
psql --version (verify 13.x/14.x/15.x), pg_dump, pg_restore,
pg_isready, createdb, dropdb, curl, wget

# ruby/3.3, ruby/3.4, ruby/4.0: Test Ruby tools + version verification
ruby --version (verify 3.3.x/3.4.x/4.0.x), irb, gem, bundle,
git, curl, wget, vim, gcc (for native gems)

# mysql/8.0, mysql/8.4, mysql/9.5.0: Test MySQL tools + version verification
mysql --version (verify 8.0.x/8.4.x/9.5.x), mysqldump, mysqladmin, curl, wget
```

**Critical rules**:
- Use `set -e` to fail fast on any error
- Test ALL tools documented in README
- Verify major/minor versions for versioned images
- Tests should complete quickly (seconds, not minutes)
- Clear output with ✓ markers for passed tests

See [.github/TESTING.md](TESTING.md) for detailed documentation.

### Renovate Configuration
- **Pinning**: All dependencies use SHA digests for reproducibility
- **Auto-merge**: Minor and patch updates auto-merge AFTER tests pass
- **Platform auto-merge**: Uses GitHub native auto-merge (platformAutomerge)
- **Manual approval**: Major version updates require review
- **Monitoring**:
  - Dockerfile base images
  - GitHub Actions versions (grouped as "GitHub Actions")
- **Safety**: Testing framework ensures no broken images reach production

## Adding New Debug Images

### Building Custom Images

1. Create directory structure organized by category and version:
```bash
mkdir -p images/redis/7.0
mkdir -p pods/redis
```

2. Add Dockerfile following best practices above in `images/redis/7.0/Dockerfile`

3. Create pod manifest template in `pods/`:
```bash
cp pods/network/debug.yml pods/redis/7.0.yml
# Edit to customize image, volumes, env vars, etc.
```

4. **Add tests to `.github/workflows/build-images.yml`** (REQUIRED):
```bash
# Add new case in "Test image" step
your-new-image)
  echo ""
  echo "=== Running Your New Image Tests ==="
  
  # Test each tool
  docker run --rm "$IMAGE_TAG" your-tool --version
  echo "✓ your-tool works"
  
  # More tests for all documented tools
  ;;
```

5. Update README.md:
   - Add section under "Available Images"
   - Document installed tools
   - Provide usage example
   - Include image reference

6. Commit and push - GitHub Actions builds, tests, and pushes

**CRITICAL**: Every new image MUST have tests added to the workflow. Images without tests will only get basic bash verification. See [.github/TESTING.md](TESTING.md) for test writing guidelines.

**Note**: Pod manifests are organized by category/version (e.g., `pods/redis/7.0.yml` for `redis/7.0` image) so deployment scripts can find them automatically using the `category/version` format.

### Using External Images

For matching production infrastructure exactly (e.g., Percona images), you can create pod manifests without building custom images:

1. Create pod manifest in `pods/`:
```bash
mkdir -p pods/mysql
touch pods/mysql/percona-8.0.yml
```

2. Reference external image with SHA pinning:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-percona-debug-8.0-pod
  labels:
    app: debug-pod
    type: mysql-percona-debug-8.0
spec:
  containers:
  - name: debug-container
    image: percona:8.0.36-28@sha256:1128d56e64711ed65cb0c57041048967ee5875a2167d708d327885fd1f995fa0
    command: ['sleep', '3600']
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
        ephemeral-storage: "128Mi"
      limits:
        memory: "128Mi"
        ephemeral-storage: "128Mi"
  restartPolicy: Never
```

3. Update README.md with external image details

4. Commit and push - no CI/CD build needed

**Benefits**:
- Exact match with production infrastructure
- No maintenance overhead of custom images
- SHA pinning ensures reproducibility
- Still compatible with deployment scripts

## Pod Manifests

Pod manifests in `pods/` directory serve as templates for the deployment script. Each manifest should:

1. Be organized as `pods/<category>/<version>.yml` (e.g., `pods/mysql/8.0.yml`)
2. Include standard labels: `app: debug-pod` and `type: <category>-<version>`
3. Set reasonable default resources (128Mi memory/ephemeral-storage)
4. Include any volumes, environment variables, or special configurations
5. Can reference either custom-built images or external images (e.g., Percona, official images)

Example structure:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-8.0-pod
  labels:
    app: debug-pod
    type: mysql-8.0
spec:
  containers:
  - name: debug-container
    image: ghcr.io/c-gerke/k8s-pods/mysql-8.0:latest
    command: ['sleep', '3600']
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
        ephemeral-storage: "128Mi"
      limits:
        memory: "128Mi"
        ephemeral-storage: "128Mi"
  restartPolicy: Never
```

**Key points about pod manifests**:
- CPU limits are omitted to allow burstable pods (only requests are set)
- Memory and ephemeral-storage have both requests and limits
- Default to 128Mi but deployment script can scale up to 1Gi based on namespace quotas
- Script steps up in 64Mi increments and reserves 20% of quota for other pods

The deployment script uses `yq` to dynamically modify:
- Pod name (based on --name flag or defaults to `<type>-pod`)
- Resource requests/limits (memory and ephemeral-storage only)

All other configurations (volumes, commands, env vars, security context, etc.) are preserved from the template.

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

## Deployment Scripts

### Overview
The `bin/` directory contains helper scripts for deploying debug pods with intelligent resource allocation. These scripts use the pod manifests in `pods/` as templates and dynamically adjust resources based on namespace quotas.

**Prerequisites**:
- `kubectl` configured with cluster access
- `yq` YAML processor (install: `brew install yq` on macOS)

### deploy-debug-pod

Deploys a debug pod using a template from `pods/<category>/<version>.yml`:

```bash
# Deploy with context/namespace
./bin/deploy-debug-pod -c <context> -n <namespace> <category>/<version>

# Deploy to current context/namespace
./bin/deploy-debug-pod mysql/8.0

# Deploy and exec in one command
./bin/deploy-debug-pod --auto postgresql/15

# Override resources
./bin/deploy-debug-pod -m 512Mi -e 512Mi ruby/3.4

# Custom pod name
./bin/deploy-debug-pod --name my-debug network/debug

# List available pod types
./bin/deploy-debug-pod --list-images
```

**How it works**:
1. Reads pod manifest template from `pods/<category>/<version>.yml`
2. Checks namespace ResourceQuota (if present)
3. Calculates appropriate resources using smart allocation:
   - Starts at 128Mi default
   - Steps up in 64Mi increments
   - Uses 80% of quota (20% buffer for other pods)
   - Caps at 1Gi maximum
4. Uses `yq` to modify only pod name and resource values
5. Checks if pod already exists:
   - **Default**: Informs user and exits (safe, non-disruptive)
   - **--force**: Deletes and recreates pod (for applying changes)
   - **--auto**: Execs into existing pod or creates if missing
6. Creates pod in cluster (if needed)
7. Preserves all other template configurations (volumes, env, etc.)

### cleanup-debug-pods

Manages cleanup of deployed debug pods:

```bash
# List debug pods in namespace
./bin/cleanup-debug-pods -n <namespace>

# Delete all debug pods (with confirmation)
./bin/cleanup-debug-pods -n <namespace> --all

# Delete specific pod
./bin/cleanup-debug-pods --name mysql-8.0-pod

# Dry run
./bin/cleanup-debug-pods -n <namespace> --all --dry-run
```

Targets pods with label `app=debug-pod` (automatically added by deployment script).

## Repository-Specific Commands

### Local Development
```bash
# Build image locally
cd images/<category>/<version>
docker build -t <category>-<version>:local .

# Test the image
docker run --rm -it <category>-<version>:local

# Analyze efficiency
dive <category>-<version>:local
dive --ci <category>-<version>:local

# Pull from registry (force amd64 on ARM Macs)
docker pull --platform linux/amd64 ghcr.io/c-gerke/k8s-pods/<category>-<version>:latest
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
  --image=ghcr.io/c-gerke/k8s-pods/<category>-<version>:latest \
  --restart=Never \
  -- /bin/bash

# Apply manifest
kubectl apply -f pods/<category>/<version>.yml
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

Format: `ghcr.io/c-gerke/k8s-pods/<category>-<version>:latest`

Examples:
- `ghcr.io/c-gerke/k8s-pods/mysql-8.0:latest`
- `ghcr.io/c-gerke/k8s-pods/mysql-9.5.0:latest`
- `ghcr.io/c-gerke/k8s-pods/postgresql-15:latest`
- `ghcr.io/c-gerke/k8s-pods/ruby-3.4:latest`
- `ghcr.io/c-gerke/k8s-pods/ruby-4.0:latest`
- `ghcr.io/c-gerke/k8s-pods/network-debug:latest`

Keep names:
- Lowercase with hyphens between category and version
- Descriptive of primary purpose and version
- Organized in filesystem as `category/version`

## Known Configurations

- **Platform**: amd64 only (no multi-arch builds)
- **Registry**: GitHub Container Registry (ghcr.io)
- **Base OS**: Debian Bookworm Slim
- **Shell**: Bash (default CMD)
- **Package manager**: apt-get
- **Deployment tool**: yq (required for bin scripts)
- **Pod resource defaults**: 128Mi memory/ephemeral, 100m CPU request (no CPU limit)
- **Resource stepping**: 64Mi increments with 20% quota buffer, 1Gi max

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

### Tests fail in CI but work locally
Check that you're testing the correct image tag (should be `sha-<commit-hash>`)

### Image builds but tests are skipped
Add test case for your image in `.github/workflows/build-images.yml` under "Test image" step

### Auto-merge not working for Renovate PRs
Ensure:
- Tests pass successfully
- Branch protection rules allow auto-merge
- Renovate has permission to auto-merge
- Update is minor/patch (majors require manual approval)

### Deployment script fails with "yq: command not found"
Install yq: `brew install yq` on macOS or download from https://github.com/mikefarah/yq/releases

### Deployment script can't find pod manifest
Ensure manifest is organized correctly (e.g., `pods/mysql/8.0.yml` for type `mysql/8.0`). Use `category/version` format when deploying.

## Future Expansion Ideas

Consider adding specialized images for:
- Database debugging (mysql-client, psql, redis-cli, influx)
- Application debugging (language-specific tools)
- Storage debugging (filesystem tools)
- Security scanning (vulnerability scanners)
- Performance profiling (perf, flamegraphs)

Each should follow the same patterns established here.

## Design Philosophy

### Template-Based Deployment
The repository uses a template-based approach where:
- Pod manifests are the source of truth
- Scripts modify only what's necessary (name, resources)
- Complex configurations (volumes, init containers, etc.) stay in YAML
- Easy to maintain and scale to many pod types

This approach ensures:
- **Maintainability**: Changes to pod structure happen in YAML, not bash
- **Scalability**: Adding new pod types is just creating a new YAML file
- **Flexibility**: Each pod type can have unique volumes, commands, security context
- **Clarity**: Pod configuration is visible and version-controlled in YAML

