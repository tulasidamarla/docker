# Docker

Containers
-
- Containers are all about applications. 
- A container is an isolated area with resource usage limits applied. 
- A container maintains this isolation using kernal's name spaces and control groups.
- Name spaces and control groups are present since the evolution of unix/linux OS, but they remained obscure because of the complexity. Docker Engine makes this very easy.
 
![containers](containers.png)

Docker Engine
-
- Docker Engine is modular.
- We interact with Docker engine either through CLI or api. Under the hood Docker engine consists of various parts like docker daemon, containerd, OCI etc.
- The command `docker container run...` to start the container, it makes a call to the api `/containers/create` in the docker engine.
- Docker engine gets all the kernal stuff like control groups and name spaces and start the container.

![docker engine](dockerengine.png)

Namespaces and Control Groups
-
- Linux containers are around even before docker. To make them working one must be an expert on the kernal.
- Name spaces and Control groups are two major building blocks for building containers.
- These two are linux kernal primitives, and now they are available on windows aswell.
  - Namespaces
	- Name spaces are about isolation and Control groups are about grouping objects and setting limits.
	- Name spaces split the Host OS into multiple isolated virtual OS's like hypervisors.
	- With hypdervisor a single physical machine resources like CPU, RAM, disk storage, networking are split into multiple VM's, such that each VM has its own Virtual CPU, Virtual RAM, Virtual disk storage, Virtual networking etc.
	- In container world, name spaces use HOST OS resources which are high level constructs like file system, process trees and users and turn them into multiple OS's called containers.
	- Each container gets it's own file system, process tree, root user etc, but share the same OS.
	- In Linux world namespaces are collection of 
	  - Process ID(pid): Each container has it's own process tree with its own pid 1. 
	  - Network(net): Each container has it's own ipaddresses, route tables etc.
	  - File system/mount(mnt): Each container has it's own root file system `/` or `C:\` 
	  - Inter-proc comms(ipc): Each container gets it's own shared memory isolated from other containers.
      - UTS(uts): Every container get's its own hostname.
	  - User(user): Specific to docker, to provide different privileges to different users with in the container.
  - Control Groups(In windows Job objects)
	- Control groups are used for grouping process and imposing resource limits like amount of CPU, RAM , disk storage.

![Control Groups](control_groups.png)

Images
-
- Image is a read only template for creating application containers. 
- Typical Image consists of OS files/objects, app files, manifests etc.
- Images are build-time construct, whereas containers are run-time siblings.
- An image is a bunch of layers stacked up on one another.
- An image can be used to run multiple containers from it.
- Images are stored in a registry(on-premise or cloud) and later can be pulled on to the hosts(`docker pull image-name`) in which container needs to be run.

![Images](images_1.png)

Images in Detail
-
- To get image from registry use the command `docker image pull` or `docker pull`. For ex: `docker pull redis`.

- An image is a bunch of independent layers that are loosely coupled by a manifest file. So, an image is not a single blob file.
- A manifest file describes the image like id, creation time and most importantly the list of layers that it stacks.
- The layers are unaware of the bigger image. So, one layer of an image has no reference to the other.
- To support multi architecture support and content addressable storage, Pulling a docker image consists of two parts.
  - Get manifest: Fat manifest and image manifest
  - Pull layers

![Docker  info](docker_info.png)

Manifest
-
- There are two types of manifest
  - Fat manifest
    - Contains details about the list of architectures supported and a manifest for each of those.
	- `docker pull` gets the fat manifest and verifies if the fat manifest contains the host system architecture in the list of supported architectures.
  - Image manifest
    - If fat manifest supports the host system architecture, then image manifest is pulled.
    - Each of the layers present in the image manifest is pulled from docker registry's blob store as shown in the image below.
	- To look at content of manifest `docker inspect imageName/imageId`

![Pull image](images_2.png)

Content addressable storage
-
- Earlier docker did not hash it's images and there was no easy way to know if the image that is pulled from registry is right.
- Docker 1.10 introducted CAS(content addressable storage)
- CAS hashes all of the content in the image and use that as image id, so when we pull image from registry, we run the hash of it and verify if it matches the image id.
- The image manifest shows path locations where the image data is stored.

Layers
-
- Base Layer
  - This is the OS layer which contains the files and os objects used to build the basic OS like ubuntu etc. If the docker container uses ubuntu, it will have common ubuntu tools and file system utilities etc. So, it is totally possible to run a centos docker host with ubuntu based containers on it. 
  - It is going to be Linux containers on Linux host and Windows containers on Windows host on Kernal(This is not applicable for Hyper-V containers).
