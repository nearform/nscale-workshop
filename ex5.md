Rollback
========

This tutorial covers:

1. Rolling a deployed system back to a known good state
2. Rolling a deployed system forward after applying a full fix

Before Making any Changes
-----------------
cd into the sudc/workspace/sudc-web directory and run a git branch command. 
If the HEAD is detached, then do 
```bash
git checkout master
```
The HEAD may already have been detached depending on how you've followed the workshops.
(more on that later)

Add a buggy alert
-----------------

Let's break something! Open up `sudc/workspace/sudc-web/web/public/js/app.js` and add an alert after the 'Your code here' comment:

```js
initialize: function () {
    // Your code here
    alert('whoops');
}
```

**cd into the sudc/workspace/sudc-web directory**, stage the changes and commit:
```bash	
git add .
git commit -m "Added buggy alert"
```

Build the container and deploy the latest revision:
```bash
nscale system compile sudc development
nscale container build sudc web
nscale revision deploy sudc latest development
```

Check the site for the buggy alert:

OS X:
open http://$(boot2docker ip):8000

Linux:
open [localhost:8000](http://localhost:8000)

![image](https://raw.githubusercontent.com/nearform/nscale-workshop/master/img/bugalert.png)

It is this workflow which allows us to makes changes and deploy them using nscale:
	
	- Make changes in code
	- Commit
	- (optional) push to Github
	- System compile
	- Build relevant container(s)
	- System Deploy

Pay particular attention to the fact that nscale will checkout repos using the sha of the latest commit, which will detach the HEAD.

**Tip:**
	Before writing code that you are building into a container, do a git status of that repo to make sure the HEAD isn't detached or else you will be writing code on an unnamed branch. Checkout to master or another branch before proceeding.

Roll back
------------

It's time to quickly rollback, picking the **second** revision id:
```bash
nscale revision list sudc
nscale revision deploy sudc <revision id> development
```

Check the site is working:
    
OS X:
open http://$(boot2docker ip):8000

Linux:
open [localhost:8000](http://localhost:8000)

Roll forward
------------

The site is back up and running so let's fix the bug in the code and apply that fix.
remove the alert statement from app.js and commit the change or:

open the sudc/workspace/sudc-web directory
```bash
git checkout master (because the HEAD is now detached from the container being built)
git revert HEAD --no-edit
```

Roll forward the change:
```bash
nscale system compile sudc development
nscale container build sudc web
nscale revision deploy sudc latest development
```	
Check the site is working:
    
OS X:
open http://$(boot2docker ip):8000

Linux:
open [localhost:8000](http://localhost:8000)

[Next up: exercise 6](https://github.com/nearform/nscale-workshop/blob/master/ex6.md)
