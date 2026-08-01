# KTFactory

KTFactory helps to bootstrap new `Kubernetes` clusters with ̀`Talos Linux` nodes
by managing reproductible Talos Machine Configuration with `make`

## Quick start

In order to start a single node cluster, run following commands:

```bash
cd clusters
# create new cluster configuration files
# enter cluster name (kube) and domain (example.org), which will be used for Kubernetes endpoint API
make new

# Prepare patches
cd kube
# The schedule on controlers patch is required on single node cluster
# It should be run on controlers node only with 'ctl.' prefix
copy ../../talos-patches/schedule-on-controlers.yml ./patches/ctl.schedule-on-controlers.yml
# For Cilium CNI, default CNI must not be installed
# Copy predefined patch and prefix its name by 'all.'
# Tha way this patch will be applied on all nodes
copy ../../talos-patches/no-cni.yml ./patches/all.no-cni.yml
# Identify target disk and network interface devices
talosctl get disks --nodes ctrl01 --insecure
talosctl get links --nodes ctrl01 --insecure
copy ../../talos-patches/node-patch ./patches/ctrl01.patch.yml # update target disk and network interface within this copy

# Generate full Talos machine configuration for our ctrl01 controler
make cp-node HOST=ctrl01
# Boot ctrlk01 from Talos Linux ISO image and Apply machine configuration
talosctl apply-config --file ctl-ctrl01.yaml --nodes ctrl01 --insecure
# Wait for etcd to be ready for bootstrap...
talosctl bootstrap --nodes ctrl01 --talosconfig gen-talosconfig

# Deploy CNI if no-cni.yml patch was applied
# Cluster should be ready
```