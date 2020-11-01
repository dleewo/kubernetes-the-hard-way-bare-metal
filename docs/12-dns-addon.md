# Deploying the DNS Cluster Add-on

In this lab you will deploy the [DNS add-on](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) which provides DNS based service discovery, backed by [CoreDNS](https://coredns.io/), to applications running inside the Kubernetes cluster.

## The DNS Cluster Add-on

Deploy the `coredns` cluster add-on.  This uses a customzied YAML file that adds `forward . /etc/resolv.conf` as follows:

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

```
kubectl apply -f https://raw.githubusercontent.com/dleewo/kubernetes-the-hard-way-bare-metal/main/deployments/coredns-1.7.0.yaml
```

> output

```
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
```

List the pods created by the `kube-dns` deployment:

```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

> output

```
NAME                       READY   STATUS    RESTARTS   AGE
coredns-5677dc4cdb-d8rtv   1/1     Running   0          30s
coredns-5677dc4cdb-m8n69   1/1     Running   0          30s
```

## Verification

Create a `busybox` deployment:

```
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```

List the pod created by the `busybox` deployment:

```
kubectl get pods -l run=busybox
```

> output

```
NAME                      READY   STATUS    RESTARTS   AGE
busybox-5d494584b-8w47z   1/1     Running   0          9s
```

Retrieve the full name of the `busybox` pod:

```
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

Execute a DNS lookup for the `kubernetes` service inside the `busybox` pod:

```
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

> output

```
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```

Try to do a lookup of an external site:

```
kubectl exec -ti $POD_NAME -- nslookup www.google.com
```

> output

```
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      www.google.com
Address 1: 2607:f8b0:4023:1004::93
Address 2: 2607:f8b0:4023:1004::69
Address 3: 2607:f8b0:4023:1004::67
Address 4: 2607:f8b0:4023:1004::68
Address 5: 172.217.12.68 dfw28s05-in-f4.1e100.net
```



Next: [Smoke Test](13-smoke-test.md)