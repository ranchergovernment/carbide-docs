# Uninstall

On the Rancher `local` cluster, and any other cluster where you installed the chart, run the following command to uninstall Carbide Offline Docs:

```bash
helm uninstall carbide-offline-docs -n carbide-docs-system
```

This removes every docs site's Deployment and Service, along with its Rancher menu link.

If nothing else runs in the namespace, you can also delete it:

```bash
kubectl delete namespace carbide-docs-system
```
