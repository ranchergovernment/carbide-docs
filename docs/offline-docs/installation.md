# Installation

Carbide Offline Docs ships as a Helm chart, `carbide-offline-docs`, plus one container image per docs site. Both are published to the Carbide as part of the [Offline Docs product](https://portal.ranchercarbide.dev/product/offline-docs):

- **Chart:** `oci://registry.ranchercarbide.dev/carbide/carbide-offline-docs`
- **Images:** `registry.ranchercarbide.dev/carbide/docs-<key>:<tag>`, for example `docs-rancher:latest`

To install, copy the chart and images into your registry, then install the chart on the Rancher `local` cluster.

## Copy the Chart and Images to Your Registry

### In Connected Environment

Log in to the CSR:

```bash
hauler login -u <username> -p <password> registry.ranchercarbide.dev
```

Fetch the `offline-docs` product into your Hauler store. The product includes the `carbide-offline-docs` Helm chart and every docs image:

```bash
hauler store sync --products offline-docs=latest --platform linux/amd64 --product-registry registry.ranchercarbide.dev
```

The images are built for `linux/amd64` and `linux/arm64`. Set `--platform` to match your cluster nodes.

If your registry can reach the CSR, copy the chart and images directly to your registry, then skip to [Install the Chart](#install-the-chart):

```bash
hauler store copy --username <redacted> --password <redacted> registry://<registry-url>
```

Otherwise, save the Hauler store to a tarball:

```bash
hauler store save --filename offline-docs.tar.zst
```

Move `offline-docs.tar.zst` into your airgapped environment.

### In Airgapped Environment

Load the content from the tarball into the Hauler store:

```bash
hauler store load offline-docs.tar.zst
```

Copy the chart and images to your registry:

```bash
hauler store copy --username <redacted> --password <redacted> registry://<registry-url>
```

The chart and images keep their `carbide/` path, so the chart lands at `oci://<registry-url>/carbide/carbide-offline-docs` and the images at `<registry-url>/carbide/docs-<key>:latest`. Run `hauler store info` to see the exact references in your store.

## Install the Chart

If your registry requires authentication, log in with Helm first:

```bash
helm registry login <registry-url> -u <redacted> -p <redacted>
```

On the Rancher `local` cluster, run the following Helm command, substituting your registry:

```bash
helm install carbide-offline-docs -n carbide-docs-system --create-namespace \
  --set "image.registry=<registry-url>" \
  --version 2.0.0-latest \
  oci://<registry-url>/carbide/carbide-offline-docs
```

Check the status of the rollout:

```bash
helm status carbide-offline-docs -n carbide-docs-system
kubectl get pods -n carbide-docs-system
```

In the Rancher UI, explore the `local` cluster. The docs sites appear in the left menu under **Airgap-Friendly Docs**, and each one opens in a new tab.

:::note
The Rancher menu links are `NavLink` resources, which need the `ui.cattle.io` CRDs that Rancher installs on the `local` cluster. To install the chart on a cluster without those CRDs, add `--set "navLinks.enabled=false"`, then reach the docs through the `<key>-offline-docs` Services, for example:

```bash
kubectl port-forward -n carbide-docs-system svc/rancher-offline-docs 8080:80
```
:::

## Selecting Documentation

There may be situations where you are running in a low-compute, low-resource environment or are not using specific products and do not want to deploy all of the docs sites. For those situations, you can disable specific products during installation, such as:

```bash
# Disable NeuVector docs
helm install -n carbide-docs-system --create-namespace \
  --set "image.registry=<registry-url>" \
  --set "docs.neuvector.enabled=false" \
  --version 2.0.0-latest \
  carbide-offline-docs oci://<registry-url>/carbide/carbide-offline-docs
```

To deploy only a few sites, use a values file that disables the rest. This example lists every docs site key:

```yaml title="offline-docs-values.yaml"
image:
  registry: <registry-url>

docs:
  rancher:
    enabled: true
  rke2:
    enabled: true
  carbide:
    enabled: true
  k3s:
    enabled: false
  longhorn:
    enabled: false
  neuvector:
    enabled: false
  harvester:
    enabled: false
  elemental:
    enabled: false
  kubewarden:
    enabled: false
  fleet:
    enabled: false
  turtles:
    enabled: false
  k3k:
    enabled: false
  hauler:
    enabled: false
  rancher-desktop:
    enabled: false
  kubernetes:
    enabled: false
```

```bash
helm install -n carbide-docs-system --create-namespace \
  -f offline-docs-values.yaml \
  --version 2.0.0-latest \
  carbide-offline-docs oci://<registry-url>/carbide/carbide-offline-docs
```

## Configuration

These are the most common chart values:

| Value | Default | Purpose |
| --- | --- | --- |
| `image.registry` | `registry.ranchercarbide.dev` | Registry the docs images are pulled from. |
| `image.project` | `carbide` | Project or path the docs images live under. |
| `image.tag` | `""` | Docs image tag. When empty, uses the chart's `appVersion`, for example `latest` for chart `2.0.0-latest`. |
| `image.pullPolicy` | `Always` | Image pull policy for every docs site. |
| `imagePullSecrets` | `[]` | Pull secrets for registries that need authentication, for example `[{name: my-registry-secret}]`. |
| `replicas` | `1` | Replicas per docs site. |
| `resources` | `10m` CPU / `32Mi` memory requests, `128Mi` memory limit | Resources per docs site. |
| `navLinks.enabled` | `true` | Adds a Rancher menu link for each docs site. |
| `navLinks.group` | `Airgap-Friendly Docs` | Rancher menu group the links appear under. |
| `docs.<key>.enabled` | `true` | Deploys that docs site. |
| `docs.<key>.label`, `.description`, `.sideLabel` | varies | Text shown on the docs site's Rancher menu link. |
| `docs.<key>.image` | unset | Full image reference for that docs site, overriding `image.registry`, `image.project` and `image.tag`. |

## Upgrading

Refreshed docs images are published under the same `latest` tag. To pick up the latest content, re-run the steps in [Copy the Chart and Images to Your Registry](#copy-the-chart-and-images-to-your-registry), then restart the docs sites so they pull the new images:

```bash
kubectl rollout restart deployment -n carbide-docs-system
```

When a new chart version is available, copy it into your registry and upgrade:

```bash
helm upgrade -n carbide-docs-system \
  --reuse-values \
  --version <chart-version> \
  carbide-offline-docs oci://<registry-url>/carbide/carbide-offline-docs
```

## Migrating from Airgapped Docs

The `carbide-offline-docs` chart replaces the older `carbide-charts/airgapped-docs` chart. The new chart uses `image.registry` instead of `global.cattle.systemDefaultRegistry`, and is installed on the `local` cluster. To migrate, uninstall the old chart, then follow the steps above:

```bash
helm uninstall -n carbide-docs-system airgapped-docs
```
