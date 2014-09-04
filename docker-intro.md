
Docker Introduction
===================

This exercise let you:

1. learn and install docker
2. build a static web site on docker

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

### Run Node.js

To run node.js with Docker, you can run:

```bash
$ docker run -it --rm dockerfile/nodejs node
```

which starts the same REPL that you can start typing `node`

Your static website on Docker
-----------------------------

### Project Setup

First of all, create a new directory, and a new git repository, you
might need this shortly:

```bash
$ mkdir mystatic
$ cd mystatic
$ git init
```

### Sample Application

Take the code that has make node famous:

```js
var http = require('http');
var port = process.env.PORT || 1337;
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(port);
console.log('Server running on port', port);
```

and write it to a file named `app.js`.

You can test it locally:
```bash
$ node app.js
```

In another shell:
```bash
$ curl http://localhost:1337
```

### Dockerize!

In order to build a container, we need a `Dockerfile`.

Here is one to build our app:

```
# The world-famous node.js hello world
#
# VERSION 0.0.1

FROM dockerfile/nodejs
MAINTAINER Spongebob

# install tools for building binary addons
RUN apt-get -y update
RUN apt-get -y install build-essential libssl-dev curl python

ADD ./ /src

EXPOSE 1337

ENTRYPOINT ["node", "/src/app.js"]
```

You can build it by typing:
```bash
$ docker build .
.. lots of stuff

Successfully built <containerid>
```

Then:
```bash
docker run -p 80:1337 <containerid>
```

(copy paste the container id from the previous command)


In another shell:
```bash
$ curl http://<DOCKER_HOST>:1337
```

you can get your DOCKER_HOST ip address by typing:
```bash
echo $DOCKER_HOST
```

(if you are on boot2docker)

The end result is here:
https://github.com/nearform/nscale-workshop-intro-docker-sample

