# PersistentVolume-hostPath
create a hostPath PersistentVolume. Kubernetes supports hostPath for development and testing on a single-node cluster. 
A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage.

# Configure a Pod to Use a PersistentVolume for Storage

1. Login to worker node:
multiple machines:
vagrant ssh worker 
or 
single machine:
vagrant ssh master

mkdir /tmp/globalipgowsrocksdb01
