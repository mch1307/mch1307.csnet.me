---
title: "On-Prem k8s | Part 3"
linktitle: "On-Prem k8s | Part 3"
description: "Generating Kubeconfig files"
type: "itemized"
author: "mch1307"
date: "2018-03-31"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 4
draft: true
---

# Kubeconfig files

[Kubeconfig files][20], or Kubernetes configuration files, enable Kubernetes client to locate and authenticate to Kubernetes API servers. 

We will use [kubectl][21] to generate config files for [kubelet][22] and [kube-proxy][23] clients.

{{% alert theme="info" %}}The Kubernetes scheduler and control manager also access the kubernetes API server but through non secured port exposed on the localhost interface. Therefore they do not require kubeconfig file.{{% /alert %}}

## kubectl

First we will install kubectl utility.

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kubectl
```
Make the kubectl binary executable

```
chmod +x kubectl
```
Move the binary in to your PATH

```
sudo mv kubectl /usr/local/bin/kubectl
```

## Client Authentication files

{{% alert theme="info" %}}As we will deploy the Kubernetes API server in HA mode, we will use the load balancer's IP (10.32.2.97) as the Kubernetes API server IP address in our kubeconfig files{{% /alert %}}

### kubelet

>Kubelet needs to be properly authorized by the [Kubernetes Node Authorizer][24]. We will include each worker node's individual client certificate in the kubeconfig file

Generate a kubelet kubeconfig file for each of our worker nodes:

```
for instance in k8swrk1 k8swrk2 k8swrk3; do
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://10.32.2.97:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

The following files have been generated:

```
k8swrk1.kubeconfig
k8swrk2.kubeconfig
k8swrk3.kubeconfig
```

### kube-proxy

Generate a kube-proxy config file for the kube-proxy service:

```
kubectl config set-cluster kubernetes \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://10.32.2.97:6443 \
  --kubeconfig=kube-proxy.kubeconfig
kubectl config set-credentials kube-proxy \
  --client-certificate=kube-proxy.pem \
  --client-key=kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

Generated file:

```
kube-proxy.kubeconfig
```

## Distribute the kubeconfig files

We now need to copy the generated kubeconfig files to each woker node:

```
for instance in k8swrk1 k8swrk2 k8swrk3; do
  scp ${instance}.kubeconfig kube-proxy.kubeconfig $instance:~/
done
```

#### [Next: Generating the Data Encryption Config][4]

#### [Previous: Provisioning TLS certificates][2]

 [1]: /k8s-thw/thw1
 [2]: /k8s-thw/thw2
 [3]: /k8s-thw/thw3
 [4]: /k8s-thw/thw4
 [5]: /k8s-thw/thw5
 [6]: /k8s-thw/thw6
 [7]: /k8s-thw/thw7
 [8]: /k8s-thw/thw8
 [9]: /k8s-thw/thw9
 [20]: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/
 [21]: https://kubernetes.io/docs/reference/kubectl/overview/
 [22]: https://kubernetes.io/docs/reference/generated/kubelet/
 [23]: https://kubernetes.io/docs/reference/generated/kube-proxy/
 [24]: https://kubernetes.io/docs/admin/authorization/node/