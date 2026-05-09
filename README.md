# AList Helm Chart

This chart installs [AList](https://alist.nn.ci/), a file list and WebDAV
program that supports multiple storage backends.

## Versions

| Component | Version |
| --- | --- |
| Chart | `0.1.6` |
| AList app | `v3.60.0` |
| Default image | `xhofe/alist:v3.60.0` |

## Repository

```console
helm repo add panghuli https://charts.panghuli.cn
helm repo update
```

## Install

Install from the Helm repository:

```console
helm install my-release panghuli/alist
```

Install from this local chart checkout:

```console
helm install my-release .
```

## Upgrade

```console
helm upgrade my-release panghuli/alist
```

To pin or override the AList image tag:

```console
helm upgrade --install my-release panghuli/alist \
  --set image.repository=xhofe/alist \
  --set image.tag=v3.60.0
```

## Persistence

Persistence is disabled by default. Enable it for real deployments so AList
configuration and runtime data survive pod restarts:

```console
helm upgrade --install my-release panghuli/alist \
  --set persistence.enabled=true \
  --set persistence.size=10Gi
```

Use an existing PVC:

```console
helm upgrade --install my-release panghuli/alist \
  --set persistence.enabled=true \
  --set persistence.existingClaim=my-alist-data
```

## Configuration

When `createConfigFile.enabled=true`, the chart renders `.Values.config` into
`config.json` and copies it into AList's data directory before the main
container starts.

Example MySQL configuration:

```yaml
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

If you manage `config.json` yourself through an existing volume or image,
disable generated config:

```console
helm upgrade --install my-release panghuli/alist \
  --set createConfigFile.enabled=false
```

## Ingress

```yaml
ingress:
  enabled: true
  className: nginx
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

## Common Values

| Value | Default | Description |
| --- | --- | --- |
| `replicaCount` | `1` | Number of pods when autoscaling is disabled. |
| `image.repository` | `xhofe/alist` | AList container image repository. |
| `image.tag` | `v3.60.0` | AList image tag. Defaults to chart `appVersion` if empty. |
| `service.type` | `ClusterIP` | Kubernetes Service type. |
| `service.port` | `80` | Service port. |
| `service.targetPort` | `5244` | AList container port. |
| `ingress.enabled` | `false` | Create an Ingress resource. |
| `persistence.enabled` | `false` | Create or use a PVC for `/opt/alist/data`. |
| `persistence.size` | `10Gi` | PVC size when creating a claim. |
| `createConfigFile.enabled` | `true` | Generate and copy `config.json` at startup. |
| `namespaceOverride` | `""` | Render namespaced resources into another namespace. |
| `autoscaling.enabled` | `false` | Create a HorizontalPodAutoscaler. |

## Test

```console
helm test my-release
```

## Uninstall

```console
helm uninstall my-release
```

The uninstall command removes Kubernetes resources managed by the release. PVCs
may remain depending on cluster reclaim policy and chart values.
