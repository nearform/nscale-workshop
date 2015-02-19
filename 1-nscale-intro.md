NScale Intro
============

This tutorial covers:

1. what `nscale` is
2. how to install it
2. how to use it locally
3. deploying a simple web app with `nscale`

What is nscale?
---------------

![image](./img/logo.png)

`nscale` is an open source deployment management toolkit. After working on several microservice projects, we found there was a need to formalize a process for deploying microservice based systems so we could stop reinventing the wheel for each new project.

`nscale` uses Docker to create and deploy containerised applications and services.

### Prerequisites

#### Docker
We should have docker installed on our system as per the previous exercise.

#### Git
`nscale` uses git to track revision history so we will need git installed on our system. We need to configure our `git` username and email for `nscale` to track revisions correctly.

Let's do the following:

Set git username
```bash
git config --global user.name "your name"
```
Set git email
```bash
git config --global user.email "you@somewhere.com"
```
### Installation
`nscale` is installed though `npm`. To install it, we run the following:
```bash
npm install -g nscale # run with sudo if needed
```
If you are running on Linux, you should also add yourself to the docker
group, so that nscale can run docker commands without root permissions.
You can do that with:

```bash
sudo usermod -G docker -a `whoami`
```

Beware that __not specifying the `-a` flag will remove you from all the
other groups__.

We can do a quick check to verify installation success by running the `nscale` command line client:
```bash
nscale help
```
We should see output similar to the following:
```bash
-[ server ]----------
server start [config]                 - start the server
server stop                           - stop the server
server logs [logfile]                 - tail the server logs

-[ login ]----------
login                                 - login to the nominated host
logout                                - logout from the nominated host
use <host> [port]                     - use the given host and optional port
help                                  - show this

-[ system ]----------
system create                         - create a new blank system
system clone                          - clone a system from an existing git
                                        remote
system list                           - list all systems in this instance
system put                            - put a new revision of the system
system deployed <sys>                 - get the deployed revision of this system
system analyze <sys>                  - run an analysis for this system and
                                        output the results
system check <sys>                    - run an analysis and check it against the
                                        expected deployed system configuration
```

### Run the server

`nscale` runs as a server with a command line client and web interface.

To start the server we run:

```bash
nscale server start
```

To stop the server we run:
```bash
nscale server stop
```
### Setting credentials
We need to tell `nscale` who you are:
```bash
nscale login
```
`nscale` will read our git credentials and use them for making system commits as we build and deploy containers.

### List available systems
`nscale` utilizes the concept of a system, which is a collection of containers (containers being Docker containers, virtual machines, physical machines or anything that can hold
our microservices) arranged according a topological deployment strategy.

A system is defined by a system definition file and holds all of the meta information required to build and deploy system containers.

Let's check that the `system list` command works.

With the `nscale server` running, let's execute the following command:
```bash
nscale system list
```
We should see the following output
```bash
Name                           Id
```
That's because there are as yet no systems defined.

### Clone a system

Let's clone an example 'hello world' system. There is one already prepared at ```git@github.com:nearform/nscaledemo.git```.

Create a clean working folder on your machine and cd into it.
```bash
  mkdir test
  cd test
```
To grab this system we run:
```bash
git clone git@github.com:nearform/nscaledemo.git
nscale system link nscaledemo
```
You should now see a folder called nscaledemo. Let's check that nscale can see this with the `list` command:
```bash
nscale system list
```
We should see the following output:
```bash
Name                           Id
nscaledemo                     e1144711-47bb-5931-9117-94f01dd20f6f
```
Then enter the nscaledemo folder:
```bash
cd nscaledemo
```
### Under the hood
`nscale` uses a configuration file to tell it where to store its data. The default configuration is kept at ~/.nscale/config/config.json. Let's take a moment to inspect the configuration.

We should see from the configuration that nscale keeps its data in ~/.nscale/data.

Now lets look at the nscaledemo repository. It contains the following files:
```bash
├── README.md 
├── definitions
│   └── services.js
└── system.js
```
#### definitions/services.js
services.js contains some javascript that defines the two containers. 
```js
exports.root = {
  type: 'blank-container'
};

exports.web = {
  type: 'docker',
  specific: {
    repositoryUrl: 'git@github.com:nearform/nscaledemoweb.git',
    execute: {
      args: '-p 8000:8000 -d',
      exec: 'node index.js'
    }
  }
};

```
#### system.js
system.js holds the system topology and identity information.
```js
exports.name = 'nscaledemo';
exports.namespace = 'nscaledemo';
exports.id = 'e1144711-47bb-5931-9117-94f01dd20f6f';

exports.topology = {
  development: {
    root: ['web']
  }
};

```
nscale 'compiles' containers defined under the definitions folder along with information in system.js into a full system definition. The result of this compilation process is help in development.json. Lets run a compile now:
```bash
nscale system compile nscaledemo development
```
This will run a compile of nscaledemo to the development target. Lets go ahead and take a look at the contents of development.json.

### Inspect the demo system
Now that we have run a compile, let's use nscale to inspect the `nscaledemo` system:
```bash
nscale container list
```
If you are running the command outside of the nscaledemo folder, you
need to:
```bash
nscale container list nscaledemo
```
We should see output similar to the following:
```bash
Name                 Type            Id                                                 
root                 blank-container 85d99b2c-06d0-5485-9501-4d4ed429799c                               
web                  docker          9ddc6c027-9ce2-5fdg-9936-696d2b3789bb             
```
There are two containers definitions: a blank root container and a docker container. Let's take a look at the revision history:
```bash
nscale revision list
```
We should see a list of system revisions for the current system along with their IDs. 
* With every compile, nscale syncs up with the latest code commits in your various repos. 
* This creates an immutable system revision which can be deployed. 
* We can use these revisions to roll an entire system back to a particular state with ease.

Once the system is compiled, we can build its containers using the latest code commits.

### Building a container
Let's now build the example web container by running the following:
```bash
nscale container build web
```
`nscale` will build the `nscaledemo` web container so that it's ready to be deployed. This will take a few mins so for the curious, open a new terminal window and execute ```nscale server logs```.

### Deploying the container
Let's deploy the container that we just built using the ```nscale deploy``` command.
There are a number of variants to the deploy command

```bash
nscale revision deploy <system_id> <revision_id> <target>
<system_id> can be replaced with the system name
<system_id> can be left out if in the system directory
<revision_id> can be subsituted with "latest"
arguments can be shortened down to 3 characters in length
```
If you are in the system directory you can run:
```bash
nscale rev dep latest dev
```
which stands for:
```bash
nscale revision deploy latest development
```

Specify the system ID or name if you aren't in the directory

`nscale` will now deploy the container. We can check that the deploy went as planned using Docker like so:
```bash
docker ps
```
We should see that there is one running container. We can further verify by opening a browser at $DOCKER_HOST (on OS X) or localhost (on Linux) port `8000`.

OS X:
open http://$(boot2docker ip):8000

Linux:
open [localhost:8000](http://localhost:8000)

The page should load and display 'Hello world!'.

Stop the running docker container before moving on to the next exercise.
```bash
docker ps
docker stop <container id>
```
[Next up: exercise 2](./2-create-a-system.md)
