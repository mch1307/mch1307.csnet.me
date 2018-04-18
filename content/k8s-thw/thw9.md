---
title: "On-Prem k8s | Part 9"
linktitle: "On-Prem k8s | Part 9"
description: "Configuring Networking"
type: "itemized"
author: "mch1307"
date: "2018-04-11"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 10
draft: true
url: /k8s-thw/part9/
---
## Weave

We will use [weave][20] version 2.3.0 as our network overlay. 

It is really easy to install as a [Kubernetes Addon][21] and does not require additional configuration.

Here is the basic command from the Weave documentation:

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

We need to customize the default configuration to reflect our custom POD_CIDR network (10.16.0.0/16). We can customize the yaml manifest by passing the **IPALLOC_RANGE** option to the HTTP get:

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.16.0.0/16"
```
Output:

```bash
serviceaccount "weave-net" created
clusterrole.rbac.authorization.k8s.io "weave-net" created
clusterrolebinding.rbac.authorization.k8s.io "weave-net" created
role.rbac.authorization.k8s.io "weave-net" created
rolebinding.rbac.authorization.k8s.io "weave-net" created
daemonset.extensions "weave-net" created
```

If you prefer checking the yaml manifest before applying to Kubernetes, just **wget** the above url.


## Checks

Verify pods are up and running:

```bash
kubectl get pod --namespace=kube-system -l name=weave-net
```

Output:

```bash
NAME              READY     STATUS    RESTARTS   AGE
weave-net-fwvsr   2/2       Running   1          4h
weave-net-v9z9n   2/2       Running   1          4h
weave-net-zfghq   2/2       Running   1          4h
```

All nodes should now be in state `Ready`

#### [Next: Deploying Cluster Add-ons >][10]

#### [< Previous: Generating kubectl config][8]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [10]: /k8s-thw/part10
 [20]: https://www.weave.works/docs/net/latest/kubernetes/kube-addon/
 [21]: https://kubernetes.io/docs/concepts/cluster-administration/addons/
