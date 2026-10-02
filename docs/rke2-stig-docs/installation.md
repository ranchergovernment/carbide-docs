import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Installation

:::info
RKE2 STIG currently supports installation only through the [air-gapped method](https://docs.rke2.io/install/airgap). Use this method even when your nodes are in a connected environment.
:::

These steps install an RKE2 STIG server or agent node. They assume you have already provisioned the node and are using the bundled containerd as the container runtime.

- **Server nodes** run the Kubernetes control plane, etcd, and workloads. Every cluster needs at least one server node.
- **Agent nodes** run workloads only and join an existing server.

Steps 1 and 2 are the same for both node types. Step 3 differs depending on whether you are installing a server or an agent. Install the first server node before installing any agent nodes.

Replace `<VERSION>` in the commands below with the RKE2 STIG version you want to install. Available versions are listed on the [Carbide Portal](https://portal.ranchercarbide.dev).

## Step 1: Get the RKE2 STIG Artifacts

Download the RKE2 STIG artifacts on a connected machine using one of the following options.

### Option A: Download from the Carbide Portal

Log in to the [Carbide Portal](https://portal.ranchercarbide.dev), select RKE2 STIG, and download the pre-built Hauler tarball for your version. Then skip to [Step 2](#step-2-load-the-artifacts-on-the-node).

### Option B: Sync with Hauler

This option uses [Hauler](https://docs.hauler.dev/docs/intro). Follow the [Hauler installation instructions](https://docs.hauler.dev/docs/introduction/install) if you do not already have it installed.

1. Download the Carbide public key, which Hauler uses to verify the artifact signatures.

    ```bash
    curl -sfOL https://raw.githubusercontent.com/ranchergovernment/carbide/main/carbide-key.pub
    ```

2. Log in to the Carbide Secured Registry with your Carbide credentials, sync the RKE2 STIG product to your Hauler store, and save the store to a tarball.

<Tabs groupId="registry">
   <TabItem value="harbor" label="Harbor Registry (Standard)" default>
        ```bash
        hauler login registry.ranchercarbide.dev -u <username> -p <password>
        hauler store sync --products rke2-stig=<VERSION> --product-registry registry.ranchercarbide.dev --key carbide-key.pub --platform linux/amd64
        hauler store save --filename rke2-stig.tar.zst
        ```
    </TabItem>
    <TabItem value="ACR" label="Azure Container Registry (Legacy)">
        ```bash
        hauler login rgcrprod.azurecr.us -u <username> -p <password>
        hauler store sync --products rke2-stig=<VERSION> --product-registry rgcrprod.azurecr.us --key carbide-key.pub --platform linux/amd64
        hauler store save --filename rke2-stig.tar.zst
        ```
    </TabItem>
</Tabs>

3. Copy `rke2-stig.tar.zst` and the Hauler binary to the RKE2 STIG node.

## Step 2: Load the Artifacts on the Node

1. On the node, load the tarball into a local Hauler store.

    ```bash
    hauler store load --filename rke2-stig.tar.zst
    ```

2. List the contents of the store to confirm the artifact references.

    ```bash
    hauler store info
    ```

3. Extract the RKE2 STIG binary tarball, checksum file, and install script into a working directory.

    ```bash
    mkdir -p /root/rke2-artifacts && cd /root/rke2-artifacts
    hauler store extract rke2:<VERSION>
    hauler store extract rke2-sha256sum:<VERSION>
    hauler store extract rke2-install:<VERSION>
    ```

4. Save the RKE2 STIG images directly into the containerd images directory.

    ```bash
    mkdir -p /var/lib/rancher/rke2/agent/images/
    hauler store save --containerd --filename /var/lib/rancher/rke2/agent/images/rke2-stig.tar.zst
    ```

## Step 3: Install RKE2 STIG

<Tabs groupId="node-type">
  <TabItem value="server" label="Server Node" default>

1. Run the install script, setting `INSTALL_RKE2_ARTIFACT_PATH` to the directory that contains the extracted artifacts.

    ```bash
    INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh
    ```

2. (Optional) Create `/etc/rancher/rke2/config.yaml` with any configuration for the node. See [Configuration](configuration.md).

3. Enable and start the `rke2-server` service.

    ```bash
    systemctl enable rke2-server
    systemctl start rke2-server
    ```

4. Follow the service logs to monitor startup.

    ```bash
    journalctl -u rke2-server -f
    ```

5. Once the server is running, retrieve the node token. You need this token to join additional server and agent nodes to the cluster.

    ```bash
    cat /var/lib/rancher/rke2/server/node-token
    ```

To add more server nodes to the cluster, create `/etc/rancher/rke2/config.yaml` on each new server **before** starting the service, pointing it at the first server:

```yaml
server: https://<FIRST_SERVER_IP>:9345
token: <NODE_TOKEN>
```

  </TabItem>
  <TabItem value="agent" label="Agent Node">

1. Run the install script with `INSTALL_RKE2_TYPE` set to `agent`, and `INSTALL_RKE2_ARTIFACT_PATH` set to the directory that contains the extracted artifacts.

    ```bash
    INSTALL_RKE2_TYPE="agent" INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh
    ```

2. Create `/etc/rancher/rke2/config.yaml` with the address of a server node and the node token retrieved from that server.

    ```bash
    mkdir -p /etc/rancher/rke2
    cat <<EOF > /etc/rancher/rke2/config.yaml
    server: https://<SERVER_IP>:9345
    token: <NODE_TOKEN>
    EOF
    ```

3. Enable and start the `rke2-agent` service.

    ```bash
    systemctl enable rke2-agent
    systemctl start rke2-agent
    ```

4. Follow the service logs to monitor startup.

    ```bash
    journalctl -u rke2-agent -f
    ```

  </TabItem>
</Tabs>

## Step 4: Validate STIG Compliance

After the cluster is running, use the Rancher Compliance Operator to scan the cluster against the RKE2 STIG profile and confirm that it is in a compliant state. See [RKE2 STIG Scanning](../compliance-operator-docs/rke2-stig-scans.md) for instructions on installing the operator and running a scan.

If you have overridden any STIG defaults in your [configuration](configuration.md), review the scan results for any findings those changes introduce.

See the upstream [RKE2 documentation](https://docs.rke2.io/install/airgap) for more information on air-gapped installation and configuration.
