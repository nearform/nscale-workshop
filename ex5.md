Rollback
========

This tutorial covers:

1. Rolling a deployed system back to a known good state
2. Rolling a deployed system forward after applying a full fix

Add a buggy alert
------------
Let's break something! Open up `~/.nscale/data/build/sudc/startupdeathclock/web/public/js/app.js` and addan alert after the 'Your code here' comment:

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

	open http://$(boot2docker ip):8000

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/bugalert.png)

Roll back
------------

It's time to quickly rollback, picking the **second** revision id:

	nsd revision list sudc
	nsd revision deploy sudc <revision id>

Check the site is working:
    
	open http://$(boot2docker ip):8000 # Mac OS X
	open http://localhost:8000 # linux

Roll forward
------------

The site is back up and running so let's fix the bug in the code and apply that fix.

Start by removing the buggy alert from the code:

	(cd ~/.nscale/data/build/sudc/startupdeathclock && git checkout web/public/js/app.js)

Roll forward the change:

	nsd container buid sudc web
	nsd revision list sudc
	nsd revision deploy sudc <revision id>
	
Check the site is working:

	open http://$(boot2docker ip):8000 # Mac OS X
	open http://localhost:8000 # linux

[Next up: exercise 6](https://github.com/nearform/nscale-workshop/blob/master/ex6.md)
