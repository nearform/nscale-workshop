
Docker Introduction
===================

This exercise let you:

1. know docker
2. install docker on OS X and Linux?
3. build a static web site on docker

What is Docker?
---------------

![Docker](https://d3oypxn00j2a10.cloudfront.net/0.9.0/images/pages/brand_guidelines/small_v.png)

Docker is coolest tech for deploying your applications, Docker allows you to write a 'recipe' for building your application in a file called `Dockerfile`. In it, you write commands using bash, the same tool you know and love!

Thanks to Docker, an application is wrapped in a _container_.

### Installation

If you are Linux, follow https://github.com/nearform/nscale/wiki/Linux-Setup-Guide

If you are on Mac OS X, follow https://github.com/nearform/nscale/wiki/OS-X-Setup-Guide

Obviously, if you still have the cited software, you can skip the relevant steps.

### Basic kwnoledge of Docker

To check if you instance works, you should be able to run:

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
`````

Which means that you have no containers running. Don't worry, we'll build one shortly.

