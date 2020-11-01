# Provisioning Pod Network

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this section, we will be installing [Flannel](https://github.com/coreos/flannel)


### Install Flannel

First, enable IP forwarding on each worker node

```
sudo sysctl net.ipv4.conf.all.forwarding=1
```

One one of the controller nodes, or from your remove server where you configure `kubectl`, apply the YAML for Flannel.

This kube-flannel.yml is idential to the one Kelsey Hightowed used except that Network was changed from `10.244.0.0/16` to `10.200.0.0/16` to match the pod CIDR.  Here is a snaippet shoiwng that section:

```
  net-conf.json: |
    {
      "Network": "10.200.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```

```
kubectl apply -f https://raw.githubusercontent.com/dleewo/kubernetes-the-hard-way-bare-metal/main/deployments/kube-flannel.yml
```
> Output

```
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

You can chekc the staytus of the Flannel daemonsets using

```
kubectl get daemonsets -n kube-system
```

> Output

```
NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-flannel-ds   3         3         0       3            0           <none>          29s
```





Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)