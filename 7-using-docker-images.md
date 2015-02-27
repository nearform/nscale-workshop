Use an image from Docker Hub
==================================

This tutorial covers:

1. Creating a new system
2. Hooking up a new container definition from the Registry
3. Building a new container
4. Run!

Create a new System
-------------------

As done in [Exercise 2](./2-create-a-system.md), let's go ahead and create a new system with:

```bash
$ nscale system create
? What is the system name? registry
? What is the system namespace? nscale
? Confirm creating system "registry" with namespace "nscale"? (Y/n) y
```

Now we can check that everything is as expected:

```bash
$ nscale system list
Name                           Id
registry                       2de30af9-fdc4-41ff-9b88-cd47eacb7f77
```

Add a registry container definition
-----------------------------------

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

exports.redis = {
  type: 'docker',
  specific: {
    name: 'redis',
    execute: {
      args: '-d -p 6379:6379'
    }
  }
};
```

Build a container
-----------------

Let's open `system.js` in you favorite editor. It currently looks like this:

```js
exports.name = 'registry';
exports.namespace = 'nscale';
exports.id = '2c03b9c3-5623-42b8-8af7-7093c6be2e70';

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
exports.name = 'registry';
exports.namespace = 'nscale';
exports.id = '2c03b9c3-5623-42b8-8af7-7093c6be2e70';

exports.topology = {
  development: {
    root: ['redis']
  }
};
```

```bash
nscale system compile registry
```

Now, let's build our containers:

```bash
$ nscale container buildall registry latest development
```

We'll check the revision list again:

```bash
$ nscale rev list workshop
revision             deployed who                                                     time                      description
15dffc828de9d971ba1… false    Matteo Collina <hello@matteocollina.com>                2014-10-30T10:54:04.000Z  built container: redis
```

Deploy
------

All we have to do now is deploy:

 ```bash
$ nscale rev deploy registry 15dff development
```

Our container should be running just fine, we can use the following to see it in action:

OS X : 
```bash
$ redis-cli -h `boot2docker ip`
```

Linux:
```bash
$ redis-cli
```
you may need to install redis-tools
  
```bash
sudo apt-get install redis-tools
```

[Next up: exercise 8] (./8-deploy-to-aws.md)
