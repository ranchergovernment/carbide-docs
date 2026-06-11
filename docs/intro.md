---
sidebar_position: 1
---

# Introduction

Carbide is Rancher Government's hardened distribution of the SUSE Rancher product portfolio. It is offered exclusively to our US Government and partner customers who value security above all else.

## What makes Carbide different

Rancher Carbide is tactically built with the following enhancements over the community version:

- **[SLSA Level 3](https://slsa.dev) secure build process** hosted on Harbor (or the legacy Azure Government registry).
- **Digitally signed container images.** Every container in our registry is digitally [signed](/docs/registry-docs/validating-images) by Rancher Government Solutions. Verifiable trust is baked into everything we do.
- **[Software Bill of Materials](https://www.cisa.gov/sbom) (SBOM)** support in every container image.
- **[Container attestations](https://www.testifysec.com/blog/what-is-a-supply-chain-attestation)** with non-falsifiable provenance.
- **Container vulnerability scans** delivered alongside every image.
- **Day 2 security operators** powered by Rancher Government's DISA STIGs.
- **Airgapped documentation** for the entire supported Rancher product portfolio.
- **Custom red-white-blue theming** and Rancher Manager white-labeling.

## Explore the docs

| Section | What you'll find |
| --- | --- |
| [Carbide Secured Registry (CSR)](/docs/registry-docs/introduction) | Acquire, verify, and seed Carbide's hardened, signed images into your own registry. |
| [RGS Application Collection](/docs/app-collection-docs/introduction) | A curated, production-ready set of trusted developer and infrastructure applications. |
| [IC Cloud Support](/docs/IC-cloud-support-docs/introduction) | Native Rancher provisioning on Intelligence Community cloud regions (Tech Preview). |
| [Rancher Compliance Operator](/docs/compliance-operator-docs/rke2-stig-scans) | Run RKE2 STIG scans against downstream clusters from the Rancher UI. |
| [Airgapped Docs](/docs/airgapped-docs/introduction) | The entire supported Rancher product portfolio, available inside an airgap. |

New to Carbide? Start with the [Carbide Secured Registry](/docs/registry-docs/introduction) — it's the acquisition point for everything else.

We are honored to serve your mission.
