# Deploy Flannel in Turing Pi

In order to get the nodes and pods interface with each other accross the cluster.
This post describes how I deployed Flannel acounting with the fact that some of the
nodes have multiple interfaces (wlan0 and eth0).


## Key Aspects

- Flannel seems to deploy ok. Looks like in trouble when multiple interfaces available
- Calico in not compiled by default for Rapsberry PI

## Flannel

### Setup through kubectl

The kubectl deployment file for flannel and adapted to kubedge is available here:
Note: Be sure to have picked the right branch (arm32v7 or arm64v8) when pulling kube-deployment

```bash
cd $HOME
cp -r proj/kubedge/kubedge_utils/kube-deployment/ .

cd kube-deployment/flannel/
kubectl apply -f flannel.yaml


### Flannel Issue 1: flannel.1. Link has incompatible address

on master-pi, both the WLAN and LAN interfaces were activated. After unplugging the CAT5,
behavior was similar. Moreover this had some impact on the kube-apiserver (see the number of restarts).

```bash
$ kubectl get all --all-namespaces

NAMESPACE     NAME                                             READY     STATUS             RESTARTS   AGE
default       pod/helm-rpi-kubeplay-arm32v7-6cb66496c6-9w97m   1/1       Running            0          4h
kube-system   pod/coredns-78fcdf6894-cw5p8                     1/1       Running            15         10d
kube-system   pod/coredns-78fcdf6894-czjcj                     1/1       Running            15         10d
kube-system   pod/etcd-master-pi                               1/1       Running            11         10d
kube-system   pod/kube-apiserver-master-pi                     1/1       Running            599        10d
kube-system   pod/kube-controller-manager-master-pi            1/1       Running            38         10d
kube-system   pod/kube-flannel-ds-bhllh                        1/1       Running            13         9d
kube-system   pod/kube-flannel-ds-q7cp2                        0/1       CrashLoopBackOff   401        9d
kube-system   pod/kube-flannel-ds-wqxsz                        1/1       Running            16         9d
kube-system   pod/kube-proxy-4chwh                             1/1       Running            9          9d
kube-system   pod/kube-proxy-6r5mn                             1/1       Running            5          9d
kube-system   pod/kube-proxy-vvj6j                             1/1       Running            11         10d
kube-system   pod/kube-scheduler-master-pi                     1/1       Running            13         10d
kube-system   pod/kubernetes-dashboard-7d59788d44-rchkk        1/1       Running            20         7d
kube-system   pod/tiller-deploy-b59fcc885-66l7s                1/1       Running            0          6h
```

```bash
$ kubectl logs pod/kube-flannel-ds-q7cp2 -n kube-system

I0716 00:42:46.596796       1 main.go:474] Determining IP address of default interface
I0716 00:42:46.598043       1 main.go:487] Using interface with name wlan0 and address 192.168.1.95
I0716 00:42:46.598138       1 main.go:504] Defaulting external address to interface address (192.168.1.95)
I0716 00:42:46.775936       1 kube.go:283] Starting kube subnet manager
I0716 00:42:46.775907       1 kube.go:130] Waiting 10m0s for node controller to sync
I0716 00:42:47.776280       1 kube.go:137] Node controller sync successful
I0716 00:42:47.776400       1 main.go:234] Created subnet manager: Kubernetes Subnet Manager - master-pi
I0716 00:42:47.776431       1 main.go:237] Installing signal handlers
I0716 00:42:47.776697       1 main.go:352] Found network config - Backend type: vxlan
I0716 00:42:47.776900       1 vxlan.go:119] VXLAN config: VNI=1 Port=0 GBP=false DirectRouting=false
E0716 00:42:47.778884       1 main.go:279] Error registering network: failed to configure interface flannel.1: link has incompatible addresses. Remove additional addresses and try again....
I0716 00:42:47.778991       1 main.go:332] Stopping shutdownHandler...
```

Deleting the pod, did not help. After recreation same issue reappeared.
```bash
$ kubectl delete pod/kube-flannel-ds-q7cp2 -n kube-system
```

```bash
$ kubectl get all --all-namespaces

NAMESPACE     NAME                                             READY     STATUS    RESTARTS   AGE
kube-system   pod/kube-flannel-ds-z7w4f                        0/1       Error     1          17s
```

Deleting the interface flannel.1 interface actually worked:
```bash
$ sudo ip link delete flannel.1
```

```bash
$ kubectl get all --all-namespaces

