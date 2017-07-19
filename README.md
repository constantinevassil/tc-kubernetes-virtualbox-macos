# kubernetes-macos

Vagrant config to run a full local Kubernetes cluster using the source directory from your Mac.

## Getting started

git clone https://github.com/topconnector/tc-kubernetes-macos.git

cd kubernetes-macos

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

Kubernetes is hard to install without using third party tools. kubeadm is an official tool for simple deployment. 

* Before you begin
	1.	One or more virtual machines running Ubuntu 16.04+
	1.	1GB or more of RAM per machine (any less will leave little room for your apps)
	1.	Full network connectivity between all machines in the cluster

* Objectives
	* Install a secure Kubernetes cluster on your machines
	* Install a pod network on the cluster so that application components (pods) can talk to each other
	* Install a sample Golang application on the cluster

Everything is done manually for a better understanding of the process. Here is Vagrantfile I used to run 2 VMs:

```javascript
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/xenial64"
 
    config.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 1
    end
 
    config.vm.define "master" do |node|
      node.vm.hostname = "master"
      node.vm.network :private_network, ip: "192.168.33.10"
      node.vm.provision :shell, inline: "sed 's/127\.0\.0\.1.*master.*/192\.168\.33\.10 master/' -i /etc/hosts"
    end
 
    config.vm.define "worker" do |node|
      node.vm.hostname = "worker"
      node.vm.network :private_network, ip: "192.168.33.20"
      node.vm.provision :shell, inline: "sed 's/127\.0\.0\.1.*worker.*/192\.168\.33\.20 worker/' -i /etc/hosts"
    end
 
    config.vm.define "worker2" do |node|
      node.vm.hostname = "worker2"
      node.vm.network :private_network, ip: "192.168.33.30"
      node.vm.provision :shell, inline: "sed 's/127\.0\.0\.1.*worker2.*/192\.168\.33\.30 worker2/' -i /etc/hosts"
    end
end
```

After all VMs are up and running the first step is to add official Kubernetes repo and to install all required packages.

### Install all required packages
#### 1. On master

```bash
vagrant ssh master
ubuntu@master:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
ubuntu@master:~$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
ubuntu@master:~$ sudo apt-get update && sudo apt-get install -y docker-engine kubelet kubeadm kubectl kubernetes-cni
ubuntu@master:~$ exit
```

Repeat above step on the workers:

#### 2. On worker

```bash
vagrant ssh worker
ubuntu@worker:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
ubuntu@worker:~$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
ubuntu@worker:~$ sudo apt-get update && sudo apt-get install -y docker-engine kubelet kubeadm kubectl kubernetes-cni
ubuntu@master:~$ exit
```

#### 3. On worker2

```bash
vagrant ssh worker2
ubuntu@worker2:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
ubuntu@worker2:~$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
ubuntu@worker2:~$ sudo apt-get update && sudo apt-get install -y docker-engine kubelet kubeadm kubectl kubernetes-cni
ubuntu@master:~$ exit
```

#### 4. Start cluster initialization on the master node.


When using flannel as the pod network (described in step 6.), specify --pod-network-cidr=10.244.0.0/16. 
```bash
vagrant ssh master
ubuntu@master:~$ sudo kubeadm init --apiserver-advertise-address 192.168.33.10 --pod-network-cidr 10.244.0.0/16 --token 8c2350.f55343444a6ffc46
```

#### 5. To start using your cluster, you need to run (as a regular user):

