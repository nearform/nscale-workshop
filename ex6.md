Check and fix
=============

This tutorial teaches you:

1. Verify that the system is running ok as expected
2. Fix the system after two containers are killed off

Verify sudc is running ok
-------------

Open the sudc application in the browser:

	echo $DOCKER_HOST
	open http://<ip>:8000
	
Let's double check that sudc is working as expected by running:

	nsd system check sudc
	
You should see the following output:

	Looking good! No deviation detected
	
The check command runs a system analysis to verify that everything is looking good! You can see the output of that analysis by running:

	nsd system analyze sudc

Kill two of the containers
-------------

Now kill the web and doc-srv containers:

	docker ps
	docker kill <web container id>
	docker kill <doc container id>

Verify sudc is not running:

	echo $DOCKER_HOST
	open http://<ip>:8000

And do the check again:

	nsd system check sudc
	
You should see some output similar to the following:

	Deviation from deployed revision detected!
	Remedial action plan as follows:

	execution plan: 
	Command                        Id                                                
	add                            f9a8f2b4-19f6-4538-97a7-4a52fdfdef4b              
	start                          f9a8f2b4-19f6-4538-97a7-4a52fdfdef4b              
	link                           f9a8f2b4-19f6-4538-97a7-4a52fdfdef4b              
	add                            7fdbeda8-affe-4e47-9b86-ca8d5b2124ff              
	start                          7fdbeda8-affe-4e47-9b86-ca8d5b2124ff              
	link                           7fdbeda8-affe-4e47-9b86-ca8d5b2124ff              

	operations: 
	Host                 Command                                                                                                                                               
	localhost            docker run -e WEB_HOST=10.75.29.243 -p 8000:8000 -d sudc/web-231 sh /web/run.sh                                                                       
	localhost            docker run -p 9002:9002 -d sudc/doc-srv-30 /usr/bin/node /srv/doc-srv                                                                                 
	
	run 'system fix' to execute

System fix and get sudc up and running again
-------------	

Let's do a system fix to get sudc up and running again:

	nsd system fix sudc
	
And check sudc is working as expected again:

	nsd system check sudc
	
	echo $DOCKER_HOST
	open http://<ip>:8000

[Next up: exercise 7](https://github.com/nearform/nscale-workshop/blob/master/ex7.md)
