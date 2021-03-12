Title:___Deployment Strategy for Hada's Vagrant Cluster___
Date:____12.03.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: Cluster, Distributed Computing, Ansible, kubernetes, kubespray, hada, Vagrant
# Deployment Strategy for Hada's Vagrant Cluster
In the past days I gathered some experience in the deployment of kubernetes with kubesprays vagrant deployment. This deployment is fairly easy. I tried to deploy it offline mode but was never successful. I have the impression that this vagrant deployment is not really meant for offline deployment. Before I get into the trouble setting up a secured registry and all the things that go into it. I want to switch the deployment to hada's vagrant cluster. Where I have an offline setup. Hopefully the setup will work there. 

# Prerequisites 
Before deploying all artifact sources have to setup and tested. 
Clone repo to the master.
## Files_Repo
	- execute download_artifact.sh to download all the necessary files and link those to the corresponding folder structure kubespray needs.

## Apt-Repo
	- Add docker repo to the apt-mirror and the master. 
	- Learn how to validate those repos for the slaves to prevent errors and warnings.

# Introduction
Here I want to list the detailed steps to deploy kubernetes with kubespray. Basis will be the [documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/) of kubernetes and the git repo [/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray) containing the playbooks.

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
	- To enable ipv4 forwarding the playbook *enable_ipv4_forwarding.yml* has been created. It uses a collection from the ansible-galaxy.
		- Install the collection with: `ansible-galaxy collection install ansible.posix`
		- Run the playbook with: `ansible-playbook enable_ipv4_forwarding.yml -u ansible --ask-become-pass`

- your ssh key must be copied to all servers part of your inventory
	  (Maybe create a new kuberntes user, with corresponding key, or check if present user can do the task)
	- The playbook will be deployed as user ansible this user has its keys distributed all over the cluster

- Firewalls are not managed, during deployment switch off the firewall, than add custom rules
	- The masters firewall will allow all outgoing traffic and block all incoming traffic excepct on port ssh port. The firewall can be disabled with `ufw disable`  

- If kubespray is ran from non-root user account, correct privilege escalation method should be configured in the target servers. Then the ansible_become flag or command parameters --become or -b should be specified

	
# Compose an inventory file

After you provision your servers, create an inventory file for Ansible. You can do this manually or via a dynamic inventory script. For more information, see "Building your own inventory".
Before creating an inventory a couple of thought have to be put in to the architecture of the cluster. [Ranger}(https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/production/recommended-architecture/) provides a very comprehensive dokumentation on the function of each node and what thoughts go into the architecture. Key takeaways are:
	- Worker nodes should not run etcd
	- Use more than one controleplane/kubernetes master node for High Availbility
	- Min etcd nodes should be (n*/2)+1 with n being the cluster nodes, for HA-Cluster
 
 
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

# Install Python Requirements
The kubespray's github readme has an additional step, which installs the required python packages with:
	`sudo pip3 install -r requirements.txt`


# Deploy a Cluster
Run the ansilbe-playbook

# Verfiy the deployment



