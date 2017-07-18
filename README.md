# kubernetes-macos

Vagrant config to run a full local Kubernetes cluster using the source directory from your Mac.

## Getting started
You must have the following installed:

* Virtualbox >= 5.1.22

  Download and install from https://www.virtualbox.org/.

* Vagrant >= 1.9.7

  Download and install from https://www.vagrantup.com/.

* vagrant-vbguest Vagrant plugin
  automatically installs the host's VirtualBox Guest Additions on the guest system.

  Install by running: 

    vagrant plugin install vagrant-vbguest

    vagrant vbguest --do install

* update Vagrant box

  Install by running: 
    
    vagrant box update
    
* run Virtual machine (VM)

  Install by running: 
  
    vagrant up

## Using kubeadm to create a cluster

* Before you begin
	1.	One or more machines running Ubuntu 16.04+
	1.	1GB or more of RAM per machine (any less will leave little room for your apps)
	1.	Full network connectivity between all machines in the cluster (public or private network is fine)

* Objectives
	* Install a secure Kubernetes cluster on your machines
	* Install a pod network on the cluster so that application components (pods) can talk to each other
	* Install a sample microservices application (a socks shop) on the cluster

