Title:___How to a private container image registry___
Date:____23.02.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: offline Cluster, Ansible, kubernetes, kubespray

# Introduction and Thoughts
The reason the a registry is implemented is that the cluster nodes are not connected to the internet and can't pull images from docker hub. Since the registry will only host images which are anyway available on docker hub and contain no private information, I will not implement any security, meaning certificates of tls connections. The registry is once started the images of interest are push into it. Then the deployment of the cluster is done. Once the deployment is finished the registry can be shutdown. So it will not cause any security issues. Since the master is anyway not accessible directly from the internet, I have a good feeling I can get away with that.
Ok, after looking at helm and how to setup a helm registry I found a very simple way to implement an authentification method. I will add this into the intructions from Exoscale. It basically creating auth.htpasswd file and adding it to the registry container.


# Setting up a private container image registry without Security
[HowTo from Exoscale]https://www.exoscale.com/syslog/setup-private-docker-registry/)

## Install latest docker release
Since a Docker registry is just a Docker image, first docker needs to be installed.

- Follow steps for docker installtion for debian from docker

- Check installtion with hello-world example

## Run registry container

	$ docker run -d -p 5000:5000 --restart=always --name registry registry:2

Adding bin-mounts to a directory on the base system seams a good idea:	
 	-v host_directory:container_directory

	$ docker run -d \
  	  -p 5000:5000 \
 	  --restart=always \
  	  --name registry \
  	  -v /mnt/registry:/var/lib/registry \
  	  registry:2

### Adding Authentification

	- Create auth.htpasswd file with user and password:
	htpasswd -cB -b auth.htpasswd myuser mypass

	- Run the registry with REGISTRY_AUTH env variable:
	docker run -dp 5000:5000 --restart=always --name registry \
  	-v $(pwd)/auth.htpasswd:/etc/docker/registry/auth.htpasswd \
  	-e REGISTRY_AUTH="{htpasswd: {realm: localhost, path: /etc/docker/registry/auth.htpasswd}}" \
  	registry


## Push custom Docker image to a remote private registry
An image accessible to remote machine this image needs to be stored in the Docker registry.

## Change container tag

	$ docker tag container-name <registry URL>:port/container-name 

docker imagae, will display two containers with different name but the same image ID

## Push the image to the registry

	$ docker push <registry URL>:Port/container-name

This will lead to an error. Since non secure connections are not allowed. Work around: Modify /etc/docker/deamon.json
with the following entry:
	{
	 "insecure-registries" : ["<registry URL>:port]
	}
If the file does not exist, create it. After that restart docker daemon. Push again!

To verify the presents of the container inside the registry we can curl the registry with this command:

	$ curl -X GET http://<registry URL>:port/v2/_catalog

Which should return:
	{"repositories":["container-name"]}

----------------------------------------------------------------------------------------------------------

# Setting up a registry according to docker docs

## Run a local registry

	$ docker run -d -p 5000:5000 --restart=always --name registry registry:2

## Copy an image from Docker Hub to your registry
This example will use an *ubuntu:16.04* image and will tag it with *my-ubuntu*

	- Pull the image from Docker Hub
	$ docker pull ubuntu:16.04

	- Tag the image
	$ docker tag ubuntu:16.04 localhost:5000/my-ubuntu

	- Push the image to registry
	$ docker push localhost:5000/my-ubuntu
	( In contrast to the instruction from Exoscale no modification to config files needed?!)
	
	- Remove local images, so we can test pulling the image from the registry
	$ docker image remove ubuntu:16.04
	$ docker image remove localhost:5000/my-ubuntu

	- Pull the image from the registry instead the docker hub
	$ docker pull localhost:5000/my-ubuntu

## Stop a local registry

	$ docker container stop registry

## Remove a local registry

	$ docker container stop registry && docker container rm -v registry

# After installation
Pulling the registry container seams not to require changes in the config files. Registry was reachable from another node via the
curl command.


	
