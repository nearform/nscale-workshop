Create a System With NScale
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
$ nscale sys create
? What is the system name? workshop
? What is the system namespace? nscale
? Confirm creating system "workshop" with namespace "nscale"? (Y/n) y
```

Now we can check that everything is as expected:

```bash
$ nscale sys list
Name                           Id
workshop                       2de30af9-fdc4-41ff-9b88-cd47eacb7f77
```

(In `nscale`, you can shorten commands down to 3 chars.)

The `list` command will return any systems that `nscale` is aware of,
including our new `nscale_workshop` system:

The `nscale sys create` creates a new git repository in the current
directory. Go ahead and have a look at the files in there.

```bash
$ cd workshop
$ tree # you may not have tree installed
.
├── definitions
│   └── services.js
└── system.js
```

1 directory, 4 files

There are two main files there:

1. `system.js` is the _source of knowledge_ of a system managed by
   nscale.
2. `definitions/services.js` contains the definitions on how to build
   our containers.

We will edit both shortly.

Preparing the Application
-------------------------

Containers should live in a git repository, this allows `nscale` to check
out the repo and build the containers for us.

Let's take the example we built in [exercise 1](./docker-intro.md), and place it in a git repository on github.

In this tutorial, our test repo will be: git@github.com:nearform/nscale-workshop-intro-docker-sample.git

Add a container definition
--------------------------

Let's open `definitions/services.js` in you favorite editor. It
currently looks like:

```js
exports.root = {
  type: 'blank-container'
};

// Example
//
// exports.web = {
//   type: 'docker',
//     specific: {
//       repositoryUrl: 'git@github.com:nearform/nscaledemoweb.git',
//       execute: {
//         args: '-p 8000:8000 -d',
//         exec: '/usr/bin/node index.js'
//       }
//     }
// };
```

To begin defining our system, we need to change it to:

```js
exports.root = {
  type: 'blank-container'
};

exports.web = {
  type: 'docker',
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
exports.id = '13fc4a5a-f1e3-4bc1-9acf-3ba8a7b65f6a';

exports.topology = {
  development: {
  }
};

// Example
//
// exports.topology = {
//   development: {
//     root: ['web']
//   }
// };
```

This system is empty, let's add our containers:

```js
exports.name = 'workshop';
exports.namespace = 'nscale';
exports.id = '2de30af9-fdc4-41ff-9b88-cd47eacb7f77';

exports.topology = {
  development: {
    root: ['web']
  }
};
```

This abstract system definition __must__ be compiled into the
`development.json`, which corresponds to the _same key used in `exports.topology`_.
We can define an unlimited number of keys there.
In order to compile, we run:

```js
nscale system compile workshop development
```

Now, let's build our containers:

```bash
$ nscale container build workshop web latest development
```

We'll check the revision list again:

```bash
$ nscale rev list workshop
revision             deployed     who                                      time                      description
f75ff4f3b2ecdf378ad…              Matteo Collina <hello@matteocollina.com> 2015-01-23T15:50:25.000Z  system compile
26490f52c61e020ffe0…              Matteo Collina <hello@matteocollina.com> 2015-01-23T15:41:16.000Z  first commit
```

Deploy
------

All we have to do now is deploy:

```bash
$ nscale rev deploy workshop f75f development
```

The f75f part is just the first chars of the revision identifier from `nscale rev list`.
We can also use the alias `latest` to point to the latest revision.

Our container should be running just fine, we can use the following to see it in action:

OS X :
```bash
$ curl http://$(boot2docker ip):1337
```

Linux:
```bash
$ curl http://localhost:1337
```

### What is deployed?

We can check which revision is deployed with:

```bash
$ nscale rev list workshop
revision             deployed     who                                      time                      description
f75ff4f3b2ecdf378ad… development  Matteo Collina <hello@matteocollina.com> 2015-01-23T15:50:25.000Z  system compile
26490f52c61e020ffe0…              Matteo Collina <hello@matteocollina.com> 2015-01-23T15:41:16.000Z  first commit
```

And then we can ask that revision `development.json`:

```js
$ nscale rev get f75ff dev # from the project folder
{
  "name": "workshop",
  "namespace": "nscale",
  "id": "98114ae6-0ab7-438f-b7a3-e7def8101117",
  "containerDefinitions": [
    {
      "type": "blank-container",
      "id": "root",
      "name": "root"
    },
    {
      "type": "docker",
      "specific": {
        "repositoryUrl":
"git@github.com:nearform/nscale-workshop-intro-docker-sample.git",
        "execute": {
          "args": "-p 1337:1337 -d"
        },
        "commit": "92ea17ace03f9184ca0818707b952f7ad64f8d1d"
      },
      "id": "web$92ea17ace03f9184ca0818707b952f7ad64f8d1d",
      "name": "web"
    }
  ],
  "topology": {
    "containers": {
      "root-16f4f95b": {
        "id": "root-16f4f95b",
        "containedBy": "root-16f4f95b",
        "containerDefinitionId": "root",
        "type": "blank-container",
        "contains": [
          "web-c31f912e$92ea17ace03f9184ca0818707b952f7ad64f8d1d"
        ],
        "specific": {}
      },
      "web-c31f912e$92ea17ace03f9184ca0818707b952f7ad64f8d1d": {
        "id": "web-c31f912e$92ea17ace03f9184ca0818707b952f7ad64f8d1d",
        "containedBy": "root-16f4f95b",
        "containerDefinitionId":
"web$92ea17ace03f9184ca0818707b952f7ad64f8d1d",
        "type": "docker",
        "contains": [],
        "specific": {
          "repositoryUrl":
"git@github.com:nearform/nscale-workshop-intro-docker-sample.git",
          "execute": {
            "args": "-p 1337:1337 -d"
          },
          "commit": "92ea17ace03f9184ca0818707b952f7ad64f8d1d"
        }
      }
    },
    "name": "development"
  }
}
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

[Next up: exercise 3](./3-deploy-larger-application.md)
