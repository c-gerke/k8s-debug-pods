# Testing Strategy

This repository implements comprehensive automated testing for all debug pod images to ensure quality and enable safe auto-merging of dependency updates.

## Test Execution

Tests run automatically on every pull request and push to main via GitHub Actions (`.github/workflows/build-images.yml`). The workflow:

1. Builds the Docker image
2. Loads it locally
3. Runs image-specific tests
4. Fails the build if any test fails
5. Only pushes the image if all tests pass

## Test Coverage by Image

### network-debug

Tests all network troubleshooting tools:

- `bash` - Shell availability
- `curl` - HTTP client
- `wget` - File downloader
- `dig` - DNS lookup
- `nslookup` - DNS query
- `ping` - ICMP connectivity
- `netstat` - Network statistics
- `ss` - Socket statistics
- `ip` - Network interface management
- `nc` (netcat) - Network utility
- `telnet` - TCP connection testing

### postgresql-debug-13, postgresql-debug-14, postgresql-debug-15

Tests PostgreSQL client tools and verifies correct version:

- `psql` - PostgreSQL interactive terminal (version verified)
- `pg_dump` - Database backup utility
- `pg_restore` - Database restoration utility
- `pg_isready` - Connection health check
- `createdb` - Database creation utility
- `dropdb` - Database deletion utility
- `curl` - HTTP client
- `wget` - File downloader

**Version Verification:** Tests ensure the correct major version is installed (e.g., postgresql-debug-15 must have PostgreSQL 15.x)

### ruby-debug-3.3, ruby-debug-3.4

Tests Ruby development tools and verifies correct version:

- `ruby` - Ruby interpreter (version verified)
- `irb` - Interactive Ruby shell
- `gem` - Ruby package manager
- `bundle` (bundler) - Dependency manager
- `git` - Version control
- `curl` - HTTP client
- `wget` - File downloader
- `vim` - Text editor
- `gcc` - C compiler (for native gem extensions)

**Version Verification:** Tests ensure the correct major.minor version is installed (e.g., ruby-debug-3.4 must have Ruby 3.4.x)

## Adding Tests for New Images

When adding a new debug image type:

1. Add a new case statement in `.github/workflows/build-images.yml` under the "Test image" step
2. Test all tools documented in the image's README
3. Include version verification if the image is version-specific
4. Use `set -e` at the start to fail on any error
5. Print clear success messages for each test

Example test structure:

```bash
your-new-image)
  echo ""
  echo "=== Running Your New Image Tests ==="
  
  # Test primary tool
  VERSION=$(docker run --rm "$IMAGE_TAG" your-tool --version)
  echo "✓ your-tool works: $VERSION"
  
  # Verify version if needed
  echo "$VERSION" | grep -q "expected-version" || (echo "❌ Expected version X" && exit 1)
  echo "✓ Version verified"
  
  # Test additional tools
  docker run --rm "$IMAGE_TAG" tool2 --version
  echo "✓ tool2 works"
  ;;
```

## Renovate Auto-Merge

The testing framework enables safe auto-merging of dependency updates:

- **Minor and patch updates:** Auto-merged after tests pass
- **Major updates:** Require manual approval

This is configured in `renovate.json`:

- Docker base images (minor/patch): Auto-merge
- Docker base images (major): Manual review required
- GitHub Actions (minor/patch): Auto-merge
- GitHub Actions (major): Manual review required

## Test Failure Handling

If tests fail:

1. The build workflow fails immediately
2. The image is NOT pushed to the registry
3. GitHub Actions provides detailed logs showing which test failed
4. The PR cannot be merged until tests pass

## Running Tests Locally

To test an image locally before pushing:

```bash
cd images/your-image-name
docker build --platform linux/amd64 -t your-image:test .

# Run your tests
docker run --rm your-image:test bash --version
docker run --rm your-image:test your-tool --version
# ... additional tests
```

## Best Practices

1. **Test all documented tools** - If a tool is listed in the README, it should be tested
2. **Verify versions** - For versioned images, ensure the correct version is installed
3. **Test real usage** - Don't just check if commands exist; verify they can actually run
4. **Clear output** - Use descriptive messages so failures are easy to diagnose
5. **Fail fast** - Use `set -e` to stop on first failure
6. **Quick tests** - Tests should complete in seconds, not minutes

## CI/CD Integration

The testing workflow integrates with:

- **Pull Requests:** Tests run on all PRs before merge is allowed
- **Main Branch:** Tests verify images before pushing to registry
- **Renovate:** Automatic updates only merge if tests pass
- **Status Checks:** GitHub branch protection can require tests to pass

This ensures that no untested or broken images ever reach the main branch or container registry.

