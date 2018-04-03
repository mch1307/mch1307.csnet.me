---
title: "On-Prem k8s | Part 5"
linktitle: "On-Prem k8s | Part 5"
description: "Bootstrapping etcd Cluster"
type: "itemized"
author: "mch1307"
date: "2018-04-03"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 6
draft: true
---

Kubernetes stores cluster states in an etcd k/v store. We will now setup a three nodes etcd cluster for high availability.

{{% alert theme="info" %}}All actions described in this section need to be performed on each controller node{{% /alert %}}

## Install binaries

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.2.18/etcd-v3.2.18-linux-amd64.tar.gz"
```

Extract and copy **etcd** and **etcdctl** binaries to your PATH

```
tar -xvf etcd-v3.2.18-linux-amd64.tar.gz
```
```
sudo mv etcd-v3.2.18-linux-amd64/etcd* /usr/local/bin
```

## Setup etcd

Create etcd directories and copy TLS certs

```
sudo mkdir -p /etc/etcd /var/lib/etcd
```

```
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

Each etcd node needs a unique name. We will use the hostname as name:

```
ETCD_NAME=`hostname -s`
```

We also need to specify the IP address we will bind etcd to:

```
IP=`nslookup $(hostname -s) | awk '/^Address: / { print $2 }'`
```

Create the systemd unit file for etcd service

```
cat > etcd.service <<EOF
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
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
  --initial-advertise-peer-urls https://${IP}:2380 \\
  --listen-peer-urls https://${IP}:2380 \\
  --listen-client-urls https://${IP}:2379,http://127.0.0.1:2379 \\
  --advertise-client-urls https://${IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster k8sctl1=https://10.32.2.91:2380,k8sctl2=https://10.32.2.92:2380,k8sctl3=https://10.32.2.93:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Bootstratp etcd cluster

Enable and start the etcd service:

```
sudo mv etcd.service /etc/systemd/system/
```

```
sudo systemctl daemon-reload
```

```
sudo systemctl enable etcd
```

```
sudo systemctl start etcd
```

> Remember to run the above commands on each controller node: **k8sctl1**, **k8sctl2**, and **k8sctl3**.

## Check etcd cluster is up and running

List the etcd cluster members:

```
ETCDCTL_API=3 etcdctl member list
```

Output:

```
b826932a88befba, started, k8sctl1, https://10.32.2.91:2380, https://10.32.2.91:2379
203a99ec1242917c, started, k8sctl2, https://10.32.2.92:2380, https://10.32.2.92:2379
4fb9793032518ec9, started, k8sctl3, https://10.32.2.93:2380, https://10.32.2.93:2379
```



#### [Next: Bootstrapping k8s Control Plane >][6]

#### [<Previous: Generating the Data Encryption Config][4]

 [1]: /k8s-thw/thw1
 [2]: /k8s-thw/thw2
 [3]: /k8s-thw/thw3
 [4]: /k8s-thw/thw4
 [5]: /k8s-thw/thw5
 [6]: /k8s-thw/thw6
 [7]: /k8s-thw/thw7
 [8]: /k8s-thw/thw8
 [9]: /k8s-thw/thw9
