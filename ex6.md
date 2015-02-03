Check and fix
=============

This tutorial covers:

1. Verifying that the system is running according to expectations
2. Fixing the system after two containers are killed off

Verify sudc is running
-------------

    OSX:
	open http://$(boot2docker ip):8000

	Linux:
	open <a href="http://localhost:8000" target="_blank">http://localhost:9000</a>
	
Let's double check that `sudc` is working as expected by running:

	nscale system check sudc development
	
You should see the following output:

	Looking good! No deviation detected
	
The `check` command runs a system analysis to verify that everything is looking good! 
We can view the output of that analysis by running:

	nscale system analyze sudc development

Kill two of the containers
-------------

Now we'll kill the `web` and `doc-srv` containers:

	docker ps
	docker kill <web container id>
	docker kill <doc container id>

Then we'll verify sudc is not running:

    OSX:
	open http://$(boot2docker ip):8000

	Linux:
	open <a href="http://localhost:8000" target="_blank">http://localhost:9000</a>

The address should be unreachable. 

Now let's do the check again:

	nscale system check sudc development
	
We should see some output similar to the following:

	Deviation from deployed revision detected!
	Remedial action plan as follows:

	execution plan: 
	Command                        Id                                                
	add                            doc-c31f912e$77c4014bef47deb4fef3af579f2959457c05…
	start                          doc-c31f912e$77c4014bef47deb4fef3af579f2959457c05…
	link                           doc-c31f912e$77c4014bef47deb4fef3af579f2959457c05…
	add                            web-5a16c094$3779d219ae5108029625d849e15bbd1ef3be…
	start                          web-5a16c094$3779d219ae5108029625d849e15bbd1ef3be…
	link                           web-5a16c094$3779d219ae5108029625d849e15bbd1ef3be…

	operations: 
	Host                 Command                                                                                                                                               
	localhost            docker run  -p 9002:9002 -d localhost:8011/sudc/doc-77c4014bef47deb4fef3af579f2959457c058ce8 node /srv/doc-srv.js && docker tag localhost:8011/sudc/d…
	localhost            docker run  -p 8000:8000 -d localhost:8011/sudc/web-3779d219ae5108029625d849e15bbd1ef3bed74e /bin/bash /web/run.sh && docker tag localhost:8011/sudc/…

	run 'system fix' to execute


System fix and sudc redeploy
-------------	

Let's do a system fix to `get` sudc up and running again:

	nscale system fix sudc development
	
And check `sudc` is working as expected again:

	nsd system check sudc development
	
	OSX:
	open http://$(boot2docker ip):8000

	Linux:
	open <a href="http://localhost:8000" target="_blank">http://localhost:9000</a>

[Next up: exercise 7] (https://github.com/nearform/nscale-workshop/edit/master/ex7.md)
