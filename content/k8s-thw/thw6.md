---
title: "On-Prem k8s | Part 6"
linktitle: "On-Prem k8s | Part 6"
description: "Bootstrapping k8s Control Plane"
type: "itemized"
author: "mch1307"
date: "2018-04-11"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 7
draft: false
url: /k8s-thw/part6/
---

We will now bootstrap the Kubernetes control plane on the three controller nodes. We will also setup the HAProxy host **haprx1** which will load balance the API server traffic over the three controllers.

# Load Balancer

## Install HAProxy

{{% alert theme="info" %}}HAProxy will be setup on the load balancer node **haprx1**{{% /alert %}}

The latest stable version is not available in the default repos

```bash
sudo add-apt-repository ppa:vbernat/haproxy-1.8
```

```bash
apt-cache policy haproxy
```

```bash
sudo apt-get install haproxy=1.8.5-1ppa1~xenial
```

## Configure

Edit the **/etc/haproxy/haproxy.cfg** file by adding the following lines at the end of the file:

```
backend k8s-api
  mode tcp
  timeout server 1h
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
  timeout client 1h
  default_backend k8s-api
```

{{% alert theme="info" %}}The default HAProxy timeouts are set to 50 seconds. We are setting them to one hour in order to allow long running kubectl commands not to be disconnected.{{% /alert %}}

## Start HAProxy

Enable and start the haproxy service

``` bash
sudo service haproxy enable
```

```bash
sudo service haproxy start
```

Check the status:

```bash
sudo systemctl status haproxy
```


# Kubernetes Control Plane

The Kubernetes control plane is made up of three components:

* Kubernetes API Server
* Kubernetes Controller
* Kubernetes Scheduler

{{% alert theme="info" %}}All actions described in this section need to be performed on each controller node{{% /alert %}}

## Install Binaries

```bash
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.1/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.1/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.1/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.10.1/bin/linux/amd64/kubectl"
```

```bash
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
```

```bash
sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```

## Global config

Create a kubernetes directory and copy the TLS files.

```bash
sudo mkdir -p /var/lib/kubernetes/
```

```bash
sudo mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem encryption-config.yaml /var/lib/kubernetes/
```

## API Server

Create the **kube-apiserver.sercice** systemd unit file

```bash
IP=`nslookup $(hostname -s)| awk '/^Address: / { print $2 }'`
```

```bash
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

```bash
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

## Checks

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


# RBAC for Kubelet Access

The Kubernetes API Server needs access to the Kubelet for executing commands and getting logs and metrics.

When setting up the worker nodes in the next part, we will secure Kubelet by setting the **--authorization-mode** flag to [Webhook][20].

We therefore need to create a [ClusterRole][21] and bind it to the kubernetes user so that Kubernetes API Server can be properly authorized by the Kubelet.


{{% alert theme="info" %}}You can run the below commands on any of the controller nodes.{{% /alert %}}

Create the ```system:kube-apiserver-to-kubelet``` ClusterRole with permissions to access the Kubelet API and perform most common tasks associated with managing pods:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

The Kubernetes API Server authenticates to the Kubelet as the kubernetes user using the client certificate as defined by the `--kubelet-client-certificate` flag specified when starting the API Server.

Bind the `system:kube-apiserver-to-kubelet` ClusterRole to the kubernetes user:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

## Checks

Verify we can access the Kubernetes API Server through the HAProxy load balancer:

```bash
curl -k https://10.32.2.97:6443/version
```

You should get a similar output:

```bash
{
  "major": "1",
  "minor": "10",
  "gitVersion": "v1.10.1",
  "gitCommit": "d4ab47518836c750f9949b9e0d387f20fb92260b",
  "gitTreeState": "clean",
  "buildDate": "2018-04-12T14:14:26Z",
  "goVersion": "go1.9.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

#### [Next: Bootstrapping k8s Worker Nodes >][7]

#### [< Previous: Bootstrapping etcd Cluster][5]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9
 [20]: https://kubernetes.io/docs/admin/authorization/webhook/
 [21]: https://kubernetes.io/docs/admin/authorization/rbac/#role-and-clusterrole