---
title: "On-prem k8s | Part 1"
linktitle: "On-prem k8s | Part 1"
description: "Planning the Installation"
type: "itemized"
author: "mch1307"
date: "2018-04-11"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 2
draft: true
url: /k8s-thw/part1/
---

## Machines

{{% alert theme="info" %}}This tutorial assumes you already have the basic infrastructure blocks like DHCP or DNS up and running.{{% /alert %}}

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

{{% alert theme="info" %}}The details of provisionning the VMs is out of scope as they depend on the infrastructure. In my case, I am using Vmware ESX and will manually provison the hosts from a template.{{% /alert %}}

## Versions

We will be using the following versions:

* Kubernetes v1.10.0
* etcd v3.2.18
* docker
* weave

## Network information

This section contain additional network information to be clear on LAN, k8s networks and other important IPs

| Network / IP | Description
| --- | ---
| 10.32.2.0/24 | LAN (csnet.me)
| 10.16.0.0/16 | k8s Pod network
| 10.10.0.0/22 | k8s Service network
| 10.10.0.1 | k8s API server
| 10.10.0.10 | k8s dns


Once the machines are ready, we can head to next part

#### [Next: Provisioning TLS certificates >][2]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9