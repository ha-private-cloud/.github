# High Availability Private Cloud Storage Project
# ClusterKeep (Service Name)

This organisation contains the repositories for my high availability private cloud storage solution. I intend to build a service that allows users to access private and secure cloud storage running on a kubernetes cluster to support the high availability aspect.

## Implementation

This project is using three Lenovo Thinkcentre computers which each deploy a control plane and two worker VMs with 4Gb of RAM and 32Gb Disk each. The two computer with larger disk spaces also deploy nfs-storage VMs. Further specs will be on the report.

Each computer is flashed with the Proxmox VE ISO to allow Talos Linux VMs to be deployed using infrastructure-as-code.

There will be 1 control node and 2 worker nodes per physical machine, with consideration for more if they can support the load.

A 8 port managed switch is being used to isolate traffic on a VLAN.

A raspberry PI is intended to host the DNS server to allow a public route.
