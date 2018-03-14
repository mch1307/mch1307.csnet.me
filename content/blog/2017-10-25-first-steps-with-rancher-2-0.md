---
title: First steps with Rancher 2.0
author: mch1307
type: post
date: 2017-10-25
url: /2017/10/25/first-steps-with-rancher-2-0/
featured: rancher.png
featuredpath: date
categories:
  - containers
tags:
  - rancher
archives: ["2017"]
---
Rancher Labs has released [Rancher 2.0 Tech Preview][1] on 26th of September. The 2.0 release is a significant one as it brings many changes compared to 1.x versions. Based on current Rancher users feedback, market trends (almost all major infrastructure providers offer &#8220;Kubernetes-As-A-Service&#8221;), and some vision (&#8220;Kubernetes everywhere&#8221;), they have re-engineered Rancher 2.0 to be fully based on [Kubernetes][2].

To me, one of the key strengths of Rancher so far was to bring easiness to deploy and manage a container orchestrator. Let&#8217;s see if this is still the case with v2.0.

### Installation:

I will use on premises VMs to host my Rancher 2.0 setup, and to be 100% rancher, I will use RancherOS 😉

As described in the [quick start guide][3], the requirements for Rancher server is a linux host with kernel version 3.10+ and Docker installed (v1.12.6, v1.13.1, v17.03-ce, or v17.06-ce). In terms of hardware resources, I have allocated 2vCPUs, 2Gb RAM and 30Gb disk.

For the hosts, I have created 2 VMs with 4vCPU, 4Gb RAM and 30Gb of disk.

#### Rancher server:

The installation (in non HA) could not be simpler:

> sudo docker run -d &#8211;restart<span class="o">=</span>unless-stopped -p 8080:8080 rancher/server:preview

This simple command will take a few minutes to complete. Under the hood, Rancher will spin up a full Kubernetes cluster for us!

![](/wp-content/uploads/2017/10/screen-shot-10-19-17-at-12-56-pm.png)

For an HA setup, we will have to wait further guidance from Rancher Labs with later pre-release or as v2 becomes GA.

#### Hosts

Once the Rancher server is up and running, you can access the UI on port 8080. Rancher will directly show a screen to add hosts. Here we have two options: add hosts or import existing k8s cluster. A third option might arrive in future release: spin up a new k8s cluster on cloud-provider platform like GKE, Aws, ..

We will choose the &#8220;add hosts&#8221;. Rancher can add host from several sources, being VM, bare-metal or cloud provided hosts (ie AWS, GCE, Azure, &#8230;).

As with previous versions, Rancher will provide you with a docker command to be executed on the target host in order to join Rancher:

![](/wp-content/uploads/2017/10/screen-shot-10-19-17-at-01-44-pm.png)

Run that command on each node you want to add. After a few minutes, head to the Host part of the UI

![](/wp-content/uploads/2017/10/screen-shot-10-19-17-at-01-49-pm.png)

That&#8217;s it, our cluster is now up and running!

### Exploring Rancher 2.0

#### Environments and Clusters:

An environment is a namespace where applications, containers and services are grouped in; a cluster is a grouping of compute resources. One environment can only run on one cluster, a cluster can host several environments.

Native Kubernetes:

The native Kubernetes tools can easily be used with Rancher 2.0

![](/wp-content/uploads/2017/10/screen-shot-10-22-17-at-08-46-pm.png)

You can use the pure Kubernetes dashboard,

![](/wp-content/uploads/2017/10/screen-shot-10-22-17-at-08-48-pm.png)

install and easily configure the kubectl CLI

![](/wp-content/uploads/2017/10/screen-shot-10-22-17-at-08-51-pm.jpg)

or launch it from your web browser:

![](/wp-content/uploads/2017/10/screen-shot-10-22-17-at-08-50-pm-001.png)

#### Catalog:

Rancher has of course kept its catalog, from which new apps can easily be deployed. The library contains, among others, [Helm][4], the &#8220;Kubernetes Package Manager&#8221; which allows easy apps deployment on Kubernetes.

![](/wp-content/uploads/2017/10/screen-shot-10-22-17-at-08-59-pm.png)

### Roadmap:

Rancher Labs targets a first GA release in early 2018. Several features are still being worked on and will be released in different alpha and/or beta releases in the following weeks:

  * Volume Support
  * RBAC
  * Load balancer
  * Secret Management
  * Integrated Logging and Monitoring
  * Integrated CI/CD

### Conclusion

While Kubernetes is becoming the de-facto containers orchestration platform, it is not the easiest platform to get started with. In Rancher 2.0, Rancher Labs has kept its willingness to bring user-friendliness to the container orchestration space. This version looks promising and I look forward next alpha releases that will include volume support, secret management and all features announced in the roadmap.

### 

&nbsp;

 [1]: http://rancher.com/announcing-rancher-2-0/
 [2]: https://kubernetes.io/
 [3]: http://rancher.com/docs/rancher/v2.0/en/quick-start-guide/
 [4]: https://github.com/kubernetes/helm