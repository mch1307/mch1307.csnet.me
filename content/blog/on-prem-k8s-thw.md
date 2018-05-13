---
title: "On-Premises Kubernetes 1.10… The Hard Way"
linktitle: "Kubernetes"
description: "Step by step Kubernetes 1.10 on-premises setup"
type: "post"
author: "mch1307"
date: "2018-04-26"
lastmod: "2018-05-12"
featured: "k8s-thw.png"
featuredpath: "date"
featuredalt: ""
categories: [""]
draft: false
url: /2018/04/on-prem-k8s-thw/
archives: "2018"
tags: ["Kubernetes","k8s","containerd","weave net","cni", "CoreDNS", "Ubuntu"]
categories: ["containers", "kubernetes"]
---

While preparing the CKA exam, I have been using [minikube][100] and [kubeadm][200] or Rancher's [rke][201] to bootstrap kubernetes clusters. Those tools are very nice but I wanted to understand all the details of a full setup. The best for this is the excellent &#8220;[Kubernetes The Hard Way][300]&#8221; tutorial from Kelsey Hightower.

I wanted to do the setup on-premises, meaning no cloud provider. So I had to &#8220;adapt&#8221; the tutorial accordingly. As I have spent some time getting everything up and running, I have decided to share my experience through a series of posts.

The tutorial is using VMs, but should be applicable any non cloud setup like bare metal or other.

This is just an adaptation of the guide and a sharing of my experience, which I hope will be helpful to others.

At the end of this guide, you will have a HA [Kubernetes][304] 1.10.1 up and running with [containerd][301] 1.1.0, [Weave Net][302] 2.3.0 and [CoreDNS][303] 1.1.1.

The tutorial is organized in the following steps:

  1. [Planning the Installation][1]
  2. [Provisioning TLS certificates][2]
  3. [Generating Kubeconfig files][3]
  4. [Generating the Data Encryption Config][4]
  5. [Bootstrapping etcd Cluster][5]
  6. [Bootstrapping k8s Control Plane][6]
  7. [Bootstrapping k8s Worker Nodes][7]
  8. [Generating kubectl config file][8]
  9. [Configuring Networking][9]
  10. [Deploying Cluster Add-ons][10]
  11. [Kubernetes Cluster Conformance][11]

#### [On-Premises Kubernetes Step by Step][11]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9
 [10]: /k8s-thw/part10
 [11]: /k8s-thw/part11
 [100]: https://github.com/kubernetes/minikube
 [200]: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/
 [201]: https://github.com/rancher/rke
 [300]: https://github.com/kelseyhightower/kubernetes-the-hard-way
 [301]: https://containerd.io/
 [302]: https://www.weave.works/docs/net/latest/kubernetes/kube-addon/
 [303]: https://coredns.io/
 [304]: https://kubernetes.io/
