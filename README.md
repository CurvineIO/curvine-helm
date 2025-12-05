# Curvine Helm Charts

Official Helm charts for Curvine projects.

## Available Charts

- **curvine-csi**: CSI Driver for Curvine storage
- **curvine-runtime**: Curvine distributed multi-level cache system

## Quick Start

Add the Helm repository:

```bash
helm repo add curvineio https://curvineio.github.io/curvine-doc/helm-charts
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

This repository uses a clean, efficient architecture:

```
┌──────────────────────────────────────────────┐
│ curvine-helm Repository (main branch)        │
│ - Source code only                           │
│ - curvine-csi/ (chart source)               │
│ - curvine-runtime/ (chart source)           │
└───────────────┬──────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────┐
│ GitHub Actions Build & Release               │
│ - Builds charts from source                  │
│ - Creates GitHub Releases                    │
│ - Uploads .tgz packages to Releases          │
└───────────────┬──────────────────────────────┘
                │
                ├────────────┬──────────────────┐
                ▼            ▼                  ▼
    ┌───────────────────┐   ┌────────────┐   ┌────────────┐
    │ GitHub Releases   │   │ curvine-doc│   │ Users      │
    │ Store .tgz files  │   │ index.yaml │   │ Download   │
    └───────────────────┘   └────────────┘   └────────────┘
```

### Key Points

1. **Chart Packages (.tgz)**: Stored in GitHub Releases
   - URL: `https://github.com/CurvineIO/curvine-helm/releases/download/{tag}/`
   - Each release (v0.1.0, v0.2.0, latest, etc.) contains chart packages
   
2. **Index File (index.yaml)**: Stored in curvine-doc repository
   - URL: `https://curvineio.github.io/curvine-doc/helm-charts/index.yaml`
   - Contains metadata for all chart versions
   - URLs point to GitHub Releases for downloads

3. **main Branch**: Clean source code only
   - No build artifacts
   - No gh-pages branch needed

## Development

### Building Charts Locally

```bash
# Lint a chart
cd curvine-csi
helm lint .

# Package a chart
helm package curvine-csi
```

### Automated Releases

Charts are automatically built and published via GitHub Actions:

#### 1. Testing (main branch)

```bash
git push origin main
```

**Result**:
- Builds fixed version `0.0.0-dev` (or similar)
- Updates GitHub Release "latest" (overwrites previous)
- Does NOT sync to curvine-doc by default
- Perfect for testing without polluting version history

#### 2. Release (version tag)

```bash
git tag v0.1.0
git push origin v0.1.0
```

**Result**:
- Builds version `0.1.0`
- Creates new GitHub Release `v0.1.0`
- Does NOT sync to curvine-doc automatically

**To publish to curvine-doc**:
- Use manual workflow trigger
- Select the tag
- Check "Sync to curvine-doc"

#### 3. Manual Trigger

Go to GitHub Actions → "Build and Release Helm Chart" → "Run workflow"

Options:
- **ref**: Branch or tag to build (e.g., `main`, `v0.1.0`)
- **Sync to curvine-doc**: Check this to update the public index

**Use cases**:
- Test a specific branch without syncing
- Update production index after releasing a tag
- Re-sync index if needed

## Version Strategy

| Trigger | Version Format | Example | Overwrites? | Syncs to curvine-doc? |
|---------|----------------|---------|-------------|-----------------------|
| main branch | `{base}-dev` | `0.0.0-dev` | Yes | No (manual only) |
| v* tag | `{tag without v}` | `0.1.0` | No | Manual only |
| Manual | Depends on ref | Varies | Depends | Optional (checkbox) |

## Chart Documentation

- [Curvine CSI Documentation](./curvine-csi/README.md)
- [Curvine Runtime Documentation](./curvine-runtime/README.md)

## Repository Structure

```
curvine-helm/
├── .github/workflows/    # Automated CI/CD
│   └── release.yml
├── curvine-csi/          # CSI Driver chart source
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
├── curvine-runtime/      # Runtime chart source
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
└── README.md             # This file
```

**Note**: No `charts/` directory or `gh-pages` branch. All build artifacts are in GitHub Releases.

## How It Works

1. **Developer pushes code** (tag or main branch)
2. **GitHub Actions builds** charts and creates Release
3. **Manual trigger** (when ready to publish):
   - Downloads current release's .tgz files
   - Downloads existing index.yaml from curvine-doc
   - Generates new index with `--merge` (updates same versions, adds new ones)
   - URLs in index point to GitHub Releases
   - Pushes updated index.yaml to curvine-doc
4. **Users install** from curvine-doc (index) + GitHub Releases (packages)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

See [LICENSE](./LICENSE) file for details.

## Links

- [Curvine Project](https://github.com/CurvineIO/curvine)
- [Helm Documentation](https://helm.sh/docs/)
- [Chart Repository Guide](https://helm.sh/docs/topics/chart_repository/)
