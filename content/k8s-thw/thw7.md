---
title: "On-Prem k8s | Part 7"
linktitle: "On-Prem k8s | Part 7"
description: "Bootstrapping k8s Worker Nodes"
type: "itemized"
author: "mch1307"
date: "2018-04-15"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 8
draft: false
url: /k8s-thw/part7/
---
In this section, we will setup the worker nodes. The following components will be installed on each node:

* [**cni plugins**][21]
* [**containerd**][20]
* [**kubelet**][22]
* [**kube-proxy**][23]


{{% alert theme="info" %}}All steps in this section need to be run on each worker node{{% /alert %}}

## Host preparation

A few points need to be addressed at host level in order to make everything work smoothly:

* socat and conntrack should be installed
    
    ```bash
    sudo apt install socat conntrack
    ```
* ipv4 packet forwarding should be enabled

      modify /etc/sysctl.conf:
    
      ```bash
      net.ipv4.ip_forward=1
      ```

      ```bash
      sysctl -p /etc/sysctl.conf
      ```

* Kubelet requires host to have no swap

    ```
    sudo swapoff -a
    ```
    Then comment the swap mount in the /etc/fstab file

    > In case swap is required on the host, Kubelet can be started with the ```--fail-swap-on=false``` flag

## CNI Plugins

### Install

The cni plugins can be downloaded from the [CNI Plugins release page][31]. We will install the latest version to date: v0.7.1


Create the default directories

```bash
sudo mkdir -p /opt/cni/bin \
  /etc/cni/net.d
```

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/containernetworking/plugins/releases/download/v0.7.1/cni-plugins-amd64-v0.7.1.tgz
```


```bash
sudo tar xvf cni-plugins-amd64-v0.7.1.tgz -C /opt/cni/bin
```

We do not need to configure cni as we will setup Weave and it will do the necessary setup automagically.


## Container runtime

Download and install containerd v1.1.0.

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/containerd/containerd/releases/download/v1.1.0/containerd-1.1.0.linux-amd64.tar.gz
```

```bash
sudo tar xvf containerd-1.1.0.linux-amd64.tar.gz -C /usr/local/
```

containerd requires [**runc**][26]

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64
```

```bash
mv runc.amd64 runc
chmod +x runc
sudo mv runc /usr/local/bin/
```

Create containerd systemd unit file

```bash
cat > containerd.service <<EOF
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

We will also install the [_crictl tool_][24], a command line tool for interacting with Container Runtime Interface.

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.0/crictl-v1.0.0-beta.0-linux-amd64.tar.gz
```

Extract and install the binary:

```bash
tar xvf crictl-v1.0.0-beta.0-linux-amd64.tar.gz
chmod +x crictl
sudo mv crictl /usr/local/bin
```

Configure ```crictl``` to connect to the ```containerd``` runtime by creating a config file:

```bash
cat > crictl.yaml <<EOF
runtime-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10
EOF
```

```bash
sudo mv crictl.yaml /etc/crictl.yaml
```

## Install Kubernetes binaries

Download 1.10.1 Kubernetes binaries:

```bash
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.1/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.1/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.1/bin/linux/amd64/kubelet
```

Make the bin executable and move them to ```/usr/local/bin```:

```bash
chmod +x kubectl kube-proxy kubelet
sudo mv kubectl kube-proxy kubelet /usr/local/bin/
```

Create additional directories for Kubernetes components:

```bash
sudo mkdir -p /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

## Kubelet

Copy the required certificate and kubeconfig files files to new directory:

```bash
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ca.pem /var/lib/kubernetes/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
``` 

Create the ```kubelet.service``` systemd unit file:

```bash
cat > kubelet.service <<EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

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
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
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

Generate the ```kube-proxy``` systemd unit file:

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
sudo mv containerd.service kubelet.service kube-proxy.service /etc/systemd/system/
sudo systemctl daemon-reload
```

Enable and start the services:

```bash
sudo systemctl enable containerd kubelet kube-proxy
sudo systemctl start containerd kubelet kube-proxy
```

## Checks

```bash
kubectl get nodes
```

```bash
kubectl get nodes
NAME      STATUS       ROLES     AGE       VERSION
k8swrk1   NotReady     <none>    1h        v1.10.1
k8swrk2   NotReady     <none>    2h        v1.10.1
k8swrk3   NotReady     <none>    2h        v1.10.1
```

{{% alert theme="info" %}}Note that nodes status is "NotReady". This is normal as we did not configure networking yet.{{% /alert %}}

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
[20]: http://containerd.io/
[21]: https://github.com/containernetworking/plugins
[22]: https://kubernetes.io/docs/reference/generated/kubelet/
[23]: https://kubernetes.io/docs/reference/generated/kube-proxy/
[24]: https://github.com/kubernetes-incubator/cri-tools
[25]: https://github.com/kubernetes-incubator/cri-o#configuration
[26]: https://github.com/opencontainers/runc