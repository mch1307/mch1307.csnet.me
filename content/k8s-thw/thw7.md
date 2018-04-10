---
title: "On-Prem k8s | Part 7"
linktitle: "On-Prem k8s | Part 7"
description: "Bootstrapping k8s Worker Nodes"
type: "itemized"
author: "mch1307"
date: "2018-03-04"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 8
draft: true
url: /k8s-thw/part7/
---
In this section, we will setup the worker nodes. We will install the following components:

* [**cri-o**][20] Version 1.10
* [**cni plugins**][21] Version 0.70
* [**kubelet**][22] v1.10
* [**kube-proxy**][23] v1.10

{{% alert theme="info" %}}All steps in this sections need to be run on each worker node{{% /alert %}}


## Container runtime

cri-o provides a ditribution package for Ubuntu, available as [ppa][30].

Add the projectatomic repo to Ubuntu Software Sources:

```
sudo add-apt-repository ppa:projectatomic/ppa
sudo apt-get update
```

Install the cri-o package:

```
sudo apt install cri-o-1.10
```

It will install **cri-o-runc** and **skopeo-containers** and other required dependencies if needed.

## CNI Plugins

### Install

The cni plugins can be downloaded from the [CNI Plugins releases page][31]. It is also available as an Ubuntu package from the ppa repo we just added in previous step. So we will install this way:

```
sudo apt install containernetworking-plugins
```

It will install the binaries in /opt/cni/bin.

### Configure

We now need to configure the cni plugin:

```
sudo mkdir -p /etc/cni/net.d
```
```
cat > 10-bridge.conf <<EOF
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "10.16.0.0/16"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

```
cat > 99-loopback.conf <<EOF
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```
Move the config files to the cni config directory:

```
sudo mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
```
## Install Kubernetes binaries

Download 1.10 Kubernetes binaries:

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kubelet
```

Make the bin executable and move them to **/usr/local/bin**:

```
chmod +x kubectl kube-proxy kubelet
sudo mv kubectl kube-proxy kubelet /usr/local/bin/
```

## Kubelet

First we will create additional directories for Kubernetes components:

```
sudo mkdir -p /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Copy the required certificate and kubeconfig files files to new directory:

```
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ca.pem /var/lib/kubernetes/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
``` 

Create the kubelet.service systemd unit file:

```
cat > kubelet.service <<EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=crio.service
Requires=crio.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --allow-privileged=true \\
  --anonymous-auth=false \\
  --authorization-mode=Webhook \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --cloud-provider= \\
  --cluster-dns=10.10.0.10 \\
  --cluster-domain=cluster.local \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/crio/crio.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --pod-cidr=10.16.0.0/16 \\
  --register-node=true \\
  --runtime-request-timeout=15m \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.pem \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## kube-proxy

Copy the kube-proxy config file:

```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Generate the kube-proxy unti file:

```
cat > kube-proxy.service <<EOF
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --cluster-cidr=10.16.0.0/16 \\
  --kubeconfig=/var/lib/kube-proxy/kubeconfig \\
  --proxy-mode=iptables \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Start the worker services

Move the systemd unit files:

```
sudo mv kubelet.service kube-proxy.service /etc/systemd/system/
sudo systemctl daemon-reload
```

Enable and start the services:

```
sudo systemctl enable crio kubelet kube-proxy
sudo systemctl start crio kubelet kube-proxy
```

#### [Next: Configuring Networking >][8]

#### [< Previous: Bootstrapping k8s Control Plane][6]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9
[30]: https://launchpad.net/~projectatomic/+archive/ubuntu/ppa
[31]: https://github.com/containernetworking/plugins/releases
[20]: http://cri-o.io/
[21]: https://github.com/containernetworking/plugins
[22]: https://kubernetes.io/docs/reference/generated/kubelet/
[23]: https://kubernetes.io/docs/reference/generated/kube-proxy/