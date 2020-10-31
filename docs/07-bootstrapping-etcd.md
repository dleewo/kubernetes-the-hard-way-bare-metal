> This is almost an identical copy of Kelsey Hightower's [Bootstrapping the etcd Cluster](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/07-bootstrapping-etcd.md) document.  The main changes are to the IP addresses and hostname
<br />

# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). In this lab you will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `khw-controller-0`, `khw-controller-1`, and `khw-controller-2`. Login to each controller instance using the `ssh` command. Example:

```
ssh controller-0
```

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. 

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [etcd](https://github.com/etcd-io/etcd) GitHub project:

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.13/etcd-v3.4.13-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.4.13-linux-amd64.tar.gz
  sudo mv etcd-v3.4.13-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
}
```

Set IP_ADDRESS to the IP address of each controller

```
IP_ADDRESS=192.168.20.31
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file.  Note the IP addresses ands hostname in the `--initial-cluster khw-controller` argument:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${IP_ADDRESS}:2380 \\
  --listen-peer-urls https://${IP_ADDRESS}:2380 \\
  --listen-client-urls https://${IP_ADDRESS}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${IP_ADDRESS}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster khw-controller-0=https://192.168.20.31:2380,khw-controller-1=https://192.168.20.32:2380,khw-controller-2=https://192.168.20.33:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

> Remember to run the above commands on each controller node: `khw-controller-0`, `khw-controller-1`, and `khw-controller-2`.

## Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

> output

```
9fe5e4802b12ddc6, started, khw-controller-2, https://192.168.20.33:2380, https://192.168.20.33:2379, false
b284866a08ae9d4a, started, khw-controller-0, https://192.168.20.31:2380, https://192.168.20.31:2379, false
b9c242770a261a8b, started, khw-controller-1, https://192.168.20.32:2380, https://192.168.20.32:2379, false
```

Next: [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)