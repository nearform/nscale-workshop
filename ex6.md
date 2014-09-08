Check and fix
=============

This tutorial covers:

1. Verifying that the system is running according to expectations
2. Fixing the system after two containers are killed off

Verify sudc is running
-------------

Open the sudc application in the browser:

	open http://$(echo $DOCKER_HOST):8000 # Mac OS X
	open http://localhost:8000 # linux
	
Let's double check that `sudc` is working as expected by running:

	nsd system check sudc
	
You should see the following output:

	Looking good! No deviation detected
	
The `check` command runs a system analysis to verify that everything is looking good! 
We can view the output of that analysis by running:

	nsd system analyze sudc

Kill two of the containers
-------------

Now we'll kill the `web` and `doc-srv` containers:

	docker ps
	docker kill <web container id>
	docker kill <doc container id>

Then we'll verify sudc is not running:

	open http://$(echo $DOCKER_HOST):8000 # Mac OS X
	open http://localhost:8000 # linux

The address should be unreachable. 

Now let's do the check again:

	nsd system check sudc
	
We should see some output similar to the following:

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

System fix and sudc redeploy
-------------	

Let's do a system fix to `get` sudc up and running again:

	nsd system fix sudc
	
And check `sudc` is working as expected again:

	nsd system check sudc
	
	open http://$(echo $DOCKER_HOST):8000 # Mac OS X
	open http://localhost:8000 # linux

[Next up: exercise 7](https://github.com/nearform/nscale-workshop/blob/master/ex7.md)