```bash
ubuntu@master:~$ mkdir -p $HOME/.kube
ubuntu@master:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
ubuntu@master:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 6. You should now deploy a pod network to the cluster.

Flannel RBAC:
```bash
ubuntu@master:~$ curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml
ubuntu@master:~$ kubectl apply -f kube-flannel-rbac.yml
```

Flannel config:
```bash
ubuntu@master:~$ curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
ubuntu@master:~$ kubectl apply -f kube-flannel.yml
```

#### 7. Check the cluster initialization:
```bash
ubuntu@master:~$ kubectl get pods -o wide --all-namespaces
```

After successfull initialization you should get:
```bash
ubuntu@master:~$ kubectl get pods -o wide --all-namespaces
NAMESPACE     NAME                             READY     STATUS    RESTARTS   AGE       IP              NODE
kube-system   etcd-master                      1/1       Running   0          8m        192.168.33.10   master
kube-system   kube-apiserver-master            1/1       Running   0          8m        192.168.33.10   master
kube-system   kube-controller-manager-master   1/1       Running   0          8m        192.168.33.10   master
kube-system   kube-dns-2425271678-d5b85        3/3       Running   0          13m       10.244.0.2      master
kube-system   kube-flannel-ds-vkcqt            2/2       Running   0          1m        192.168.33.10   master
kube-system   kube-proxy-vthjs                 1/1       Running   0          13m       192.168.33.10   master
kube-system   kube-scheduler-master            1/1       Running   0          8m        192.168.33.10   master
ubuntu@master:~$ exit
```

#### 8. Adding Kubernetes nodes. It is easy - just one command.

```bash
vagrant ssh worker
ubuntu@worker:~$ sudo kubeadm join --token 8c2350.f55343444a6ffc46 192.168.33.10:6443 --skip-preflight-checks
ubuntu@worker:~$ exit
```
```bash
vagrant ssh worker2
ubuntu@worker2:~$ sudo kubeadm join --token 8c2350.f55343444a6ffc46 192.168.33.10:6443 --skip-preflight-checks
ubuntu@worker2:~$ exit
```

#### 9. Check the nodes creation:

```bash
vagrant ssh master
ubuntu@master:~$ kubectl get nodes
```

After successfully adding nodes you should get:
```bash
ubuntu@master:~$ kubectl get nodes
NAME      STATUS    AGE       VERSION
master    Ready     19h       v1.7.1
worker    Ready     19h       v1.7.1
worker2   Ready     19h       v1.7.1
```

#### 10. Check the pods creation:

ubuntu@master:~$ kubectl get pods -o wide --all-namespaces

After successfull initialization you should get:
```bash
ubuntu@master:~$ kubectl get pods -o wide --all-namespaces
NAMESPACE     NAME                             READY     STATUS    RESTARTS   AGE       IP              NODE
kube-system   etcd-master                                1/1       Running   0          19h       192.168.33.10   master
kube-system   kube-apiserver-master                      1/1       Running   0          19h       192.168.33.10   master
kube-system   kube-controller-manager-master             1/1       Running   0          20h       192.168.33.10   master
kube-system   kube-dns-2425271678-nj435                  3/3       Running   0          20h       10.244.0.2      master
kube-system   kube-flannel-ds-cj62d                      2/2       Running   0          19h       192.168.33.20   worker
kube-system   kube-flannel-ds-dsrrj                      2/2       Running   0          19h       192.168.33.10   master
kube-system   kube-proxy-pr1qd                           1/1       Running   0          19h       192.168.33.20   worker
kube-system   kube-proxy-xtbbr                           1/1       Running   0          20h       192.168.33.10   master
kube-system   kube-scheduler-master                      1/1       Running   0          20h       192.168.33.10   master
```

## Testing kubernetes

### 1. Create a Deployment that manages a Pod

deploy topconnector/helloworld-go-ws

```bash
vagrant ssh master
ubuntu@master:~$ kubectl run tc-helloworld-go-ws --image=topconnector/tc-helloworld-go-ws:v1 --port=8080 --record
```

Check rollout status:

```bash
ubuntu@master:~$ kubectl rollout status deployment/tc-helloworld-go-ws
deployment "helloworld-go-ws" successfully rolled out
```

View the Deployment:
```bash
ubuntu@master:~$ kubectl get deployments
NAME                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
tc-helloworld-go-ws          1         1         1            1           3m
```

View the Pods:

```bash
ubuntu@master:~$ kubectl get pods -o wide
NAME                                       READY     STATUS    RESTARTS   AGE       IP           NODE
tc-helloworld-go-ws-495672996-nt1m9           1/1       Running   0          5m        10.244.1.4   worker
```

### 2. Scaling:
```bash
ubuntu@master:~$ kubectl scale --replicas=2 deployment/tc-helloworld-go-ws --record
deployment "helloworld-go-ws" scaled
```

### 3. Create a service:
```bash
ubuntu@master:~$ kubectl expose deployment tc-helloworld-go-ws --type=NodePort
service "helloworld-go-ws" exposed
```

### 4. Access the service:

1. get node "worker"'s IP address:
```bash
ubuntu@master:~$ kubectl describe nodes
...
Addresses:
  InternalIP:	192.168.33.20
  Hostname:	worker
...  
```

IP address:192.168.33.20

2. get service port number

View the services:
```bash
ubuntu@master:~$ kubectl get services
NAME                  CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes            10.96.0.1       <none>        443/TCP          8m
tc-helloworld-go-ws   10.104.31.142   <nodes>       8080:32658/TCP   1m
```
service port number:30817 

### 5. Test the service:

The http address of the service: 192.168.33.20:32658

```bash
ubuntu@master:~$ curl http://192.168.33.20:32658
Hello World from Go in minimal Docker container(4.28MB) v.1.0, it took 78ns to run
```



