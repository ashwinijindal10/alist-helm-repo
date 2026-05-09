# Alist Helm Chart

- Installs the storage manager system [Alist](https://alist.nn.ci/)。

## Get Repo Info

```console
helm repo add panghuli https://charts.panghuli.cn
helm repo update
```

## Installing the Chart

To install the chart with the release name `my-release`:

```console
helm install my-release panghuli/alist
```

## Uninstalling the Chart

To uninstall/delete the my-release deployment:

```console
helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.
# alist-helm-repo
