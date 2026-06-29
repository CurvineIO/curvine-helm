# Helm Chart Release Workflow

This repository publishes chart packages from source directories and serves the public Helm index from `curvine-doc`.

## Published Charts

| Source Directory | Published Chart Name | Package Pattern |
| --- | --- | --- |
| `curvine-runtime/` | `curvine` | `curvine-<version>.tgz` |
| `curvine-csi/` | `curvine-csi` | `curvine-csi-<version>.tgz` |

## Architecture

- Chart source code lives in this repository.
- Packaged charts are uploaded to GitHub Releases.
- The public Helm index lives at `https://curvineio.github.io/curvine-doc/helm-charts/index.yaml`.
- The workflow does not commit packaged charts or `index.yaml` back into this repository.

## Triggers

### Push to `main`

- Packages both charts with a fixed `0.0.0-dev` chart version
- Updates the GitHub Release named `latest`
- Does not sync `index.yaml` to `curvine-doc`

Use this for testing only.

### Push a `v*` tag

- Packages both charts with the tag version
- Creates or updates the matching GitHub Release
- Does not sync `index.yaml` automatically

### Manual trigger

Inputs:

- `ref`: branch or tag to build
- `sync_to_doc`: whether to sync the public `index.yaml` to `curvine-doc` for a version tag build

Behavior:

- A tag ref such as `v0.2.0` packages charts with version `0.2.0`
- A branch ref packages charts as `0.0.0-dev` and updates the `latest` test release
- `sync_to_doc=true` is only valid when `ref` is a `v*` tag; the workflow fails fast for branch refs
- When `sync_to_doc` is enabled for a tag build, the workflow rebuilds `index.yaml` with release URLs and pushes it to `curvine-doc`

## Versioning Rules

| Trigger | Chart Version | App Version | Default Image Tag |
| --- | --- | --- | --- |
| `main` push | `0.0.0-dev` | `latest` | `latest` |
| `v*` tag push | `<tag without v>` | `<tag without v>` | `v<tag without v>` |
| Manual branch build | `0.0.0-dev` | `latest` | `latest` |
| Manual tag build | `<tag without v>` | `<tag without v>` | `v<tag without v>` |

`values.yaml` leaves `image.tag` empty by default. Templates resolve it to `latest` when
`Chart.AppVersion` is `latest`, otherwise `v{Chart.AppVersion}`.

## Operator Installation

End users should install from the public Helm repository:

```bash
helm repo add curvineio https://curvineio.github.io/curvine-doc/helm-charts
helm repo update

helm upgrade --install curvine curvineio/curvine
helm upgrade --install curvine-csi curvineio/curvine-csi
```

## Release Checklist

1. Update chart source and documentation.
2. Push to `main` if you need a test package in the `latest` release.
3. Create and push a `v*` tag for a versioned release.
4. Manually run the workflow with `ref=<v-tag>` and `sync_to_doc=true` when you are ready to publish the updated index.

## Notes

- The runtime source directory is `curvine-runtime/`, but the published chart name is `curvine`.
- The public repository URL should always point to `curvine-doc`.
- GitHub Releases are the source of truth for chart package files.
- Manual branch builds are for release validation only and must not be synced into the public Helm index.
