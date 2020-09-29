# Persistent Volume

In order to install Prometheus, NATS, Cassandra using Kubernetes, we need to first create Persistency Volumes


## Procedures

The procedure assumes that you have 32G SD Card that you can partition into one 16G disk and seven 7G disks.

### Partition the SD cards

Use the Rescue dongle to create the partition. The ideal solution would be to update the cloud-init
of the HypriotOS to do that automatically.

### Mount the disks

Mounting the partitions, involves creating new directories on each node and adding ".mount" files under systemd
This kind of done automatically under by the mount-disks playbook

```
cd $HOME/mgt/
ansible picluster -i inventory/ -m shell -a "df -kh"
ansible-playbook -i inventory/ playbooks/mount_disks.yaml
ansible picluster -i inventory/ -m shell -a "df -kh"
```

### Create the Persistency Volumes in Kubernetes

The kubectl deployment file for flannel and adapted to kubedge is available here:
Note: Be sure to have picked the right branch (arm32v7 or arm64v8) when pulling kube-deployment

Because we did not convert the go code to create the volumes automatically from the /mnt/disks/...,
we have to kind of create the PV by hand.
```bash
cd $HOME
cp -r proj/kubedge/kubedge_utils/kube-deployment/ .

cd $HOME/kube-deployment/local-storage
kubectl apply -f local-storageclass.yaml
kubectl apply -f admin_account.yaml
kubectl apply -f kubemaster-pi-volumes.yaml
kubectl apply -f kube-node01-volumes.yaml
kubectl apply -f kube-node02-volumes.yaml
kubectl apply -f kube-node03-volumes.yaml
kubectl apply -f kube-node04-volumes.yaml
```

## Results

- ![](/images/kubernetes/cluster2_volumes.png)

### Reference Links

- TBD