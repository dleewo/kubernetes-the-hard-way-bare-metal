# Kubernetes The Hard Way - On Bare Metal

I wanted to setup a Kubernetes cluster "the hard way".  Kelsey Hightower has an [excellent guide](https://github.com/kelseyhightower/kubernetes-the-hard-way) for doing this, but it created it in Google Cloud, but I wanted to create it locally on bare metal machines.  Or more accurately, on virtual machines.

There are a couple other guides out there for bare metal installs like [this](https://github.com/Praqma/LearnKubernetes/blob/master/kamran/Kubernetes-The-Hard-Way-on-BareMetal.md), [this](https://github.com/oahcran/kubernetes-the-hard-way-bare-metal), and [this](https://medium.com/@DrewViles/kubernetes-the-hard-way-on-bare-metal-vms-fdb32bc4fed0) but they are all older documentats that install older versions of Kubernetes.

I have tried to keep the same stucture and even some of the same content as Kelsey Hightower's instructions.

This tutorial walks you through setting up Kubernetes the hard way on virtual machines or bare metal. This guide is not for people looking for a fully automated command to bring up a Kubernetes cluster. If that's you then check out [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine), or the [Getting Started Guides](https://kubernetes.io/docs/setup).

Kubernetes The Hard Way is optimized for learning, which means taking the long route to ensure you understand each task required to bootstrap a Kubernetes cluster.

> The results of this tutorial should not be viewed as production ready, and may receive limited support from the community, but don't let that stop you from learning!

## Copyright

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

Many parts of this document is copied from, or based on, Kelsey Hightower's [instructions](https://github.com/kelseyhightower/kubernetes-the-hard-way)


## Target Audience

The target audience for this tutorial is someone planning to support a production Kubernetes cluster and wants to understand how everything fits together.

## Cluster Details

Kubernetes The Hard Way guides you through bootstrapping a highly available Kubernetes cluster with end-to-end encryption between components and RBAC authentication.

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.19.3
* [containerd](https://github.com/containerd/containerd) v1.4.1
* [coredns](https://github.com/coredns/coredns) v1.7.0
* [cni](https://github.com/containernetworking/cni) v0.8.7
* [etcd](https://github.com/coreos/etcd) v3.4.13
* [nginx](https://www.nginx.com/) v1.18.0

## Labs



* [Prerequisites](docs/01-prerequisites.md)
* [Installing the Client Tools](docs/02-client-tools.md)
* [Provisioning Compute Resources](docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](docs/04-certificate-authority.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/05-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/06-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/07-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Provisioning Pod Network Routes](docs/11-pod-network-routes.md)
* [Deploying the DNS Cluster Add-on](docs/12-dns-addon.md)
* [Smoke Test](docs/13-smoke-test.md)
* [Cleaning Up](docs/14-cleanup.md)


<br/>

## Problems and Issues Encountered

This section is a summary of the problems and issues I encountered and how I solved them.

### kube-scheduler Would Not Start

The first problem I ran into is that when it came to running this command to check the control plane in [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md) by running:

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```
I received the following output - the scheduler was failing the health check

```
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused   
controller-manager   Healthy     ok                                                                                            
etcd-1               Healthy     {"health":"true"}                                                                             
etcd-2               Healthy     {"health":"true"}                                                                             
etcd-0               Healthy     {"health":"true"}   
```

I then checked the status of the `kube-scheduler` by running

```
sudo systemctl status kube-scheduler
```

and got output as follows:

```
$ sudo systemctl status kube-scheduler
● kube-scheduler.service - Kubernetes Scheduler
     Loaded: loaded (/etc/systemd/system/kube-scheduler.service; enabled; vendor preset: enabled)
     Active: activating (auto-restart) (Result: exit-code) since Sat 2020-10-31 03:30:38 UTC; 1s ago
       Docs: https://github.com/kubernetes/kubernetes
    Process: 88968 ExecStart=/usr/local/bin/kube-scheduler --config=/etc/kubernetes/config/kube-scheduler.yaml --v=2 (code=exited, >
   Main PID: 88968 (code=exited, status=1/FAILURE)

Oct 31 03:30:38 khw-controller-1 systemd[1]: kube-scheduler.service: Main process exited, code=exited, status=1/FAILURE
Oct 31 03:30:38 khw-controller-1 systemd[1]: kube-scheduler.service: Failed with result 'exit-code'.
```

The `kube-scheduler` would start and then exit.  So I then tried to run the `kube-scheduler `manually off the command line and got:

```
$ sudo /usr/local/bin/kube-scheduler --config=/etc/kubernetes/config/kube-scheduler.yaml --v=2
I1031 03:34:33.747160   89911 registry.go:173] Registering SelectorSpread plugin
I1031 03:34:33.747320   89911 registry.go:173] Registering SelectorSpread plugin
I1031 03:34:33.747614   89911 flags.go:59] FLAG: --add-dir-header="false"
I1031 03:34:33.747709   89911 flags.go:59] FLAG: --address="0.0.0.0"

    ...[SNIP]...

I1031 03:34:33.750067   89911 flags.go:59] FLAG: --use-legacy-policy-config="false"
I1031 03:34:33.750092   89911 flags.go:59] FLAG: --v="2"
I1031 03:34:33.750117   89911 flags.go:59] FLAG: --version="false"
I1031 03:34:33.750144   89911 flags.go:59] FLAG: --vmodule=""
I1031 03:34:33.750169   89911 flags.go:59] FLAG: --write-config-to=""
I1031 03:34:34.261500   89911 serving.go:331] Generated self-signed cert in-memory
no kind "KubeSchedulerConfiguration" is registered for version "kubescheduler.config.k8s.io/v1alpha1" in scheme "k8s.io/kubernetes/pkg/scheduler/apis/config/scheme/scheme.go:30"
```

The last line provided the clue I needed to figure out what was going wrong.  SOme Google searching pointed me to the corretc `apiVersion` to use.  Kelsey Hightower's `kube-scheduler.yaml` file had `apiVersion` as follows:

```
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
```

but in kubernetes 1.19, it needed to be:

```
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
```
Once I made that change, the `kube-scheduler` started to run just fine


