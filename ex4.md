Fixing a bug
============

This tutorial covers:

1. The nscale configuration file
2. How to view `nscale` server logs
3. Deploying a bugfix to the startup death clock

Configuration
-------------
Before we proceed let's just take a look at the `nscale` configuration file. By default this is placed in `~/.nscale/config/config.json`. When we open this file up and inspect it we'll see a JSON file containing several sections that control various aspects of `nscale`.

The kernel section defines the following:

	port - the port the nscale kernel should run on
	systemsRoot - the full path where system data will reside
	buildRoot - the full path where build data will reside
	targetRoot - the full path where target data will reside

	"kernel": {
	  "port": "8010",
	  "systemsRoot": "/Users/me/.nscale/data/systems",
	  "buildRoot": "/Users/me/.nscale/data/build",
	  "targetRoot": "/Users/me/.nscale/data/targets"
	}

The api section defines the following:

	port - the port the nscale api should run on
	sdkhost and sdkport - the host and port where the nscale sdk is running on
	https - true if running on https, false otherwise
	cors and github - required config settings when github login feature has been released

	"api": {
	  "port": 3000,
	  "sdkhost": "localhost",
	  "sdkport": "3223",
	  "https": false,
	  "cors": {
	    ...
	  },
	  "github": {
	    ...
	  }
	}

The web section defines the following:

	ip and port - the ip and port the nscale web gui should run on
	apiserver and apibase - the host, port and base where the nscale api is running on

	"web": {
	  "ip": "0.0.0.0",
	  "port": 9000,
	  "apiserver": "http://localhost:3000",
	  "apibase": "/api/1.0"
	}

Note that the web and api sections are optional, by omitting these from the configuration nscale will not start a GUI.

The modules section defines the following:


	"modules": {
	  "protocol": {
	    "require": "nscale-protocol",
	    "specific": {}
	  },
	  "authorization": {
	    "require": "nscale-noauth",
	    "specific": {
	      "credentialsPath": "/Users/me/.nscale/data"
	    }
	  },
	  "analysis": {
	    "require": "nscale-local-analyzer",
	    "specific": {}
	  }
	}

Right now we the auth module just picks up your git credentials, however this is open for extension with and can be replaced with other authentication strategies.

The analysis module implements the logic that queries a running system for comparison against the desired system state. Right now nscale has a local boot2docker analyzer and also and Amazon web services analyzer.

The containers section defines the following:

	containers - list of supported container types.

	"containers": [
	  {"require": "blank-container",
	    "type": "blank-container",
	    "specific": {}
	  },
	  {"require": "docker-container",
	    "type": "docker",
	    "specific": {"imageCachePath": "/tmp"}
	  }
	]

For this workshop we are using just the virtualbox and boot2docker containers. Nscale also has additional containers for AWS deployment. The intent is that this provides an open framework for extension into other platforms.

The `nscale` root folder looks as follows:

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/configdir.png)

Viewing the logs
----------------
We can view the nscale logs at any time by running

	nsd server logs
	nsd server logs api.log
	nsd server logs web.log

Make the fix
------------
There is a problem with our Startup Death Clock application. The home page reads 'Click' instead of 'Clock'.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/click.png)

We need to get this fixed and deployed right away. Let's go ahead and make the fix. Let's replace 'Click' with 'Clock':

	sed -i .bak -e s/Click/Clock/g -e s/click/clock/g sudc-system/workspace/sudc-web/web/public/templates/index.dust
	(cd sudc-system/workspace/sudc-web/web && git diff)

And run the following commands:

	nsd container build sudc web

	nsd revision list sudc

	nsd revision preview sudc <revision id>

	nsd revision deploy sudc <revision id>

	echo $DOCKER_HOST
	open http://<ip>:8000

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/clock.png)

In a real system, fixes would be made and committed to a git repository and nscale would pull the latest changes before building a container.

[Next up: exercise 5](https://github.com/nearform/nscale-workshop/blob/master/ex5.md)
