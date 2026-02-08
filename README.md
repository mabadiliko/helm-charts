# helm-charts

A collection of usable (?) home grown helm charts

## Available Charts

| Chart | Description |
|-------|-------------|
| [mediawiki](charts/mediawiki/) | A Helm chart for deploying MediaWiki |

## Usage

Charts are published as OCI artifacts to GitHub Container Registry.

```bash
helm install my-release oci://ghcr.io/mabadiliko/helm-charts/mediawiki --version 0.1.0
```

## Adding a New Chart

1. Create a new directory under `charts/` with your chart name
2. Add the standard Helm chart files (`Chart.yaml`, `values.yaml`, `templates/`)
3. Push to `main` — the GitHub Action will automatically package and publish it

## Development

Lint charts locally:

```bash
helm lint charts/*
```
