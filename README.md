# TKGI/PKS pressure watcher 

**This was incorporated upstream in PKS/TKGI 1.8+, is here for historical purposes**

## What does this do?

TKGI/PKS relies on airgapped, preloaded images for system daemons (e.g. CoreDNS, Observability, etc.).   During DiskPressure conditions, the default Kubelet setup is to garbage collect all images, evict all pods, and cordon the node.   After DiskPressure is lifted, any Pods on this node that rely on an offline image will likely fail with `ErrImagePull` as their Image Pull policy is "Never" and certain images aren't registered on Docker Hub as they were intended to never need to be downloaded over the Internet.

This script ensures any offline system images are reloaded on that node after the DiskPressure condition is lifted, and future system pods will succeed in starting.   Note that any evicted Pods will still need to be manually deleted.


## How do I install it?

1. Open a shell prompt on a BOSH CLI workstation with network access to your PKS bosh director.  Eg. Ops Manager .
2. Export your BOSH credentials to the enviornment.  These can be accessed via the Ops Manager GUI -> BOSH Director Tile -> Credentials Tab -> Bosh Commandline Credentials.    

e.g.
```
export BOSH_CLIENT=ops_manager BOSH_CLIENT_SECRET=fakesecret BOSH_CA_CERT=/var/tempest/workspaces/default/root_ca_certificate  BOSH_ENVIRONMENT=10.0.0.10
```
3. Copy or clone this repository onto this BOSH CLI workstation and create+upload the BOSH release to the director

```
git clone https://github.com/svrc-pivotal/pks-disk-pressure-watch && cd pks-disk-pressure-watch
bosh create-release
bosh upload-release ./dev_releases/pks-disk-pressure-watch/pks-disk-pressure-watch-0+dev.1.yml 

```
4. Configure the addon from this repo
```
bosh -n update-config --name=pks-disk-pressure-watch --type=runtime ./pks-dpw-manifest.yml
```
5. Update your PKS clusters via the PKS CLI and/or Ops Manager "Apply Pending Changes" button with the PKS upgrade errand enabled.  This addon will automatically be installed on all worker nodes with the default manifest `pks-dpw-manifest.yml`



