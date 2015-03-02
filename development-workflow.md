Development Workflow
====================

Can we use nScale during the development workflow? Yes, with
process-containers!

A process container is a 'fake' container, a standard unix process on
nScale host, managed by nScale. All the repositories defined in your
topology will be checked out from GIT for you in workspace/\<SERVICE\>,
and you can make your edit there.

The most important feature for the development work it is the onboarding
of new developers into the project and manage any changes in the
dependencies easily.

This guide is composed of two sections:

* [Editing an existing system](#existing)
* [Setting up a new system with a dockerized database](#new-system)


<a name="existing"></a>
Editing an existing system
--------------------------
It is this workflow which allows us to makes changes and deploy them using nscale:
  
  - Make changes in code
  - Commit
  - (optional) push to Github
  - System compile
  - Build relevant container(s)
  - System Deploy

First, clone a system:

```bash
git clone git@github.com:nearform/sudc-system.git
cd sudc-system
nscale sys link .
```

The system will be set up for local development by issuing:

```bash
nscale sys compile development
```

Then, to download and install dependencies, launch:

```bash
nscale cont buildall latest development
```

Finally, to start the system:

```bash
nscale rev dep head development
```
**Make the Edit**
Pay particular attention to the fact that when building containers nscale, will checkout repos using the sha of the latest commit. This will detach the HEAD your repos. The danger is that you build containers which detaches the HEAD, edit code on an unreferenced branch, and then do a system compile which checks out the master branch in each repo. You could potentially lose work.

**Tip:**
  Before writing code that you are building into a container, do a git status of that repo to make sure the HEAD isn't detached or else you will be writing code on an unnamed branch. Checkout to master or another branch before proceeding.

cd into the workspace/sudc-web directory and run a git branch command. 
If the HEAD is detached, then do 
```bash
git checkout master
```
Open up `workspace/sudc-web/web/public/js/app.js` and add an alert after the 'Your code here' comment:

```js
initialize: function () {
    // Your code here
    alert('Hello World!');
}

**cd into the sudc/workspace/sudc-web directory**, stage the changes and commit:
```bash 
git add .
git commit -m "added alert"
```
Compile the system:
```bash
nscale sys compile sudc latest development
```

build the Web container:
```bash
nscale cont build sudc web latest development
```

redeploy:
```bash
nscale rev deploy sudc latest development
```

<a name="new-system"></a>
Setting up a new system with a dockerized database
--------------------------------------------------

### Create a new app

We are going to create a _extremely simple_ REST key/value store on top of
Redis.

Launch:

```bash
$ nsd sys create
? What is the system name? nscale-kv
? What is the system namespace? nscalekv
? Confirm creating system "nscale-kv" with namespace "nscale-kv"? Yes
```

Then:

```bash
cd nscale-kv
```

### Add a database

Fire up your editor, and create a new `definitions/databases.js` file
with the content:

```js
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

Then, edit the topology section of `system.js` into:

```js
exports.topology = {
  local: {
  },
  development: {
    root: ['redis']
  }
};
```

Then, compile and build this topology:

```bash
nsd sys comp process
nsd cont buildall
nsd rev dep head
```

To check that everything is fine, launch:
```bash
docker ps
```

And note down your container name, then:

```bash
docker run -it --link IMAGE_NAME:redis --rm redis sh -c 'exec redis-cli
-h "$REDIS_PORT_6379_TCP_ADDR" -p "$REDIS_PORT_6379_TCP_PORT"'
```

Where you should replace IMAGE\_NAME with the name you just noted down.

nscale can now spin up a Redis server for your local developement!

### Add your app

First, create a git repository on Github/BitBucket/whatever for our new
project `nscale-kv-demo`.

Then, add to `definitions/services.js` the following:

```
exports.web = {
  type: 'process',
  specific: {
    // replace repositoryUrl with the repo you just created
    repositoryUrl: 'git@github.com:nearform/nscale-kv-demo.git',
    processBuild: 'npm install',
    execute: {
      process: './server.js'
    }
  }
};
```

and edit the topology section of `system.js` into:

```js
exports.topology = {
  local: {
  },
  process: {
    root: ['web', 'redis']
  }
};
```

Then launch:

``` bash
nsd sys comp process
nsd cont build web
```

The latest command will fail, because we did not add a `package.json`:
```
--> finding container
--> synchronizing repository...
Cloning into 'nscale-kv-demo'...

--> initiating container build
--> executing container specific build for web
building
npm
 ERR! install Couldn't read dependencies

npm
 ERR! Darwin 14.0.0

npm ERR! argv "node" "/Users/matteocollina/.nvm/v0.10.33/bin/npm"
"install"

npm ERR! node v0.10.33
npm ERR! npm
 v2.1.10
npm ERR! path
/Users/matteocollina/Temp/nscalekv/workspace/nscale-kv-demo/package.json

npm ERR! code ENOPACKAGEJSON
npm ERR! errno 34


npm ERR! package.json ENOENT, open
'/Users/matteocollina/Temp/nscalekv/workspace/nscale-kv-demo/package.json'
npm ERR! package.json This is most likely not a problem with npm itself.
npm ERR! package.json npm can't find a package.json file in your
current directory.


npm ERR! Please include the following file with any support request:
npm ERR!
/Users/matteocollina/Temp/nscalekv/workspace/nscale-kv-demo/npm-debug.log

{ cmd: 'npm install', code: 34 }
{ cmd: 'npm install', code: 34 }
command failed
```

We should initialize our project, so cd into
`workspace/nscale-kv-demo`, and launch:

```
npm init
```

Which will create a package.json for us:

```
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sane defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg> --save` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
name: (nscale-kv-demo)
version: (1.0.0)
description:
entry point: (index.js) server.js
test command:
git repository: (https://github.com/nearform/nscale-kv-demo.git)
keywords:
author: Matteo Collina <matteo.collina@nearform.com>
license: (ISC) MIT
About to write to
/Users/matteocollina/Temp/nscalekv/workspace/nscale-kv-demo/package.json:

{
  "name": "nscale-kv-demo",
  "version": "1.0.0",
  "description": "nscale-kv-demo ==============",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/nearform/nscale-kv-demo.git"
  },
  "author": "Matteo Collina <matteo.collina@nearform.com>",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/nearform/nscale-kv-demo/issues"
  },
  "homepage": "https://github.com/nearform/nscale-kv-demo"
}


Is this ok? (yes) yes
```

Then, you must install redis:

```
npm install redis --save
```

Fire up your editor, and add a `server.js` file with the content:

```
#! /usr/bin/env node

var http    = require('http');
var redis   = require('redis');
var concat  = require('concat-stream');
var client  = redis.createClient(6379, process.env.DOCKER_HOST_IP ||
'localhost');
var server  = http.createServer();
var errors  = 0;

client.on('error', function(err) {
  console.log(err);
  if (++errors === 10) {
    process.exit(1);
  }
});

server.on('request', function(req, res) {
  if (req.method === 'GET') {
    client.get(req.path, function(err, value) {
      if (err) {
        res.statusCode = 500;
        res.end(err.message);
        return;
      }
      res.end(value);
    });
  } else {
    req.pipe(concat(function(list) {
      client.set(req.path, list.toString(), function(err) {
        if (err) {
          res.statusCode = 500;
          res.end(err.message);
          return;
        }
        res.statusCode = 204;
        res.end();
      })
    }));
  }
});

server.listen(3000);
```

Finally:

```bash
chmod +x server.js
```

Test it with:

```bash
DOCKER_HOST_IP=192.168.59.103 ./server.js # if your are on Linux, omit
DOCKER_HOST_IP
curl -X POST -d 'hello' http://127.0.0.1:3000/
curl http://127.0.0.1:3000/
curl -X POST -d 'some content' http://127.0.0.1:3000/a/resource
curl http://127.0.0.1:3000/a/resource
```

Our little REST key/value store is ready to go!

### Wire everything up!

Go back to your system folder (`cd ../../`) and then run:

```
nsd cont build web
nsd rev dep head
```

And then you can test your little REST server as above!

If you change your `server.js`, you will see it is automatically
reloaded!
