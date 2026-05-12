# Install RKE2 from the Carbide Registry

## Save the Artifacts

The airgap tool [Hauler](https://docs.hauler.dev/docs/intro) allows you to aggregate and transport the RKE2 artifacts to disconnected environments, where you can seed a private registry or serve a registry of the Hauler store. Follow the [Hauler installation instructions](https://docs.hauler.dev/docs/introduction/install), then proceed with the following steps. This pattern is repeatable for other artifacts you may want to bring into your airgapped environment.

These steps assume you have already created nodes in your air-gap environment, and are using the bundled containerd as the container runtime.

1. On your connected machine, sync the required artifacts for your desired RKE2 version. The Carbide product manifest includes the images and binary you'll need to run RKE2 in airgap. You may also add extra artifacts such as the install script and shasum file if you'd like to use that install method in the airgap.

```bash
 hauler store sync --products rke2=v1.34.5+rke2r1 --platform linux/amd64 --product-registry registry.ranchercarbide.dev
```

```bash
hauler store add file https://get.rke2.io --name install.sh

hauler store add file https://github.com/rancher/rke2/releases/download/v1.35.3%2Brke2r3/sha256sum-amd64.txt --name sha256sum.txt
```

Sync commands and image lists can be found on the [Carbide portal](https://portal.ranchercarbide.dev/product/rke2). The portal also has pre-complied Hauler tarballs that can be downloaded directly to skip the sync step.

2. Save the Hauler store to a file. The `--containerd` flag ensures the tarball is compatible if copied directly into containerd. 

```bash
hauler store save --filename haul.tar.zst --containerd
```

## Copy the Images

The tarball resulting from `hauler store save` with the containerd flag can be dropped directly into the containerd directory at `/var/lib/rancher/rke2/agent/images/`. Alternatively, you can use Hauler on the airgapped node to copy to the directory or a private registry. 

1. On your airgap machine with the Hauler CLI installed, and .tar.zst file available, load the stored content.

```bash
hauler store load haul.tar.zst
```
2. You copy the Hauler store content directly to the images directory. For RKE2, ensure the directory `/var/lib/rancher/rke2/agent/images/` exists on the node. Then run:

```bash
hauler store copy dir://var/lib/rancher/rke2/agent/images/
```

3. If you have a private registry available, you can also copy the artifacts using Hauler. If the registry is authenticated, login with `hauler store login <airgap.private.registry> -u <username> -p <password>` first.

```bash
hauler store copy registry://airgap.private.registry
```

## Script Install

1. After loading the Hauler store, extract the RKE2 install script from the Hauler store and copy to a `rke2-artifacts` directory. 

Show artifact references: 

```bash
hauler store info
```

Extract files to your desired output directory:

```bash
hauler store extract install.sh --output /root/rke2-artifacts

hauler store extract sha256sum.txt --output /root/rke2-artifacts
```

2. Run the install script, setting `INSTALL_RKE2_ARTIFACT_PATH` to the directory which contains your RKE2 install artifacts from the Hauler store. 

```bash
INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh
```

3. Enable and start the rke2 server service.

```bash 
systemctl enable rke2-server.service && systemctl start rke2-server.service
```

4. Watch the logs:

```bash
journalctl -u rke2-server -f
```

## Binary Install

1. Place the binary file `rke2-binary:v1.34.5-rke2r1` in /usr/local/bin on each node and ensure it's executable.

2. Run the binary with desired parameters. If using a private registry, set:

```bash
system-default-registry: "airgap.private.registry"
```

See the [upstream docs](https://docs.rke2.io/install/airgap) for more information on installing and configuring RKE2.