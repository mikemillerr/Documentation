Title:___How to deploy kubernetes with kubespray___
Date:____23.02.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: Cluster, Distributed Computing, Ansible, kubernetes, kubespray

# Introduction
Here I want to list the detailed steps to deploy kubernetes with kubespray. Basis will be the documentation of kubernetes and the git repo /kubernetes-sigs/kubespray containing the playbooks.

# Meet the unterlaying requirements

- Install docker on the master node, accoring to the docs using the repository
[Install Docker Engine on Debian using the repository](https://docs.docker.com/engine/install/debian/#install-using-the-repository)

- Ansible v2.9 and python-netaddr in installed on the machine that will run Ansible commands
	- Installed: Ansible 2.9.17
	- **python-netaddr is installed** 
	
- Installed jinja >2.11 is required to run the Ansible playbook
	- Debian Strech jinja2 version is 2.8 installed pip, and then pip install jinja2 to update to version 2.11, Confirmed with 
	custom playbook, based on [stackoverflow question](https://stackoverflow.com/questions/49040013/how-can-i-know-what-version-of-jinja2-my-ansible-is-using)

- The Target server must have internet access to pull docker images 
	- Since this is not the case in our cluster I have to configure the offline environment
	[How to setup a private container image registry](./How_to_setup_a_private_container_image_registry.md)
- The Target server are configured to allow IPv4 forwarding

- your ssh key must be copied to all servers part of your inventory
	  (Maybe create a new kuberntes user, with corresponding key, or check if present user can do the task)

- Firewalls are not managed, during deployment switch off the firewall, than add custom rules

- If kubespray is ran from non-root user account, correct privilege escalation method should be configured in the target servers. Then the ansible_become flag or command parameters --become or -b should be specified

	
# Compose an inventory file

After you provision your servers, create an inventory file for Ansible. You can do this manually or via a dynamic inventory script. For more information, see "Building your own inventory".


# Plan your cluster deployment
Kubespray provides the ability to customize many aspects of the deployment:

	- Choice deployment mode: kubeadm or non-kubeadm
    	- CNI (networking) plugins
    	- DNS configuration
    	- Choice of control plane: native/binary or containerized
    	- Component versions
    	- Calico route reflectors
    	- Component runtime options
        	- Docker
         	- containerd
        	- CRI-O
    	- Certificate generation methods

Kubespray customizations can be made to a variable file. If you are just getting started with Kubespray, consider using the Kubespray defaults to deploy your cluster and explore Kubernetes.

# Deploy a Cluster
Run the ansilbe-playbook

# Verfiy the deployment



