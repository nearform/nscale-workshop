Rollback
========

This tutorial teaches you:

1. Roll system back to known good state
2. Roll system forward after applying a full fix


open ~/.nscale/data/build/sudc/startupdeathclock/web/public/js/app.js

insert the bug dialog

	require(['config', 'death'], function (config, death) {
	console.log(death)
	var app = {
        initialize: function () {
			alert('whoops');
            // Your code here
        }
    };
	app.initialize();
	});

Build the container

	nsd container buid sudc web
	
Deploy the system

	nsd revision list sudc
	nsd revision deploy sudc <revision id>

Check the site out http://<boot2dockerip>:8000

See the error dialog

Rollback

	nsd revision list sudc
	nsd revision deploy sudc <revision id>

edit the code again and fix it

Roll forward

	nsd revision list sudc
	nsd revision deploy sudc <revision id>


