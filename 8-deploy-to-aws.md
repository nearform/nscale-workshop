Deploying on AWS
================

This tutorial covers:

1. Installing nscale on an EC2 instance
2. AWS configuration file updates
3. Deployment into AWS
4. Check and fix on AWS

During this tutorial we will assume deployment onto a linux based AMI. Specifically we assume use of ubuntu as the base operating system.

Installing nscale on AWS
------------------------
nscale should be installed on AWS in a similar manner to a direct linux install:

* boot up a community base VM. Do this using the ubuntu [EC2 AMI locator](http://cloud-images.ubuntu.com/locator/ec2/). We recommend using a 64 bit Trusty image.

* connect to your newly booted VM and install docker as per the instructions [here](http://docs.docker.com/installation/ubuntulinux/)

* add yourself to the docker group using:
```bash
$sudo usermod -G docker -a $(whoami)
```
For these changes to come into effect you usually need to log out and log back in.
close the ssh connection with your instance with ```exit``` and reconnect.

* It is recommended you install the build-essentials package. Many default AMIs do not come with packages such as make and g++. This will ensure you have the packages required for some npm modules to work. Failing to do so may lead to problems installing nscale depending on the AMI you launched.

	`$sudo apt-get install build-essential`

* Install node and npm from [here](http://nodejs.org/download/)

* Install git and configure:
    * git config --global user.name "your name"
    * git config --global user.email "you@somewhere.com"

Enable SSH Agent Forwarding
----------------------------
create a file `config` in the ~/.ssh folder on your local machine and insert the following:
```
host *
 ForwardAgent yes
```
This allows the remote instance to use your local ssh key for cloning Github repos. You may need to log out and back in to see the effect of this.

Once the above dependencies have been met you can proceed to install nscale using 
```bash
$sudo npm install -g nscale
```

Recommended
----------------------
To make life easier, there is a powerful tool called SSHFS which allows you to mount a remote file system locally. This lets you view files on your remote instance in your file browser and you'll be able to edit them using graphical editors. Check out the guide to using it [here](http://www.emreakkas.com/linux-tips/how-to-mount-amazon-ec2-drive-locally-fuse-sshfs)

Configuring AMIs for use with nscale
------------------------------------
A base AMI configured for management by nscale should be created. We can just create a reusable image based off our newly configured instance as per above. An easy way to do this is through the EC2 Dashboard:

* Find the instance you've just set up and right click.
* Image > Create Image.
* Follow the steps provided.

See [here.](http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/ApiReference-cmd-CreateImage.html) if you would prefer to use the AWS Command Line Interface tools.   

Once the server has been imaged, make a note of the ami identifier and log back into the running management system.

AWS configuration file updates
------------------------------
In order to operate correctly on AWS the nscale coniguration file (~/.nscale/config/config.json) requires some additional parameters. These are as follows:

* Kernel section
  * user - the username to use when connecting to remove systems (ubuntu)
  * identityFile - path to the ssh key file required to connect to remote systmes
  * region - the AWS region you are using
  * accessKeyId - your AWS API access key
  * secretAccessKey- your AWS secret access key
* Modules section
  * analysis section
    * set the analyzer to - nscale-aws-analyzer
  * Containers section
    * add aws-elb-container - specifiy the default subnet and vpc ID
    * add aws-sg-container - specifiy the default subnet and vpc ID
    * add aws-ami-container - specifiy the default subnet and vpc ID and also the ami id 

A full AWS confiuration file should look similar to the following:

```js
{
  "kernel": {
    "port": "8010",
    "root": "/home/ubuntu/.nscale",
    "user": "ubuntu",
    "identityFile": "FULL_PATH_TO_YOUR_KEY.pem",
    "region": "us-west-2",
    "accessKeyId": "XXXXXXXXXXXXXXXXX",
    "secretAccessKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  },
  "api": {
  },
  "web": {
  },
  "modules": {
    "protocol": {
      "require": "nscale-protocol",
      "specific": {
      }
    },
    "authorization": {
      "require": "nscale-noauth",
      "specific": {
        "credentialsPath": "/home/ubuntu/.nscale/data"
      }
    },
    "analysis": {
      "require": "nscale-aws-analyzer",
      "specific": {
      }
    }
  },
  "containers": [{
    "require": "aws-elb-container",
    "type": "aws-elb",
    "specific": {
      "region": "us-west-2",
      "defaultSubnetId": "subnet-xxxxxxxx",
      "defaultVpcId": "vpc-xxxxxxxx"
      }
  },
  {
    "require": "aws-sg-container",
    "type": "aws-sg",
      "specific": {
      "region": "us-west-2",
      "defaultSubnetId": "subnet-xxxxxxxx",
      "defaultVpcId": "vpc-xxxxxxxx"
    }
  },
  {
    "require": "aws-ami-container",
    "type": "aws-ami",
    "specific": {
      "region": "us-west-2",
      "defaultImageId": "ami-xxxxxxxx",
      "defaultSubnetId": "subnet-xxxxxxxx",
      "defaultVpcId": "vpc-xxxxxxxx"
    }
  },
  {
    "require": "blank-container",
    "type": "blank-container",
    "specific": {}
  },
  {
    "require": "docker-container",
    "type": "docker",
    "specific": {
      "imageCachePath": "/tmp"
    }
  },
  {
    "require": "process-container",
    "type": "process",
    "specific": {}
  }]
}
```

Deploying into AWS
------------------
Now that we have configured our AWS setup lets proceed to deploy a system into AWS. To do this we will clone and deploy the startup death clock.

Lets get started by cloning the repository onto our AWS management system. Firstly cd into a working folder lets say /home/ubuntu/work/sudc.
```bash
git clone git@github.com:nearform/sudc-system.git
nscale system link sudc-system
```
This will pull down the code for the Startup Death Clock system. In order to work with the AWS version of the system we will need to switch branches. to do this:
```bash
cd /home/ubuntu/work/sudc/sudc-system
git checkout v0.14
```
Lets take a look at the differences between the local configuration (on the master branch) and the AWS version:

####Infrastructure
The AWS version contains an additional file awsInfrastructure.js, which contains some AWS specific component definitions

 * awsWebElb - elastic load balancer definition
 * awsWebSg - security group definition
 * awsMachine - base AMI definition

####Topology
The AWS version also contains an updated topology in system.js which is specific for deployment into our AWS infrastructure. The topology section is as follows:

```
  aws: {
    awsWebElb: [{
      awsWebSg:[{awsMachine: ['web']},
                {awsMachine: ['doc', 'hist', 'real']}]
    }]
  },
```

The topology section defines how the service containers are distributed amongst machine instances.

####Edit
In order to make this work we need to make some minor adjustments to the ids specified in the configuration files:

* replace AMI-ID - open the file definitions/awsInfrastructure
  * under awsWebElb change the AvailabilityZone setting to match your availability zone
  * under awsWebSg change the VpcId setting to match your VPC
  * under awsMachine change the ImageId to match your ami id



####Compile

Lets now go ahead and compile for aws
```bash
nscale system compile sudc
```
Now let's take a look at the system definition:
```bash
nscale container list sudc
```
You should see the following containers:

	web                  docker
	hist                 docker
	real                 docker
	doc                  docker

####Build the system
Let's go ahead and build the containers ready for deployment:
```bash
nscale container buildall sudc latest aws
```
Alternatively, you can build all the containers by themselves:

	nscale container build sudc hist latest aws
	nscale container build sudc real latest aws
	nscale container build sudc doc latest aws
	nscale container build sudc web latest aws

After those have all completed we should have four containers ready for deployment.

####Deploy the system
Now that we have an AWS system definintion and a set of containers to deploy, we can go ahead and push our system out onto AWS. 

Lets first run a preview:
```bash
nscale revision preview sudc latest aws
```
This will run an analysis against your configured AWS region and may take several seconds to complete. The preview should determine that a full deployment is required. nscale will show you a list of commands that will be executed against your infrastrcutre upon deployment. You should review this list to ensure that you are comfortable with the changes. If you are lets not go ahead and deploy.
```bash
nscale revision deploy sudc latest aws
```
nscale will no go ahead and deploy the SUDC system into your infrastructure. The following actions will be taken:

* create a new load balancer awsWebElb
* create a new security group awsWebSg
* spin up two machine instances
* deploy front end container to one of the machine instances and start it
* deploy 3 service containers to the other machine instnace and start them

Deployment may take several minutes depending on AWS machine instance spin up time. Once complete you should be able to point a web browser at port 8000 on the front end instance and view the startup death clock front end.

Check and Fix on AWS
--------------------
Now that we have a deployed working system on AWS, lets just check that everything is in order by running an nscale check:
```bash
nscale system check sudc aws
```
The check command will run an analysis against the deployed system and compare it to your desired system configuration. This may take a few moments to run, but nscale should now respond that all is well with your deployment. Lets break something!

Go into the AWS management console and kill the SUDC front end machine. Now run the check again on the management host:
```bash
nscale system check sudc aws
```
This time nscale should report that the system is broked and present a remedial action plan. To go ahead and fix the system execute:
```bash
nscale system fix sudc aws
```
nscale will now boot up a replacement machine and deploy the appropriate containers to it, onece the fix has completed double check it by running:
```bash
nscale system check sudc aws
```
Congratulations - you are now an nscale AWS ninja!
