A Larger Application
=======================

This tutorial teaches you:

1. How to deploy a larger application
2. Run a preview before deploying
2. nscale web gui

Clone the Application
---------------------
Lets get started by cloning the repository for a larger application. Run the following command:

	nsd system clone git@github.com:nearform/sudclocal.git

This will pull down the code for the Startup Death Clock. Lets take a look at the system definition:

	nsd container list sudc
	
You should see the following containers:

	web                  boot2docker    
	hist-srv             boot2docker     
	real-srv             boot2docker     
	doc-srv              boot2docker

The application is composed of a web front end and three addtional services.

Build the system
----------------
Lets go ahead and build the containers ready for deployment, run the following:

	nsd container build sudc hist-srv
	nsd container build sudc real-srv
	nsd container build sudc doc-srv
	nsd container build sudc web

After those have all completed you should have four containers ready for deployment.

GUI
---
Lets now take a look at the startup death clock system using the graphical interface. Point your browser to <a href="http://localhost:9000" target="_blank">http://localhost:9000</a>.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/systems.png)

Click on the sudclocal link to go to the container view.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/containers.png)

Show the revision list by clicking on the Revisions tab.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/revisions.png)

Show the system state by clicking on the System state tab.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/topology.png)

Show the timeline by clicking on the timeline tab.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/timeline.png)
	
Previewing the deployment
-------------------------
Before we deploy the system lets take a look at the commands that will be executed on deployment. Fist off check the revision list:

	nsd revision list sudc

Next lets preview what a deploy of the latest build would look like:

	nsd revision preview sudc <revision id>

You should see some output similar to the following:

	OUTPUT HERE

to produce this view nscale has computed a delta between what the latest reivions of the system should look like compared with what is actually running on your system. 

Run the deployment
------------------
Lets go ahead and run the deployment:

	nsd revision deploy sudc <revision id>

nscale will now execute the deployment that we previewed in the last step. Once this completes you shuold have a running system composed of four docker containers. To check that all is working point your browser to http://<dockerhost>:8000. You should see the running startup death clock:

	IMG HERE

[Next up: exercise 4](https://github.com/nearform/nscale-workshop/blob/master/ex4.md)
