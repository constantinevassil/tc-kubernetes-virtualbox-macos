# PersistentVolume-hostPath
Create a hostPath PersistentVolume. Kubernetes supports hostPath for development and testing on a single-node cluster. 
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
ubuntu@master:~$ mkdir /tmp/www-01/
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
4. Create deployment for ver. 1.0

go back to PersistentVolume-hostPath

```bash
cd ..
cd PersistentVolume-hostPath
```

create deployment:

```bash
cd ..
kubectl apply -f tc-tiny-go-ws-deployment01.yaml 
kubectl get pods
NAME                                       READY     STATUS    RESTARTS   AGE
tc-tiny-go-ws-deployment-715299535-mgq2l   1/1       Running   0          12s
tc-tiny-go-ws-deployment-715299535-nm04x   1/1       Running   0          12s
```

create service:

```bash
kubectl expose deployment tc-tiny-go-ws-deployment --type=NodePort
kubectl get services
NAME                       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes                 10.96.0.1       <none>        443/TCP          1d
tc-tiny-go-ws-deployment   10.110.226.49   <nodes>       8080:31449/TCP   39s
```

test service:

```bash
curl 192.168.33.10:31449
Hello from Kubernetes storage of www-01
```

replace deployment with v 2.0:

```bash
cd ..
kubectl replace -f tc-tiny-go-ws-deployment02.yaml 
kubectl get pods
NAME                                       READY     STATUS    RESTARTS   AGE
tc-tiny-go-ws-deployment-715299535-ccfx3   1/1       Running   0          16s
tc-tiny-go-ws-deployment-715299535-qhpcm   1/1       Running   0          16s
```

test service:

```bash
curl 192.168.33.10:31449
Hello from Kubernetes storage of www-02
```

undo deployment:

```bash
kubectl get deployments
NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
tc-tiny-go-ws-deployment   2         2         2            2           11m
kubectl rollout undo deployment tc-tiny-go-ws-deployment
```

test service:

```bash
curl 192.168.33.10:31449
Hello from Kubernetes storage of www-01
```

