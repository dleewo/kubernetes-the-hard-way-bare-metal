> This is almost an identical copy of Kelsey Hightower's [Smoke Test](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/13-smoke-test.md) document.  The main changes are to the IP addresses and hostname when generating the certificates

# Smoke Test

In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.

## Data Encryption

In this section you will verify the ability to [encrypt secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

Create a generic secret:

```
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```

Print a hexdump of the `kubernetes-the-hard-way` secret stored in etcd. On one of the controller nodes, run:

```
sudo ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C
```

> output

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a 5d ee 64 b1 e4 2f 69  |:v1:key1:].d../i|
00000050  7f 6a 5d 40 e0 78 7e 92  7a ff 9f 4a 11 78 c1 b2  |.j]@.x~.z..J.x..|
00000060  6a 45 b4 f6 5f fe 16 40  b0 0e e1 a4 b5 4f f5 6c  |jE.._..@.....O.l|
00000070  ad 85 90 42 9f fb f9 4e  ff 8d e3 13 d3 7d d7 7c  |...B...N.....}.||
00000080  a6 31 22 0e 42 1a 0b 0e  f8 de d7 2d 4f ae df 4c  |.1".B......-O..L|
00000090  6f 3c 0c f7 bc 48 55 ad  17 a9 01 b7 f7 e6 be fc  |o<...HU.........|
000000a0  48 a2 60 a5 7f 10 dc 56  5c 98 5c 4f 19 b1 4a b1  |H.`....V\.\O..J.|
000000b0  4b ad 44 e0 b0 48 16 93  43 d5 ca 86 19 d6 ca 84  |K.D..H..C.......|
000000c0  54 a4 d1 3c 5c d5 c8 74  0f 5a 3a 8d 2d 78 26 2c  |T..<\..t.Z:.-x&,|
000000d0  75 d9 62 52 43 e4 14 e9  f5 db 54 9e 4d c4 c1 5a  |u.bRC.....T.M..Z|
000000e0  a6 74 66 66 d1 d6 38 e0  e4 a4 60 14 ad b2 2a e8  |.tff..8...`...*.|
000000f0  20 29 13 2d 46 f8 3a b2  8f 8d 38 42 d3 ca cf e0  | ).-F.:...8B....|
00000100  74 2d 36 f3 0a aa 0d ba  69 4f ed b8 65 f3 72 f1  |t-6.....iO..e.r.|
00000110  94 9a ed 05 bf e8 ce e6  a1 16 24 4b ab f9 a6 3b  |..........$K...;|
00000120  54 6b 0d 4d f3 bd 31 78  67 78 d2 64 12 c3 bf 56  |Tk.M..1xgx.d...V|
00000130  3a 68 83 a1 a9 dd 84 8e  cd dc 33 15 65 32 5d 74  |:h........3.e2]t|
00000140  ac be d3 1c 3b c2 5f 6f  c0 0a                    |....;._o..|
0000014a
```

The etcd key should be prefixed with `k8s:enc:aescbc:v1:key1`, which indicates the `aescbc` provider was used to encrypt the data with the `key1` encryption key.

## Deployments

In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

Create a deployment for the [nginx](https://nginx.org/en/) web server:

```
kubectl create deployment nginx --image=nginx
```

List the pod created by the `nginx` deployment:

```
kubectl get pods -l app=nginx
```

> output

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-b94cv   1/1     Running   0          7s
```

### Port Forwarding

In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

Retrieve the full name of the `nginx` pod:

```
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
```

Forward port `8080` on your local machine to port `80` of the `nginx` pod:

```
kubectl port-forward $POD_NAME 8080:80
```

> output

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

In a new terminal make an HTTP request using the forwarding address:

```
curl --head http://127.0.0.1:8080
```

> output

```
HTTP/1.1 200 OK
Server: nginx/1.19.3
Date: Sun, 01 Nov 2020 02:25:26 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 29 Sep 2020 14:12:31 GMT
Connection: keep-alive
ETag: "5f7340cf-264"
Accept-Ranges: bytes
```

Switch back to the previous terminal and stop the port forwarding to the `nginx` pod:

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

### Logs

In this section you will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Print the `nginx` pod logs:

```
kubectl logs $POD_NAME
```

> output

```
...
127.0.0.1 - - [01/Nov/2020:02:25:26 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.65.3" "-"
```

### Exec

In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Print the nginx version by executing the `nginx -v` command in the `nginx` container:

```
kubectl exec -ti $POD_NAME -- nginx -v
```

> output

```
nginx version: nginx/1.19.3
```

## Services

In this section you will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

Expose the `nginx` deployment using a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) service:

```
kubectl expose deployment nginx --port 80 --type NodePort
```

> The LoadBalancer service type can not be used because this is a local cluster and there is nothing to handle LoadBalancer requests.  metallb can be instaled at a later time is needed.

Retrieve the node port assigned to the `nginx` service:

```
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

Node ports are exposed on each worker node so you can execute curl against any or all of the worker nodes and even though there is only one pod running on one of the nodes, all should work.  In my case, the pod is on khw-worker-2:

```
(base) Dereks-MacBook-Pro:deployments derek$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
nginx-6799fc88d8-b94cv   1/1     Running   0          7m26s   10.200.2.10   khw-worker-2   <none>           <none>
```

Execute the curl command:

```
$ curl -I http://khw-worker-0:$NODE_PORT
HTTP/1.1 200 OK
Server: nginx/1.19.3
Date: Sun, 01 Nov 2020 02:31:23 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 29 Sep 2020 14:12:31 GMT
Connection: keep-alive
ETag: "5f7340cf-264"
Accept-Ranges: bytes

$ curl -I http://khw-worker-1:$NODE_PORT
HTTP/1.1 200 OK
Server: nginx/1.19.3
Date: Sun, 01 Nov 2020 02:31:26 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 29 Sep 2020 14:12:31 GMT
Connection: keep-alive
ETag: "5f7340cf-264"
Accept-Ranges: bytes

($ curl -I http://khw-worker-2:$NODE_PORT
HTTP/1.1 200 OK
Server: nginx/1.19.3
Date: Sun, 01 Nov 2020 02:31:31 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 29 Sep 2020 14:12:31 GMT
Connection: keep-alive
ETag: "5f7340cf-264"
Accept-Ranges: bytes

```