- App Layer
  - On top of Base Layer, there will be bunch of other layers which are required to run the application like program runtimes(jvm), build tools(maven), servers(tomcat), updates etc.
- Each of the layers are present in the directory /var/lib/docker/aufs/diff for linux, for windows C:\programData\docker\overlay2\diff.
- When building images some of the commands build new layers and some of them add content to the image config file. 
- The command `ls -l /var/lib/docker/aufs/diff` all layers as shown below.

![Layers](layers.png)  

- The command `docker history imageName/imageId` shows the commands that formed the image.

![Docker history](docker_history.png)  

- To remove the docker image `docker rm imageName`

Registries
-
- Images are pulled from registry to docker host.
- The default registry for docker is dockerhub. Other registries are available from google, amazon also.
- Docker provides an on-premise registry called Docker trusted registry(DTR) with it's enterprise edition.
- Docker maintains a local registry on each docker host. The path of local registry for linux is /var/lib/docker/storage-drive and for windows C:\programData\docker\windowsfilter.
- Docker registry maintains two levels of images.
  - Official images
    - These images reside at the top level of hub namespace(docker.io)
	- We name these images redis, nginx etc, but they are actually not the images.
	- The actual image name can be identified with REGISTRY/REPO/TAG
	  - Registry can be docker hub, google, amazon etc.
	  - Repo's can be redis, nginx , fedora, ubuntu etc.
	  - Tag's are the original images. In the redis ex, tag name is `latest`.
	  - Registry defaults to docker hub, tag defaults to latest if we don't mention explicitly.
	  - Full command to pull image is `docker pull REGISTRY/REPO:TAG`. For ex, `docker pull docker.io/redis:4.0.1`
	  - An image can have different tag ids. For ex, if latest version of redis is 4.0.1, then both will refer to the same image id.
  - Unofficial images
	- These images reside inside an organization name or an user name.
	- The unofficial images reside at /REGISTRY/(ORG_NAME/USER_NAME)/REPO/TAG
	- The command used to pull these images `docker pull (ORG_NAME/USER_NAME)/REPO:TAG`. For ex, `docker pull tulasiramdamarla/helloworld:1.0`.
- Pushing images to the registry
  - Each of the layers in an image contains data. For each layer a hash is computed using its content, which is nothing but the layer id. This is called as content hash.
  - When pushing images, we define a manifest that defines the layers in it and push the manifest and layers independently.
  - Distributed Hash
    - When pushing the layers over internet, they are compressed. As we know, layer id's are content hashes, when the data is compressed it's content will be changed. So, when these arrive at registry, verification fails.
    - To get around the problem of hashes, when maifest is created layers are compressed and new hash is created. This hash is called distributed hash.

Dockerfile
-
- Dockerfile is a list of instructions to build an image with the runtime required along with application inside.
- It is a convention to write all docker instructions in capital.
- An instruction if of the format `<Instruction> <value>`.
- Dockerfile always start with the FROM instruction which defines the base image.
- It is a good practice to list maintainer.
- Use RUN command to execute a command and build a layer.
- Use COPY command to copy code into image as a new layer.
- Some instructions like WORKDIR can add metadata to image configuration instead of creating layers.
- Use ENTRYPOINT to start the application inside the image.
- Let's see an example on how to write a Dockerfile for a node application.

		# Test web-app to use with Pluralsight courses and Docker Deep Dive book
		# Linux x64
		FROM alpine

		LABEL maintainer="tulasiram.damarla@gmail.com"

		# Install Node and NPM
		RUN apk add --update nodejs nodejs-npm

		# Copy app to /src
		COPY . /src

		WORKDIR /src

		# Install dependencies
		RUN  npm install

		EXPOSE 8080

		ENTRYPOINT ["node", "./app.js"]  
  
  

  






























In the initial days every application needs an infracture.
To run an application it needs hardware, os and installing lots of software, licenses etc. Each of the applications we run on the hardware is very less utilized. Average utilization for many of those is under 10%. It led to virtualization.

we can run multiple apps on a single physical machine using hyperviser virtualization. It improved utilization to more than 50%. Even though it is far better than previous models, it is not perfect.

All applications running inside a VM needs a OS. The OS's only purpose is to run the app. For ex, if we run 10 apps in a single physical machine, say each os require 1GB of RAM, 10GB of Harddrive. Without running any app, these vm's consume about 10GB of RAM and 100GB of hard drive. Again it's a waste of lot of resources.

