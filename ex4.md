Fixing a bug
============

This tutorial teaches you:

1. The nscale configuration file
2. How to view nscale server logs
3. Deploy a bugfix to the startup death clock

Configuration
-------------
Before we proceed lets just take a look at the nscale configuration file. By default this is placed in ~/.nscale/config/config.json. Open this file up and inspect it. You should see that this file has several sections each of which controls various aspects of nscale.

The kernel section in the config defines:

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

The nscale root folder looks as follows.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/configdir.png)

Viewing the logs
----------------
You can view the nscale logs at any time by running 

	nsd server logs
	nsd server logs api.log
	nsd server logs web.log

Make the fix
------------
There is a problem with our Starup Death Clock application. The home pages reads 'click' instead of 'clock'. We need to get this fixed and deployed right away. Lets go ahead and make the fix. Open up the following file: 

.nscale/data/build/sudc/startupdeathclock/web/public/templates/index.dust

change click to clock

	nsd container build sudc web
	
	nsd revision list sudc
	
	nsd revision previes sudc
	
	nsd revision deploy sudc
	
Note that in a real system fixes would be made and committed to a git repository, nscale will pull the latest changes before building a container.


