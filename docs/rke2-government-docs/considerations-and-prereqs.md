# Considerations and Prerequisites

RKE2 Government comes preconfigured in a fully hardened state. Settings that are optional in standard RKE2 are enforced by default in RKE2 Government, which can change the behavior of existing clusters and workloads. Review the prerequisites and considerations below before installing RKE2 Government or migrating an existing cluster.

## Prerequisites

### Hauler

The installation steps assume you are using [Hauler](https://docs.hauler.dev/docs/intro) to fetch, transfer, and load the RKE2 Government artifacts. Hauler must be installed on the connected machine you use to download the artifacts and on each node you install RKE2 Government on. See the [Hauler installation instructions](https://docs.hauler.dev/docs/introduction/install).

### jq (Recommended)

[jq](https://jqlang.org/) is not required, but it is recommended on each node. The installation steps use it to filter the Hauler store contents down to the RKE2 Government artifacts that need to be extracted. Without `jq`, you can identify those artifacts manually from the output of `hauler store info`.

### Host Preparation

RKE2 Government applies its own SELinux policies and hardened configuration, but some host-level prerequisites are outside of what the installation manages. Complete the following on each node **before** installing RKE2 Government.

#### SELinux Packages

RKE2 Government installs its own RKE2 SELinux policies, but those policies depend on additional packages that must be present on the node. Which packages you need depends on how the cluster is deployed.

| Package | Required When |
|---|---|
| `container-selinux` | The node has SELinux enabled. |
| `rancher-selinux` | The cluster is running under Rancher Manager. |
| `harvester-selinux` | The cluster runs as a downstream (guest) cluster on Harvester. |

Install the required packages on every server and agent node **before** installing RKE2 Government.

##### container-selinux

If you are running on an SELinux-enabled system, ensure the `container-selinux` package is installed. It provides the base container SELinux policy that the RKE2 policies build on, and is available from your operating system's package repositories:

```bash
sudo dnf install -y container-selinux
```

##### rancher-selinux

If you are running RKE2 Government under Rancher, you must also install the `rancher-selinux` package. It provides the SELinux policies required by Rancher components, such as monitoring and logging, that run on the cluster. See the Rancher [rancher-selinux](https://ranchermanager.docs.rancher.com/reference-guides/rancher-security/selinux-rpm/about-rancher-selinux) documentation for installation steps.

##### harvester-selinux

If you are running RKE2 Government as a downstream cluster on Harvester, you must also install the `harvester-selinux` package. It provides the SELinux policies required by Harvester components, such as the Harvester CSI driver and cloud provider, that run on the cluster.

##### Verify the Packages

Confirm that the required packages are installed before continuing:

```bash
rpm -q container-selinux rancher-selinux harvester-selinux
```

Only the packages that apply to your deployment need to be present.

:::note
RKE2 Government supports only the air-gapped installation method. If your nodes cannot reach a package repository, download the required RPMs on a connected system and transfer them to each node, or install them from an internal repository mirror.
:::

#### Kernel Parameters

Because RKE2 Government runs in CIS mode by default, the kubelet expects specific kernel parameters to be set on the host and will not start if they are missing. Set them on every server and agent node:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/60-rke2-cis.conf
vm.panic_on_oom=0
vm.overcommit_memory=1
kernel.panic=10
kernel.panic_on_oops=1
EOF
sudo sysctl -p /etc/sysctl.d/60-rke2-cis.conf
```

#### etcd User

In CIS mode, etcd runs as the `etcd` user and group rather than as root, and RKE2 will not start on a server node if they do not exist. Create them on every server node:

```bash
sudo useradd -r -c "etcd user" -s /sbin/nologin -M etcd -U
```

Agent nodes do not run etcd and do not need this user.

## Considerations

### Standard RKE2 Remains Available

RKE2 Government does not replace standard RKE2. Standard RKE2 is still available as part of Carbide and can still be configured into a fully compliant STIG state by following the DISA RKE2 STIG. See [Install Standard RKE2 from the Carbide Registry](../rke2/install.md) for installation steps.

Choose RKE2 Government when you want the hardened configuration applied and maintained as part of the product. Choose standard RKE2 when you prefer to apply and manage the hardening settings yourself.

### CIS Mode and Restricted Pod Security Admission

Standard RKE2 only runs in CIS mode when the `profile` option is set in the RKE2 configuration file. RKE2 Government runs in CIS mode by default.

If your existing cluster runs in non-CIS mode and your configuration file does not specify a CIS profile, switching to RKE2 Government will:

- Enable CIS mode.
- Enforce the `restricted` Pod Security Admission (PSA) standard.

Workloads that do not meet the `restricted` Pod Security Standard, such as pods that run as root, use host namespaces, or require privileged containers or added capabilities, may be rejected after the transition.

### Test Before Production

:::warning
Always validate the transition to RKE2 Government in a test environment before deploying to production. Confirm that your workloads, Helm charts, and operators run as expected under CIS mode and restricted PSA before migrating production clusters.
:::

If a workload or environment requires a setting that differs from the RKE2 Government defaults, you can override it in the RKE2 configuration file. See [Configuration](configuration.md).

See the upstream [RKE2 Pod Security Standards](https://docs.rke2.io/security/pod_security_standards) documentation for more on PSA configuration and exemptions.
