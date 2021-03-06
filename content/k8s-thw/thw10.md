---
title: "On-Prem k8s | Part 10"
linktitle: "On-Prem k8s | Part 10"
description: "Deploying Cluster Add-ons"
type: "itemized"
author: "mch1307"
date: "2018-04-18"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 11
draft: false
url: /k8s-thw/part10/
---

## CoreDNS

[CoreDNS][20] is a dns server written in Go. It is a [Cloud Native Computing Foundation][21] incubating project and an eventual replacement to **kube-dns**. It has been promoted to beta with Kubernetes 1.10.

CoreDNS is configured through a _Corefile_, and in Kubernetes, we will use a _ConfigMap_

We will simply deploy it using the following [manifest file][22]

```bash
kubectl apply -f https://raw.githubusercontent.com/mch1307/k8s-thw/master/coredns.yaml
```

```bash
serviceaccount "coredns" created
clusterrole.rbac.authorization.k8s.io "system:coredns" created
clusterrolebinding.rbac.authorization.k8s.io "system:coredns" created
configmap "coredns" created
deployment.extensions "coredns" created
service "kube-dns" created
```

### Checks

Verify pods are up and running:

```bash
kubectl get pod --all-namespaces -l k8s-app=coredns -o wide
```

```bash
NAMESPACE     NAME                       READY     STATUS    RESTARTS   AGE       IP             NODE
kube-system   coredns-85646c4d6b-2bhzf   1/1       Running   0          9m        10.16.128.14   k8swrk1
kube-system   coredns-85646c4d6b-2cdgq   1/1       Running   0          3m        10.16.0.13     k8swrk2
```

Verify DNS resolution:

Launch a simple busybox deployment: 

```bash
kubectl run busybox --image=busybox --command -- sleep 3600
```

Get the pod's exact name:

```bash
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

Check DNS resolution from within the busybox pod:

```bash
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

```bash
Server:    10.10.0.10
Address 1: 10.10.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.10.0.1 kubernetes.default.svc.cluster.local
```


#### [Next: Kubernetes Cluster Conformance >][10]

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
 [10]: /k8s-thw/part11
 [20]: https://github.com/coredns/coredns
[21]: https://www.cncf.io/
[22]: https://github.com/mch1307/k8s-thw/blob/master/coredns.yaml