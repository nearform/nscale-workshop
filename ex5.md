Rollback
========

This tutorial teaches you:

1. Roll the system back to a known good state
2. Roll the system forward after applying a full fix

Add a buggy alert
------------

Open ~/.nscale/data/build/sudc/startupdeathclock/web/public/js/app.js and add an alert after the 'Your code here' comment:

	...
        initialize: function () {
          // Your code here
          alert('whoops');
        }
	...

Build the container and deploy the latest revision:

	nsd container buid sudc web
	nsd revision list sudc
	nsd revision deploy sudc <revision id>

Check the site for the buggy alert:

	echo $DOCKER_HOST
	open http://<ip>:8000

Roll back
------------

It's time to rollback, picking the **second** revision id:

	nsd revision list sudc
	nsd revision deploy sudc <revision id>

Check the site is working:

	echo $DOCKER_HOST
	open http://<ip>:8000

Roll forward
------------

Remove the buggy alert:

	(cd ~/.nscale/data/build/sudc/startupdeathclock && git checkout web/public/js/app.js)

Roll forward the change:

	nsd container buid sudc web
	nsd revision list sudc
	nsd revision deploy sudc <revision id>
	
Check the site is working:

	echo $DOCKER_HOST
	open http://<ip>:8000

[Next up: exercise 6](https://github.com/nearform/nscale-workshop/blob/master/ex6.md)
