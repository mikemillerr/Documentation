Title:__hadoop_installation_steps_of_teralitcs_ansible_playbook___
Date:____16.02.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: Hadoop, Cluster, Ansible, playbook, teralytics


# Introduction
I used teralytics playbook to install hadoop on a virtual cluster I made with vagrant on the hada server. But I has many shortcomings, its also too complex and only work with an old version of hadoop. The idea of this dokumentation is to take this playbook as a basis and combine it with the instruction I used to install hadoop on an earlier cluster. For that I will extract the steps into this dokument.

# File Structure
The various tasks a structured into various playbooks. This structure is implemented by using an main-playbook which calls the other playbooks if needed.

## Variables
This file contains all variables to be adjusted for the installation
	
	ansible-hdfs/default/main.yml

## Tasks
This directory contains the playbooks.

	ansible-hdfs/task/

This is the main playbook, which call the task inside the other playbooks.

	ansible-hdfs/task/main.yml
	
	- user.yaml

	- native.yml

	- base.yml

	- config.yml

	- datanode.yml

	- namenode.yml

	- secondarynamenode.yml

	- journalnode.yml

	- bootstrap_spof.yml
	  when hdfs_bootstrap and not hdfs_ha_enabled

	- bootstrap_ha.yml
	  when hdfs_bootstrap and hdfs_ha_enabled

	- upgrade.yml
	  when hdfs_upgrade

	- scripts.yml

# user.yaml

- Add hadoop group on all machines
  
- Add hadoop user on the first namenode only and generate an ssh key 

- Create user on all machines

- ssh_fence.yml
	- when hdfs_ssh_fence and invetory_hostnam in hdfs_namenodes

## ssh_fence.yml

- Check if ssh keys should be distributed

- Set distribute keys variable

- Fetch private key

- Fetch public key

- Create .ssh directory for {{hdfs_user}}

- Copy private key to all machines

- Copy pubkeys to master server

- Make sure the known hosts file exists

- Add long names to namenodes for proper key deployment

- Check host name availability

- Scan the puplic key

- Delete key locally


# native.yml
Is only necessary if hadoop should be build from source

# base.yml

- Install some packages needed for native use

- Make sure parent directory exists

- Download Hadoop .tgz to {{hdfs_parent_dir}}

- Unarchive downloaded Hadoop

- Link hadoop version to {{hdfs_hadoop_home}}

- Create folder /etc/hadoop

- Create hadoop link for conf to /etc/hadoop

- Create link for hdfs to /usr/local/bin

- Create link for hadoop to /usr/local/bin

- Export hadoop variables

- Allow hadoop variables keeping for sudoers

- Create rack awareness script

- Create hadoop tmp dir

- Create hadoop log dir

- Create directory for unix sockets

# config.yml

- Configure hadoop-env.sh

- Configure core-site.xml

- Configure hdfs-site.xml

- Configure log4j.properties

# datanode.yml

- Create datanode directories

- Set program variable to 'datanode'

- Deploy init.d service for datanode

- Deploy systemd service for datanode

- Reload systemd daemon

# namenode.yml

- Configure mapred-site.xml

- Configure slaves

- Create namenode directories

- Create exclude file

- Set program variable to 'namenode'

- Deploy init.d service for namenode

- Deploy systemd service for namenode

- Set program variable to 'zkfc'

- Deploy init.d service for zkfc

- Deploy systemd service for zkfc

- Reload systemd daemon

- Register namenode service

- Register zkfc service

# secondarynamenode.yml

- Set program variable to 'secodarynamenode'

- Create directory for namenode checkpoints

- Deploy init.d service for secondarynamenode

- Deploy systemd service for secondarynamenode

- Reload systemd daemon

# journalnode.yml

- Create journalnode edits dir

- Set program variable to journalnode

- Deploy init.d service for journalnode

- Deploy systemd service for journalnode

- Reload systemd daemon

# bootstrap_ha.yml

- Pause - Bootstrapping is about to begin

- Ensure that zookeeper is running

- Ensure that journalnodes are running

- Format namenode {{hdfs_namenode[0]}}

- Start namenode {{hdfs_namenode[0]}}

- Wait for the namenode {{ hdfs_namenode{0]}} to come online

- Bootstrap the standby namenode ({{hdfs_namenode[1]}})

- Start namenode {{hdfs_namenodes[1]}}

- Fromat ZK for zkfc

- Start zkfs service

- Start data nodes

- Bootstrapping complete


# scripts.yml

- Create log comrpress and rotate script on {{ hdfs_bin_dir }}

