# Docker

Containers are all about applications. 

In the initial days every application needs an infracture.
To run an application it needs hardware, os and installing lots of software, licenses etc. Each of the applications we run on the hardware is very less utilized. Average utilization for many of those is under 10%. It led to virtualization.

we can run multiple apps on a single physical machine using hyperviser virtualization. It improved utilization to more than 50%. Even though it is far better than previous models, it is not perfect.

All applications running inside a VM needs a OS. The OS's only purpose is to run the app. For ex, if we run 10 apps in a single physical machine, say each os require 1GB of RAM, 10GB of Harddrive. Without running any app, these vm's consume about 10GB of RAM and 100GB of hard drive. Again it's a waste of lot of resources.

Containers are more light weight than VM's. Unlike hyperviser virtualization , containers use os virtualization. Say an Unix os is running on a physical machine. Unix kernal manages the underlying hardware. Containers hold apps using a construct called user space. Containers create multiple isolated areas of disk user spaces. This is called container virtualization or os level virtualization. Every container shares the underlying OS. so containers consume less CPU and RAM.

Containers provide isolated instances of things like root file system, process tree, networking etc. If there are 10 containers each will have its own view of the root file system. same goes with the processes. A process inside one container cannot send signal to another process in another container. But, how does this work? There are various kernal features like name spaces, cgroups and capabilities to do this. 

Linux kernal uses various name spaces like pid (for processes), net (for networking), mnt (for mount and storage) and user (for users to have root previliges inside the container but not outside containers).

control groups (cgroups) used to map containers to cgroups. we can set limits to memory, cpu etc that can be used by a container with cgroups. If we run multiple containers on a machine these are very useful.

capabilities give fine grained control of what previliges an user or a process gets. Instead of having root(all) and non root (nothing), it provides lot of things like the following.

CAP_AUDIT_CONTROL<br>
CAP_CHOWN<br>
CAP_DAC_OVERRIDE<br>
CAP_KILL<br>
CAP_NET_BIND_SERVICE<br>
CAP_SETUID<br>
.<br>
.<br>
.<br>
etc..<br>

Instead of giving root previleges to all users, capabilities allow an user to have more fine grained control.

Docker Images
-------------
Images are like templates for a container. An Image contains multiple layers. A running Docker image is a container. 

For ex to start a container from an image,
-----------------------------------------
docker run -it fedora /bin/bash

The above command runs fedora image(downloaded from docker repository). All the repositories present in the docker registry https://hub.docker.com. Inside a repository there would be different versions of images.

i --> interactive<br>
t --> sudo tty

/bin/bash --> to open bash terminal inside the fedora image that was run.

To run a container in detached mode 
-----------------------------------
docker run -d fedora 

For ex, If you want to run a container, execute a command and then exit, 
-----------------------------------------------------------------------
docker run -d ubuntu /bin/bash -c "echo 'cool content' > /tmp/coolfile"

Note: The above command will pull the docker image fedora from https://hub.docker.com and saves it locally if not already exist.

you can pull images from docker public registry using docker pull command like below.
------------------------------------------------------------------------------------
docker pull fedora 

Note:The pull command will always pull latest from repository. 

If you want to pull all versions of fedora,
-----------------------------------
docker pull -a fedora

If you want to see all images of fedora,
----------------------------------------
docker images fedora

To see list of docker processes currently running,
-------------------------------------------------
docker ps

docker ps -a  (shows all container that were run on the host including the currently running ones)

To attach to a running container,
--------------------------------
docker attach CONTAINER_ID

CONTAINER_ID can be fetched from 'docker ps' command.

quit the container, without killing it by pressing CTRL+p+q.

creating new image from existing.
--------------------------------
get the CONTAINER_ID from the command 'docker ps -a'.

docker commit CONTAINER_ID NEW_IMAGE_NAME

To see the history of an image how it was formed
------------------------------------------------
docker history IMAGE_NAME/IMAGE_ID

To save docker image 
--------------------
docker save -o /path/filename IMAGE_NAME/IMAGE_ID

