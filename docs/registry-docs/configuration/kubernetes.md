# RKE2/k3s Configuration

This page walks you through configuring the Rancher Kubernetes distributions (RKE2 and k3s) to use Carbide Secured Registry images instead of the upstream Docker Hub images.

## Usage with k3s

To change the system images that `k3s` pulls at bootstrap, configure k3s' mirror settings as described in the [k3s private registry docs](https://docs.k3s.io/installation/private-registry#mirrors).

Set the mirrors in `/etc/rancher/k3s/registries.yaml`. A full example using an authenticated registry is below:

```yaml
mirrors:
  docker.io:
    endpoint:
      - "https://<registry-url>"

configs:
  "<registry-url>":
    auth:
      username: <redacted>
      password: <redacted>
```

You will also need to utilize the `system-default-registry` flag when installing k3s to ensure it uses the registry. For instance:

```bash
curl -sfL https://get.k3s.io | sh -s - --system-default-registry=<registry-url>
```

## Usage with RKE2

As with k3s, RKE2 pulls Carbide Secured Registry (CSR) images at bootstrap once you configure its mirroring, as described in the [RKE2 private registry docs](https://docs.rke2.io/install/private_registry#mirrors).

Set the mirrors in `/etc/rancher/rke2/registries.yaml`. A full example using an authenticated registry is below:

```yaml
mirrors:
  docker.io:
    endpoint:
      - "https://<registry-url>"

configs:
  "<registry-url>":
    auth:
      username: <redacted>
      password: <redacted>
```

You'll also need to set the `system-default-registry` flag when installing RKE2 so it uses the registry. For example, in the configuration file `/etc/rancher/rke2/config.yaml`:

```bash
node-name: controlplane1
write-kubeconfig-mode: 0640
system-default-registry: <registry-url>
...
```

### `registries.yaml` Strategy Approaches

| Scenario                        | Best practice                                                                     |
| ------------------------------- | --------------------------------------------------------------------------------- |
| Use of a 'golden machine image' | Pre-configure `registries.yaml` on the golden machine image before host provisioning |
| Rancher-provisioned cluster     | Embed a `cloud-init` file into cluster provisioning (example below)                |
| Ansible/Saltstack/Manual        | Pre-configure `registries.yaml` on the host before cluster provisioning           |

### Example `cloud-init` (RKE2)

```yaml
runcmd:
  - mkdir -p /etc/rancher/rke2
write_files:
  - path: /etc/rancher/rke2/registries.yaml
    permissions: '0644'
    content: |
      mirrors:
        docker.io:
          endpoint:
            - "https://<registry-url>"

      configs:
        "<registry-url>":
          auth:
            username: <redacted>
            password: <redacted>
```