NAMESPACE     NAME                                             READY     STATUS    RESTARTS   AGE
kube-system   pod/kube-flannel-ds-z7w4f                        1/1       Running   5          3m
```

```bash
$ kubectl logs pod/kube-flannel-ds-z7w4f -n kube-system

I0716 00:52:14.555290       1 main.go:474] Determining IP address of default interface
I0716 00:52:14.564490       1 main.go:487] Using interface with name wlan0 and address 192.168.1.95
I0716 00:52:14.564578       1 main.go:504] Defaulting external address to interface address (192.168.1.95)
I0716 00:52:14.802491       1 kube.go:130] Waiting 10m0s for node controller to sync
I0716 00:52:14.802544       1 kube.go:283] Starting kube subnet manager
I0716 00:52:15.803114       1 kube.go:137] Node controller sync successful
I0716 00:52:15.803308       1 main.go:234] Created subnet manager: Kubernetes Subnet Manager - master-pi
I0716 00:52:15.803909       1 main.go:237] Installing signal handlers
I0716 00:52:15.804662       1 main.go:352] Found network config - Backend type: vxlan
I0716 00:52:15.804985       1 vxlan.go:119] VXLAN config: VNI=1 Port=0 GBP=false DirectRouting=false
I0716 00:52:15.875242       1 main.go:299] Wrote subnet file to /run/flannel/subnet.env
I0716 00:52:15.875367       1 main.go:303] Running backend.
I0716 00:52:15.875489       1 main.go:321] Waiting for all goroutines to exit
I0716 00:52:15.875559       1 vxlan_network.go:56] watching for new subnet leases
```

### Flannel issue 2: Multiple interfaces

Some of the PI have two interfaces running: wlan0 and eth0. The internal cluster network is using eth0.
We need to force Flannel to use it.

### Setup through kubectl

Realize I was using v0.9.1 instead of v0.10.0. Let's update the file
```bash
$ mkdir -p $HOME/kube-deployments/flannel
$ cd $HOME/kube-deployments/flannel
$ curl -sSL https://rawgit.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml | sed "s/amd64/arm/g" > flannel.yaml
```

Let's add --iface=eth0 to the flanneld to in the flannel.yaml
```bash
```

Let's update flannel from 0.9.1 to 0.10.0 at the same time we specify which interface to use.
```bash
$ kubectl apply -f flannel.yaml

clusterrole.rbac.authorization.k8s.io/flannel configured
clusterrolebinding.rbac.authorization.k8s.io/flannel configured
serviceaccount/flannel unchanged
configmap/kube-flannel-cfg configured
daemonset.extensions/kube-flannel-ds configured
```

It seems it solved the flannel issue. The bug in kube 1.11.0 still there (restart of kube-apiserver)
Will update to 1.11.1 when it is published
```bash
$ kubectl get all --all-namespaces

NAMESPACE     NAME                                        READY     STATUS             RESTARTS   AGE
kube-system   pod/coredns-78fcdf6894-bn6wl                1/1       Running            0          6d
kube-system   pod/coredns-78fcdf6894-k52xb                1/1       Running            0          6d
kube-system   pod/etcd-kubemaster-pi                      1/1       Running            3          6d
kube-system   pod/kube-apiserver-kubemaster-pi            1/1       Running            3          6d
kube-system   pod/kube-controller-manager-kubemaster-pi   0/1       CrashLoopBackOff   1740       6d
kube-system   pod/kube-flannel-ds-62fz9                   1/1       Running            984        6d
kube-system   pod/kube-flannel-ds-gwzdt                   1/1       Running            0          6d
kube-system   pod/kube-flannel-ds-h7ln5                   1/1       Running            0          6d
kube-system   pod/kube-flannel-ds-qs9lf                   1/1       Running            0          6d
kube-system   pod/kube-flannel-ds-vwsjk                   1/1       Running            0          6d
kube-system   pod/kube-proxy-45z5s                        1/1       Running            0          6d
kube-system   pod/kube-proxy-4trsd                        1/1       Running            0          6d
kube-system   pod/kube-proxy-ksj7c                        1/1       Running            4          6d
kube-system   pod/kube-proxy-t7gmc                        1/1       Running            0          6d
kube-system   pod/kube-proxy-tfmqb                        1/1       Running            0          6d
kube-system   pod/kube-scheduler-kubemaster-pi            1/1       Running            4          6d

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP         6d
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   6d

