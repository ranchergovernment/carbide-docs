# Known Limitations & Caveats

We continually improve Carbide based on customer feedback and requirements while striving to stay agile in our development. The known limitations and caveats with the Secured Registry are listed below. We intend to address these as Carbide continues to evolve.

## Rancher Managing Cloud-Hosted Kubernetes

- Rancher managing cloud-hosted Kubernetes (EKS, AKS, GKE, etc.) does not currently support private, secured registries for the agent that gets installed onto the downstream cluster for Rancher to manage it.
- Rancher Manager does not currently support managing cloud-hosted solutions in an airgap, including serving images from a private registry.

:::danger
If you are managing cloud-hosted Kubernetes from Rancher in any capacity, **do not update your Rancher's `systemDefaultRegistry` to point to a private registry holding the Carbide images**. Doing so will break Rancher's ability to manage those downstream clusters.
:::
