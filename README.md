# Curvine Helm Charts

Source repository for the Curvine Helm charts.

## Published Helm Repository

Public index:

```bash
helm repo add curvineio https://curvineio.github.io/curvine-doc/helm-charts
helm repo update
```

Published charts:

| Source Directory | Published Chart Name | Purpose |
| --- | --- | --- |
| `curvine-runtime/` | `curvine` | Curvine runtime cluster |
| `curvine-csi/` | `curvine-csi` | Curvine CSI driver |

## Quick Start

Install a minimal runtime cluster:

```bash
helm upgrade --install curvine curvineio/curvine \
  -n curvine \
  --create-namespace
```

Install the CSI driver:

```bash
helm upgrade --install curvine-csi curvineio/curvine-csi \
  -n curvine-system \
  --create-namespace
```

Install the CSI chart only after the Curvine runtime cluster is reachable and you are ready to create a `StorageClass`.

Before you install the runtime chart, make sure one of these is true:

- Your cluster has a default `StorageClass`
- You pass explicit `storageClass` values
- You use a `hostPath`-based example on bare metal

## Documentation

- Runtime chart: [curvine-runtime/README.md](./curvine-runtime/README.md)
- CSI chart: [curvine-csi/README.md](./curvine-csi/README.md)
- Release workflow: [.github/workflows/README.md](./.github/workflows/README.md)

## Repository Layout

```text
curvine-helm/
├── .github/workflows/        # Chart packaging and release workflow
├── curvine-runtime/          # Source for the published `curvine` chart
├── curvine-csi/              # Source for the published `curvine-csi` chart
└── README.md                 # Repository entrypoint
```

## Release Architecture

- This repository stores chart source code only.
- Chart packages are published to GitHub Releases.
- The public Helm index is served from `curvine-doc`.
- End users should install from `https://curvineio.github.io/curvine-doc/helm-charts`.

## Local Development

Lint and package from source directories:

```bash
helm lint ./curvine-runtime
helm lint ./curvine-csi

mkdir -p dist
helm package ./curvine-runtime --destination dist
helm package ./curvine-csi --destination dist
```

Install locally from source:

```bash
helm upgrade --install curvine ./curvine-runtime -n curvine --create-namespace
helm upgrade --install curvine-csi ./curvine-csi -n curvine-system --create-namespace
```

## Links

- Curvine project: https://github.com/CurvineIO/curvine
- Helm documentation: https://helm.sh/docs/
