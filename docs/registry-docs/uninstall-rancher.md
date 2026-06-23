# Rancher Uninstall

This page will walk you through how to uninstall Carbide Registry Images from Rancher, both for its own components and downstream Rancher Kubernetes clusters (RKE2/K3s).

## Reverting Cert Manager

As Rancher has a dependency on Cert Manager, you'll need to update your Helm install of Cert Manager to use the default images.

### Using Your Own Registry

If using your own registry, you simply need to [collect the necessary images](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/publish-images#2-collect-the-cert-manager-image) for cert-manager and overwrite those images in your registry.

As pods cycle, the new image should propagate across the cluster.

### Using Docker Hub Images

If you want to go back to using Docker Hub images directly, you'll need to upgrade the cert-manager installation to revert pointing to the private registry hosting the Carbide images:

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4
```

## Registry Auth Scenarios
### Global Registry

#### Uninstall Carbide Images on Rancher (Private Registry)

If using your own registry, you simply need to collect the necessary images for Rancher and overwrite those images in your registry.

As pods cycle, the new image should propagate across the cluster.

#### Uninstall Carbide Images on Rancher (Docker Hub)

If you want to switch back to Docker Hub images for Rancher:

1. Log into Rancher and configure the default administrator password.
1. Click **☰ > Global Settings**.
1. Go to the setting called `system-default-registry` and choose **⋮ > Edit Setting**.
1. Click the `Use the default value` button.

**Result:** Rancher will use the default Docker Hub images to pull system images.

#### Reverting Downstream Clusters to use Docker Hub Images

If you want to revert downstream RKE2/K3s clusters to use Docker Hub images:

1. Click **☰ > Cluster Management**.
2. On the **Clusters** page, select the 3-dot menu to the right of the downstream cluster you'd like to revert, and select **Edit Config**.
3. In the **Cluster Configuration** section, go to the **Registries** tab and click **Use default global registry for Rancher System Container Images**.
4. Click **Save**.

**Result:** The cluster will update its node pools to use Docker Hub images.


### Manual `registries.yaml` configuration (`RKE2`/`k3s`)

See the [RKE2/K3s Uninstall](uninstall-kubernetes.md) documentation.

### Reverting `Rancher` Chart

Following Rancher's [Installation Guide](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster), you can revert to using Docker Hub images by removing the Carbide values from the Helm upgrade command.

```bash
helm upgrade rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set replicas=3
```