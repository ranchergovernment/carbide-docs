# Prerequisites

In order to utilize Carbide Offline Docs, you will need the following prerequisites:

- **Rancher v2.7.0 or higher:** https://portal.ranchercarbide.dev/product/rancher
- **Helm v3.8.0 or higher:** https://helm.sh/docs/intro/install (needed to install charts from OCI registries)
- **Hauler:** https://docs.hauler.dev/docs/introduction/install, or any tool that can copy OCI images and charts
- Access to the Rancher `local` cluster, to be able to install and deploy the app.
- Credentials for the Carbide Secured Registry (CSR).

## Resource requirements

Each docs site runs as a single lightweight NGINX pod. By default, each pod requests `10m` CPU and `32Mi` memory, with a `128Mi` memory limit. With all 15 docs sites enabled, plan for roughly `150m` CPU and `480Mi` of requested memory. See [Selecting Documentation](installation.md#selecting-documentation) to deploy fewer sites in low-compute environments.
