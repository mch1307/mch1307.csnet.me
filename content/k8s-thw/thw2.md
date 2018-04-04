---
title: "On-Prem k8s | Part 2"
linktitle: "On-Prem k8s | Part 2"
description: "Provisioning TLS certificates"
type: "itemized"
author: "mch1307"
date: "2018-03-04"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 3
draft: true
url: /k8s-thw/part2/
---

# PKI Infrastructure

We will use `cfssl` and `cfssljson` utilities from [CloudFlare’s open source PKI toolkit][11].

If you are interested in building a complete PKI infrastructure, I invite you to read [this interesting post][10] on CloudFlare's blog

## Tools Installation

The easiest way to install the two utilities is to download the prebuilt packages.

```
curl -o cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
curl -o cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
```

`chmod +x cfssl cfssljson`

`sudo mv cfssl cfssljson /usr/local/bin/`

An alternate way is to build them from source code as decribed [here][12].

## CA provisioning

We will setup a Certificate Authority that will allow us to generate TLS certifiates.


Create the CA configuration file
```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "26280h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "26280h"
      }
    }
  }
}
EOF
```

Create the CA cert request

```
cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LU",
      "L": "Luxembourg",
      "O": "Kubernetes",
      "OU": "CA"
    }
  ]
}
EOF
```

Finally, generate the CA certificate and private key

`cfssl gencert -initca ca-csr.json | cfssljson -bare ca`

The following files have been generated:

```
ca.pem
ca-key.pem
```

# TLS cerificates generation

## Admin certificate

Create the admin client certificate signing request:

```
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LU",
      "L": "Luxembourg",
      "O": "system:masters",
      "OU": "Kubernetes"
    }
  ]
}
EOF
```

Generate the admin client certificate and private key:

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```

The following files have be generated:

```
admin-key.pem
admin.pem
```

## Kubelet certificate

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

We will generate a certificate and private key pair for each worker node:

```
for instance in k8swrk1 k8swrk2 k8swrk3; do
cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LU",
      "L": "Luxembourg",
      "O": "system:nodes",
      "OU": "Kubernetes"
    }
  ]
}
EOF

IP=`nslookup ${instance} | awk '/^Address: / { print $2 }'`

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},$IP \
  -profile=kubernetes \
  ${instance}-csr.json | cfssljson -bare ${instance}
done
```

The following files have been generated:

```
k8swrk1.pem
k8swrk1-key.pem
k8swrk2.pem
k8swrk2-key.pem
k8swrk3.pem
k8swrk3-key.pem
```

## Kube-proxy certificate

```
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LU",
      "L": "Luxembourg",
      "O": "system:node-proxier",
      "OU": "Kubernetes"
    }
  ]
}
EOF
```
Generate the kube-proxy certificate

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
```

Generated files: 

```
kube-proxy-key.pem
kube-proxy.pem
```

## API Server Certificate

{{% alert theme="info" %}}The certificate's SAN will include the following IPs:

    10.32.2.91/2/3 -> the 3 k8s control plane hosts
    10.32.2.97 -> the Kube-api load balancer
    10.10.0.1 -> the Kube-api ("internal") service IP
    {{% /alert %}}


Create the Kubernetes API Server certificate signing request:

```
cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LU",
      "L": "Luxembourg",
      "O": "Kubernetes",
      "OU": "Kubernetes"
    }
  ]
}
EOF
```
Generate the Kubernetes API Server certificate and private key:

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.2.91,10.32.2.92,10.32.2.93,10.32.2.97,10.10.0.1,127.0.0.1,kubernetes.default \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
```

Generated files:

```
kubernetes-key.pem
kubernetes.pem
```

## Distribute the certificates

Now it's time to copy the generated cert/keys to our hosts:

Master nodes:

```
for instance in k8sctl1 k8sctl2 k8sctl3; do
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem $instance:~/
done
```

Worker nodes:

```
for instance in k8swrk1 k8swrk2 k8swrk3; do
scp ca.pem ${instance}-key.pem ${instance}.pem $instance:~/
done
```


#### [Next: Generating Kubeconfig files >][3]

#### [< Previous: Planning the Installation and Provisioning VMs][1]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9
 [10]: https://blog.cloudflare.com/how-to-build-your-own-public-key-infrastructure/
 [11]: https://www.cloudflare.com/
 [12]: https://github.com/cloudflare/cfssl
 