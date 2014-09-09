
Boot2docker OSX Installation
===================

Setting up boot2docker and an initial docker image will take a while as it needs to download large files from the Internet. So it's best to use the USB to get started quickly:

Install boot2docker
---------------

Go to the USB in Finder and double click on the Boot2Docker-1.2.0.pkg file.

Follow the installation steps and install boot2docker.

Locate the Boot2Docker app in your Applications folder and run it.

A terminal window will open and you'll see the virtual machine starting up.

Set DOCKER_HOST env in the terminal and in your ```~/.bash_profile```:

```bash
export DOCKER_HOST=tcp://$(boot2docker ip 2>/dev/null):2375
```

A terminal window will open and you'll see the virtual machine starting up. Once you have an initialized virtual machine, you can control it with ```boot2docker stop``` and ```boot2docker start```.

Check docker is working:

```bash
docker version
```

```bash
docker version
Client version: 1.1.2
Client API version: 1.13
Go version (client): go1.3
Git commit (client): d84a070
Server version: 1.2.0
Server API version: 1.14
Go version (server): go1.3.1
Git commit (server): fa7b24f
```

You can control boot2docker with ```boot2docker stop``` and ```boot2docker start```.

Load ubuntu docker images
---------------

Copy ubuntu_docker_images.tar from the USB into /tmp folder.

```bash
cp /Volumes/<usb>/ubuntu_docker_images.tar /tmp/
```

Load the ubuntu docker images into docker (this will take a few mins):

```bash
docker load < /tmp/ubuntu_docker_images.tar
```

List the docker images:

```bash
docker images
```

And you should see the following output:

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              trusty              826544226fdc        4 days ago          194.2 MB
ubuntu              14.04               826544226fdc        4 days ago          194.2 MB
ubuntu              latest              826544226fdc        4 days ago          194.2 MB
ubuntu              14.04.1             826544226fdc        4 days ago          194.2 MB
ubuntu              14.10               245ce11c1f25        4 days ago          202.5 MB
ubuntu              utopic              245ce11c1f25        4 days ago          202.5 MB
ubuntu              12.04               c17f3f519388        4 days ago          106.7 MB
ubuntu              12.04.5             c17f3f519388        4 days ago          106.7 MB
ubuntu              precise             c17f3f519388        4 days ago          106.7 MB
ubuntu              12.10               c5881f11ded9        11 weeks ago        172.2 MB
ubuntu              quantal             c5881f11ded9        11 weeks ago        172.2 MB
ubuntu              13.04               463ff6be4238        11 weeks ago        169.4 MB
ubuntu              raring              463ff6be4238        11 weeks ago        169.4 MB
ubuntu              13.10               195eb90b5349        11 weeks ago        184.7 MB
ubuntu              saucy               195eb90b5349        11 weeks ago        184.7 MB
ubuntu              10.04               3db9c44f4520        4 months ago        183 MB
ubuntu              lucid               3db9c44f4520        4 months ago        183 MB
```
