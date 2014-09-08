Check and fix
=============

Verify sudc is running
-------------

	echo $DOCKER_HOST
	open http://<ip>:8000
	
Check sudc is working as expected with the following commands:

	nsd system analyze sucd
	nsd system check sudc

Kill two of the containers
-------------

Now kill the web and doc-srv containers:

	docker ps
	docker kill <web container id>
	docker kill <doc container id>
	
Check again:

	nsd system check sudc
	
You should see some output similar to the following:


Fix sudc and get it up and running again
-------------	

Fix it using:

	nsd system fix
	
And check sudc is working as expected again:

	nsd system check
	
	echo $DOCKER_HOST
	open http://<ip>:8000

[Next up: exercise 7](https://github.com/nearform/nscale-workshop/blob/master/ex7.md)