For ex, docker save -o /tmp/fridge.tar fridge

To load the file created in the above step
------------------------------------------
docker load -i /tmp/fridge.tar

To see the processes running inside a container
-----------------------------------------------
docker top CONTAINER_ID

docker run memory=1g (gives container 1GB memory)

To get complete information of a container,
-------------------------------------------
docker inspect CONTAINER_ID ( we can run this against IMAGE_ID also)

Starting and stopping containers
--------------------------------
For ex, to start an ubuntu container in interactive mode, use the following command<br>
1)docker run -it ubuntu /bin/bash <br>
2)To quit the container or to dettach from a container without stopping use CTRL+P+Q<br>
3)docker stop CONTAINER_ID (to stop the container) or docker kill CONTAINER_ID (brute force)<br>
we can start the container again using the CONTAINER_ID using<br>
4)docker start CONTAINER_ID

"docker info" gives the no of images and containers present in the local system.<br>
"docker rm CONTAINER_ID" to delete a container. you can't delete a container that is currently running. To remove currently running container use flag '-f'.

Getting logs
-------------
docker logs CONTAINER_ID --> gives list of commands ran inside the container

To remove image
---------------
docker rmi IMAGE_ID --> To remove the image(Container has to be removed first).

Shell Access
------------
All the containers may not provide you a shell access when you run "docker attach". This is because not all containers provide shell. For ex, apache webserver running in a container may not provide shell access. To do this, follow the below steps.

docker inspect CONTAINER_ID | grep Pid --> This will give Pid say 1923.

nsenter -m -u -n -p -i -t 1923 /bin/bash

m --> mount name space
u --> uts name space
n --> network name space
p --> process name space
i --> ipc name space
t --> target

once you enter the shell , you can use normal shell commands. There is a simple way to enter the shell using " docker exec -it CONTAINER_ID /bin/bash".

Working with Dockerfile
-----------------------
Dockerfile contains all the commands a user could call on the command line to assemble an image. Using "docker build" users can create images. For ex,

docker build -t helloworld:0.1 .

t --> target<br>
helloworld -> is the name of the image with version number followed<br>
. --> to consider all the files from current directory for the build

working with repositories
-------------------------
To pull image , we use docker pull command. To push an image to a repository create an account followedby create a repository at hub.docker.com

After creating the repository, you need to tag your image to the repository like this.

docker tag IMAGE_ID username/repositoryname:version

For ex,

docker tag dec1bc188001 tulasiramdamarla/helloworld:1.0

you can see the changes with "docker images" command. Now you can use the following command.

docker push tulasiramdamarla/helloworld:1.0

Dockerfile contains the following important commands
----------------------------------------------------
FROM --> used to pull the base image from repository<br>
MAINTAINER --> Info of the user who maintains the Dockerfile<br>
RUN --> It is a build time construct. Add layers to images. Generally used to install applications.<br>
CMD --> It is a run time construct. used to run commands in containers at launch time. This is equivalent of "docker run <args> <command>" . We have used /bin/bash before. This is nothing but a command given to the container to execute after it is launched.

Note: Any command line argument with "docker run", will override CMD instruction of Dockerfile. Also, there can be only one CMD instruction per Dockerfile. If there are more CMD instructions, the last one is considered.

There are two ways we can work with CMD instructions. Shell form and Exec form.

Shell form --> CMD echo "Hello World " (These commands are prepended by "/bin/sh -c"<br>
Exec form --> CMD ["command","args"]

ENTRYPOINT --> unlike CMD, any argumnets passed with docker run command, goes to ENTRYPOINT command as an argument. For ex, dockerfile image name is test and the following command is present in Dockerfile.

ENTRYPOINT ["echo"]

Run the image using the following command prints "Hello World" on the screen.

docker run -d test Hello World

Note: Unlike CMD, you cannot override ENTRYPOINT command. you can pass argument to the ENTRYPOINT command but can't override. 

ENV --> used to enter environment variables in name=value format.

Volumes
-------
 




