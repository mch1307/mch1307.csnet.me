---
title: "On-Prem k8s | Part 6"
linktitle: "On-Prem k8s | Part 6"
description: "Bootstrapping k8s Control Plane"
type: "itemized"
author: "mch1307"
date: "2018-04-04"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 7
draft: true
url: /k8s-thw/part6/
---

We will now bootstrap the Kubernetes control plane on the three controller nodes. We will also setup the HAProxy host **haprx1** which will load balance the API server traffic over the three controllers.

# Load Balancer

## Install HAProxy

{{% alert theme="info" %}}HAProxy will be setup on the load balancer node **haprx1**{{% /alert %}}

The latest stable version is not available in the default repos

```
sudo add-apt-repository ppa:vbernat/haproxy-1.8
```

```
apt-cache policy haproxy
```

```
sudo apt-get install haproxy=1.8.5-1ppa1~xenial
```

## Configure

Edit the **/etc/haproxy/haproxy.cfg** file by adding the following lines at the end of the file:

```
backend k8s-api
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-api-1 10.32.2.91:6443 check
  server k8s-api-2 10.32.2.92:6443 check
  server k8s-api-3 10.32.2.93:6443 check

frontend k8s-api
  bind 0.0.0.0:6443
  mode tcp
  option tcplog
  default_backend k8s-api
```

## Start HAProxy

Enable and start the haproxy service

```
sudo service haproxy enable
```

```
sudo service haproxy start
```

Check the status:

```
sudo systemctl status haproxy
```


# Kubernetes Control Plane

The Kubernetes control plane is made up of three components:

* Kubernetes API Server
* Kubernetes Controller
* Kubernetes Scheduler

{{% alert theme="info" %}}All actions described in this section need to be performed on each controller node{{% /alert %}}

## Install Binaries

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kubectl"
```

```
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
```

```
sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```

## Global config

Create a kubernetes directory and copy the TLS files.

```
sudo mkdir -p /var/lib/kubernetes/
```

```
sudo mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem encryption-config.yaml /var/lib/kubernetes/
```

## API Server

Create the **kube-apiserver.sercice** systemd unit file

```
IP=`nslookup $(hostname -s)| awk '/^Address: / { print $2 }'`
```

```
cat > kube-apiserver.service <<EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --admission-control=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --advertise-address=${IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://10.32.2.91:2379,https://10.32.2.92:2379,https://10.32.2.93:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --insecure-bind-address=127.0.0.1 \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/ca-key.pem \\
  --service-cluster-ip-range=10.10.0.0/22 \\
  --service-node-port-range=30000-32767 \\
  --tls-ca-file=/var/lib/kubernetes/ca.pem \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```



## Controller

Create the **kube-controller-manager.service** systemd unit file:

```
cat > kube-controller-manager.service <<EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.16.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --leader-elect=true \\
  --master=http://127.0.0.1:8080 \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/ca-key.pem \\
  --service-cluster-ip-range=10.10.0.0/22 \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Scheduler

Create the **kube-scheduler.service** systemd unit file:

```
cat > kube-scheduler.service <<EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --leader-elect=true \\
  --master=http://127.0.0.1:8080 \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Start the services

```
sudo mv kube-apiserver.service kube-scheduler.service kube-controller-manager.service /etc/systemd/system/
```

```
sudo systemctl daemon-reload
```

```
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
```

```
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

{{% alert theme="info" %}}The Kubernetes API Server takes some time to fully initialize.{{% /alert %}}

## Verification

```
kubectl get componentstatuses
```

The following output should be displayed if everything goes well.

```
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true"}
```

#### [Bootstrapping k8s Worker Nodes >][7]

#### [< Bootstrapping etcd Cluster][5]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9
