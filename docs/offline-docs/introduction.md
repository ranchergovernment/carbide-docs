# Introduction

This section walks through installing and using Carbide Offline Docs, the airgap-friendly documentation component of Rancher Government Carbide.

:::tip Airgapped Docs have been refreshed and renamed to Offline Docs. For migration instructions, see the [installation page](./installation#migrating-from-airgapped-docs). 
:::

## What's included?

In an airgap, access to documentation can be critical to mission success, especially while troubleshooting problems.

Carbide Offline Docs runs a self-hosted copy of the documentation for Carbide and the supported RGS product portfolio inside your cluster. Each product's docs site is its own container image, so you can deploy only the docs you need. The sites include search and copy/paste shortcuts.

## How it works

The `carbide-offline-docs` Helm chart creates a Deployment and a Service named `<key>-offline-docs` for each docs site you enable. When installed on the Rancher `local` cluster, it also adds a Rancher navigation link for each site, grouped under **Airgap-Friendly Docs**, so users can open the docs straight from the Rancher UI. The default chart values includes all documentation sites.

## Releases

The offline docs images are updated periodically. Each release publishes:

- the docs images, tagged `latest` and `2`, for example `docs-rancher:latest`
- two chart versions, `2.0.0-latest` and `2.0.0-2`, which deploy the matching image tag by default

Periodically pull the refreshed versions of the chart and images to keep documentation up-to-date.
