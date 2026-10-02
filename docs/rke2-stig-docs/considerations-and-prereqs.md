# Before You Begin

RKE2 STIG comes preconfigured in a fully hardened state. Settings that are optional in standard RKE2 are enforced by default in RKE2 STIG, which can change the behavior of existing clusters and workloads.

## Standard RKE2 Remains Available

RKE2 STIG does not replace standard RKE2. Standard RKE2 is still available as part of Carbide and can still be configured into a fully compliant STIG state by following the DISA RKE2 STIG. See [Install RKE2 from the Carbide Registry](../rke2/install.md) for installation steps.

Choose RKE2 STIG when you want the hardened configuration applied and maintained as part of the product. Choose standard RKE2 when you prefer to apply and manage the hardening settings yourself.

## CIS Mode and Restricted Pod Security Admission

Standard RKE2 only runs in CIS mode when the `profile` option is set in the RKE2 configuration file. RKE2 STIG runs in CIS mode by default.

If your existing cluster runs in non-CIS mode and your configuration file does not specify a CIS profile, switching to RKE2 STIG will:

- Enable CIS mode.
- Enforce the `restricted` Pod Security Admission (PSA) standard.

Workloads that do not meet the `restricted` Pod Security Standard, such as pods that run as root, use host namespaces, or require privileged containers or added capabilities, may be rejected after the transition.

## Test Before Production

:::warning
Always validate the transition to RKE2 STIG in a test environment before deploying to production. Confirm that your workloads, Helm charts, and operators run as expected under CIS mode and restricted PSA before migrating production clusters.
:::

If a workload or environment requires a setting that differs from the RKE2 STIG defaults, you can override it in the RKE2 configuration file. See [Configuration](configuration.md).

See the upstream [RKE2 Pod Security Standards](https://docs.rke2.io/security/pod_security_standards) documentation for more on PSA configuration and exemptions.
