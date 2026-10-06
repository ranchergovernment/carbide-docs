# Overview

## What is RKE2 STIG?

RKE2 STIG is a build of [RKE2](https://docs.rke2.io) provided by Rancher Government Solutions (RGS) that ships in a hardened, STIG-compliant configuration out of the box. It is available exclusively to RGS customers.

Upstream RKE2 provides a hardened baseline that administrators finish configuring for their environment. RKE2 STIG applies the required Kubernetes security configuration as part of the product itself, so administrators do not need to interpret, apply, and maintain individual hardening settings.

## Features

- **STIG-compliant by default.** The Kubernetes control plane and supporting components are configured for STIG compliance automatically. No additional hardening configuration is required.
- **Integrated SELinux enablement.** The RKE2 SELinux policies are installed and applied during the normal installation process, so there is no separate `rke2-selinux` package to locate, install, or maintain. See [SELinux Packages](considerations-and-prereqs.md#selinux-packages) for the base packages your nodes still need.
- **FIPS 140-3.** RKE2 STIG is FIPS 140-3 certified, verified, and validated.
- **Rancher compatibility.** RKE2 STIG is designed and tested both for hosting Rancher and for running as a downstream cluster managed by Rancher. The required Pod Security Admission (PSA) configuration and other compatibility settings are included.

## Next Steps

- Review [Considerations and Prerequisites](considerations-and-prereqs.md) for node prerequisites and important considerations when moving from standard RKE2 to RKE2 STIG.
- Follow the [Installation](installation.md) guide to deploy RKE2 STIG.
- See [Configuration](configuration.md) to customize or override the STIG defaults.
