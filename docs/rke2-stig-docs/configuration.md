# Configuration

RKE2 STIG ships with STIG-compliant defaults built in, so a node runs in a hardened state with no configuration file at all. When your environment needs something different, you can still change it: RKE2 STIG is configured the same way as standard RKE2, and **any value you set in the RKE2 configuration file overrides the corresponding STIG default**.

## How Configuration Is Applied

RKE2 STIG reads its configuration from `/etc/rancher/rke2/config.yaml`, and from any files in `/etc/rancher/rke2/config.yaml.d/`. Settings are applied in the following order:

1. **RKE2 STIG defaults.** The hardened, STIG-compliant values built into the product.
2. **Configuration file.** Any option set in `config.yaml` replaces the default for that option.
3. **Command-line flags and environment variables.** As with standard RKE2, these take precedence over the configuration file.

Options that you do not set keep their STIG default. You only need to specify the settings you want to change.

See the upstream [RKE2 configuration documentation](https://docs.rke2.io/install/configuration) for the full list of options and how multiple configuration files are merged.

## Overriding a Default

1. Create or edit `/etc/rancher/rke2/config.yaml` on the node.

    ```bash
    mkdir -p /etc/rancher/rke2
    vi /etc/rancher/rke2/config.yaml
    ```

2. Add the options you want to change. For example, to supply your own Pod Security Admission configuration with exemptions for specific namespaces:

    ```yaml
    pod-security-admission-config-file: /etc/rancher/rke2/custom-psa.yaml
    ```

    Or to change a Kubernetes component argument:

    ```yaml
    kube-apiserver-arg:
      - "audit-log-maxage=60"
    ```

3. Restart the RKE2 service for the change to take effect.

    ```bash
    # Server nodes
    systemctl restart rke2-server

    # Agent nodes
    systemctl restart rke2-agent
    ```

Apply the same change to every node that needs it. Server-level options, such as control plane arguments, should be consistent across all server nodes.

## Staying Compliant

:::warning
Overriding a STIG default can take the cluster out of STIG compliance. Before changing a hardened setting, confirm that the new value still satisfies the applicable STIG controls, or document the deviation according to your organization's process.
:::

Common reasons to override a default include:

- Adding Pod Security Admission exemptions for system namespaces or workloads that cannot meet the `restricted` standard.
- Adjusting audit log retention or location to match your logging infrastructure.
- Tuning component arguments for your environment's scale or network.

As with any change, validate configuration overrides in a test environment before applying them to production clusters.
