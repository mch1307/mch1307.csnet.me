---
title: Setting up CI/CD pipeline for Golang using Travis-CI, Coveralls, goreleaser and Docker
author: mch1307
type: post
date: 2017-09-11T21:11:16+00:00
url: /2017/09/11/setting-up-cicd-pipeline-for-golang-using-travis-ci-coveralls-goreleaser-and-docker/
featured: ci.png
featuredpath: date
categories:
  - go
  - CI/CD
tags:
  - coveralls
  - docker
  - github releases
  - go
  - goreleaser
  - travis-ci
archives: ["2017"]
---
## Overview

In the [previous post][1], I was introducing my personal project &#8220;gomotics&#8221;, a domotics API for Niko Home Control, written in Go. In this post I will detail how I setup the &#8220;build &#8211; test &#8211; release&#8221; pipeline for this project.

## Objectives

The objectives for this &#8220;pipeline&#8221; are:

  * build the code for different platforms
  * run the unit tests (sometimes they are more integration tests)
  * measure the coverage
  * in case of release tag, release the built binaries
  * build a docker image and release it on Docker Hub

To achieve those objectives we will use [GitHub][2] for sources, [Travis-CI][3] for building, [Coveralls][4] for coverage, [goreleaser][5] to automate the releases to GitHub pages and [Docker Hub][6] for our container image release.

## Build

The first step is to be able to build the code. Doing this with Travis-CI is pretty straight forward.

If you don&#8217;t have an account on Travis, simply signup using your GitHub account and accept the GitHub access permissions. Wait for your account to be synchronized with GitHub and then enable the repository you want to build. Next, create a [repo token on GitHub][7] save it and set it up in your [Travis-CI environment variables][8].

Add a &#8220;.travis.yml&#8221; file to the root of your project as follows

<div>
  <blockquote>
    <pre># .travis.yml
language: go
go:
  - 1.8.3</pre>
  </blockquote>
</div>

<div>
</div>

## Coverage

We will use Coveralls and the [go plugin goveralls][9] for the code coverage. Coveralls will allow you to compute coverage and followup on their web interface:

![](/wp-content/uploads/2017/09/screen-shot-09-11-17-at-07-01-pm.png)


Signup to [Coveralls.io][4] using your GitHub account and add your project&#8217;s GitHub repo.

Then add the following lines to your .travis.yml file:

> <div>
>   <pre>before_install:
  - go get github.com/mattn/goveralls
script:
  - $HOME/gopath/bin/goveralls -v -service=travis-ci</pre>
> </div>

## Release

Once the binaries are built, we need to release them. We will publish those to GitHub release page using goreleaser. This tool will publish our binaries to GitHub Release Page and create a nice looking changelog with the commit comments:

![](/wp-content/uploads/2017/09/screen-shot-09-09-17-at-10-54-pm.png)


To implement goreleaser, add a .goreleaser.yml file to the root of your project:

<div>
  <blockquote>
    <div>
      <pre># .goreleaser.yml
# Build customization
builds:
  - main: app.go
binary: gomotics
goos:
  - windows
  - darwin
  - linux
goarch:
  - amd64
  - arm64
# Archive customization
archive:
  format: tar.gz</pre>
    </div>
  </blockquote>
</div>

<div>
  The add the following in your travis file:
</div>

<div>
  <blockquote>
    <pre>after_success:
- test -n "$TRAVIS_TAG" && curl -sL https://git.io/goreleaser | bash</pre>
  </blockquote>
</div>

## Docker

We also want to deploy the application as a Docker image. Let&#8217;s use Travis to build our image and then push it to Docker Hub.

Setup the docker hub credentials on Travis-CI web interface. Password should be secured:

![](/wp-content/uploads/2017/09/screen-shot-09-11-17-at-01-00-pm.png)


Then add the following lines to the travis file in your project:

> <div>
>   <pre>services:
  - docker</pre>
>   
>   <div>
>     <pre>after_success:
  - mkdir -p dist
  - CGO_ENABLED="0" GOARCH="amd64" GOOS="linux" go build -a -installsuffix cgo -o ./dist/gomotics
  - docker login -u $DOCKER_USER -p $DOCKER_PASSWORD
  - export REPO=$DOCKER_USER/gomotics
  - export TAG=`if [ "$TRAVIS_BRANCH" == "master" ]; then echo "latest"; else echo $TRAVIS_TAG ; fi`
  - echo $REPO:$TAG
  - docker build -f Dockerfile -t $REPO:$TAG .
  - docker push $REPO</pre>
>   </div>
> </div>

Those lines instruct the following actions:

  * get docker available in Travis environment
  * create a dist directory
  * build the project for the specified target (linux amd64) to the dist folder
  * compile the release version for Docker Hub image
  * build and push the image to Docker Hub

## Improve your project &#8220;look&#8221;

Once you have everything in place, you can improve you project &#8220;look&#8221; by using badges. Those allow any visitor to have a quick overview on how the project is being developed/maintained.

![](/wp-content/uploads/2017/09/screen-shot-09-11-17-at-01-35-pm.png)

And then the same for Docker Hub 😉

![](/wp-content/uploads/2017/09/screen-shot-09-11-17-at-01-37-pm.png)


## Conclusion

We have seen that setting up an automated CI/CD pipeline is not very complex and that using open source and/or free tools can definitely ease your life and improve you development project life cycle.

 [1]: http://blog.csnet.me/2017/09/06/gomotics-a-go-rest-api-for-niko-home-control/
 [2]: http://github.com
 [3]: http://travis-ci.org
 [4]: http://coveralls.io
 [5]: https://github.com/goreleaser/goreleaser
 [6]: http://hub.docker.com
 [7]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/#
 [8]: https://docs.travis-ci.com/user/environment-variables/
 [9]: https://github.com/mattn/goveralls