Containers are more light weight than VM's. Unlike hyperviser virtualization , containers use os virtualization. Say an Unix os is running on a physical machine. Unix kernal manages the underlying hardware. Containers hold apps using a construct called user space. Containers create multiple isolated areas of disk user spaces. This is called container virtualization or os level virtualization. Every container shares the underlying OS. so containers consume less CPU and RAM.

Containers provide isolated instances of things like root file system, process tree, networking etc. If there are 10 containers each will have its own view of the root file system. same goes with the processes. A process inside one container cannot send signal to another process in another container. But, how does this work? There are various kernal features like name spaces, cgroups and capabilities to do this. 

Linux kernal uses various name spaces like pid (for processes), net (for networking), mnt (for mount and storage) and user (for users to have root previliges inside the container but not outside containers).

control groups (cgroups) used to map containers to cgroups. we can set limits to memory, cpu etc that can be used by a container with cgroups. If we run multiple containers on a machine these are very useful.

capabilities give fine grained control of what previliges an user or a process gets. Instead of having root(all) and non root (nothing), it provides lot of things like the following.

    CAP_AUDIT_CONTROL
    CAP_CHOWN
    CAP_DAC_OVERRIDE
    CAP_KILL
    CAP_NET_BIND_SERVICE
    CAP_SETUID
    .
    .
    .


Instead of giving root previleges to all users, capabilities allow an user to have more fine grained control.

Docker Images
-------------
Images are like templates for a container. An Image contains multiple layers. A running Docker image is a container. 

For ex to start a container from an image,
-----------------------------------------
      docker run -it fedora /bin/bash
      //i --> interactive
      //t --> sudo tty

The above command runs fedora image(downloaded from docker repository). All the repositories present in the docker registry https://hub.docker.com. Inside a repository there would be different versions of images.


/bin/bash --> to open bash terminal inside the fedora image that was run.

To run a container in detached mode 
-----------------------------------
     docker run -d fedora 

Note: To run a container, execute a command and then exit<br>

     docker run -d ubuntu /bin/bash -c "echo 'cool content' > /tmp/coolfile"


Note: The above command will pull the docker image fedora from https://hub.docker.com and saves it locally if not already exist.

Pull images from docker public registry 
---------------------------------------
     docker pull fedora
     docker pull -a fedora // pulls all versions

Note: The pull command will always pull latest from repository. 

To fetch all images of fedora
---------------------------
    docker images fedora

To fetch list of docker processes currently running
---------------------------------------------------
    docker ps
    docker ps -a  (shows all container that were run on the host including the currently running ones)

Note: CONTAINER_ID can be fetched from 'docker ps' command.

To attach to a running container,
--------------------------------
    docker attach CONTAINER_ID

Note: To quit the container shell, without killing it by pressing CTRL+p+q.

creating new image from existing
--------------------------------
Get CONTAINER_ID 'docker ps' and execute the command "docker commit CONTAINER_ID NEW_IMAGE_NAME".

To fetch the history of an image
--------------------------------
    docker history IMAGE_NAME/IMAGE_ID

To save docker image to local file system
-----------------------------------------
    docker save -o /path/filename IMAGE_NAME/IMAGE_ID
    Ex: docker save -o /tmp/fridge.tar fridge

To load an image file
---------------------
    docker load -i /tmp/fridge.tar

To see the processes running inside a container
-----------------------------------------------
    docker top CONTAINER_ID

To allocate memory for container
--------------------------------
    docker run memory=1g (Allocates 1GB memory)

To get complete information of a container,
-------------------------------------------
    docker inspect CONTAINER_ID ( we can run this against IMAGE_ID also)

Starting and stopping containers
--------------------------------
a)To start an ubuntu container in interactive mode, `docker run -it ubuntu /bin/bash`<br>
b)To quit the container or to dettach from a container without stopping use `CTRL+P+Q<br>`<br>
c)To stop the container `docker stop CONTAINER_ID` or `docker kill CONTAINER_ID` (brute force)<br>
4)To start the cotainer again `docker start CONTAINER_ID`

`docker info` gives the no of images and containers present in the local system.<br>
`docker rm CONTAINER_ID` to delete a container. you can't delete a container that is currently running. To remove currently running container use flag '-f'.

Getting logs
-------------
    docker logs CONTAINER_ID --> gives list of commands ran inside the container

To remove image
---------------
    docker rmi IMAGE_ID --> To remove the image(Container has to be removed first).

Shell Access
------------
All containers may not provide you a shell access when you run "docker attach". This is because not all containers provide shell. For ex, apache webserver running in a container may not provide shell access. To do this, follow the below steps.

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
 




