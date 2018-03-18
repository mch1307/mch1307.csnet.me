---
title: "On-prem k8s | Part 1"
linktitle: "On-prem k8s | Part 1"
description: "Planning the Installation"
type: "itemized"
author: "mch1307"
date: "2018-03-04"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 2
draft: true
---

## Machines

We will setup a HA Kubernetes cluster, with 3 control plane nodes and 3 worker nodes.

We will also need a load balancer in front of the Kubernetes API server. We will use HAProxy.

The OS will be Ubuntu 16.04 for all the hosts.

The following table contains the list of hosts to be provisioned.

| Name     | IP address  | Role          |
|----------|-------------|---------------|
| k8sctl1  | 10.32.2.91  | control plane |
| k8sctl2  | 10.32.2.92  | control plane |
| k8sctl3  | 10.32.2.93  | control plane |
| k8swrk1  | 10.32.2.94  | worker        |
| k8swrk2  | 10.32.2.95  | worker        |
| k8swrk3  | 10.32.2.96  | worker        |
| haprx1  | 10.32.2.97  | load balancer |

The details of provisionning the VMs is out of scope as they depend on the infrastructure. In my case, I am using Vmware ESX and will manually provison the hosts from a template.

    This tutorial assumes you have the basic infrastructure blocks like DNS service.

## Versions

We will be using the following versions:

* Kubernetes 1.9.3
* docker 
* weave

## Network information

| Network / IP | Description
| --- | ---
| 10.32.2.0/24 | LAN (csnet.me)
| 10.16.0.0 | k8s CIDR network
| 10.10.0.0 | k8s Services network
| 10.10.0.1 | k8s API server
| 10.10.0.10 | k8s dns


#### [Next: Provisioning TLS certificates >][2]

 [1]: /k8s-thw/thw1
 [2]: /k8s-thw/thw2
 [3]: /k8s-thw/thw3
 [4]: /k8s-thw/thw4
 [5]: /k8s-thw/thw5
 [6]: /k8s-thw/thw6
 [7]: /k8s-thw/thw7
 [8]: /k8s-thw/thw8
 [9]: /k8s-thw/thw9