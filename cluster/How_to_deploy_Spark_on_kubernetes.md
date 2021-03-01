Title:___How to deploy spark on kubernetes with kubespray___
Date:____23.02.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: Spark, Cluster, Distributed Computing, Ansible, kubernetes, kubespray

# Introduction
Here I want to list the detailed steps to deploy kubernetes with kubespray. Basis will be the documentation of kubernetes and the git repo /kubernetes-sigs/kubespray containing the playbooks.

# Meet the unterlay requirements

	- Ansible v2.9 and python-netaddr in installed on the machine that will run Ansible commands
		- Installed: Ansible 2.9.17
		- **python-netaddr is installed** 

		- Installed jinja >2.11 is required to run the Ansible playbook
		- Debian Strech jinja2 version is 2.8 installed pip, and then pip install jinja2 to update to version 2.11, Confirmed with custom playbook, based on [stackoverflow question](https://stackoverflow.com/questions/49040013/how-can-i-know-what-version-of-jinja2-my-ansible-is-using)

		- The Target server must have internet access to pull docker images 
		- Since this is not the case in our cluster I have to configure the offline environment
