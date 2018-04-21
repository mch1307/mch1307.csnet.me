---
title: "On-Prem k8s | Part 8"
linktitle: "On-Prem k8s | Part 8"
description: "Generating kubectl config"
type: "itemized"
author: "mch1307"
date: "2018-04-16"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 9
draft: false
url: /k8s-thw/part8/
---

In this section we will generate a kubeconfig file for the **kubectl** k8s utility based on the **admin** user credentials.

{{% alert theme="info" %}}Run the commands from the same directory used to generate the admin client certificates in <a href=" /k8s-thw/part2">Part 2</a>.{{% /alert %}}

## kubeconfig file

The kubeconfig file contains the information allowing to remotely connect to a Kubernetes cluster. It stores the following information:

```bash
cluster
    Name
    API server URL
    CA data
user
    Name
    Client cert data
context
    Name
    Cluster name
    User name
```

The **context** links the cluster information to the user information.

The config file is generated with **kubectl** and is usually stored in ~/.kube

As we have setup the API server in HA mode, we will use the load balancer's IP (**10.32.2.97**) as the API server IP.


```bash
kubectl config set-cluster kubernetes \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://10.32.2.97:6443
```

```bash
kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem
```

```bash
kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=admin
```

```bash
kubectl config use-context kubernetes
```

## Tests

Check the health of the remote Kubernetes cluster:

```bash
kubectl get componentstatuses
```

Result:

```bash
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

Result:

```bash
NAME      STATUS       ROLES     AGE       VERSION
k8swrk1   NotReady     <none>    1h        v1.10.1
k8swrk2   NotReady     <none>    2h        v1.10.1
k8swrk3   NotReady     <none>    2h        v1.10.1
```

## Distribute

Now that we have our working kubeconfig file, we can copy it to the home folder in the .kube sub-directory.

If you already have some Kubernetes cluster configured, you will need to either "merge" your current config file with the new one or specify the config file used when invoking **kubectl**

More information on organizing access to Kubernetes clusters is available [here][20]

#### [Next: Configuring Networking >][9]

#### [< Previous: Bootstrapping k8s Worker Nodes][7]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9
[20]: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/