# Curvine Runtime Helm Chart

Helm chart for deploying a Curvine runtime cluster on Kubernetes.

## Naming

- Source directory: `curvine-runtime/`
- Published chart name: `curvine`
- Recommended release name: `curvine`
- Recommended namespace: `curvine`

When installing from a Helm repository, use `curvineio/curvine`. When installing from this repository, use `./curvine-runtime`.

## Versioning and Images

This chart leaves `values.image.tag` empty by default. Templates resolve the effective image tag
from `Chart.appVersion`:

| `Chart.appVersion` | Default image tag |
| --- | --- |
| `0.3.0` | `v0.3.0` |
| `latest` | `latest` |

On versioned releases, `Chart.version` and `Chart.appVersion` match. On `main` branch test
packages, `Chart.version` is `<base>-dev` while `appVersion` is `latest`.

See the repository [README](../README.md#versioning-model) for the full version mapping and
release workflow.

Confirm these prerequisites first:

- Kubernetes 1.20+
- Helm 3.x
- One of these storage options:
  - a default `StorageClass`
  - explicit `storageClass` values in your values file
  - `hostPath` storage on bare metal
- Enough cluster capacity for the requested CPU and memory
- If you use the production or bare-metal examples:
  - node labels already exist
  - required taints are tolerated
  - privileged Pods and `hostNetwork` are allowed when applicable

### OpenKruise (optional)

By default `openKruise.enabled=false`. The chart uses standard `apps/v1` StatefulSets and does not install OpenKruise.

Set `openKruise.enabled=true` when you need:

- Advanced StatefulSet features (in-place image updates)
- PersistentPodState topology pinning for master pods

Enabling OpenKruise installs the `kruise` subchart (v1.9.0) into `kruise-system` as part of the same `helm upgrade --install` command. Override subchart settings under the top-level `kruise:` values key.

If OpenKruise is already installed cluster-wide, set `openKruise.enabled=true` for Curvine Kruise API usage and consider `kruise.crds.managed=false` to avoid reinstalling CRDs.

The chart defaults are intentionally conservative so a first deployment is less likely to end in `Pending`.
The chart also includes `values.schema.json` so invalid values can be rejected before rendering.

## Install

### From The Public Helm Repository

```bash
helm repo add curvineio https://curvineio.github.io/curvine-doc/helm-charts
helm repo update

helm upgrade --install curvine curvineio/curvine \
  -n curvine \
  --create-namespace
```

### From Local Source

```bash
helm upgrade --install curvine ./curvine-runtime \
  -n curvine \
  --create-namespace
```

### Bootstrap Empty Master Storage

`cluster.formatMaster` defaults to `false` to protect existing master metadata
and Raft journals. Empty master storage needs an explicit bootstrap:

```bash
helm upgrade --install curvine ./curvine-runtime \
  -n curvine \
  --create-namespace \
  --set cluster.formatMaster=true

kubectl rollout status statefulset/curvine-master -n curvine

helm upgrade --install curvine ./curvine-runtime \
  -n curvine \
  --set cluster.formatMaster=false
```

Do this only for a new cluster or a rebuild where data loss is acceptable. For
an existing HA master, never clear one ordinal and restart it empty with
`cluster.formatMaster=false`; copy a consistent meta and journal snapshot from a
healthy master or rebuild the whole cluster.

### With Example Values

Development:

```bash
helm upgrade --install curvine ./curvine-runtime \
  -n curvine \
  --create-namespace \
  -f ./curvine-runtime/examples/values-dev.yaml
```

Production:

```bash
helm upgrade --install curvine ./curvine-runtime \
  -n curvine \
  --create-namespace \
  -f ./curvine-runtime/examples/values-prod.yaml
```

Bare metal:

```bash
helm upgrade --install curvine ./curvine-runtime \
  -n curvine \
  --create-namespace \
  -f ./curvine-runtime/examples/values-baremetal.yaml
```

When Master metadata or journal storage uses `hostPath`, the chart treats the
data as node-local. For multi-Master deployments, it adds required Pod
anti-affinity so each Master ordinal stays on a distinct node and two Master
pods cannot share the same node-local RocksDB paths.

To pin master pods to their original nodes across restarts, set
`openKruise.enabled=true` and configure `master.persistentTopology` (see the
bare-metal example).

Master startup is intentionally protected by `master.startupProbe`. A Master
does not open the RPC port until Raft snapshot restore and metadata tree rebuild
finish. In production, a 700 MiB checkpoint has taken about 4 minutes to rebuild,
so short liveness-only probing can kill a healthy restore loop before the process
can become ready.

Do not use the production or bare-metal examples unchanged on a generic cluster. Edit storage classes, labels, and paths first.

## Verify

Run these commands immediately after install or upgrade:

```bash
helm status curvine -n curvine
kubectl get statefulset,pod,pvc -n curvine
kubectl get events -n curvine --sort-by='.lastTimestamp'
```

Access the master web UI:

```bash
kubectl port-forward -n curvine svc/curvine-master 9000:9000
```

## Standard Operator Workflows

### Upgrade In Place

```bash
helm upgrade --install curvine curvineio/curvine \
  -n curvine \
  -f values.yaml
```

Use this flow for:

- image changes
- resource tuning
- worker replica changes
- config changes

Do not change `master.replicas` during an in-place upgrade.

### Migrating from implicit OpenKruise usage

Older chart versions rendered master StatefulSets as `apps.kruise.io/v1beta1` when
using `hostPath` storage with `master.persistentTopology.enabled=true`, even if
`openKruise.enabled=false`. Current versions only use the Kruise API when
`openKruise.enabled=true`.

If you upgraded from that behavior:

- Set `openKruise.enabled=true` before upgrading to keep Advanced StatefulSet semantics, or
- Accept that the master StatefulSet apiVersion may change to `apps/v1` and plan accordingly

### Roll Back

```bash
helm history curvine -n curvine
helm rollback curvine <revision> -n curvine
```

### Re-Deploy While Preserving Data

This path keeps StatefulSet PVCs:

```bash
helm uninstall curvine -n curvine

helm upgrade --install curvine curvineio/curvine \
  -n curvine \
  --create-namespace \
  -f values.yaml
```

Rules:

- reuse the same release name
- reuse the same namespace
- do not delete PVCs

If old PVCs were already unresolved, the re-deployed Pods may remain `Pending` for the same reason.

### Destroy And Rebuild

This path deletes data:

```bash
helm uninstall curvine -n curvine
kubectl delete pvc -n curvine -l app.kubernetes.io/instance=curvine
kubectl delete namespace curvine
```

Use this only when data loss is acceptable.

## Values And Examples

Inspect current defaults:

```bash
helm show values curvineio/curvine
```

Or locally:

```bash
sed -n '1,220p' ./curvine-runtime/values.yaml
```

Included example files:

- `examples/values-dev.yaml`
- `examples/values-prod.yaml`
- `examples/values-baremetal.yaml`

## Troubleshooting

If a Pod is `Pending`, start with:

```bash
kubectl describe pod <pod-name> -n curvine
kubectl describe pvc <pvc-name> -n curvine
kubectl get storageclass
kubectl get nodes --show-labels
kubectl get events -n curvine --sort-by='.lastTimestamp'
```

## Important Behavior Notes

- `helm uninstall` removes workloads but leaves StatefulSet PVCs behind
- `master.replicas` must stay stable across in-place upgrades
- Logs are often not useful for `Pending` Pods because containers may never start

## Support

- Curvine project: https://github.com/CurvineIO/curvine
- Helm repo index: https://curvineio.github.io/curvine-doc/helm-charts/index.yaml
