# Getting Started

## Using the Carbide Secured Registry (CSR)

As with the other products in the Carbide suite, you can pull application collection artifacts from the Carbide Secured Registry to seed your private registry. This works in both connected and airgapped environments.

Make sure you've completed the [Prerequisites](/docs/registry-docs/prereqs.md), then follow the [Seeding a Registry](/docs/registry-docs/copying-images.md) steps.

:::note
With the `hauler store sync` commands below, Hauler automatically pulls both the Helm chart and the images required for the application. The `hauler store copy` command places Helm charts into a `/charts` folder and container images into `/containers`.
:::

### Copy Images in a Connected Environment using Hauler

Sync app:

```bash
hauler store sync --store application-store --products apps-<application-name>=0.28.1 --key carbide-key.pub --platform <platform/arch>
```

Copy to registry:

```bash
hauler store copy --store application-store --username <redacted> --password <redacted> registry://<registry-url>
```

### Transfer Images to an Airgapped Environment using Hauler

In your connected environment, download the artifacts:

```bash
hauler store sync --store application-store --products apps-<application-name>=0.28.1 --key carbide-key.pub --platform <platform/arch>

hauler store save --store application-store --filename application.tar.zst
```

If you plan to load images directly into `containerd`, set the `--containerd` flag when you save the tarball to ensure compatibility:

:::note
The `--containerd` flag is available in Hauler v1.4.1+.
:::

```bash
hauler store save --store application-store application.tar.zst --containerd
```

In the airgap, load images and copy to your registry:

```bash
hauler store load --store application-store application.tar.zst

hauler store copy --store application-store --username <redacted> --password <redacted> registry://<registry-url>
```