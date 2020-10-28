# Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. These will be provisioned in virtual machines.

A totla of 7 VMs will be used as follows:

1 x Load Balancer
3 x Control plane nodes
3 x Worker nodes

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



## Networking

Kubernetes uses threee different network CIDRs.  My infrastructure network is ont he 192.168.20.0 subnet so that differs from Kelsey Hightower's subnet, but for the other CIDRs, I use the same subnets.  So in summary, the subnects are as follows:

|Network|Subnet|
|-------|------|
|Infrastructure|`192.168.20.0/24`|
|POD Network|`10.200.0.0/16`|
|Service Network|`10.32.0.0/24`|





### Firewall Rules



### Kubernetes Public IP Address

KeySey Hightowers uses an external load balancer to expose the cluster.  That loadbalancer would distrbute requests amonst the three control plane nodes.  Since this install is local on bare metal, I will use a single server running NGINX that will act as the load balancer.


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


## Configuring SSH Access

SSH will be used to configure the controller and worker instances. When connecting to compute instances for the first time SSH keys will be generated for you and stored in the project or instance metadata as described in the [connecting to instances](https://cloud.google.com/compute/docs/instances/connecting-to-instance) documentation.

Test SSH access to the `controller-0` compute instances:

```
gcloud compute ssh controller-0
```

If this is your first time connecting to a compute instance SSH keys will be generated for you. Enter a passphrase at the prompt to continue:

```
WARNING: The public SSH key file for gcloud does not exist.
WARNING: The private SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

At this point the generated SSH keys will be uploaded and stored in your project:

```
Your identification has been saved in /home/$USER/.ssh/google_compute_engine.
Your public key has been saved in /home/$USER/.ssh/google_compute_engine.pub.
The key fingerprint is:
SHA256:nz1i8jHmgQuGt+WscqP5SeIaSy5wyIJeL71MuV+QruE $USER@$HOSTNAME
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|                 |
|        .        |
|o.     oS        |
|=... .o .o o     |
|+.+ =+=.+.X o    |
|.+ ==O*B.B = .   |
| .+.=EB++ o      |
+----[SHA256]-----+
Updating project ssh metadata...-Updated [https://www.googleapis.com/compute/v1/projects/$PROJECT_ID].
Updating project ssh metadata...done.
Waiting for SSH key to propagate.
```

After the SSH keys have been updated you'll be logged into the `controller-0` instance:

```
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-1019-gcp x86_64)
...
```

Type `exit` at the prompt to exit the `controller-0` compute instance:

```
$USER@controller-0:~$ exit
```
> output

```
logout
Connection to XX.XX.XX.XXX closed
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
