Title:___Choosing a Kubernetes deployment method___
Date:____22.02.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: Spark, Cluster, Distributed Computing, Ansible, kubernetes

# Introduction
After the deloyment of hdfs on the virtual cluster was successfull, the next step is to deploy spark. I want to use kubernetes as cluster manager. This dokument should give an overview of how the deployment works.

# Getting Started
The kubernetes doku distiguishes a Learning environment and a Production environment. Since we are planing to use the cluster productivly I will try to deploy one of the Production enviroments.

# Container runtimes
This is the basis for the Pods to run on. There are three different runtimes:
	- containerd
	- CRI-O
	- Docker

Kubernetes is not directly compatible with docker. So when using docker, the additional docker-shim has to be implemented. For ease of use and compatability reasons I will use conainerd or CRI-O. Installation of the runtimes is pretty straight forward. Later in the research proccess I found that CRI-O is not supported for Offline installation. See this [issue](https://github.com/kubernetes-sigs/kubespray/issues/6233)
	- Setup the repository
	- Add Docker GPG key
	- Add Docker apt repo
	- Install containerd
	- Set default containerd configuration
	- Restart containerd 
	- configure /etc/containerd/config.toml

# Cgroup drivers
Control groups are used to contrain resources that are allocated to processes. In other words its limiting how much resources you can use. While the namespace limits what you can see and therefore use.
This video explains the basics of containers pretty good: [video](https://www.youtube.com/watch?v=el7768BNUPw)
So either resources are managed by systemd of the main system. Or one can configure the container runtime and kubelet to use *cgroupfs* to do that. In this case the system would have two different cgroup managers. A single cgroup manager makes it easier to overview the allocated resoureces. Having two managers also tends to be problematic in the case of resource presure. So based on that, I will configure kubernetes to use systemd as cgroup manager. 


# Checking Prerequisites

- Spark 2.3 or above / I want to deploy the latest version -- 3.0.2 
- Kubernetes cluster version >=1.5 with access configured to it using **kubectl**
	- Usage of latest minikube release with DNS addon enabled recommended
	- 3 CPUs and 4g memory per single executor min. 
- you must have appropirate permissions to list, create, edit and delete pods in your cluster.
- You must have **Kubernetes DNS** configured in your cluster


# Choosing the deployment method
The are multiple tools to deploy an kubernetes cluster on hardware: 
[good overview of deployment tools}(https://medium.com/@m.k.joerg/overview-of-kubernetes-installers-8f06437d215a)

- kubeadm
- kops
- kubespray

In the next sections I will shortly give an overview of the different method and decide which one to pick.

## Bootstraping cluster with kubeadmn
[link](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)
This is default option by kubernetes. Setup is realtive easy. Only high availability cluster setup is complicated.
Steps: 
	- Install kubeadm
		- prerequisites
		- Install runtime  
		- Install kubeadm, kubelet, kubectl
		- Configure cgroup driver used by kubelet on control-plane node
	- Create a cluster with kubeadm
		- Initializing your control-plane node
			 - This is the node where the controle plane components run, including the etcd (the cluster database) and theAPI Server (which kubectl communicates with)
		- Install a Pod network on the cluster so that the Pods can talk to each other
		- Join your nodes
		

## Installing Kubenetes with kops

This option is focused on cloud deployment. Therefore it can't be used on premise. 


## Installing Kubernetes with Kubespray

Kubespray is a composition of Ansible playbooks, inventory, provisioning tools, and domain knowledge for generic OS/Kubernetes clusters configuration management tasks.s
Installation Steps:

	- Meet the underlay requirements
		- Ansible 2.9 and python-netaddr on ansible master
		- Jinja > 2.11
		- Target Servers must be able to pull docker images from internet. Otherwise Offline Environment has to be configured
		- Target Server must allow IPv4 forwarding
		- SSh key must be copied to all server in inventory
		- Disable firewalls during deployment
		- Correct privilege escalation method should be used

	- Compose an inventory file

	- Plan your cluster deployment
		- Choose deployment mode
		- CNI plugins
		- DNS configurations
		- Choose control plane
		- Choose Runtime option

	- Deploy a Cluster
		- run ansible-playbook 

	- Verify the deployment

** This will be my deployment method of choice. Since I have experience in the the deployment of hdfs with ansible. I think I can try this method.**

# Next steps
In the next steps I will decribe the steps for the deployment in detail. Have a look at the actual playbook in the kubespray git repo and check out other repos I find online...

 
## Prerequisites

- debian 9+ - debian9 present
- 2GB or more RAM per machine 
- 2 CPUs or more 
- Full network connectivity between all machines in the cluster
- Unique hostname, MAC address, and product_uuid for every node
- Certain ports are open on your machines
- Swap disabled 

I will create a new k8s cluster based on the vagrantfile I made for the hadoop cluster. At this moment I plan to run the hadoop cluster along the k8s cluster. So each node will not be share between those services. I will keep the same limitation in the k8s cluster as in the hadoop cluster. So only the master will have internet access. 

I have changed my mind about creating a new cluster. The issue is that creating an setting up a new apt-mirror might take a lot of time. I will use snapshots as a way to switch between clusters. Snapshot 4 contains the fully deployed hdfs cluster. Snapshot 3 contains the fully prepared cluter and is the starting point for the k8s cluster. 

### Verify the MAC address and product_uuid are unique for every node

| name | ip address | MAC address | product_uuid |
|:-----|:-----------|:------------|:-------------|
|master | 192.168.51.3 | 08:00:27:5d:95:79 | 651DC0DE-E7AF-4F5E-86E1-CFF0728BA96B|
|nnode1 | 192.168.51.4 | 08:00:27:89:8a:7d | 909BE965-5047-4AF9-AA0A-073D3496B7D0|
|nnode2 | 192.168.51.5 | 08:00:27:55:ac:74 | DA7BADD2-85AA-4E78-AC84-EC01D379A3CD|
|zookeeper | 192.168.51.6 | 08:00:27:36:aa:93 | F6B3BEE2-6EDC-4ACF-A100-7545E8D7B2D5|
|data1 | 192.168.51.7 | 08:00:27:ab:fb:42 | D8E547FF-1CE6-43BC-BE18-8B918ED91DF1|
|data2 | 192.168.51.8 | 08:00:27:b2:c3:87 | 6369F33D-C7E4-454F-9042-51F721484FBF|
|data3 | 192.168.51.9 | 08:00:27:b2:da:34 | 9DC47E38-D4AC-4B5A-B914-08B246718188|


All machines satify this requirements. Consideration: All machines run two ethernet devices eth0 and eth1. I only check the addresses from the eth1 devices!

