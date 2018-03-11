+++
title = "On-prem k8s | Part 1"
linktitle = "On-prem k8s | Part 1"
description = "Planning the Installation and Provisioning VMs"
type = "itemized"
author = "mch1307"
date = "2018-03-04"
featured = ""
featuredpath = ""
featuredalt = ""
categories = ["kubernetes"]
#format = "Android"
link = "#"
share = false
sharesocial = false
socialshare = false
+++

## Machines

We will setup a HA Kubernetes cluster, with 3 control plane nodes and 3 worker nodes.

The OS will be Ubuntu 16.0.4. 

The following table contains the list of hosts to be provisioned.

| Name     | IP address  | Role          |
|----------|-------------|---------------|
| k8sctl1  | 10.32.2.91  | control plane |
| k8sctl2  | 10.32.2.92  | control plane |
| k8sctl3  | 10.32.2.93  | control plane |
| k8swrk1  | 10.32.2.94  | worker        |
| k8swrk2  | 10.32.2.95  | worker        |
| k8swrk3  | 10.32.2.96  | worker        |

We will also need a load balancer in front of the Kubernetes API server. We will use HAProxy 

## Versions



## IPs


### [Next: Provisioning TLS certificates >][2]

 [1]: /k8s-thw/thw1
 [2]: /k8s-thw/thw2
 [3]: /k8s-thw/thw3
 [4]: /k8s-thw/thw4
 [5]: /k8s-thw/thw5
 [6]: /k8s-thw/thw6
 [7]: /k8s-thw/thw7
 [8]: /k8s-thw/thw8
 [9]: /k8s-thw/thw9