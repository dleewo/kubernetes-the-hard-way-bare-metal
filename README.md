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



<br/>

## Problems and Issues Encountered

This section is a summary of the problems and issues I encountered and how I solved them.  For any problem encountered, the instrcution in the labs have been updated to include any fixes.

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


### Flannel Failed to get the POD CIDR

The deployment of flannel went fine.  Looking atht ethe daemonset, I saw:

```
$ kubectl get daemonsets -n kube-system
NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-flannel-ds   3         3         0       3            0           <none>          97m
```
but when I examined the pods, I saw:

```
(base) Dereks-MacBook-Pro:~ derek$ kubectl get pods -n kube-system
NAME                    READY   STATUS             RESTARTS   AGE
kube-flannel-ds-8x86x   0/1     CrashLoopBackOff   21         61m
kube-flannel-ds-cdhnw   0/1     CrashLoopBackOff   21         63m
kube-flannel-ds-l4fp8   0/1     CrashLoopBackOff   15         53m
```

The pods were failing to run.  Looking tah telogs or the one of them showed:

```
(base) Dereks-MacBook-Pro:~ derek$ kubectl logs kube-flannel-ds-8x86x -n kube-system
I1031 20:27:00.726041       1 main.go:518] Determining IP address of default interface
I1031 20:27:00.726479       1 main.go:531] Using interface with name ens160 and address 192.168.20.34
I1031 20:27:00.726494       1 main.go:548] Defaulting external address to interface address (192.168.20.34)
W1031 20:27:00.726514       1 client_config.go:517] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I1031 20:27:00.735383       1 kube.go:119] Waiting 10m0s for node controller to sync
I1031 20:27:00.736097       1 kube.go:306] Starting kube subnet manager
I1031 20:27:01.736204       1 kube.go:126] Node controller sync successful
I1031 20:27:01.736226       1 main.go:246] Created subnet manager: Kubernetes Subnet Manager - khw-worker-0
I1031 20:27:01.736230       1 main.go:249] Installing signal handlers
I1031 20:27:01.736315       1 main.go:390] Found network config - Backend type: vxlan
I1031 20:27:01.736371       1 vxlan.go:121] VXLAN config: VNI=1 Port=0 GBP=false Learning=false DirectRouting=false
E1031 20:27:01.736643       1 main.go:291] Error registering network: failed to acquire lease: node "khw-worker-2" pod cidr not assigned
I1031 20:27:01.736679       1 main.go:370] Stopping shutdownHandler...
```

The logs indicated that the pod cidr was not assigned.

The `kubelet-config.yaml` definitely had the podCIDR:

```
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "10.200.2.0/24"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/khw-worker-2.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/khw-worker-2-key.pem"
```
After much Google searching, I found where some said they had to add the `--allocate-node-cidrs=true` flag to the kube-controller-manager.  So I modified the creation of file `/etc/systemd/system/kube-controller-manager.service` so it was crerated as follows:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --allocate-node-cidrs=true \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Once I did that, flannel started up just fine.  I think it's because I read that in cluster mode, the pod CIDRs come from the master nodes.

```
(base) Dereks-MacBook-Pro:~ derek$ kubectl get pods -n kube-system -o wide
NAME                    READY   STATUS    RESTARTS   AGE    IP              NODE           NOMINATED NODE   READINESS GATES
kube-flannel-ds-8x86x   1/1     Running   37         146m   192.168.20.34   khw-worker-0   <none>           <none>
kube-flannel-ds-cdhnw   1/1     Running   37         148m   192.168.20.36   khw-worker-2   <none>           <none>
kube-flannel-ds-l4fp8   1/1     Running   31         138m   192.168.20.35   khw-worker-1   <none>           <none>
```

### CoreDNS Pods would not start

After deploying CoreDNS, the pods would start, but never actaully get to a fully running state.  When I examined the logs for one of the pods, I noticed the following error:

```
E1101 01:21:15.769797       1 reflector.go:178] pkg/mod/k8s.io/client-go@v0.18.3/tools/cache/reflector.go:125: Failed to list *v1.Service: Get "https://10.32.0.1:443/api/v1/services?limit=500&resourceVersion=0": dial tcp 10.32.0.1:443: i/o timeout
[INFO] plugin/ready: Still waiting on: "kubernetes"
[INFO] plugin/ready: Still waiting on: "kubernetes"
[INFO] plugin/ready: Still waiting on: "kubernetes"
[INFO] plugin/ready: Still waiting on: "kubernetes"
```

The pod was not able to access the kubernetes API

After much digging and investigating, I found out that the problem wasn't a CoreDNS problem, but rather a pod networking problem.  While pods were able to communicate with other pods, even on other nodes, pods could not access any other subnet, only the `10.200.0.0/16` subnet.  

I found others with a simialr issue and one reponse from someone mentioned that the issue was that flannel was using a different netwrok to the POD CIDR.  I did notice that the kube-flannel.yml file had a section azs follows:

```
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```

I decided to try changing that to

```
  net-conf.json: |
    {
      "Network": "10.200.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```

and once I got it all re-deployed, CoreDNS started to work.

As a test, I deployed busybox and execed into and tried a couple (nslookup and curl) commands as followed:

```
$ kubectl exec -ti busybox-6bbcc6579-hkvnt -- sh
/ # nslookup kubernetes
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local


/ # curl https://192.168.20.30:6443
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
/ # exit
command terminated with exit code 60
```

The `nslookup` was able to resolve `kubernetes` which is the kubernetes API service name.  I was also able to access the localbalancer IP address which is external to the system.

I did discover another problem though where while CoreDNS could resolve kubernetes services, it was unabel to resovle exteranl hostname, e.g `nslookup www.yahoo.com` would fail.  This is covered in the next section


### CoreNDS Cannot Resolve External Sites

CoreNDS is unable to resolve external sites.  In looking at the logs for the pods, I noticed:

```
[ERROR] plugin/errors: 2 www.yahoo.com. AAAA: plugin/loop: no next plugin found
[ERROR] plugin/errors: 2 www.yahoo.com. A: plugin/loop: no next plugin found
[ERROR] plugin/errors: 2 www.yahoo.com. AAAA: plugin/loop: no next plugin found
[ERROR] plugin/errors: 2 www.yahoo.com. A: plugin/loop: no next plugin found
```

To resolve this, I had to cusotmize the coredns-1.7.0.yaml file to add `forward . /etc/resolv.conf` here:

```
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        prometheus :9153
        cache 30
        loop
        reload
        loadbalance
    }
```

After re-deploying, I used the busybox pod and was able to resolve external sites:

 ```
$ kubectl exec -ti busybox-6bbcc6579-c7fsr -- sh
/ # nslookup www.yahoo.com
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      www.yahoo.com
Address 1: 74.6.231.20 media-router-fp73.prod.media.vip.ne1.yahoo.com
Address 2: 74.6.143.25 media-router-fp73.prod.media.vip.bf1.yahoo.com
Address 3: 74.6.231.21 media-router-fp74.prod.media.vip.ne1.yahoo.com
Address 4: 74.6.143.26 media-router-fp74.prod.media.vip.bf1.yahoo.com
Address 5: 2001:4998:124:1507::f000 media-router-fp73.prod.media.vip.bf1.yahoo.com
Address 6: 2001:4998:44:3507::8000 media-router-fp73.prod.media.vip.ne1.yahoo.com
Address 7: 2001:4998:44:3507::8001 media-router-fp74.prod.media.vip.ne1.yahoo.com
Address 8: 2001:4998:124:1507::f001 media-router-fp74.prod.media.vip.bf1.yahoo.com
```

