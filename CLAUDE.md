# CLAUDE.md -- Helm Charts

## What this repo is

A collection of Helm charts published as OCI artifacts to GitHub Container Registry (`ghcr.io/mabadiliko/helm-charts/<chart>`). Currently contains one chart: `mediawiki`.

## Repo structure

```
charts/
  mediawiki/           # The MediaWiki Helm chart
    Chart.yaml
    values.yaml        # Default values with all configurable parameters
    templates/
      _helpers.tpl              # Reusable template helpers (fullname, labels, etc.)
      configmap-localsettings.yaml  # Generates LocalSettings.php from values
      deployment.yaml           # MediaWiki Deployment
      service.yaml              # ClusterIP Service
      ingress.yaml              # Optional Ingress with TLS
      pvc.yaml                  # PVC for uploads (when objectStore.type=pvc)
      job-update.yaml           # Helm hook Job for maintenance/update.php
      serviceaccount.yaml       # ServiceAccount
      NOTES.txt                 # Post-install notes
.github/
  workflows/
    lint-charts.yaml     # PR lint via chart-testing
    release-charts.yaml  # Auto-package and push to GHCR on merge to main
ct.yaml                  # chart-testing config
```

## MediaWiki chart architecture

The chart deploys MediaWiki as a stateless pod. All state lives externally:

- **Database**: External MySQL/MariaDB. Password mounted from a Kubernetes Secret at `/etc/mediawiki/secrets/db/password` and read via `file_get_contents()`.
- **Uploads**: PVC (default), MinIO/S3, or Azure Blob Storage -- selected via `objectStore.type`.
- **Configuration**: `LocalSettings.php` is generated from a ConfigMap template. The `localSettings.extraPhp` field allows injecting arbitrary PHP (extension loading, skin config, OIDC setup, etc.).
- **OIDC**: Optional secret mount at `/etc/mediawiki/secrets/oidc/clientSecret`. Activated by setting `oidc.existingSecret`.

### Key design decisions

- **Secrets are mounted as files, not environment variables.** The DB password, OIDC client secret, and storage credentials are all mounted from Kubernetes Secrets and read via `file_get_contents()` or projected volumes. They never appear in ConfigMaps or environment variable lists.
- **The update Job is a Helm hook** with `before-hook-creation` delete policy, so it gets recreated on every `helm upgrade` without manual cleanup.
- **No `runAsNonRoot`**: The official MediaWiki image requires starting as root (Apache binds port 80, then drops to www-data). Setting `runAsNonRoot: true` or `runAsUser: 65534` will crash Apache.
- **`extraPhp` is the primary extension point.** Rather than templating every possible MediaWiki setting, the chart provides a freeform PHP block for callers to configure extensions, skins, and any `$wg*` variables.

## Release process

Push to `main` with changes under `charts/` triggers the GitHub Action which:
1. Packages each chart with `helm package`
2. Pushes to `oci://ghcr.io/mabadiliko/helm-charts/<chart-name>`

Consumers install with:
```bash
helm install my-wiki oci://ghcr.io/mabadiliko/helm-charts/mediawiki --version 0.1.0
```

Bump `version` in `Chart.yaml` to release a new version.

## Related repos

- **MediaWiki image** (public): Builds the custom Docker image with bundled extensions
- **ScoutWiki config** (private): Instance-specific values, secret references, and OIDC configuration for the Trollbäckens ScoutWiki deployment
