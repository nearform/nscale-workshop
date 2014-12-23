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


