
Docker Introduction
===================

This exercise walks through:

1. installing Docker
2. grasping Docker fundamentals
3. deploying a static website inside a Docker container

What is Docker?
---------------

![Docker](https://d3oypxn00j2a10.cloudfront.net/0.9.0/images/pages/brand_guidelines/small_v.png)

Docker is an awesome tool we can use to deploy production systems. It allows us to isolate
our code in completely clean system environments. The `Dockerfile` is a sort of installation
'recipe' that can be used to initialize a system environment. A Docker container is a 
fresh Linux environment that bootstraps from a host systems Linux kernel - this gives us a lot
of the benefits of the VM but without the slow execution and bulk often associated with
VMs.

### Installation

If you are Linux, follow https://github.com/nearform/nscale/wiki/Linux-Setup-Guide

If you are on Mac OS X, follow https://github.com/nearform/nscale/wiki/OS-X-Setup-Guide

If any software listed in the guides is already installed on your system, you can
of course skip those steps. 

### Basic knowledge of Docker

Let's do a quick sanity check:

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
`````

Docker hasn't complained so we're good to go, the output shows
that we have no containers running. Don't worry, we'll build one shortly.

### Run Node.js in a Docker container

The following command will start a Node REPL inside a Docker container

```bash
$ docker run -it --rm node node
```

This command pulls in the official `node` image from the Docker hub and
tells Docker to open an interactive terminal into the
container and runs the `node` executable, thus giving us the 
Node REPL.


Our static website on Docker
-----------------------------

### Project Setup

First let's create a new directory and initialize a new git repo
inside it:

```bash
$ mkdir mystatic
$ cd mystatic
$ git init
```

### Sample Application

Let's use the canonincal Node.js web server example:

```js
var http = require('http');
var port = process.env.PORT || 1337;
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(port);
console.log('Server running on port', port);
```

We'll save that in a file named `app.js`.

We can test it in the usual way:
```bash
$ node app.js
```

Then in another shell:
```bash
$ curl http://localhost:1337
```

### Dockerize!

Now we're going to build a container around the web server.

To get started we need a recipe list: the `Dockerfile`:

```
# The world-famous node.js hello world
#
# VERSION 0.0.1

FROM node
MAINTAINER Spongebob

ADD ./ /src

EXPOSE 1337

ENTRYPOINT ["node", "/src/app.js"]
```

We can build it by typing:
```bash
$ docker build .
```

Which will eventually output:
```bash
Successfully built <containerid>
```

Next we run our container with:

```bash
docker run -p 80:1337 <containerid>
```
(copy paste the container id from the previous command)


Finally we can make a request to our container, in another
shell.

If we're using OS X we're using boot2docker which is actually a Linux VM,
we need to use the $DOCKER_HOST environment variable to access the VM's
localhost.

```bash
$ curl http://$(boot2docker ip):80
```
Otherwise, if we're using Linux we simply request localhost:

```bash
$ curl http://localhost:80
```


Here's one we made earlier: 
<https://github.com/nearform/nscale-workshop-intro-docker-sample>

[Next up: nscale intro](./nscale-intro.md)

