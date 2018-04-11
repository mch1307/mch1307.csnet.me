---
title: "On-Premises Kubernetes… The Hard Way"
linktitle: "Kubernetes"
description: "Step by step k8s on-premises deployment"
type: "itemized"
author: "mch1307"
date: "2018-04-11"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 1
draft: true
url: /k8s-thw/intro/
tags: ["kubernetes","k8s","cri-o"]
---

While preparing the CKA exam, I have been using [minikube][100] and [kubeadm][200] to bootstrap kubernetes clusters. Those tools are very nice but I wanted to understand all the details of a full setup. The best for this is the excellent &#8220;[Kubernetes The Hard Way][300]&#8221; tutorial from Kelsey Hightower.

I wanted to do the setup on-premises (using VMs), meaning no cloud provider. So I had to &#8220;adapt&#8221; the tutorial accordingly. As I have spent some time getting everything up and running, I have decided to share my experience through a series of posts.

This is just an adaptation of THE guide and a sharing of my experience, which I hope will be helpful to others.

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

#### [Next: Plan the Installation and Provisioning VMs >][1]

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
 [100]: https://github.com/kubernetes/minikube
 [200]: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/
 [300]: https://github.com/kelseyhightower/kubernetes-the-hard-way
