Title:___HowToSetupaSparkCluster__
Date:____03.12.2020__
Author:__Gabriel Mendoza Reyes__
Keyword:_Spark, Hadoop, Cluster, Distributed Computing, Arch

# How to create a virtual Spark Cluster using Virtualbox
This HowTo should help you to set up a couple of virtual machines in virtualbox. On this machines we will install Spark on top of hadoop. On the masternode a jupyter notebook will be installed to execute pyspark-code on the cluster. The idea behind this setup is firstly to get a better understanding of setting a cluster and secondly to have a way to experiment with code before running it on the entire cluster.

This HowTo is based on this [medium article](https://medium.com/@pavelpokrovsky/apache-spark-archlinux-on-laptop-part-1-73a82527ea7), with some adjustments and corrections.

## Setting Up the Virtual Machine
For this HowTo we will be using Oracles Virtualbox. Its free and avialable for all OSes and linux distributions. The vm will run Arch linux. The VMs will use two networks, on NAT to connect to the internet, and a host-only network to connect to each other.

- Create a new vm with NAME: spark-master, OS: arch64, about 20GB of hdd space, use Ram accorindg to your preferences. (Tried with min. 1024mb) Leave all other option in default.

- In Virtualbox *File* go to Host Network Manager Create a new Network with Name *vboxnet*. Leave all in default. Once we have set fixed Ip adresses we will come back here to disable the dhcp Server.

- Go to the setting of the just created Arch-VM and go to Network. Enable Adapter 2, Attach to Host-Only Adapter and choose the Network you just created.

- Download the [arch-linux.iso](https://www.archlinux.org/download/) Kernel Version when writing this: (5.9.11). Perform Checksum Integrity for extra points!

- Start the created VM and Select the downloaded iso as start-up disk.

## Installing the Operating System
Now we are going to install and adjust the operating system so that we can use it as a cluster.
We should be greeted with the following prompt: root@archiso ~# _

- Change keyboard layout to german(/ is next to the right shift key):

```console
# loadkeys /usr/share/kbd/keymaps/i386/qwertz/de-latin1
```
- Create a partition with cfdisk:
	
	# cfdisk /dev/sda

	- Choose dos

	- New, default partition size, primary, Partition type: Linux (83)

	- Make it Bootable, see the star in the Boot row

	- Write Changes and Confirm with yes and quit

- Format the partition with:
	
	# mkfs.ext4 /dev/sda1

- mount the partition to /mnt
	
	# mount /dev/sda1 /mnt

- Install the Arch on the now mounted partition
	
	# pacstarp /mnt base base-devel

- Generate a fstab for the new system
	
	# genfstab -U /mnt >> /mnt/etc/fstab

- Change into the newly installed system
	
	# arch-chroot /mnt
	(Notice the change of the input prompt)

- Adjust locales and make keyboard settings permanent

	- Set time zone with symlink:
	
		# ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime

	- Sync hw-clock:

		# hwclock --systohc (double dash)

	- Install vim or other editor to modify the following textfiles:
		
		# pacman -S vim

	- Uncomment the line en_US.UTF-8 UTF-8 in /etc/locale.gen and generate the locale with:
	
		# locale-gen

	- Make keyboard changes permanent by adding the following line to a newly created /etc/vconsole.conf

		KEYMAP=de-latin1
	
- sudo Install all needed packages
	
	# pacman -S grub linux linux-headers virtualbox-guest-dkms networkmanager python3 python-virtualenv python-pip vim openssh git jdk8-openjdk inetutils
	(Choose mkinitcpio --  default-option)

- Install grub bootloader
	
	# grub-install /dev/sda

	# grub-mkconfig -o /boot/grub/grub.cfg

- Create a new user called spark and add him to the wheel group:
	
	# useradd -d /home/spark -m -G wheel spark

- Give the spark user password-less sudo rights. This is not very save and only recommended for non productive setups.  
	
	# visudo
	
	Uncomment the line:

	%wheel ALL=(ALL) NOPASSWD: ALL

	- Save and quit with :wq

- Set password for spark user:
	
	# passwd spark

- Exit chroot
	
	#  exit

- Shutdown the System
	
	# shutdown and get a coffee.

## Setting up the Operating System

- In the Storage option of the VM remove the boot disk. So that we
 boot into the installed OS.

- Boot the VM and login with the spark user

 ## Configure Network in the VM

- Enable the networkmanger:

	$ sudo systemctl enable NetworkManager --now

- Enable the ssh Server:
	
	$ sudo systemclt enable sshd --now:
	
- Give VM static Ip address. As mentioned above, the VM has two network devices. enp0s3 for communication with the internet, and enp0s8 for host communication. This is the connection where we want to keep a static ip, so that the machines will always find each other after reboot. Also we need to set the hostname of the vm so that we can create aliases for the machines. To set this up we use NM tui:
	
	$ sudo nmtui
	
	- Edit a connection:
	
	- Choose Wired Connection 2, confirm that the Device is enp0s8, if not try Connection 1

	- In IPv4 Configuration switch to manual and enter the spark-master ip-adress: 192.168.56.200. This address corresponds to the address of the host-only network created in the beginning. In our case the vboxnet* adapter should have the ip address 192.168.56.1. If its not the first adapter created adjust the addresses accordingly. Leave the rest in default and quit.

	- Go to Set system hostname and set it to *spark-master*
	
	- Reboot, now the VM should have the given internal ip.
	
- In the next step we setupt /etc/hosts so all machines know of each other. Modify the given examples for how many VMs you want to deploy. In this example we will created a spark-master, and will will later create two spark-worker nodes.

	- Open /etc/hosts with sudo rights
	
	- Add the following lines

	192.168.56.200 master spark-master
	192.168.56.201 worker1 spark-worker1
	192.168.56.202 worker2 spark-worker2

	- Save and quit
 
- Download spark:
	
	- Go to the download [link](https://spark.apache.org/downloads.html) on the apache spark site. Choose the version you want to use. In this example we use *spark.3.0.1-bin-hadoop2.7.tgz*. Get the downloadlink for the server given on top. And download the file into the home folder using curl:
	
	[spark@spark-master ~]$ curl "https://ftp.fau.de/apache/spark/spark-3.0.1/spark-3.0.1-bin-hadoop2.7.tgz" -o spark.tgz

- Untar with:
	
	tar zxf

- Remove the tarbal with:
	
	rm spark.tgz

- Change the directory name with:
	
	mv spark-3.0ß.1.-bin-hadoop2.7/ spark

At this point we can clone the spark-master. Next we will make small changes to the clones and exchange ssh keys with the VMs so that the master can ssh into the worker without a password. Finally we have to make some changes to some spark config files and install the notebook. In the end we will run some example code on the cluster to verify that the installation was successful. 

## Clone the spark-master to create the workers

- Shutdown the vm

- Go to the VM in Virtualbox and choose clone, repeate this for the number of clone you want to use:
	- Name: spark-worker1
	- Generate new MAC addresses for all network adapters
	- Keep disk names and harware UUIDs
	- Clone
	- If you want you can now adjust the memory of the workers

- Now we need to adjust the hostname and  their ip adresses. Use nmtui like before. Set ip address for worker1 to 192.168.56.201 and worker2 to .202, and their hostnames accordingly. Reboot, and verify the changes with *hostname* and *ip addr*.

- Verfiy the connection by pinging the workers from the master with:

	$ ping 192.168.56.201

- Now let's make ssh passwordless by placing the masters public key into the authorized_keys folder

	- Generate a ssh key in the master with:

	$ ssh-keygen (leave passphrase blank, and everything at default)

		- Create and a .ssh folder in the home directory of the worker:

		$ ssh worker1 'mkdir ~/.ssh'	

		- Now place the generated key into the workers:

		$ cat ./ssh/id_rsa.pub | ssh worker1 'cat >> .ssh/authorized_keys'

		- Verify that you have passwordless ssh access:

		$ ssh worker1

		- Repeate procedure for the other workers


- In the following steps we need to tell spark about our cluster setup. First we inform the slaves about their master address and then the master of the address of the slaves.

	- Go to worker1, you can do this via the VM windows, but also via ssh, since we just finished setting it up . The advantage is that you can now use your hosts terminal and use copy/paste.

	- go to the conf directory of spark

	$ cd /home/spark/spark/conf

	- cp the spark-env.sh.template

	$ cp spark-env.sh.template spark-env.sh
	
	- Add the following lines:
	
	SPARK_MASTER_IP=192.168.56.200

	SPARK_LOCAL_IP=<ip address of the workers host-only adapter (92.168.56.201-for worker1)>

	- Repeate this for the other worker

- Tell the master about the slaves:

	- Go to the master

	- go to the conf directory

	$ cd  /home/spark/spark/conf

	- Cp the slaves.template

	$ cp slaves.template slaves

	- Modfiy slaves, remove the localhost and add the following lines:

	worker1
	worker2
	
- Start the cluster

	- Go to sbin folder

	$ cd /home/spark/spark/sbin

	- execute the start-all script

	$ ./start-all.sh

This should start the spark cluster on our VMs. We can verify the successful deployment by inspecting the Web-UI on port 8080 of the master. The deployed workers should be visible in the Workers section. If not: Recheck if ssh access form master to workers is working. Check ip addresses in the spark config files. Also check the log files shown after runing the start-all.sh script for hints. 

## Installing jupyter notebook and verifing the cluster deployment

- On the master create a virtualenv 
	
	$ mkdir ~/sparkvenv

- Create a virtualenvironment in the created folder

	$ virtualenv ~/sparkvenv

- Start enter the venv 

	$ source ~/sparkvenv/bin/activate

- Install jupyter with pip

	$ pip install jupyter

- Setting up jupyter so we can access is from the host

	- Generate a jupyter config directory with

	$ jupyter notebook --generate-config

	- Set a passwort for jupyter so we don't need to deal with tokens

	$ jupyter notebook password

	- Modify Jupyter configuration file to enable access from the host ~/.jupyter/jupyter_notebook_config.py add the following lines:

	c.NotebookApp.ip = '192.168.56.200'

	c.NotebookApp.open_browser = False

	
Now we need to set some enviroment variable so that pyspark will find our spark installation and so that we can start pyspark together with the notebook and install pyspark into the created environment.

	- Add the following lines to ~/.bashrc

	export SPARK_HOME="/home/spark/spark"
	export PYSPARK_DRIVER_PYTHON=jupyter
	export PYSPARK_DRIVER_PYTHON_OPTS="notebook"

	- Reload .bashrc with

	source ~/.bashrc
	
	Install pyspark into the enviroment, make sure you are inside the enviroment indicated by command prompt.

	pip install pyspark

Finally we can run the cluster with the follwing command, before change into the home folder or create a folder we to save the notebooks to:
	
	(sparkvenv) [spark@spark-master sparkscripts]$ pyspark --master spark://master:7077

This command will execute a notebook server listening on 192.168.56.200:8888, on the host we can go to that address and log into jupyter notebook with the given password. After that we can create a new python3 file and execute the following example code which calculates *pi*.

	from pyspark import SparkContext
	import random as rnd
		
	-- new cell --

	%%timeit
	SAMPLES = 10000000

	def inside(p):     
		x, y = rnd.random(), rnd.random()
  		return x*x + y*y < 1

	count = sc.parallelize(range(0, SAMPLES)).filter(inside).count()
	pi = 4 * count / SAMPLES
	
	print(pi)

You can compare your execution times with the ones given in the medium article. Check the masterUI if all workers are active. Also spark creates a applicationUI on port 4040 for each job, here you can get a lot of other information on the task the cluster is working on. 

