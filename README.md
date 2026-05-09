# AList Helm Chart

This chart installs [AList](https://alist.nn.ci/), a file list and WebDAV program that supports multiple storage backends with comprehensive security and performance tuning capabilities.

## Versions

| Component | Version | Notes |
| --- | --- | --- |
| Chart | `0.3.0` | Latest (production-ready) |
| AList app | `v3.60.0` | Upstream application version |
| Default image | `xhofe/alist:v3.60.0` | Standard deployment |
| Helm API | `v2` | Helm 3.0+ required |
| Kubernetes | `>=1.23.0` | Minimum version |

## Repository

```console
helm repo add alist https://ashwinijindal10.github.io/alist-helm-repo/charts
helm repo update
```

## Quick Start

Install with default settings:

```console
helm install alist alist/alist
```

Install from local chart:

```console
helm install alist .
```

Upgrade existing installation:

```console
helm upgrade alist alist/alist
```

## Building the Chart

### Prerequisites

- Helm 3.0+
- kubectl configured to your cluster
- Git (for version management)

### Lint and Validate

Validate chart syntax and structure:

```console
helm lint .
```

Expected output:
```
==> Linting .

1 chart(s) linted, 0 chart(s) failed
```

### Template Preview & Testing

Preview rendered Kubernetes manifests:

```console
# Preview with default values
helm template alist . --values values.yaml

# Preview with custom values
helm template alist . -f custom-values.yaml

# Preview for dry-run
helm install alist . --dry-run --debug
```

### Run Chart Locally

Test chart installation on local cluster (Minikube/Kind):

```console
# 1. Lint first
helm lint .

# 2. Dry-run to validate
helm install alist . --dry-run --debug

# 3. Install to cluster
helm install alist .

# 4. Verify installation
kubectl get pods -l app.kubernetes.io/name=alist
kubectl logs deployment/alist

# 5. Port-forward for testing
kubectl port-forward svc/alist 8080:80
# Access: http://localhost:8080

# 6. Uninstall
helm uninstall alist
```

### Package the Chart

Create a `.tgz` chart package for distribution:

```console
# Package chart
helm package . -d docs/charts/

# List packaged charts
ls -lah docs/charts/*.tgz
```

### Generate Repository Index

Update repository index for GitHub Pages hosting:

```console
helm repo index docs/charts/ \
  --url https://ashwinijindal10.github.io/alist-helm-repo/charts
```

## Releasing a New Version

### 1. Update Chart Version

Edit `Chart.yaml`:

```yaml
version: 0.3.0          # Increment from previous (e.g., 0.2.0 → 0.3.0)
appVersion: v3.60.0     # Update if AList app version changed
```

### 2. Update README Version Table

Update version table at top of README.md:

```markdown
| Chart | `0.3.0` | Latest (production-ready) |
| AList app | `v3.60.0` | Upstream application version |
```

### 3. Update Chart.yaml Annotations

Document changes in `Chart.yaml` under `artifacthub.io/changes`:

```yaml
annotations:
  artifacthub.io/changes: |
    - kind: added
      description: Your new feature description
    - kind: fixed
      description: Bug fix description
    - kind: changed
      description: Breaking/non-breaking change
```

### 4. Run Quality Checks

```console
# Lint the chart
helm lint .

# Preview templates
helm template alist . > /tmp/manifest.yaml
echo "✅ Template valid"

# Test install (dry-run)
helm install alist . --dry-run --debug
```

### 5. Build and Package

```console
# Package the chart
helm package . -d docs/charts/

# Regenerate index
helm repo index docs/charts/ \
  --url https://ashwinijindal10.github.io/alist-helm-repo/charts

# Verify
ls -lah docs/charts/
cat docs/charts/index.yaml
```

### 6. Commit and Release

```bash
# Stage changes
git add Chart.yaml README.md values.yaml (any modified files)

# Commit with semantic version message
git commit -m "chore: bump chart version to 0.3.0

- Updated Chart.yaml with new version
- Updated README with version table
- Added v0.3.0 features in annotations"

# Create git tag
git tag chart-0.3.0

# Push to repository
git push origin main --tags

# Push packaged charts
git add docs/charts/
git commit -m "build: package helm chart v0.3.0"
git push origin main
```

### 7. Verify Release

```console
# Check git history
git log --oneline -5
git tag -l

# Verify chart package
ls -lah docs/charts/alist-*.tgz

# Check repository index
head docs/charts/index.yaml
```

### 8. Update Helm Repo (for users)

Users can now fetch the new version:

```bash
helm repo update alist
helm install alist alist/alist --version 0.3.0
```

### Full Release Workflow Example

```bash
#!/bin/bash
set -e

echo "🔄 Starting chart release process..."

# 1. Update version
NEW_VERSION="0.3.0"
sed -i "" "s/version: .*/version: $NEW_VERSION/" Chart.yaml
echo "✅ Updated Chart.yaml to v$NEW_VERSION"

# 2. Lint
helm lint .
echo "✅ Chart linting passed"

# 3. Template validation
helm template alist . > /dev/null
echo "✅ Template validation passed"

# 4. Package
helm package . -d docs/charts/
echo "✅ Chart packaged"

# 5. Generate index
helm repo index docs/charts/ \
  --url https://ashwinijindal10.github.io/alist-helm-repo/charts
echo "✅ Repository index generated"

# 6. Git operations
git add Chart.yaml README.md values.yaml docs/charts/
git commit -m "chore: release chart v$NEW_VERSION"
git tag "chart-$NEW_VERSION"
git push origin main --tags
echo "✅ Released to GitHub"

echo "🎉 Chart v$NEW_VERSION released successfully!"
```

Save as `release.sh` and run:

```console
chmod +x release.sh
./release.sh
```

## Testing Locally

```console
# Lint
helm lint .

# Validate against cluster
helm install alist . --dry-run --debug

# Install to local cluster
helm install alist .

# Verify installation
kubectl get pods -l app.kubernetes.io/name=alist
kubectl logs deployment/alist

# Uninstall
helm uninstall alist
```

### Full Build Workflow Example

```bash
#!/bin/bash

# 1. Validate chart
helm lint .

# 2. Template preview
helm template alist . > /tmp/alist-manifest.yaml
echo "✅ Template generated"

# 3. Package chart
helm package . -d docs/charts/
echo "✅ Chart packaged"

# 4. Generate repository index
helm repo index docs/charts/ \
  --url https://ashwinijindal10.github.io/alist-helm-repo/charts
echo "✅ Index regenerated"

# 5. Commit and push
git add docs/charts/
git commit -m "build: update helm chart package"
git push origin main
echo "✅ Released to repository"
```

## Key Features

- ✅ **Persistence enabled by default** — Data survives pod restarts
- ✅ **Security controls** — JWT, admin auth, CORS, protocol selection
- ✅ **Multi-database support** — SQLite, MySQL, PostgreSQL
- ✅ **Protocol flexibility** — WebDAV (default), FTP, SFTP, S3
- ✅ **Performance tuning** — Connection limits, rate limiting, task workers
- ✅ **Health checks** — Liveness and readiness probes configured
- ✅ **Logging** — Configurable log levels and rotation

## Persistence

**Persistence is enabled by default** to preserve AList data across restarts.

Disable for temporary testing:

```console
helm install alist alist/alist \
  --set persistence.enabled=false
```

Use existing PVC:

```console
helm install alist alist/alist \
  --set persistence.existingClaim=my-alist-data
```

Increase storage size:

```console
helm install alist alist/alist \
  --set persistence.size=50Gi
```

## Image Variants

Select from multiple image variants:

| Tag | Description |
| --- | --- |
| `v3.60.0` (default) | Standard AList |
| `v3.60.0-aio` | All-in-one with ffmpeg + aria2 |
| `v3.60.0-ffmpeg` | With ffmpeg for thumbnails |
| `v3.60.0-aria2` | With aria2 for offline downloads |
| `latest` | Latest stable |
| `beta` | Development version |

Use the all-in-one variant:

```console
helm install alist alist/alist \
  --set image.tag=v3.60.0-aio
```

## Security Configuration

### Admin Password

Set initial admin password:

```console
helm install alist alist/alist \
  --set security.adminPassword='your-secure-password'
```

**Recommended:** Use Kubernetes Secrets for sensitive data:

```bash
kubectl create secret generic alist-admin \
  --from-literal=password='your-secure-password'
```

### JWT Secret

Generate a secure JWT secret:

```bash
openssl rand -base64 32
```

Configure in values:

```yaml
security:
  jwtSecret: "your-generated-secret-here"
  adminPassword: "your-admin-password"
  forceHttps: true
```

### TLS/HTTPS

Enable HTTPS with manual certificate:

```yaml
security:
  tls:
    enabled: true
```

Enable HTTPS with cert-manager:

```yaml
security:
  tls:
    enabled: true
    certManager:
      enabled: true
      issuer: "letsencrypt-prod"
```

## Database Configuration

### SQLite (Default)

No configuration needed. Data stored in persistence volume at `data/data.db`.

Recommended for: Single-node deployments, small installations, development.

### MySQL

Enable remote MySQL database:

```yaml
database:
  type: mysql
  remote:
    enabled: true
    host: mysql.default.svc.cluster.local
    port: 3306
    user: alist
    password: "secure-password"
    database: alist
    tablePrefix: x_
```

### PostgreSQL

Enable remote PostgreSQL database with environment-specific SSL settings:

#### Development (No SSL)

```yaml
database:
  type: postgres
  remote:
    enabled: true
    host: postgres.default.svc.cluster.local
    port: 5432
    user: alist
    password: "secure-password"
    database: alist
    sslMode: "disable"
    tablePrefix: x_
```

#### Staging (Optional SSL, Prefer)

```yaml
database:
  type: postgres
  remote:
    enabled: true
    host: postgres.staging.example.com
    port: 5432
    user: alist
    password: "secure-password"
    database: alist
    sslMode: "prefer"
    tablePrefix: x_
```

#### Production (Mandatory SSL)

```yaml
database:
  type: postgres
  remote:
    enabled: true
    host: postgres.prod.example.com
    port: 5432
    user: alist
    password: "secure-password"
    database: alist
    sslMode: "require"
    tablePrefix: x_
```

#### Production (Maximum Security)

```yaml
database:
  type: postgres
  remote:
    enabled: true
    host: postgres.prod.example.com
    port: 5432
    user: alist
    password: "secure-password"
    database: alist
    sslMode: "verify-full"
    tablePrefix: x_
```

**SSL Mode Reference:**

| Mode | Description | Security | Use Case |
| --- | --- | --- | --- |
| `disable` | No encryption | Low | Local/trusted networks |
| `allow` | SSL optional, fallback | Medium | Backward compatibility |
| `prefer` | Try SSL, fallback to plain | Medium | Staging environments |
| `require` | SSL mandatory, no CA check | High | **Production (Recommended)** |
| `verify-ca` | SSL + CA certificate verify | Very High | High-security production |
| `verify-full` | SSL + CA + hostname verify | Maximum | Maximum security production |

## Protocol Configuration

### WebDAV (Default)

WebDAV is enabled by default. Disable if not needed:

```yaml
protocols:
  webdav:
    enabled: false
```

### FTP/SFTP

Enable FTP for file transfer:

```yaml
protocols:
  ftp:
    enabled: true
  sftp:
    enabled: true
```

### S3-Compatible API

Enable S3 API support:

```yaml
protocols:
  s3:
    enabled: true
```

## Performance Tuning

### Connection Limits

Adjust for your deployment size:

```yaml
performance:
  maxConnections: 100      # Total DB connections
  maxConcurrency: 50       # Concurrent operations
  maxDevices: 10           # Max devices per user
  deviceSessionTtl: 2592000 # Session timeout (seconds)
```

### Task Workers

Configure parallel task processing:

```yaml
performance:
  taskWorkers:
    download: 2
    upload: 2
    copy: 2
    decompress: 1
```

### Rate Limiting

Limit per-client bandwidth (0 = unlimited):

```yaml
performance:
  streamMaxClientDownloadSpeed: 0
  streamMaxClientUploadSpeed: 0
  streamMaxServerDownloadSpeed: 0
```

## Logging Configuration

Configure log verbosity and retention:

```yaml
logging:
  enabled: true
  level: "info"      # debug, info, warn, error
  maxSize: 10        # MB before rotation
  maxBackups: 5      # Number of backup files
  maxAge: 30         # Retention days
```

Logs are stored at `/opt/alist/data/alist.log` in the container.

## Environment Variables

Additional runtime variables:

```yaml
env:
  PUID: "1000"          # Container process UID
  PGID: "1000"          # Container process GID
  UMASK: "022"          # File creation mask
  RUN_ARIA2: "true"     # Enable aria2 (requires -aria2 image)
  TZ: "America/New_York" # Timezone
```

Set via CLI:

```console
helm install alist alist/alist \
  --set env.PUID=1000 \
  --set env.PGID=1000 \
  --set env.TZ=America/New_York
```

## Access Control

Configure default guest permissions:

```yaml
access:
  guestPermission: 7      # Bitmask: 0=none, 7=read-only, 255=most, 65535=admin
  cors:
    enabled: false
```

## Service & Networking

### Default (ClusterIP)

```console
helm install alist alist/alist
# HTTP accessible within cluster at alist:80
```

### LoadBalancer (External Access)

```console
helm install alist alist/alist \
  --set service.type=LoadBalancer
```

### NodePort

```console
helm install alist alist/alist \
  --set service.type=NodePort \
  --set 'service.ports.http.port=30080'
```

## Ingress

Example nginx ingress with cert-manager:

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: alist.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: alist-tls
      hosts:
        - alist.example.com
```

## Complete Configuration Example

```yaml
# values.yaml
replicaCount: 1

image:
  tag: "v3.60.0-aio"

service:
  type: ClusterIP
  ports:
    http:
      port: 80
      enabled: true

persistence:
  enabled: true
  size: 50Gi
  storageClassName: fast-ssd

security:
  jwtSecret: "your-jwt-secret"
  adminPassword: "your-admin-password"
  forceHttps: false

database:
  type: mysql
  remote:
    enabled: true
    host: mysql.default.svc.cluster.local
    port: 3306
    user: alist
    password: "db-password"
    database: alist

protocols:
  webdav:
    enabled: true
  ftp:
    enabled: false
  sftp:
    enabled: false
  s3:
    enabled: false

performance:
  maxConnections: 100
  maxConcurrency: 50
  maxDevices: 10
  taskWorkers:
    download: 2
    upload: 2
    copy: 2
    decompress: 1

logging:
  enabled: true
  level: "info"
  maxSize: 10
  maxBackups: 5
  maxAge: 30

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: alist.example.com
      paths:
        - path: /
          pathType: Prefix
```

Install with custom configuration:

```console
helm install alist alist/alist -f custom-values.yaml
```

## Access AList

Get the service endpoint:

```console
kubectl get svc alist
```

Port-forward for local access:

```console
kubectl port-forward svc/alist 8080:80
# Access at http://localhost:8080
```

Get admin credentials:

```console
# AList will prompt for admin password on first login
# Or check pod logs: kubectl logs deployment/alist
```

## Troubleshooting

View pod logs:

```console
kubectl logs deployment/alist
```

Check resource usage:

```console
kubectl top pod -l app.kubernetes.io/name=alist
```

Verify persistence:

```console
kubectl get pvc
kubectl describe pvc alist
```

## Resources & Documentation

- [AList Official Documentation](https://alist.nn.ci/)
- [AList GitHub Repository](https://github.com/AlistGo/alist)
- [AList Helm Chart Repository](https://github.com/phenix3443/charts)
- [AList Docker Hub](https://hub.docker.com/r/xhofe/alist)

## License

This Helm chart is licensed under AGPL-3.0. AList itself is licensed under AGPL-3.0-only.
    password: change-me
    name: alist
```

Example SQLite configuration:

```yaml
config:
  site_url: "https://alist.example.com"
  database:
    type: sqlite3
    db_file: data/data.db
    table_prefix: x_
```

## Health Checks

The chart includes automatic health checks (probes) to detect and recover from AList failures:

- **Liveness Probe:** Restarts unhealthy pods (30s initial delay, 10s periodic check)
- **Readiness Probe:** Removes unhealthy pods from load balancer (10s initial delay, 5s periodic check)
- Both check `/api/v3/server/info` endpoint on port 5244

Health checks are automatically configured. Customize timing if needed in values.yaml.

## Common Values

| Value | Default | Description |
| --- | --- | --- |
| `replicaCount` | `1` | Number of pods when autoscaling is disabled. |
| `image.repository` | `xhofe/alist` | AList container image repository. |
| `image.tag` | `v3.60.0` | AList image tag. Supports `-aio`, `-ffmpeg`, `-aria2` variants. |
| `service.type` | `ClusterIP` | Kubernetes Service type. |
| `service.ports.http.enabled` | `true` | Enable HTTP port (5244). |
| `service.ports.https.enabled` | `false` | Enable HTTPS port (5245). |
| `env` | `{}` | Environment variables: PUID, PGID, UMASK, RUN_ARIA2, TZ. |
| `ingress.enabled` | `false` | Create an Ingress resource. |
| `persistence.enabled` | `true` | Create or use a PVC for `/opt/alist/data`. |
| `persistence.size` | `10Gi` | PVC size when creating a claim. |
| `persistence.inMemory.enabled` | `false` | Use emptyDir with Memory medium. |
| `createConfigFile.enabled` | `false` | Generate and copy `config.json` at startup. |
| `namespaceOverride` | `""` | Render namespaced resources into another namespace. |
| `autoscaling.enabled` | `false` | Create a HorizontalPodAutoscaler. |

## Advanced Configuration

### Custom Resource Limits

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

### Node Selection

```yaml
nodeSelector:
  disk: fast

tolerations:
  - key: "workload"
    operator: "Equal"
    value: "media"
    effect: "NoSchedule"
```

### Auto-Scaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

## Test

Verify the deployment:

```console
helm test alist
```

## Uninstall

Remove the release:

```console
helm uninstall alist
```

**Note:** PVCs may remain depending on cluster reclaim policy and chart values. Manually delete if needed:

```console
kubectl delete pvc alist
```

## Troubleshooting

### Check Pod Status

```console
kubectl get pods -l app.kubernetes.io/name=alist
kubectl logs -l app.kubernetes.io/name=alist
```

### View Configuration

```console
kubectl exec -it <pod-name> -- cat /opt/alist/data/config.json
```

### Reset Admin Password

```console
kubectl exec -it <pod-name> -- ./alist admin random
```

## Support

- [AList Documentation](https://alist.nn.ci/)
- [AList GitHub](https://github.com/AlistGo/alist)
- [Helm Chart Issues](https://github.com/phenix3443/charts/issues)
