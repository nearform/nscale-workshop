Check and fix
=============

with the sudc running:

	nsd system analyze sucd
	
	nsd system check sudc

Now kill the web and doc-srv containers

	docker ps
	docker kill <web id>
	docker kill <doc id>
	
check again

	nsd system check sudc
	
Describe outupt

Fix it using

	nsd system fix
	
check OK using

	nsd system check


