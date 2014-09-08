NScale Intro
============

This tutorial teaches you:

1. what is nscale
2. how to install it
2. how to use it locally
3. deploy a simple web app on nscale

What is nscale?
---------------
nscale is an opensource deployment project. It grew out of a need to deploy microservice based systems. After having created custom scripts for deployment on several projects, nscale was created to stop us having to re-invent the wheel every time.

nscale uses docker to create and deploy containerised applications and services.

### Prerequisits

#### Docker
you should have docker installed on your system as per the previous exercise. 

#### Git
nscale uses git to track revision history so you will need git installed on your system. In order for nscale to track revisions correctly you will need to ensure that you have configured your git username and email. Do the following:

Set git username

	git config --global user.name "your name"

Set git email

	git config --global user.email "you@somewhere.com"

### Installation
nscale is installed though npm. To install it run the following:

	sudo npm install -g nscale

Check that the installation went OK by running the nscale command line client:

	nsd help
	
You should see output similar to the following:

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
 

### Run the server
nscale runs as a server with a command line client and web interface. To start the server run:

	nsd server start

to stop the server run

	nsd server stop

### Setting credential
First off we need to tell nscale who you are. To do this run:

	nsd login
	
nscale will read your git credentials and use them for making system commits as you build and deploy containers.

### List available systems
nscale works with the concept of a system, a system is defined by a system definition file and holds all of the meta information required to build and deploy system containers. Lets check that the system list command works first. With the server running, execute the following command:

	nsd system list
	
You should see the following output

	Name                           Id

Thats because there are as yet no systems defined.

### Clone a system
Lets clone an example 'hello world' system. There is one already prepared at:

	git@github.com:nearform/nscaledemo.git

To grab this system run:

	nsd clone git@github.com:nearform/nscaledemo.git

Let's check that this all went OK by running a system list command

	nsd system list

You should see the following output:

	Name                           Id
	nscaledemo                     e1144711-47bb-5931-9117-94f01dd20f6f

### Under the hood
nscale uses a configuration file to tell it where to store its data. The default configuration is kept at /usr/local/etc/nscale/config.json. Take a moment to inspect the configuration.

You should see from the configuration that nscale keeps its data in /usr/local/var/nscale/data. If you look in this directory you should see a directory named nscaledemo. This is the git repostory that we cloned in the previous step.

If you take a look in the nscaledemo repository you will see a file named system.json. Open this file and inspect the contents. System.json holds the configuration for the nscaledemo system and tells nscale the containers that it should build and deploy.

### Inspect the demo system
Lets take a look at the nscaledemo system. Run the following command:

	nsd conainer list nscaledemo

You shoudld see the following output

	Name                 Type            Id                                                 	Version         Dependencies
	Machine              virtualbox      85d99b2c-06d0-5485-9501-4d4ed429799c                               ""
	web                  boot2docker     9ddc6c027-9ce2-5fdg-9936-696d2b3789bb              0.0.1           {}
	
There are two containers definitions, a virtual box host and a boot2docker container. Lets take a look at the revision history:

	nsd revision list nscaledemo

You should see a list of system revisions.

### Building a container
Lets now build the example web container. Run the following:

	nsd container build nscaledemo web

nscale will build the nscaledemo web container ready for deployment. Once the command completes check the revision history:

	nsd revision list nscaledemo

You should see an updated commit, i.e. a new immutable system revision that includes the freshly built container.

### Deploying the container
Lets deploy the container that we just built, run:

	nsd revision deploy nscale <revisionid>

giving the revision id from the top of the revision list. nscale will now deploy the container. You can check that the deploy went OK by running:

	docker ps

You should see that there is one running container. Check that everything is OK by pointing your browser to the boot2docker ip address port 8000. You should see that the page loads and displays 'Hello world!'.
