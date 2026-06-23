# Uninstall

## Local Cluster

### Uninstalling Liz UI Plugin

1. Open the Rancher Manager UI and navigate to the 'Extensions' page.

<Image src="/img/liz-ai-agent/ui-extensions-button.png" className="indented-image" title="Extensions" />

2. Find and click the AI Assistant card, click the 3 dot then click 'Uninstall'. Confirm you want to uninstall in the popup.

<Image src="/img/liz-ai-agent/ui-plugin-uninstall.png" className="indented-image box-shadow" title="Uninstall UI Plugin" />

4. Once the extension has finished installing, click the 'Reload' button that appears at the top of the page.

<Image src="/img/liz-ai-agent/reload-ui.png" className="indented-image" title="Reload UI" />

### Uninstalling the Liz MCP Server & Agent on the Local Cluster

1. Run the uninstall Helm command against the rancher-ai-agent chart:

```bash
helm uninstall --namespace cattle-ai-agent-system \
  rancher-ai-agent
```