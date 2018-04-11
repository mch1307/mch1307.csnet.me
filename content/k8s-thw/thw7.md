---
title: "On-Prem k8s | Part 7"
linktitle: "On-Prem k8s | Part 7"
description: "Bootstrapping k8s Worker Nodes"
type: "itemized"
author: "mch1307"
date: "2018-04-11"
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
In this section, we will setup the worker nodes. The following components will be installed on each node:

* [**cri-o**][20]
* [**cni plugins**][21]
* [**kubelet**][22]
* [**kube-proxy**][23]


{{% alert theme="info" %}}All steps in this section need to be run on each worker node{{% /alert %}}

{{% alert theme="info" %}}OS dependencies: _socat_ and _conntrack_ should be installed on the host{{% /alert %}}

## Container runtime

### cri-o install

cri-o provides a distribution package for Ubuntu, available as [ppa][30].

Add the projectatomic repo to Ubuntu Software Sources:

```bash
sudo add-apt-repository ppa:projectatomic/ppa
sudo apt-get update
```

Install the cri-o package:

```bash
sudo apt install cri-o-1.10
```

_cri-o-runc_, _skopeo-containers_ and other required dependencies will be installed if needed.

We will also install the [_cri-tools_][24]

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.0/crictl-v1.0.0-beta.0-linux-amd64.tar.gz
```

Extract and install the binary:

```bash
tar -xvzf crictl-v1.0.0-beta.0-linux-amd64.tar.gz
chmod +x crictl
sudo mv crictl /usr/local/bin/
```

### cri-o basic configuration

The config file comes with default setup. We will only change the __registry__ config. More information on the configuration options is available [here][25]

Edit `/etc/crio/crio.conf` and add docker.io in the list of registry:

```toml
# registries is used to specify a comma separated list of registries to be used
# when pulling an unqualified image (e.g. fedora:rawhide).
registries = [
  "docker.io"
]
```


## CNI Plugins

### Install

The cni plugins can be downloaded from the [CNI Plugins release page][31]. It is also available as an Ubuntu package from the ppa repo we just added in previous step. We will install this way:

```bash
sudo apt install containernetworking-plugins
```

It will install the binaries in /opt/cni/bin.

### Configure

We now need to configure the cni plugin:

```bash
sudo mkdir -p /etc/cni/net.d
```

```bash
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

```bash
cat > 99-loopback.conf <<EOF
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```
Move the config files to the cni config directory:

```bash
sudo mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
```
## Install Kubernetes binaries

Download 1.10 Kubernetes binaries:

```bash
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kubelet
```

Make the bin executable and move them to **/usr/local/bin**:

```bash
chmod +x kubectl kube-proxy kubelet
sudo mv kubectl kube-proxy kubelet /usr/local/bin/
```

## Kubelet

{{% alert theme="info" %}}Kubelet requires the host to have no swap. Alternatively, Kubelet can be started with the **--fail-swap-on=false** flag{{% /alert %}}

First we will create additional directories for Kubernetes components:

```bash
sudo mkdir -p /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Copy the required certificate and kubeconfig files files to new directory:

```bash
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ca.pem /var/lib/kubernetes/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
``` 

Create the **kubelet.service** systemd unit file:

```bash
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
  --cgroup-driver=systemd \\
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

```bash
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Generate the **kube-proxy** systemd unit file:

```bash
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

```bash
sudo mv kubelet.service kube-proxy.service /etc/systemd/system/
sudo systemctl daemon-reload
```

Enable and start the services:

```bash
sudo systemctl enable crio kubelet kube-proxy
sudo systemctl start crio kubelet kube-proxy
```

## Checks

```bash
kubectl get nodes
```

```bash
kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
k8swrk1   Ready     <none>    1m        v1.10.0
k8swrk2   Ready     <none>    7h        v1.10.0
k8swrk3   Ready     <none>    7h        v1.10.0
```

#### [Next: Generating kubectl config >][8]

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
[24]: https://github.com/kubernetes-incubator/cri-tools
[25]: https://github.com/kubernetes-incubator/cri-o#configuration