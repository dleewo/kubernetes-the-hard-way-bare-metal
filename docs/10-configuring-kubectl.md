> This is almost an identical copy of Kelsey Hightower's [Configuring kubectl for Remote Access](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/10-configuring-kubectl.md) document.  The only chage is to the setting of the KUBERNETES_PUBLIC_ADDRESS
<br />

# Configuring kubectl for Remote Access

In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> Run the commands in this lab from the same directory used to generate the admin client certificates.

## The Admin Kubernetes Configuration File

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```
{
  KUBERNETES_PUBLIC_ADDRESS=192.168.20.30

  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

After executing the aboce, a `config` file will be written to `~/.kube`.  If yoiu examine the file, the end would be somethign like this:

```
users:
- name: admin
  user:
    client-certificate: /Users/derek/Source/kubernetes-the-hard-way-bare-metal/tmp/certs/admin.pem
    client-key: /Users/derek/Source/kubernetes-the-hard-way-bare-metal/tmp/certs/admin-key.pem
```

In my case, I can created the certificates in a temporary directory so once I clean that up, the location of the certs will be invalid.  I decide to copy the files to the `.kube` folder and edited the config file so the `users` section looks like this:

```
users:
- name: admin
  user:
    client-certificate: /Users/derek/.kube//admin.pem
    client-key: /Users/derek/.kube//admin-key.pem
```




## Verification

Check the health of the remote Kubernetes cluster:

```
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-1               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"}   
etcd-2               Healthy   {"health":"true"}  
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME           STATUS   ROLES    AGE   VERSION
khw-worker-0   Ready    <none>   17m   v1.19.3
khw-worker-1   Ready    <none>   17m   v1.19.3
khw-worker-2   Ready    <none>   17m   v1.19.3
```

Next: [Provisioning Pod Network Routes](11-pod-network-routes.md)