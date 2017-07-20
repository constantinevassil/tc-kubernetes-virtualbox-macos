# PersistentVolume-hostPath
create a hostPath PersistentVolume. Kubernetes supports hostPath for development and testing on a single-node cluster. 
A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage.

# Configure a Pod to Use a PersistentVolume for Storage

1. Login to worker node:
multiple machines:
```bash
vagrant ssh worker 
```
or 
single machine:
```bash
vagrant ssh master 
```

```bash
ubuntu@master:~$ mkdir /tmp/www
```

In the /tmp/www directory, create an index.html file:

```bash
ubuntu@master:~$ echo 'Hello from Kubernetes storage of www' > /tmp/www/index.html
ubuntu@master:~$ cat /tmp/www/index.html

```
