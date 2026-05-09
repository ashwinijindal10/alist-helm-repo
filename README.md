# AList Helm Chart

This chart installs [AList](https://alist.nn.ci/), a file list and WebDAV program that supports multiple storage backends.

## Versions

| Component | Version |
| --- | --- |
| Chart | `0.1.7` |
| AList app | `v3.60.0` |
| Default image | `xhofe/alist:v3.60.0` |

## Repository

```console
helm repo add alist https://ashwinijindal10.github.io/alist-helm-repo/charts
helm repo update
```

## Quick Start

Install with default settings (persistence enabled):

```console
helm install alist alist/alist
```

Install from local chart checkout:

```console
helm install alist .
```

## Upgrade

```console
helm upgrade alist alist/alist
```

Override specific values:

```console
helm upgrade --install alist alist/alist \
  --set image.tag=v3.60.0 \
  --set persistence.size=20Gi
```

## Persistence

**Persistence is enabled by default** to ensure AList configuration and runtime data survive pod restarts.

Disable persistence only for temporary testing:

```console
helm install alist alist/alist \
  --set persistence.enabled=false
```

**WARNING:** If persistence is disabled and `inMemory.enabled=true`, all data is lost on pod restart.

Use a custom PVC:

```console
helm install alist alist/alist \
  --set persistence.existingClaim=my-alist-data
```

## Image Variants

AList provides image variants with pre-installed tools:

- `v3.60.0` (default) — Standard image
- `v3.60.0-aio` — All-in-one with ffmpeg + aria2
- `v3.60.0-ffmpeg` — With ffmpeg for thumbnail generation (local storage only)
- `v3.60.0-aria2` — With aria2 for offline downloads
- `latest` — Latest stable version
- `beta` — Development version

Use the all-in-one variant:

```console
helm install alist alist/alist \
  --set image.tag=v3.60.0-aio
```

Use ffmpeg for thumbnail generation:

```console
helm install alist alist/alist \
  --set image.tag=v3.60.0-ffmpeg
```

## Environment Variables

Configure AList runtime behavior with environment variables:

```yaml
env:
  PUID: "1000"          # Container process UID (default: 0 = root)
  PGID: "1000"          # Container process GID (default: 0 = root)
  UMASK: "022"          # File creation mask (default: 022)
  RUN_ARIA2: "true"     # Enable aria2 daemon (requires -aria2 image)
  TZ: "Asia/Shanghai"   # Timezone (default: UTC)
```

Set via CLI:

```console
helm install alist alist/alist \
  --set env.PUID=1000 \
  --set env.PGID=1000 \
  --set env.TZ=America/New_York
```

## Service & Networking

### HTTP (Default)

HTTP is enabled by default on port 80 (maps to container port 5244):

```console
helm install alist alist/alist
```

### HTTPS (Optional)

Enable optional HTTPS on port 443 (maps to container port 5245):

```console
helm install alist alist/alist \
  --set 'service.ports.https.enabled=true'
```

Via values file:

```yaml
service:
  ports:
    https:
      enabled: true
      port: 443
      targetPort: 5245
```

### Custom Service Type

For LoadBalancer or NodePort:

```console
helm install alist alist/alist \
  --set service.type=LoadBalancer
```

## Ingress

Example nginx ingress with TLS:

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

## Configuration

By default, `createConfigFile.enabled=false`, so AList automatically generates a full `config.json` on first startup in `/opt/alist/data`.

Enable config injection only if you want to pre-configure AList:

```console
helm install alist alist/alist \
  --set createConfigFile.enabled=true
```

### Database Configuration

Example MySQL configuration (requires `createConfigFile.enabled=true`):

```yaml
createConfigFile:
  enabled: true

config:
  site_url: "https://alist.example.com"
  database:
    type: mysql
    host: mysql.default.svc.cluster.local
    port: 3306
    user: alist
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
