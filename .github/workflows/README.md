# Helm Chart Release Workflow

This document explains how to use the automated Helm chart release workflow.

## Overview

The workflow supports two modes:
1. **Automatic trigger** (push events) - Updates curvine-helm repository only
2. **Manual trigger** (workflow_dispatch) - Optional sync to curvine-doc repository

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│ curvine-helm Repository                                     │
│                                                              │
│ charts/                                                      │
│ ├── index.yaml          ← Always updated                    │
│ ├── curvine-csi-*.tgz   ← All versions stored here         │
│ └── curvine-runtime-*.tgz                                   │
│                                                              │
│ Served via: https://curvineio.github.io/curvine-helm/charts│
└─────────────────────────────────────────────────────────────┘
                            │
                            │ Optional sync (manual control)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│ curvine-doc Repository                                      │
│                                                              │
│ static/helm-charts/                                         │
│ └── index.yaml          ← Synced only when requested        │
│                                                              │
│ Served via: https://curvineio.github.io/curvine-doc/...    │
└─────────────────────────────────────────────────────────────┘
```

## Usage

### 1. Testing (Push to main branch)

**Purpose**: Quick testing without version pollution

```bash
git add .
git commit -m "test: some changes"
git push origin main
```

**Results**:
- ✅ Builds charts with fixed version `x.x.x-dev`
- ✅ Updates `curvine-helm/charts/` with packages and index.yaml
- ✅ Overwrites GitHub Release "latest"
- ❌ Does NOT sync to curvine-doc (testing only)

**Version behavior**:
- Always uses the same version (e.g., `0.0.0-dev`)
- Each push overwrites the previous dev release
- Keeps version history clean

### 2. Manual Build (with optional sync)

**Purpose**: Build any branch/tag and optionally sync to curvine-doc

**Steps**:
1. Go to GitHub Actions page
2. Select "Build and Release Helm Chart"
3. Click "Run workflow"
4. Fill in the form:
   - **ref**: Branch or tag name (e.g., `main`, `v0.1.0`)
   - **sync_to_doc**: Check this to sync index.yaml to curvine-doc

**Results**:
- ✅ Builds charts from specified ref
- ✅ Updates `curvine-helm/charts/` (always)
- ✅ Syncs to `curvine-doc` (if checked)

**Use cases**:
- Test a specific branch: ref=`feature-branch`, sync=unchecked
- Update production index: ref=`main`, sync=checked
- Release to both repos: ref=`v0.1.0`, sync=checked

### 3. Version Release (Push tag)

**Purpose**: Official version release

```bash
# Tag the release
git tag v0.1.0
git push origin v0.1.0
```

**Results**:
- ✅ Builds charts with version `0.1.0`
- ✅ Updates `curvine-helm/charts/`
- ✅ Creates new GitHub Release "v0.1.0"
- ❌ Does NOT auto-sync (use manual trigger to sync)

**To sync after tagging**:
```bash
# After tag is pushed, manually trigger workflow
# Go to Actions → Run workflow → Select tag → Check sync_to_doc
```

## Version Strategy

| Trigger | Version Format | Example | Overwrites? |
|---------|----------------|---------|-------------|
| main branch | `{base}-dev` | `0.0.0-dev` | Yes (same version) |
| v* tag | `{tag without v}` | `0.1.0` | No (unique version) |
| Manual (main) | `{base}-dev` | `0.0.0-dev` | Yes (same version) |
| Manual (tag) | `{tag without v}` | `0.1.0` | No (unique version) |

## Sync Control

| Event Type | Auto Sync | Manual Sync | Notes |
|------------|-----------|-------------|-------|
| push (main) | ❌ | ❌ | Testing only, no inputs available |
| push (tag) | ❌ | ✅ | Use manual trigger after push |
| workflow_dispatch | ❌ | ✅ | Check "sync_to_doc" option |

## Best Practices

### For Development

```bash
# Frequent testing (doesn't pollute versions)
git push origin main

# Result: 0.0.0-dev (always the same, gets overwritten)
```

### For Release Preparation

```bash
# 1. Create and push tag
git tag v0.2.0
git push origin v0.2.0

# 2. Verify the release in GitHub
# 3. Manually trigger workflow to sync:
#    - ref: v0.2.0
#    - sync_to_doc: ✅ checked

# Result: Both curvine-helm and curvine-doc are updated
```

### For Emergency Index Update

```bash
# Use manual trigger:
# - ref: main (or any other ref)
# - sync_to_doc: ✅ checked

# This will sync the current index.yaml to curvine-doc
```

## File Locations

### In curvine-helm repository

```
charts/
├── index.yaml                    # Always updated
├── curvine-csi-0.0.0-dev.tgz    # Dev version (overwritten)
├── curvine-csi-0.1.0.tgz        # Release version
├── curvine-csi-0.2.0.tgz        # Release version
├── curvine-runtime-0.0.0-dev.tgz # Dev version (overwritten)
├── curvine-runtime-0.1.0.tgz    # Release version
└── curvine-runtime-0.2.0.tgz    # Release version
```

### In curvine-doc repository

```
static/helm-charts/
└── index.yaml                    # Synced only when manually triggered with sync_to_doc=true
```

## User Installation

End users can install charts from curvine-helm repository:

```bash
# Add repository
helm repo add curvineio https://curvineio.github.io/curvine-helm/charts

# Update repository index
helm repo update

# Search available charts
helm search repo curvineio

# Install a chart
helm install curvine curvineio/curvine-runtime
helm install curvine-csi curvineio/curvine-csi
```

## Troubleshooting

### Q: I pushed a tag but it's not in curvine-doc

**A**: Tag pushes don't auto-sync. Use manual trigger with sync_to_doc checked.

### Q: How to test without creating lots of versions?

**A**: Push to main branch. It always uses the same dev version (e.g., `0.0.0-dev`).

### Q: How to sync latest index.yaml to curvine-doc?

**A**: Use manual trigger with:
- ref: main (or the branch/tag you want)
- sync_to_doc: ✅ checked

### Q: Will frequent pushes to main create version pollution?

**A**: No. Main branch always uses the same fixed dev version (e.g., `0.0.0-dev`), which gets overwritten each time.

## Security

The workflow requires `CURVINE_DOC_TOKEN` secret to push to curvine-doc repository. This token should have:
- `repo` scope (full control of private repositories)

To create the token:
1. GitHub Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. Select `repo` scope
4. Copy and add to curvine-helm repository secrets as `CURVINE_DOC_TOKEN`

