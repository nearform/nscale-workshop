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


Editing an existing system
--------------------------

First, clone a system:

```bash
git clone git@github.com:nearform/sudc-system.git
cd sudc-system
nsd sys link .
```

The system will be setted up for local development by issuing:

```bash
nsd sys compile process
```

Then, to download and install dependencies, launch:

```bash
nsd cont buildall
```

Finally, to start the system:

```bash
nsd rev dep head
```

Point your browser to http://localhost:8000 to see the Startup Death
Clock.

As pointed out in the output of `nsd rev dep head`, you can launch

```
tail -f ~/.nscale/log/998e0589-0936-4102-b859-d6192011c355.log
```

Then, if you edit `sudc-system/workspace/sudc-web/web/index.js` and
add a `console.log('hello world')` at the the end, like so:

```
echo "console.log('hello world')" >> workspace/sudc-web/web/index.js
```

You will see in the tailing log that the process have been automatically
restarted.


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
  process: {
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

