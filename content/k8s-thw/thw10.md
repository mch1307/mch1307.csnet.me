---
title: "On-Prem k8s | Part 10"
linktitle: "On-Prem k8s | Part 10"
description: "Deploying Cluster Add-ons"
type: "itemized"
author: "mch1307"
date: "2018-04-11"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 11
draft: true
url: /k8s-thw/part10/
---

## CoreDNS

```bash
kubectl get pod --all-namespaces -l k8s-app=coredns -o wide
```

```bash
NAMESPACE     NAME                       READY     STATUS    RESTARTS   AGE       IP             NODE
kube-system   coredns-85646c4d6b-2bhzf   1/1       Running   0          9m        10.16.128.14   k8swrk1
kube-system   coredns-85646c4d6b-2cdgq   1/1       Running   0          3m        10.16.0.13     k8swrk2
kube-system   coredns-85646c4d6b-l7dnt   1/1       Running   0          26s       10.16.224.11   k8swrk3
```

#### [< Configuring Networking][9]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9
