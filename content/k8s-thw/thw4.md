---
title: "On-Prem k8s | Part 4"
linktitle: "On-Prem k8s | Part 4"
description: "Generating the Data Encryption Config"
type: "itemized"
author: "mch1307"
date: "2018-04-11"
featured: ""
featuredpath: ""
featuredalt: ""
categories: [""]
#format: "Android"
link: "#"
weight: 5
draft: false
url: /k8s-thw/part4/
---

# Secret data encryption

Kubernetes supports [data encryption at rest][20] to securely store data in the etcd k/v database.

In this section, we will create a Kubernetes encryption config manifest to specify the resource we want to be encrypted, the encryption mechanism and key.

Later, the kube-api server will be started with the `--experimental-encryption-provider-config` flag in order to enable data encryption at rest

## Encryption key

First, we generate a random key, base64 encoded:

```
ENCRYPTION_KEY=`head -c 32 /dev/urandom | base64`
```

## Kubernetes manifest file

Generate the Kubernetes yaml file that will later be used for enabling secret data encryption. Head [here][21] if you want more information about encryption at rest.

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

## Distribute the file

Copy the generated file to the three controller nodes

```
for instance in k8sctl1 k8sctl2 k8sctl3; do
    scp encryption-config.yaml $instance:~/
done
```



#### [Next: Bootstrapping etcd Cluster >][5]

#### [< Previous: Generating Kubeconfig files][3]

 [1]: /k8s-thw/part1
 [2]: /k8s-thw/part2
 [3]: /k8s-thw/part3
 [4]: /k8s-thw/part4
 [5]: /k8s-thw/part5
 [6]: /k8s-thw/part6
 [7]: /k8s-thw/part7
 [8]: /k8s-thw/part8
 [9]: /k8s-thw/part9
[20]: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
[21]: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration