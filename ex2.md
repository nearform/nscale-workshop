Build a system with NScale
===================

This tutorial covers:

1. Creating a new system
2. Hooking up a new container definition
3. Building a new container
4. Run!

Create a new System
-------------------

A system is just a git repository, like the one you cloned in
the previous step.
That system definition contains everything `nscale` needs to
know to deploy your application.

Let's go ahead and create a new system with:

```bash
$ nsd sys create
prompt: name:  workshop
prompt: namespace:  nscale
create system: workshop with namespace: nscale?
prompt: confirm (y/n):  y
ok
```

Now we can check that everything is as expected:

```bash
$ nsd sys list
Name                           Id
workshop                       2de30af9-fdc4-41ff-9b88-cd47eacb7f77
```

(In `nscale`, you can shorten commands down to 3 chars.)

The `list` command will return any systems that `nsd` is aware of, 
including our new `nscale_workshop` system:

The `nsd sys create` creates a new git repository in the current
directory. Go ahead and have a look at the files in there.

```
$ cd workshop
$ tree # you may not have tree installed
.
├── definitions
│   └── services.js
├── deployed.json
├── map.js
├── system.js
├── system.json
└── timeline.json
```

1 directory, 6 files

There are three files:

1. `system.js` is the _source of knowledge_ of a system managed by
   nscale. We will edit this shortly.
2. `definitions/services.js` contains the definitions on how to build
   our containers. We will edit this shortly.
3. `deployed.json` contains the currently deployed system, you should
   not edit this file directly.
4. `system.json` contains the system definition, all the containers and
   how to build them, you should not edit this file directly.
5. `timeline.json` contains all the events that happened in the system,
    you should not edit this file directly.

Preparing the Application
-------------------------

Containers should live in a git repository, this allows `nscale` to check 
out the repo and build the containers for us.

Let's take the example we built in [exercise 1](https://github.com/nearform/nscale-workshop/blob/master/docker-intro.md), and place it in a git repository on github.

In this tutorial, our test repo will be: git@github.com:nearform/nscale-workshop-intro-docker-sample.git

Add a container definition
--------------------------

Let's open `definitions/services.js` in you favorite editor. It
currently looks like:

```js
exports.root = {
  type: 'container'
};

// add here more definitions
```

To begin defining our system, we need to change it to:

```js
exports.root = {
  type: 'container'
};

exports.web = {
  type: 'process',
  specific: {
    repositoryUrl: 'git@github.com:nearform/nscale-workshop-intro-docker-sample.git',
    execute: {
      args: '-p 1337:1337 -d',
    }
  }
};
```

Build a container
-----------------

Let's open `system.js` in you favorite editor. It currently looks like this:

```js
exports.name = 'workshop';
exports.namespace = 'nscale';
exports.id = '2de30af9-fdc4-41ff-9b88-cd47eacb7f77';

exports.topology = {
  local: {
  }
};
```

This system is empty, let's add our containers:

```js
exports.name = 'workshop';
exports.namespace = 'nscale';
exports.id = '2de30af9-fdc4-41ff-9b88-cd47eacb7f77';

exports.topology = {
  local: {
    root: ['web']
  }
};
```

In order for everything to work fine, we need to update one more file, `map.js`, to:

```js
exports.types = {
  local: {
  }
}

exports.ids = {
  local: {
    root: { id: '10' },
    web: { name: 'web' }
  }
}
```

This file helps with the matching between the _symbolic names_ used in
the definitions and the analysis run by the nscale.

This abstract system definition __must__ be compiled into the
`system.json`. In order to do so, run

```js
nsd system compile workshop local
```

Now, let's build our containers:

```bash
$ nsd container build workshop web
```

We'll check the revision list again:

```bash
$ nsd rev list workshop
revision             deployed who                                                     time                      description
06a25df3d163818f104… false    Matteo Collina <hello@matteocollina.com>                2014-10-17T10:53:23.000Z  built container: web
5a1f0c5ba2b675ebf07… false    Matteo Collina <hello@matteocollina.com>                2014-10-17T10:33:11.000Z  system compile
475f94e42644a0b66cd… false    Matteo Collina <hello@matteocollina.com>                2014-10-17T10:32:32.000Z  system compile
23f506e70ef36e707b8… false    Matteo Collina <hello@matteocollina.com>                2014-10-17T10:31:24.000Z  system compile
f62a1f316b41e04fb75… false    Matteo Collina <hello@matteocollina.com>                2014-10-17T10:29:57.000Z  system compile
daed56b003f6e83502a… false    Matteo Collina <hello@matteocollina.com>                2014-10-17T10:29:41.000Z  system compile
b37e43f8e21b1159bd1… false    Matteo Collina <hello@matteocollina.com>                2014-10-17T10:29:30.000Z  system compile
940eb3f5a4b977835ba… false    Matteo Collina <hello@matteocollina.com>                2014-10-17T10:20:44.000Z  first commit
```

Deploy
------

All we have to do now is deploy:

```bash
$ nsd rev deploy workshop 06a2
```

The 06a2 part is just the first chars of the revision identifier from `nsd rev list`

Our container should be running just fine, we can use the following to see it in action:

OS X : 
```bash
$ curl http://$(boot2docker ip):1337
```

Linux:
```bash
$ curl http://localhost:1337
```

Custom build directory
----------------------

If we are building multiple services from a single GIT repository,
We __must__ also include a `bash` script in the repository for each of the services.
In this way we can customize how the containers will be built.

Here is an example of the build script:
```bash
#!/bin/bash

echo TARGET:<PATH-TO-THE-SERVICE-FOLDER>
```

We'll name this `build.sh`, this file will tell nscale where the
Dockerfile is located, plus it can be used to do some steps locally to
prepare the build.

Thanks to this file, we can build multiple containers from the same git repository.

[Next up: exercise 3](https://github.com/nearform/nscale-workshop/blob/master/ex3.md)
