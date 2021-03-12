Title:___How to prepare for offline environment___
Date:____05.03.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: offline Cluster, Ansible, kubernetes, kubespray, private registry

# Introduction 
The default installation process of kubernetes expects that all nodes have access to the internet and with that to the known repositories to pull all kinds of container, files in whatnot - aka artifacts. This documents collects all the measures that I have taken to prepare to prepare for the offline environment and to document the steps taken and the thoughts made. 

The kubernets documentation has an entry for [offline environment]{../kubespray/docs/offline-environment.md} which list the main components which need to be made accessible and what to setup. 

- various static files as zips and binaries through an HTTP mirror
- an Deb repository for OS packages
- an internal container image registry that need to be populated with all images used by Kubespray Exhaustive list depends on your setup 
- optional an internal PyPi server for kubespray python packages 
- optional an internal Helm Registry 

# The files_repo
In the inventory directory is a file called offline.yaml, here we find various parameters which need to be tweaked for the offline deployment. To pull the various file I wrote an [script]{../kubespray/scripts/download_artifacts.sh}. When updating kubernetes, the versions of the corresponding files need to be adjusted. These files need to be put in the correct folder of the apache server `/var/www/html/...` or softlinked.

# Registry_host
All container images used by the cluster need to be made available for the nodes. I could have looked through the playbooks and config files and complied a list of the images. But this process is very tedious and error prone. Luckily  kubespray provides a [script]{../kubespray/contrib/offline/manage-offline-container-images.sh} for this purpose. This script compiles a list of used images from an existing kubernetes cluster and pulls those images from the docker repository. In a second step a registry is setup and the images are pushed into it. 
So for this method to work I need an existing cluster with the same images. The quickest way to deploy a kubernetes cluster with kubespray is to use vagrant to spin up a couple of vms and to deploy kubernetes on those. Kubespray also has a documentation entry for [this]{../kubespray/docs/vagrant.md}. The only thing missing in this documentation was to publish the path to the `admin.conf` of the cluster. Which can be done with `export KUBECONFIG= .../kubespray/.vagrant/provisioners/ansible/inventory/artifacts/admin.conf` After that I was able to access the cluster using `kubectl cluster-info`. And the script could be executed. 
TODO:
The creation of the registry is very vanilla, so it includes no adjustment for dns or authentication. For the final deployment some form of authentification needs to be implemented, either through certificates of user-password combination. 

# Deb Repository
An apt-mirror is already implemented. So all nodes have access to the debian repository. 
TODO:
The docker repository needs to be added.

# Helm Registry
As Helm Registry I will use Chart Museum as registry. [Here]{https://github.com/helm/chartmuseum#how-to-run} are the instruction to install and configure it. 