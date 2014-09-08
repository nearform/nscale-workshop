Fixing a bug
============

This tutorial teaches you:

1. The nscale configuration file
2. How to view nscale server logs
3. Deploy a bugfix to the startup death clock

Configuration
-------------
Before we proceed lets just take a look at the nscale configuration file. By default this is placed in ~/.nscale/config/config.json. Open this file up and inspect it. You should see that this file has several sections each of which controls various aspects of nscale.

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

The modules section defines the following:

	protocol - server side command protocol
	authorization - nscale auth module
	analysis - nscale analyzer module
	
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
	    "require": "nscale-boot2docker-analyzer",
	    "specific": {}
	  }
	}
	
The containers section defines the following:

	containers - list of supported containers

	"containers": [
	  {"require": "virtualbox-container",
	    "type": "virtualbox",
	    "specific": {}
	  },
	  {"require": "boot2docker-container",
	    "type": "boot2docker",
	    "specific": {"imageCachePath": "/tmp"}
	  }
	]

The nscale root folder looks as follows:

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/configdir.png)

Viewing the logs
----------------
You can view the nscale logs at any time by running 

	nsd server logs
	nsd server logs api.log
	nsd server logs web.log

Make the fix
------------
There is a problem with our Startup Death Clock application. The home pages reads 'Click' instead of 'Clock'. We need to get this fixed and deployed right away. Let's go ahead and make the fix. Replace 'Click' with 'Clock' in the following file: 

	sed -i .bak -e s/Click/Clock/g -e s/click/clock/g ~/.nscale/data/build/sudc/startupdeathclock/web/public/templates/index.dust
	(cd ~/.nscale/data/build/sudc/startupdeathclock && git diff)

And run the following commands:

	nsd container build sudc web
	
	nsd revision list sudc
	
	nsd revision preview sudc <revision id>
	
	nsd revision deploy sudc <revision id>
	
	echo $DOCKER_HOST
	open http://<ip>:8000
	
Note that in a real system fixes would be made and committed to a git repository, and nscale will pull the latest changes before building a container.

[Next up: exercise 5](https://github.com/nearform/nscale-workshop/blob/master/ex5.md)
