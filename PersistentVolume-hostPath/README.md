# PersistentVolume-hostPath
create a hostPath PersistentVolume. Kubernetes supports hostPath for development and testing on a single-node cluster. 
A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage.

# Configure a Pod to Use a PersistentVolume for Storage

1. Login to worker node:

multiple machines:
```bash
cd ..
cd multiple-machines
vagrant ssh worker 
```
or 
single machine:
```bash
cd ..
cd single-machine
vagrant ssh master 
```

2. Create directory for ver. 1.0
```bash
ubuntu@master:~$ mkdir /tmp/www-01
```

In the /tmp/www-01 directory, create an index.html file:

```bash
ubuntu@master:~$ echo 'Hello from Kubernetes storage of www-01' > /tmp/www-01/index.html
ubuntu@master:~$ cat /tmp/www-01/index.html
exit
```
3. Create directory for ver. 2.0
```bash
ubuntu@master:~$ mkdir /tmp/www-02
```

In the /tmp/www-01 directory, create an index.html file:

```bash
ubuntu@master:~$ echo 'Hello from Kubernetes storage of www-02' > /tmp/www-02/index.html
ubuntu@master:~$ cat /tmp/www-02/index.html
exit
```
4. Create deployment for ver. 2.0

go back to PersistentVolume-hostPath

```bash
cd ..
cd PersistentVolume-hostPath
```

create deployment:

```bash
cd ..
kubectl apply -f tc-tiny-go-ws-deployment01.yaml 
```

create service:

```bash
vagrant ssh master 
kubectl expose deployment tc-tiny-go-ws-deployment --type=NodePort
```
