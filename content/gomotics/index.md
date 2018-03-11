---
title: gomotics
author: mch1307
type: page
date: 2017-08-27T20:36:33+00:00

---
[![][1]][2] [![][3]][4] [![Coverage Status][5]][6]

## Overview

Gomotics is a small program written in Go that aims offering easy to consume Rest API endpoints to Niko Home Control and act as interface between Niko Home Control and Jeedom. It can be used to solely link NHC to Jeedom or to build an UI to perform simple, day to day operations.

## Features

  * NHC zero conf: automatically discovers the Niko Home Control
  * NHC switches
  * NHC dimmers
  * Interface with Jeedom (matches  NHC/Jeedom on name and location)
  * Automatically create NHC rooms, switches and dimmers in Jeedom

## Demo

{{< vimeo 242817325 >}}

## Installation

Navigate to the gomotics releases page. Select the latest version and download the gomotics executable for your platform. Unzip and save the executable file.

A docker image is automatically build with Travis-CI. It is available on [docker hub][7].

## Configuration

Gomotics can run with minimal configuration. It will automatically discover the NHC controller on the LAN and populate other parameters with default value. If you want gomotics to connect to Jeedom, you can either edit a config file in TOML or set the parameters as env variables. More detailed information on the [GitHub repository][8]

## Running

#### Binary

Gomotics only takes one optional parameter: the config file. As explained in the configuration section, all the parameters can also be setup as env variables.

> <pre>gomotics -conf path_to_config.file</pre>

#### Docker

<p class="code-line">
  A docker image is automatically build with Travis-CI. It is available on <a href="https://hub.docker.com/r/mch1307/gomotics/">Docker Hub</a>
</p>

<blockquote class="code-line">
  <pre class="code-line">docker run -d -P --net host --name gomotics \
-e JEE_URL=http://jeedom-host/core/api/jeeApi.php \
-e JEE_APIKEY=abcdegf1234 \
-e LOG_PATH=stdout \
mch1307/gomotics</pre>
</blockquote>

## API

Gomotics offers the following http endpoints:

#### /health (GET)

Returns a simple JSON if up and running:

<div>
  <blockquote>
    <pre>{"alive":true}</pre>
  </blockquote>
</div>

#### /api/v1/nhc (GET)

Returns a JSON list of registered items

> <pre>[{"provider":"NHC","id":1,"name":"Light","location":"Kitchen","state":100",value2":0,"value3":0,"JeedomID":"","JeedomUpdState":"","JeedomSubType":""},{"provider":"NHC","id":2,"name":"Light","location":"Office","state":100,"value2":0,"value3":0,"JeedomID":"","JeedomUpdState":"","JeedomSubType":""}]</pre>

#### /api/v1/nhc/{id} (GET)

Return a JSON containing the item id provided in the url

> <pre>{"provider":"NHC","id":1,"name":"Light","location":"Kitchen","state":100,"value2":0,"value3":0,"JeedomID":"","JeedomUpdState":"","JeedomSubType":""}</pre>

#### /api/v1/nhc/{id}/{value} (POST)

Send an action to NHC controller (switch, dimmer):

> <pre>/api/v1/nhc/1/100</pre>

#### /api/v1/jeedom/{id}/{value} (GET)

Endpoint to be called from Jeedom in order to drive Niko device. id is the Jeedom equipment id and value is the desired state

> <pre>/api/v1/jeedom/1/100</pre>

#### /events

Websocket endpoints. Provides a JSON message each time an item is updated:

> <pre>{"provider":"NHC","id":30,"name":"Light","location":"Kitchen","state":0,"value2":0,"value3":0,"JeedomID":"","JeedomUpdState":"","JeedomSubType":""}</pre>

 [1]: https://img.shields.io/github/tag/mch1307/gomotics.svg
 [2]: https://github.com/mch1307/gomotics/releases
 [3]: https://travis-ci.org/mch1307/gomotics.svg?branch=master
 [4]: https://travis-ci.org/mch1307/gomotics
 [5]: https://coveralls.io/repos/github/mch1307/gomotics/badge.svg?branch=master
 [6]: https://coveralls.io/github/mch1307/gomotics?branch=master
 [7]: https://hub.docker.com/r/mch1307/gomotics/
 [8]: https://github.com/mch1307/gomotics