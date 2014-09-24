A Larger Application
=======================

This tutorial covers:

1. Deploying a larger application
2. Previewing before deploying
2. The nscale web gui

Clone the Application
---------------------
Lets get started by cloning the repository for a larger application by running the following:

	nsd system clone git@github.com:nearform/sudclocal.git

This will pull down the code for the Startup Death Clock system. Let's take a look at the system definition:

	nsd container list sudc

You should see the following containers:

	web                  docker
	hist-srv             docker
	real-srv             docker
	doc-srv              docker

The application is composed of a web front end and three additional services.

Build the system
----------------
Let's go ahead and build the containers ready for deployment:

	nsd container build sudc hist-srv
	nsd container build sudc real-srv
	nsd container build sudc doc-srv
	nsd container build sudc web

After those have all completed we should have four containers ready for deployment.

GUI
---
Now we can take a look at the startup death clock system using the graphical interface. We just need to point our browser to <a href="http://localhost:9000" target="_blank">http://localhost:9000</a>.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/systems.png)

We can go to the container view via the sudclocal link,

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/containers.png)

show the revision list by clicking on the Revisions tab,

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/revisions.png)

show the system state by clicking on the System state tab,

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/topology.png)

and show the timeline by clicking on the System timeline tab.

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/timeline.png)

Previewing the deployment
-------------------------
Before we deploy the system let's take a look at the commands that will be executed on deployment. Fist we can check the revision list:

	nsd revision list sudc

We should see some output similar to the following:

	revision             deployed who                                                     time                      description
	136c840f016c57d0e23… false    John Doe <john.doe@gmail.com>                           2014-09-08T12:10:11.000Z  built container: 2b36df5faa5c92262aa675cd0a07312a…
	31d0cff07829dc15e29… false    John Doe <john.doe@gmail.com>                           2014-09-08T12:09:29.000Z  built container: 51df875511be6f4951a1bd00610db2a9…
	7e48d13a98746c8356a… false    John Doe <john.doe@gmail.com>                           2014-09-08T12:09:09.000Z  built container: f34344ef6f773c3e59b9cf84d01bf0ff…
	1483ec749e9202dde10… false    John Doe <john.doe@gmail.com>                           2014-09-08T12:08:46.000Z  built container: 25a6d9868347b906345513aaf99e45ad…
	5cab4567fef325bba9f… false    John Doe <john.doe@gmail.com>                           2014-09-08T12:06:24.000Z  first commit
	3ebb8b5b986d76e59e9… false    Peter Elger <elger.peter@gmail.com>                     2014-09-08T08:16:00.000Z  added system definition
	644891e6df77a8de7b2… false    Peter Elger <elger.peter@gmail.com>                     2014-09-07T18:56:23.000Z  first commit

Next let's preview what a deploy of the latest build would look like by previewing the revision id from the top of the revision list:

	nsd revision preview sudc <revision id>

We should see some output similar to the following:

	execution plan:
	Command                        Id
	unlink                         1973bbff-6f80-4f40-ae79-805f04f81690
	stop                           1973bbff-6f80-4f40-ae79-805f04f81690
	remove                         1973bbff-6f80-4f40-ae79-805f04f81690
	add                            a8602f4d-16ff-4dfa-a6d7-3ed79662d50f
	start                          a8602f4d-16ff-4dfa-a6d7-3ed79662d50f
	link                           a8602f4d-16ff-4dfa-a6d7-3ed79662d50f

	operations:
	Host                 Command
	localhost            docker kill 81bbf1284e5fee5b5f3e73c49b27c76b7b6303b2f738fb4d83b7a2fcd03c18fd
	localhost            docker ps -a --no-trunc | grep Exit | awk '{print $1}' | xargs -I {} docker rm {}
	localhost            docker images --no-trunc| grep none | awk '{print $3}' | xargs -I {} docker rmi {}
	localhost            docker run -e WEB_HOST=10.75.29.243 -p 8000:8000 -d sudc/web-223 sh /web/run.sh

In order to produce this view, `nscale` has computed a delta between what the latest revisions of the system should look like compared with what is actually running on your system.

Run the deployment
------------------
Let's go ahead and run the deployment:

	nsd revision deploy sudc <revision id>

`nscale` will now execute the deployment that we previewed in the last step. Once this completes we should have a running system composed of four docker containers. We van verify everything's working by pointing our browser to the docker host ip address port 8000 on Mac OS X or localhost on linux.

OS X:
	open http://$(boot2docker ip):8000

Linux
	open http://localhost:8000

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/sudc.png)

[Next up: exercise 4](https://github.com/nearform/nscale-workshop/blob/master/ex4.md)
