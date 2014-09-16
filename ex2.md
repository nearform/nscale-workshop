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
workshop                       521a3a73-7d5d-434d-96c5-e4732db307be
```

(In `nscale`, you can shorten commands down to 3 chars.)

The `list` command will return any systems that `nsd` is aware of, 
including our new `nscale_workshop` system:

The `nsd sys create` creates a new git repository in the current
directory. Go ahead and have a look at the files in there.

There are three files:

1. `deployed.json` contains the currently deployed system.
2. `system.json` contains the system definition, all the containers and
   how to build them.
3. `timeline.json` contains all the events that happened in the system.

Preparing the Application
-------------------------

Containers should live in a git repository, this allows `nscale` to check 
out the repo and build the containers for us.

Let's take the example we built in [exercise 1](https://github.com/nearform/nscale-workshop/blob/master/docker-intro.md), and place it in a git repository on github.

We'll also include an `bash` script in our new repo, that can customize
how the container will be built. In our case, this is really simple:

```bash
#!/bin/bash

echo TARGET:.
```

We'll name this `build.sh`

Now we can build multiple containers from the same git repository.

In this tutorial, our test repo will be: git@github.com:nearform/nscale-workshop-intro-docker-sample.git

Add a container definition
--------------------------

Let's open `system.json` in you favorite editor. It looks like this:

```js
{
  "name": "workshop",
  "namespace": "nscale",
  "id": "521a3a73-7d5d-434d-96c5-e4732db307be",
  "containerDefinitions": [
  ],
  "topology": {
    "containers": {}
  }
}
```

To begin creating a system, we need to add the container definitions:

```js
{
  // the other stuff
  "containerDefinitions": [
    {
      "name": "Machine",
      "type": "virtualbox",
      "specific": {
        "repositoryToken": "04551b154404a852e663aba4c3fa299e04f6e8a5"
      },
      "id": "85d99b2c-06d0-5485-9501-4d4ed429799c"
    },
    {
      "name": "web",
      "type": "boot2docker",
      "dependencies": {},
      "specific": {
        "repositoryUrl": "git@github.com:nearform/nscale-workshop-intro-docker-sample.git",
        "buildScript": "build.sh",
        "arguments": "-p 1337:1337 -d __TARGETNAME__"
      },
      "id": "9ddc6c027-9ce2-5fdg-9936-696d2b3789bb",
      "version": "0.0.1"
    }
  ]
  // other stuff
}
```

Build a container
-----------------

In order to build a container, we need to initialize our topology
section:

```js
{
  // other stuff
  "topology": {
    "containers": {
      "10": {
        "id": "10",
        "containerDefinitionId": "85d99b2c-06d0-5485-9501-4d4ed429799c",
        "containedBy": "10",
        "contains": [
          "8bd49b5b-3bd9-4314-8895-a2ec26e28175"
        ],
        "specific": {
          "ipaddress": "localhost"
        }
      },
      "8bd49b5b-3bd9-4314-8895-a2ec26e28175": {
        "id": "8bd49b5b-3bd9-4314-8895-a2ec26e28175",
        "containerDefinitionId": "9ddc6c027-9ce2-5fdg-9936-696d2b3789bb",
        "containedBy": "10",
        "contains": [],
        "specific": {
          "dockerLocalTag": "nfddemo/web-1",
          "version": "0.0.1"
        }
      }
    }
  }
}
```

Finally for nscale to be aware of our new containers, we create a new
git commit in the system git repo. 

```bash
$ git commit -a -m 'new containers'
```

Let's check if nscale detected the changes:

```bash
$ nsd rev list workshop
revision             deployed who                                                     time                      description
6cd3208029943c209a5… false    Matteo Collina <matteo.collina@gmail.com>               2014-09-08T22:03:06.000Z  Updated container definition.
f398e76c26114173a18… false    Matteo Collina <matteo.collina@gmail.com>               2014-09-08T21:45:22.000Z  first commit
revision             deployed who                                                     time                      description
6cd3208029943c209a5… false    Matteo Collina <matteo.collina@gmail.com>               2014-09-08T22:03:06.000Z  Updated container definition.
f398e76c26114173a18… false    Matteo Collina <matteo.collina@gmail.com>               2014-09-08T21:45:22.000Z  first commit
```

Now, let's buid our containers:

```bash
$ nsd container build workshop web
```

We'll check the revision list again:

```bash
$ nsd rev list workshop
revision             deployed who                                                     time                      description
8735764f9e57b9838b1… false    Matteo Collina <matteo.collina@gmail.com>               2014-09-08T22:08:11.000Z  built container: cf28904e57b0c4b84b24d0297820e8a6…
6cd3208029943c209a5… false    Matteo Collina <matteo.collina@gmail.com>               2014-09-08T22:03:06.000Z  Updated container definition.
f398e76c26114173a18… false    Matteo Collina <matteo.collina@gmail.com>               2014-09-08T21:45:22.000Z  first commit
```

Deploy
------

All we have to do now is deploy:

```bash
$ nsd rev deploy workshop 8735
```

The 8735 part is just the first chars of the revision identifier from `nsd rev list`

Our container should be running just fine, we can use the following to see it in action:

OS X : 
```bash
$ curl http://$(boot2docker ip):1337
```

Linux:
```bash
$ curl http://localhost:1337
```

[Next up: exercise 3](https://github.com/nearform/nscale-workshop/blob/master/ex3.md)
