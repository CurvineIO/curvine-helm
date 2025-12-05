# Curvine Helm Charts Repository

This directory serves as the central storage for all Curvine Helm chart packages.

## Available Charts

- **curvine-csi**: CSI Driver for Curvine storage
- **curvine-runtime**: Curvine distributed multi-level cache system

## Usage

Add the Helm repository:

```bash
helm repo add curvineio https://curvineio.github.io/curvine-helm/charts
helm repo update
```

Search available charts:

```bash
helm search repo curvineio
```

Install charts:

```bash
# Install Curvine Runtime
helm install curvine curvineio/curvine-runtime

# Install Curvine CSI Driver
helm install curvine-csi curvineio/curvine-csi
```

## Architecture

This repository uses a distributed architecture:

- **Chart Packages (.tgz)**: Stored in this directory (`curvine-helm/charts/`)
  - Served via GitHub Pages: `https://curvineio.github.io/curvine-helm/charts/`
  
- **Index File (index.yaml)**: 
  - Generated in this directory
  - Also synced to `curvine-doc/static/helm-charts/index.yaml` for redundancy
  - Served via: `https://curvineio.github.io/curvine-doc/helm-charts/index.yaml`

## Directory Structure

```
charts/
├── README.md                    # This file
├── index.yaml                   # Helm repository index (auto-generated)
├── curvine-csi-*.tgz           # CSI chart packages (all versions)
└── curvine-runtime-*.tgz       # Runtime chart packages (all versions)
```

## Automated Updates

This directory is automatically updated by GitHub Actions when a version tag (v*) is pushed.

**Workflow:**
1. Developer pushes a version tag (e.g., `v0.1.0`)
2. GitHub Actions builds chart packages
3. Chart packages (.tgz) are stored in this directory
4. `index.yaml` is generated using `helm repo index`
5. `index.yaml` is synced to `curvine-doc` repository
6. Changes are committed with `[skip ci]` to prevent loops

**DO NOT manually edit files in this directory.**

All chart packages and the index.yaml are automatically generated and maintained by the CI/CD pipeline.

## Source Code

Chart source code is located in the repository root:
- `curvine-csi/` - CSI Driver chart source
- `curvine-runtime/` - Runtime chart source

## More Information

- [Curvine Documentation](https://github.com/CurvineIO/curvine)
- [Helm Documentation](https://helm.sh/docs/)

