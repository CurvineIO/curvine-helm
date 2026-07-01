# Curvine CSI Helm Chart

Helm chart for deploying the Curvine CSI driver.

## Naming

- Source directory: `curvine-csi/`
- Published chart name: `curvine-csi`
- Recommended release name: `curvine-csi`
- Recommended namespace: `curvine-system`

## Versioning and Images

This chart leaves `values.image.tag` empty by default. Templates resolve the effective image tag
from `Chart.appVersion`:

| `Chart.appVersion` | Default image tag |
| --- | --- |
| `0.3.0` | `v0.3.0` |
| `latest` | `latest` |

On versioned releases, `Chart.version` and `Chart.appVersion` match. On `main` branch test
packages, `Chart.version` is `0.0.0-dev` while `appVersion` is `latest`.

See the repository [README](../README.md#versioning-model) for the full version mapping and
release workflow.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.x
- A reachable Curvine runtime cluster
- Privileged node Pods and required kubelet host paths must be allowed

The chart includes `values.schema.json` so invalid values can be rejected before rendering.

## Install

### From The Public Helm Repository

```bash
helm repo add curvineio https://curvineio.github.io/curvine-doc/helm-charts
helm repo update

helm upgrade --install curvine-csi curvineio/curvine-csi \
  --namespace curvine-system \
  --create-namespace
```

### From Local Source

```bash
helm upgrade --install curvine-csi ./curvine-csi \
  --namespace curvine-system \
  --create-namespace
```

## Configuration Notes

- Service account names are derived from the release name by default.
- You can override them with:
  - `serviceAccount.controller.name`
  - `serviceAccount.node.name`
- Embedded mount mode is the default through `node.mountMode=embedded`.

## Core Values

| Key | Description | Default |
| --- | --- | --- |
| `image.repository` | CSI image repository | `ghcr.io/curvineio/curvine-csi` |
| `image.tag` | CSI image tag (empty uses `latest` when `Chart.AppVersion` is `latest`, otherwise `v{Chart.AppVersion}`) | `""` |
| `csiDriver.name` | CSI driver name | `curvine` |
| `controller.replicas` | Controller replica count | `1` |
| `node.mountMode` | FUSE mount strategy | `embedded` |
| `node.priorityClassName` | Node DaemonSet priority class | `system-node-critical` |
| `rbac.create` | Create service accounts, cluster roles, and bindings | `true` |

For the full values surface, inspect:

```bash
helm show values curvineio/curvine-csi
```

## StorageClass Example

Create a StorageClass after the runtime cluster is available.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: curvine-sc
provisioner: curvine
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
parameters:
  master-addrs: "curvine-master-0.curvine-master.curvine.svc.cluster.local:8995"
  fs-path: "/data"
  path-type: "DirectoryOrCreate"
```

If your runtime cluster has multiple masters, add them as a comma-separated list in `master-addrs`.

## PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: curvine-sc
```

## Verify

```bash
kubectl get csidriver curvine
kubectl get deployment -n curvine-system -l app.kubernetes.io/instance=curvine-csi,app.kubernetes.io/component=controller
kubectl get daemonset -n curvine-system -l app.kubernetes.io/instance=curvine-csi,app.kubernetes.io/component=node
kubectl get pods -n curvine-system -l app.kubernetes.io/instance=curvine-csi
```

To inspect RBAC and service accounts created by the chart:

```bash
kubectl get sa -n curvine-system -l app.kubernetes.io/instance=curvine-csi
kubectl get clusterrole,clusterrolebinding -l app.kubernetes.io/instance=curvine-csi
```

## Logs

```bash
kubectl logs -n curvine-system -l app=curvine-csi-controller -c csi-plugin
kubectl logs -n curvine-system -l app=curvine-csi-controller -c csi-provisioner
kubectl logs -n curvine-system -l app=curvine-csi-node -c csi-plugin
kubectl logs -n curvine-system -l app=curvine-csi-node -c node-driver-registrar
```

## Uninstall

```bash
helm uninstall curvine-csi --namespace curvine-system
```

Delete the namespace only if no other resources still depend on it.

## Support

- Curvine project: https://github.com/CurvineIO/curvine
- Helm repo index: https://curvineio.github.io/curvine-doc/helm-charts/index.yaml
