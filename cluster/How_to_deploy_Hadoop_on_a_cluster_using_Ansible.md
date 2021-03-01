Title:___How to deploy hadoop on a cluster using Ansible___
Date:____04.01.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: Hadoop, Cluster, Distributed Computing, Ansible, Playbook

# How to deploy hadoop on a cluster using Ansible
This howTo documents how to deploy hadoop on the nodes in our cluster using Ansbile playbook. We plan to use hadoops hdfs for storing the data in our cluster. In the experimental setup I follow [this](https://www.linode.com/docs/guides/how-to-install-and-set-up-hadoop-cluster/) instructions. Which contains the ssh access, downloading the hadoop binaries, setting up some environment variables and configure couple of hadoop relates config files. In general the installation and execution of hadoop is very similar to that of Spark. Now I want to automate the installation of hadoop on the node with the help of Ansible playbooks. 

# Procedure
So first I will search for tutorials on the topic and for existing playbooks. And adjust aspect of our special usecase. Next I need to create virtual machines identical to the systems in our existing cluster. Then I find an existing playbook and modify it to my need. Next I test run the playbook on my virual cluster. Finally I run the playbook on the cluster. 
The next step is to ensure the installation was successfull by developing some kind of test method. So check if the data is correctly distributed of the cluster and so on. But this will be done in an other document.

# Sources on how to setup hadoop with Ansible on the internet
	- most popular Hadoop playbook in Ansible Galaxy (Playbook repo)[link](https://galaxy.ansible.com/andrewrothstein/hadoop)
	- Block on the matter from 2015 but explained pretty good [link](https://software.danielwatrous.com/install-and-configure-a-multi-node-hadoop-cluster-using-ansible/)
	- Medium Article with medium quality [link](https://yadav12122000.medium.com/create-a-hadoop-cluster-by-ansible-b6e806fd5b34)
	- another Medium Article but with pretty bad quality [link](https://medium.com/hackcoderr/configure-hadoop-and-start-the-cluster-services-using-the-ansible-79a77e7a5999)

# Major Steps when installing hadoop with prerequisits

	- Confirm password less ssh between hosts and master is possible
	- Set Environment Variable with path to hadoop directory (in ~/.profile and ~/.bashrc
	- Install openjdk 8 
	- Set Environment Variable for JAVA_HOME 
	- Download hadoop
	- Make entries in config files
	- Set Memory and other resources for the nodes
	- hdfs format
	- start dfs and yarn

# Aspect to consider when deploying hadoop on our premise
	
	- Only the masternode has access to the internet

# Example Playbook by dwatrous
A *role* in Ansible is the mechanism to break a playbook into multiple files in order to reduce complexity and improve resuablity.

	---
	- name: Install hadoop master node # Name of the play
	  hosts: hadoop-master # hosts defines the maschines on which the play is performed, this is based on the entries in the hosts files
	  remote_user: ubuntu
	  sudo: yes

[ inside the repo of this playbook there is a directory called roles with in turn contain directories common oraclejava8 and master, each of them contains again subdirectories called, task, templates and vars. The task directory contains the a main.yml with the instructions of this role. The template contains files which are being used in the main.yml for example an ssh pub key. Vars contains another main.yml with contains variables use in the main.yml of task. In common it's things like user names.] 
	  roles: 
	    - common
[ What does common do?]
 - set group, user and ssh key
 - download hadoop 
 - untar tarball
 - mv hadoop directory
 - add environment variable to .bashrc
 - Build /etc/hosts file 
 - add service scripts for hadoop
	- core-site.xml
	- hdfs-site.xml
	- yarn-site.xml
	- mapred-site.xml
 - ensure that hostkeys is a known host

	    - oraclejava8
[ What does oraclejava8 role do?]
 - adds repository
 - installs oracle java8 
 - Adds JAVA_HOME to .bashrc

	    - master
[What does master do?]
 - Copies private ssh key into places
 - Copies slaves into places
 - Prepare known-hosts (something with rsa keys)
 - add 0.0.0.0 to known_hosts


	- name: Install hadoop data nodes # Same as before but for the nodes
	  hosts: hadoop-data
	  remote_user: ubuntu
	  sudo: yes

	  roles:
	    - common
	    - oraclejava8
	    - data
