### GHA Cache Server Configuration

Using Documentation from [GHA Cache Server - Getting Started](https://gha-cache-server.falcondev.io/getting-started)

For now, using [hostpath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) storage until additional storage arrives and we can use a [local PersistentVolume](https://kubernetes.io/docs/concepts/storage/volumes/#local) instead.

Apply those in this directory with:

```
kubectl apply -f pv.yaml
```

This will create a pv and pvc on `sr630-node-2`. The GHA actions cache has been configured to also be on the same node to access the storage.

```
helm install gha-actions-cache oci://ghcr.io/falcondev-oss/charts/github-actions-cache-server \
  --namespace actions-cache \
  --create-namespace \
  -f values.yaml
```

Our cache-server is not exposed on the following URL within the cluster:
http://gha-actions-cache-gha-actions-cache-server.actions-cache.svc.cluster.local:3000

Accomplished via a ClusterIP service.


**NOTE**
This is a temporary configuration until we have replicated `local` `PersistentVolumes`.
