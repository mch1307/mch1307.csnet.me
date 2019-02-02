---
title: "vaultlib: a Go library for reading secrets from Vault"
linktitle: "vaultlib"
description: "A simple, lightweight Go library for getting secrets from Hashicorp Vault"
type: "post"
author: "mch1307"
date: "2019-02-02"
lastmod: "2019-02-02"
featured: "vault-go.png"
featuredpath: "date"
featuredalt: ""
draft: false
url: "/2019/02/vaultlib/"
archives: "2019"
tags: ["go","golang","vault"]
categories: ["go"]
---

Moving applications to containers can require some considerable development efforts. One of the important transitions is the application's configuration. In traditional deployments, configuration is usually performed "individually" on the target server. Some organizations use deployment or automation tools, others might even do that manually.

As part of their configuration, most applications will require different credentials for accessing services they consume (database, APIs,.. ). HashiCorp's Vault is a great tool to manage such data.

In some cases, an application can hardly be changed in order to be able to get its configuration from another source than the traditional server config file. You might then want to develop a utility that can prepare a configuration file before the application get started. Having to develop such utilities led me to write my first go library: [vaultlib][1]. 

`vaultlib` is a simple, lightweight go library allowing to easily read secrets from Vault KV using it's HTTP APIs.

It currently offers the following features:

* Can be configured through environment variables or programatically.
* Vault connection through AppRole or token.
* Vault `KV` secrets (v1 and v2)
* Token renewal management
* Execute raw queries against Vault

The project is available [here][1]. It contains a [sample folder][2] with working example.
You can consult the [GoDoc](https://godoc.org/github.com/mch1307/vaultlib) for the complete package documentation.


Hope it's usefull ;)


 [1]: https://github.com/mch1307/vaultlib
 [2]: https://github.com/mch1307/vaultlib/tree/master/sample
