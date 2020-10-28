# Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. These will be provisioned in virtual machines.

A total of 7 VMs will be used as follows:

- 1 x Load Balancer
- 3 x Control plane nodes
- 3 x Worker nodes

The VM I provisioned are as follows:

|Hostname|IP Address|Num vCPUs|RAM|Disk|
| :-- | :-- | :-: | --: | --:|
|`khw-loadbalancer`|`192.168.20.30`|1|1GB|32GB|
|`khw-controller-0`|`192.168.20.31`|2|2GB|32GB|
|`khw-controller-1`|`192.168.20.32`|2|2GB|32GB|
|`khw-controller-2`|`192.168.20.33`|2|2GB|32GB|
|`khw-worker-0`|`192.168.20.34`|2|3GB|128GB|
|`khw-worker-1`|`192.168.20.35`|2|3GB|128GB|
|`khw-worker-2`|`192.168.20.36`|2|3GB|128GB|

Each server has Ubuntu 20.04.1 and I did

```
apt update
apt upgrade
```

on each server to ensure all packages are up-to-date.  While not absoutely necessary, I also installed the following packages

```
apt install net-tools
```

For the virtual machines, thet are several options available icnlduing using toos like VMWare Workstation or Oracle VirtualBox.  For my purposes, I already had VMWare ESXi 7.0 installed on an unused PC.  You can obtain a free license from VMware.  There are limitations like the maxium number of vCPUs that can be assigned to a VM, but you will not hit any of those limits for this cluster.



## Networking

Kubernetes uses threee different network CIDRs.  My infrastructure network is ont he 192.168.20.0 subnet so that differs from Kelsey Hightower's subnet, but for the other CIDRs, I use the same subnets.  So in summary, the subnects are as follows:

|Network|Subnet|
|:--|:--|
|Infrastructure|`192.168.20.0/24`|
|POD Network|`10.200.0.0/16`|
|Service Network|`10.32.0.0/24`|





### Firewall Rules

The variousnodes must be able to communicate with each other over a variety of ports.  It would easiest to not have any firewall enabled.  By default, a new install of Ubuntu wll not have any firewall enabled.



### Kubernetes Public IP Address

Keylsey Hightower uses an external load balancer to expose the cluster.  That load balancer would distrbute requests amonst the three control plane nodes.  Since this install is local on bare metal, I will use a single server running NGINX that will act as the load balancer.


## Servers

As mentioned above, each server will run Ubuntu 20.04.1. This document will not go through the installation and setup of these servers and will assume they are ready.

They should all have static IP addresses and you should add the following to each of the `/etc/hosts` files on each server:

```
192.168.20.30    khw-loadbalancer
192.168.20.31    khw-controller-0
192.168.20.32    khw-controller-1
192.168.20.33    khw-controller-2
192.168.20.34    khw-worker-0
192.168.20.35    khw-worker-1
192.168.20.36    khw-worker-2
```

Replace with your actual IP adrdesses and hostnames.

Swap must be disabled on the controller and work nodes.  This can be done as follows

```
sudo swapoff -a
```

This will disable swap immediatly, but it is only temporary and will be reenabled if you reboot.  To disable swap permanently, edit the `/egtc/fstab` file and comment out the swap line:
 
<img src="https://github.com/dleewo/kubernetes-the-hard-way-bare-metal/raw/main/images/fstab-swap.png" width="300" />

> TIP: Most VM hypervisors will alow you to take a snapshot of a VM and then in the future, you can revert to that sanpshot.  Once I got the base Ubuntu all configured with host files, ssh keys, updates, etc., I took snapshots of each server.  By doing this, I can always start over if I wanted, or start from a clean slate if I wanted to rebuild the cluster from scratch


## Configuring SSH Access

SSH will be used to configure the controller and worker instances. Be sure to insatll and configure SSH on each server.  It is also recommendeded that you use SSH keys to make the use of ssh a lot easier.  Setting up the use of SSH keys is beyond hte scope of this document, but there are many guies on how to do that on the internet.


Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
