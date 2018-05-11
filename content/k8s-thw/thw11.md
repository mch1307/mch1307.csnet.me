---
title: "On-Prem k8s | Part 11"
linktitle: "Kubernetes Cluster Conformance"
description: "A note on Kubernetes Cluster Conformance"
type: "itemized"
author: "mch1307"
date: "2018-05-11"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 12
draft: false
url: /k8s-thw/part11/
---

After having setup the Kubernetes cluster as described in this guide, I have tried to validate it against the ["CNCF K8s Conformance Tests"][20].
The easiest and standard tool to do that is [Heptio's Sonobuoy][21]. Head to ["Sonobuoy Scanner tool"][22] web site, Click on "Scan your cluster", copy the generated `kubectl` command and run it on your cluster.

Unfortunately the result was not the one I was expecting :disappointed:

The first runs were timing out, so I added more resource to my VMs and finally changed the timeout parameter in the downloaded sonobuoy yaml file. Which lead me to this result:

![](/img/2018/05/sonobuoy-ko.png)


After some long investigations I found out that those failure were "just" due to the fact that the kube-apiserver running on the master nodes were not able to access the pod and service network.

The only way I found to fix this is to setup `containerd`, `kubelet` and `kube-proxy` on the master nodes so that the Weave DaemonSet will launch a Weave pod on your master. After that, your master nodes will be able to access the pod and network networks. After that, the Sonobuoy result were ok:

![](/img/2018/05/sonobuoy-ok.png)

In order to avoid regular pods to be scheduled on the control plane nodes, I have setup taints on those nodes and added toleration in the Weave DaemonSet so that only this pod will eventually be run on the master nodes.




#### [< Deploying Cluster Add-ons][9]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part10
 [20]: https://github.com/cncf/k8s-conformance
 [21]: https://github.com/heptio/sonobuoy
 [22]: https://scanner.heptio.com/