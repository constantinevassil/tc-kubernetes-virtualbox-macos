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

```bash
ubuntu@master:~$ mkdir /tmp/www
```

In the /tmp/www directory, create an index.html file:

```bash
ubuntu@master:~$ echo 'Hello from Kubernetes storage of www' > /tmp/www/index.html
ubuntu@master:~$ cat /tmp/www/index.html
exit
```

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


