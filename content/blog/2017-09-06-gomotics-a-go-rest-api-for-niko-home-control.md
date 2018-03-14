---
title: gomotics, a go rest API for Niko Home Control
author: mch1307
type: post
date: 2017-09-06T10:53:02+00:00
url: /2017/09/06/gomotics-a-go-rest-api-for-niko-home-control/
categories:
  - gomotics
archives: ["2017"]

---
I am developing a small domotics back-end in Go, mainly as a learning method. I already did that [in NodeJs][1] and as I wanted to learn Golang, I decided to re-develop same kind of tool. This time, I am doing it more &#8220;properly&#8221; in terms of testing coverage. And as Go is a compiled language, I have setup an automated build, test and release process using Travis CI, Coveralls and [goreleaser][2]. Those tools are amazing!

The code is available [here][3] and the small doc [here][4]

As this is in the early stages, I would welcome anyone wanting to do some tests 😉

#### Current features:

  * Niko Home Control: 
      * zero conf, auto detects the NHC controller on the network (LAN)
      * switches
      * dimmer

#### Roadmap:

  * Support NHC thermostat and energy modules
  * API endpoint to log external domotics components
  * GUI?
  * Link with Jeedom

&nbsp;

 [1]: https://github.com/mch1307/jeedom-nhc
 [2]: https://github.com/goreleaser/goreleaser
 [3]: https://github.com/mch1307/gomotics
 [4]: https://blog.csnet.me/gomotics/