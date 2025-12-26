# Debug Pod Deployment Scripts

Helper scripts for deploying and managing debug pods in Kubernetes clusters.

## Prerequisites

- `kubectl` configured with access to your clusters
- `yq` (YAML processor) - Install with `brew install yq` on macOS

## Scripts

### deploy-debug-pod

Deploy a debug pod with intelligent resource allocation based on namespace quotas.

**Basic Usage:**
```bash
./bin/deploy-debug-pod network/debug
```

**With Context and Namespace:**
```bash
./bin/deploy-debug-pod -c wc-beta-px -n example-rails-app-pr1110 mysql/8.0
```

**Auto-exec into Pod:**
```bash
./bin/deploy-debug-pod --auto postgresql/15
```

**Override Resources:**
```bash
./bin/deploy-debug-pod -m 512Mi -e 512Mi ruby/4.0
```

**Custom TTL (Time-to-Live):**
```bash
# Deploy with 4 hour TTL
./bin/deploy-debug-pod --ttl 14400 network/debug

# Deploy with no TTL (unlimited lifetime)
./bin/deploy-debug-pod --ttl 0 mysql/8.0
```

**Force Recreation of Existing Pod:**
```bash
./bin/deploy-debug-pod --force network/debug
```

**Exec into Existing Pod (if exists):**
```bash
./bin/deploy-debug-pod --auto network/debug
```

**List Available Pod Types:**
```bash
./bin/deploy-debug-pod --list-images
```

**Important:** If a pod with the same name already exists, the script will **NOT** recreate it by default. Instead, it will inform you and provide options. Use `--force` to explicitly delete and recreate, or `--auto` to exec into the existing pod.

### cleanup-debug-pods

Clean up debug pods from a namespace.

**List Debug Pods:**
```bash
./bin/cleanup-debug-pods
```

**Delete All Debug Pods:**
```bash
./bin/cleanup-debug-pods --all
```

**Delete Specific Pod:**
```bash
./bin/cleanup-debug-pods --name network-debug-pod
```

**Dry Run:**
```bash
./bin/cleanup-debug-pods --all --dry-run
```

## How It Works

### Resource Allocation

The deployment script intelligently calculates resource allocations:

1. **Manifest Minimum**: Reads minimum resource requirements from each pod's manifest file
2. **Namespace Aware**: Checks ResourceQuota if present
3. **Smart Stepping**: Steps up in 64Mi increments from the manifest minimum
4. **Conservative Buffer**: Uses only 80% of quota (leaves 20% for other pods)
5. **Maximum Cap**: Never exceeds 512Mi even if quota allows more
6. **Override**: Manual overrides with `-m` and `-e` flags

Each pod type defines its own minimum resources in its manifest:
- `network/debug`: 32Mi (lightweight debugging)
- `mysql/8.0`, `mysql/8.4`, `mysql/9.5.0`: 128Mi (database tools)
- `postgresql/15`: 128Mi (database tools)
- `ruby/3.3`, `ruby/3.4`, `ruby/4.0`: 128Mi (Rails debugging)

**Example allocation with network/debug (32Mi minimum) and 2Gi namespace quota:**
- Manifest minimum: 32Mi
- Usable quota: 2Gi × 80% = 1.6Gi
- Allocation: 32Mi → 96Mi → 160Mi → ... → 512Mi (capped at max)

**Example allocation with mysql/8.0 (128Mi minimum) and 500Mi namespace quota:**
- Manifest minimum: 128Mi
- Usable quota: 500Mi × 80% = 400Mi
- Allocation: 128Mi → 192Mi → 256Mi → 320Mi → 384Mi (stops before exceeding)

### Template-Based

The script uses YAML manifests in `pods/` directory as templates:

- Each pod type has its own manifest file organized by category/version
- Script uses `yq` to modify resources dynamically
- Preserves volumes, commands, and other custom configurations
- Easy to add new pod types - just create a new YAML file in the appropriate category folder
- **Safe by default:** Won't recreate existing pods unless `--force` is used
- Use `--auto` to exec into existing pod or create if missing

### Automatic Cleanup

Debug pods automatically clean themselves up using Kubernetes' native `activeDeadlineSeconds`:

- **Default TTL**: 2 hours (7200 seconds)
- **Configurable**: Use `--ttl` flag to set custom lifetime in seconds
- **Automatic Termination**: Kubernetes terminates pods when TTL expires
- **Resource Protection**: Prevents forgotten debug pods from consuming cluster resources
- **Native Feature**: No CronJob or external cleanup required
- **Disable TTL**: Use `--ttl 0` for unlimited lifetime

When a pod reaches its deadline, Kubernetes marks it as `Failed` and it will be garbage collected.

**Examples:**
```bash
# Default 2-hour TTL
./bin/deploy-debug-pod network/debug

# 1-hour TTL (3600 seconds)
./bin/deploy-debug-pod --ttl 3600 mysql/8.0

# 8-hour TTL for long debugging sessions
./bin/deploy-debug-pod --ttl 28800 postgresql/15

# No TTL - pod runs until manually deleted
./bin/deploy-debug-pod --ttl 0 ruby/4.0
```

## Adding New Pod Types

1. Create a new manifest in `pods/` organized by category:
```bash
mkdir -p pods/redis
cp pods/network/debug.yml pods/redis/7.0.yml
```

2. Customize the manifest (image, volumes, commands, etc.)

3. Deploy using the script with category/version format:
```bash
./bin/deploy-debug-pod redis/7.0
```

The script automatically:
- Finds the matching YAML file in the category folder
- Calculates appropriate resources
- Applies customizations
- Deploys to your cluster

## Examples

### Quick Network Debugging
```bash
# Deploy and exec in one command
./bin/deploy-debug-pod --auto -n production network/debug
```

### Custom Resources for Large Namespace
```bash
# Override with 1GB memory and storage
./bin/deploy-debug-pod -m 1Gi -e 1Gi -n large-namespace postgresql/15
```

### Multiple Contexts
```bash
# Deploy to staging
./bin/deploy-debug-pod -c staging -n my-app mysql/9.5.0

# Deploy to production
./bin/deploy-debug-pod -c production -n my-app ruby/4.0
```

### Using Percona Images
```bash
# Deploy Percona MySQL debug pod
./bin/deploy-debug-pod --auto mysql/percona-8.0

# Deploy Percona PostgreSQL debug pod
./bin/deploy-debug-pod --auto postgresql/percona-13

# With specific context and namespace
./bin/deploy-debug-pod -c production -n my-app mysql/percona-8.0
```

### Cleanup After Debugging
```bash
# List what's running
./bin/cleanup-debug-pods -n my-app

# Remove all debug pods
./bin/cleanup-debug-pods -n my-app --all
```

