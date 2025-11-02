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
./bin/deploy-debug-pod network-debug
```

**With Context and Namespace:**
```bash
./bin/deploy-debug-pod -c wc-beta-px -n example-rails-app-pr1110 network-debug
```

**Auto-exec into Pod:**
```bash
./bin/deploy-debug-pod --auto network-debug
```

**Override Resources:**
```bash
./bin/deploy-debug-pod -m 512Mi -e 512Mi network-debug
```

**List Available Pod Types:**
```bash
./bin/deploy-debug-pod --list-images
```

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

1. **Default**: Starts with 128Mi for memory and ephemeral storage
2. **Namespace Aware**: Checks ResourceQuota if present
3. **Scaling**: Can allocate up to 1Gi based on available quota
4. **Override**: Manual overrides with `-m` and `-e` flags

### Template-Based

The script uses YAML manifests in `pods/` directory as templates:

- Each pod type has its own manifest file
- Script uses `yq` to modify resources dynamically
- Preserves volumes, commands, and other custom configurations
- Easy to add new pod types - just create a new YAML file

## Adding New Pod Types

1. Create a new manifest in `pods/`:
```bash
cp pods/network-debug.yml pods/database-debug.yml
```

2. Customize the manifest (image, volumes, commands, etc.)

3. Deploy using the script:
```bash
./bin/deploy-debug-pod database-debug
```

The script automatically:
- Finds the matching YAML file
- Calculates appropriate resources
- Applies customizations
- Deploys to your cluster

## Examples

### Quick Network Debugging
```bash
# Deploy and exec in one command
./bin/deploy-debug-pod --auto -n production network-debug
```

### Custom Resources for Large Namespace
```bash
# Override with 1GB memory and storage
./bin/deploy-debug-pod -m 1Gi -e 1Gi -n large-namespace network-debug
```

### Multiple Contexts
```bash
# Deploy to staging
./bin/deploy-debug-pod -c staging -n my-app network-debug

# Deploy to production
./bin/deploy-debug-pod -c production -n my-app network-debug
```

### Cleanup After Debugging
```bash
# List what's running
./bin/cleanup-debug-pods -n my-app

# Remove all debug pods
./bin/cleanup-debug-pods -n my-app --all
```

