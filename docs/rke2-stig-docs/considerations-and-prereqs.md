# Considerations and Prerequisites

RKE2 STIG comes preconfigured in a fully hardened state. Settings that are optional in standard RKE2 are enforced by default in RKE2 STIG, which can change the behavior of existing clusters and workloads. Review the prerequisites and considerations below before installing RKE2 STIG or migrating an existing cluster.

## Prerequisites

### SELinux Packages

RKE2 STIG installs its own RKE2 SELinux policies, but those policies depend on additional packages that must be present on the node. Which packages you need depends on how the cluster is deployed.

| Package | Required When |
|---|---|
| `container-selinux` | The node has SELinux enabled. |
| `rancher-selinux` | The cluster is running under Rancher Manager. |
| `harvester-selinux` | The cluster runs as a downstream (guest) cluster on Harvester. |

Install the required packages on every server and agent node **before** installing RKE2 STIG.

#### container-selinux

If you are running on an SELinux-enabled system, ensure the `container-selinux` package is installed. It provides the base container SELinux policy that the RKE2 policies build on, and is available from your operating system's package repositories:

```bash
sudo dnf install -y container-selinux
```

#### rancher-selinux

If you are running RKE2 STIG under Rancher, you must also install the `rancher-selinux` package. It provides the SELinux policies required by Rancher components, such as monitoring and logging, that run on the cluster. See the Rancher [rancher-selinux](https://ranchermanager.docs.rancher.com/reference-guides/rancher-security/selinux-rpm/about-rancher-selinux) documentation for installation steps.

#### harvester-selinux

If you are running RKE2 STIG as a downstream cluster on Harvester, you must also install the `harvester-selinux` package. It provides the SELinux policies required by Harvester components, such as the Harvester CSI driver and cloud provider, that run on the cluster.

#### Verify the Packages

Confirm that the required packages are installed before continuing:

```bash
rpm -q container-selinux rancher-selinux harvester-selinux
```

Only the packages that apply to your deployment need to be present.

:::note
RKE2 STIG supports only the air-gapped installation method. If your nodes cannot reach a package repository, download the required RPMs on a connected system and transfer them to each node, or install them from an internal repository mirror.
:::

## Considerations

### Standard RKE2 Remains Available

RKE2 STIG does not replace standard RKE2. Standard RKE2 is still available as part of Carbide and can still be configured into a fully compliant STIG state by following the DISA RKE2 STIG. See [Install RKE2 from the Carbide Registry](../rke2/install.md) for installation steps.

Choose RKE2 STIG when you want the hardened configuration applied and maintained as part of the product. Choose standard RKE2 when you prefer to apply and manage the hardening settings yourself.

### CIS Mode and Restricted Pod Security Admission

Standard RKE2 only runs in CIS mode when the `profile` option is set in the RKE2 configuration file. RKE2 STIG runs in CIS mode by default.

If your existing cluster runs in non-CIS mode and your configuration file does not specify a CIS profile, switching to RKE2 STIG will:

- Enable CIS mode.
- Enforce the `restricted` Pod Security Admission (PSA) standard.

Workloads that do not meet the `restricted` Pod Security Standard, such as pods that run as root, use host namespaces, or require privileged containers or added capabilities, may be rejected after the transition.

### Test Before Production

:::warning
Always validate the transition to RKE2 STIG in a test environment before deploying to production. Confirm that your workloads, Helm charts, and operators run as expected under CIS mode and restricted PSA before migrating production clusters.
:::

If a workload or environment requires a setting that differs from the RKE2 STIG defaults, you can override it in the RKE2 configuration file. See [Configuration](configuration.md).

See the upstream [RKE2 Pod Security Standards](https://docs.rke2.io/security/pod_security_standards) documentation for more on PSA configuration and exemptions.