NAMESPACE     NAME                             DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
kube-system   daemonset.apps/kube-flannel-ds   5         5         5         5            5           beta.kubernetes.io/arch=arm   6d
kube-system   daemonset.apps/kube-proxy        5         5         5         5            5           beta.kubernetes.io/arch=arm   6d

NAMESPACE     NAME                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2         2         2            2           6d

NAMESPACE     NAME                                 DESIRED   CURRENT   READY     AGE
kube-system   replicaset.apps/coredns-78fcdf6894   2         2         2         6d
```


```bash
$ kubectl logs pod/kube-flannel-ds-62fz9 -n kube-system

I0714 14:34:26.719081       1 main.go:474] Determining IP address of default interface
I0714 14:34:26.728184       1 main.go:487] Using interface with name wlan0 and address 192.168.1.94
I0714 14:34:26.728273       1 main.go:504] Defaulting external address to interface address (192.168.1.94)
I0714 14:34:26.942686       1 kube.go:283] Starting kube subnet manager
I0714 14:34:26.943203       1 kube.go:130] Waiting 10m0s for node controller to sync
I0714 14:34:27.943672       1 kube.go:137] Node controller sync successful
I0714 14:34:27.943771       1 main.go:234] Created subnet manager: Kubernetes Subnet Manager - kubemaster-pi
I0714 14:34:27.943819       1 main.go:237] Installing signal handlers
I0714 14:34:27.944064       1 main.go:352] Found network config - Backend type: vxlan
I0714 14:34:27.944222       1 vxlan.go:119] VXLAN config: VNI=1 Port=0 GBP=false DirectRouting=false
I0714 14:34:28.004675       1 main.go:299] Wrote subnet file to /run/flannel/subnet.env
I0714 14:34:28.004748       1 main.go:303] Running backend.
I0714 14:34:28.004880       1 main.go:321] Waiting for all goroutines to exit
I0714 14:34:28.004931       1 vxlan_network.go:56] watching for new subnet leases
I0714 14:34:28.049933       1 iptables.go:114] Some iptables rules are missing; deleting and recreating rules
I0714 14:34:28.050202       1 iptables.go:136] Deleting iptables rule: -s 10.244.0.0/16 -j ACCEPT
I0714 14:34:28.053918       1 iptables.go:114] Some iptables rules are missing; deleting and recreating rules
I0714 14:34:28.054003       1 iptables.go:136] Deleting iptables rule: -s 10.244.0.0/16 -d 10.244.0.0/16 -j RETURN
I0714 14:34:28.057332       1 iptables.go:136] Deleting iptables rule: -d 10.244.0.0/16 -j ACCEPT
I0714 14:34:28.061665       1 iptables.go:136] Deleting iptables rule: -s 10.244.0.0/16 ! -d 224.0.0.0/4 -j MASQUERADE
I0714 14:34:28.066452       1 iptables.go:124] Adding iptables rule: -s 10.244.0.0/16 -j ACCEPT
I0714 14:34:28.069910       1 iptables.go:136] Deleting iptables rule: ! -s 10.244.0.0/16 -d 10.244.0.0/24 -j RETURN
I0714 14:34:28.075067       1 iptables.go:136] Deleting iptables rule: ! -s 10.244.0.0/16 -d 10.244.0.0/16 -j MASQUERADE
I0714 14:34:28.078310       1 iptables.go:124] Adding iptables rule: -d 10.244.0.0/16 -j ACCEPT
I0714 14:34:28.082389       1 iptables.go:124] Adding iptables rule: -s 10.244.0.0/16 -d 10.244.0.0/16 -j RETURN
I0714 14:34:28.098375       1 iptables.go:124] Adding iptables rule: -s 10.244.0.0/16 ! -d 224.0.0.0/4 -j MASQUERADE
I0714 14:34:28.111379       1 iptables.go:124] Adding iptables rule: ! -s 10.244.0.0/16 -d 10.244.0.0/24 -j RETURN
I0714 14:34:28.122424       1 iptables.go:124] Adding iptables rule: ! -s 10.244.0.0/16 -d 10.244.0.0/16 -j MASQUERADE
```

## Calico

### Compile Calico for Raspberry PI

- WIP

### Deploy on Raspberry PI

- WIP


## Results

- WIP

### Reference Links

- [Flannel Issue](https://github.com/coreos/flannel/issues/883)
- [Flannel Issue2](https://stackoverflow.com/questions/47845739/configuring-flannel-to-use-a-non-default-interface-in-kubernetes)
- [Flannel Issue3](https://github.com/coreos/flannel/issues/620)