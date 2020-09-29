# Kubernetes Installation

Building Kubernetes on top of Turing Pi brings another dimension to the edge computing and learning, from setting up the OS,
partitionning the OS, DHCP, NAT, cross compiling for the ARM32V7.


### Key Aspects

- Assemble a Turing Pi Cluster
- Deploy Kubernetes on Turing Pi

### Prepare control-plane and workers

```bash
sudo -i

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
sudo apt-key add - && echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list && sudo apt-get update -q

sudo apt-get install -qy kubelet kubectl kubeadm
```

To help during the initialization phase, get kubeadm to download the images onto docker

```bash
kubeadm config images pull
```

### Initialize the Kubernetes control-plane node

```bash
sudo kubeadm init --token-ttl=0 --pod-network-cidr=10.244.0.0/16

[init] using Kubernetes version: ...
[preflight] running pre-flight checks
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 18.04.0-ce. Latest validated version: 18.06
[preflight/images] Pulling images required for setting up a Kubernetes cluster
[preflight/images] This might take a minute or two, depending on the speed of your internet connection
[preflight/images] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[preflight] Activating the kubelet service
[certs] Generated ca certificate and key.
[certs] Generated apiserver-kubelet-client certificate and key.
[certs] Generated apiserver certificate and key.
[certs] apiserver serving cert is signed for DNS names [kubemaster-pi kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.2.1]
[certs] Generated front-proxy-ca certificate and key.
[certs] Generated front-proxy-client certificate and key.
[certs] Generated etcd/ca certificate and key.
[certs] Generated etcd/peer certificate and key.
[certs] etcd/peer serving cert is signed for DNS names [kubemaster-pi localhost] and IPs [192.168.2.1 127.0.0.1 ::1]
[certs] Generated etcd/server certificate and key.
[certs] etcd/server serving cert is signed for DNS names [kubemaster-pi localhost] and IPs [127.0.0.1 ::1]
[certs] Generated etcd/healthcheck-client certificate and key.
[certs] Generated apiserver-etcd-client certificate and key.
[certs] valid certificates and keys now exist in "/etc/kubernetes/pki"
[certs] Generated sa key and public key.
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[controlplane] wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests"              
[init] this might take a minute or longer if the control plane images have to be pulled
[apiclient] All control plane components are healthy after 88.010108 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.12" in namespace kube-system with the configuration for the kubeletsin the cluster
[mark-control-plane] Marking the node kubemaster-pi as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node kubemaster-pi as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "kubemaster-pi" asan annotation
[bootstraptoken] using token: vej1mx.6qf2xljr39rr1i14
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.2.1:6443 --token vej1mx.yyyyyy --discovery-token-ca-cert-hash sha256:xxxxxx
```

#### Initialize kubectl configuration

As normal user (not root)

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```

### Add worker nodes

```bash
kubeadm join 192.168.2.1:6443 --token vej1mx.yyyyyy --discovery-token-ca-cert-hash sha256:xxxxxx
```

### Setup Container Network Interface (CNI)

At that point, nodes are not in ready state yet and CoreDNS in pending state

```bash
kubectl get nodes
kubectl get all -n kube-system
```

The kubectl deployment file for flannel and adapted to kubedge is available here:
Note: Be sure to have picked the right branch (arm32v7 or arm64v8) when pulling kube-deployment

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

The nodes should turn ready and CoreDNS should start

```bash
kubectl get nodes
kubectl get all -n kube-system
```

### 5 node cluster

~~~
kubectl get nodes

NAME            STATUS    ROLES     AGE       VERSION
kube-node01     Ready     <none>    23d       v1.9.8
kube-node02     Ready     <none>    23d       v1.9.8
kube-node03     Ready     <none>    23d       v1.9.8
kube-node04     Ready     <none>    23d       v1.9.8
kubemaster-pi   Ready     master    23d       v1.9.8
~~~

~~~
kubectl get all --all-namespaces

NAMESPACE     NAME                                        READY     STATUS    RESTARTS   AGE
kube-system   pod/etcd-kubemaster-pi                      1/1       Running   0          23d
kube-system   pod/kube-apiserver-kubemaster-pi            1/1       Running   0          23d
kube-system   pod/kube-controller-manager-kubemaster-pi   1/1       Running   0          23d
kube-system   pod/kube-dns-7b6ff86f69-l7lf6               3/3       Running   0          23d
kube-system   pod/kube-flannel-ds-8xbx4                   1/1       Running   0          23d
kube-system   pod/kube-flannel-ds-9cz9f                   1/1       Running   0          23d
kube-system   pod/kube-flannel-ds-rgpcq                   1/1       Running   0          23d
kube-system   pod/kube-flannel-ds-xnjtz                   1/1       Running   0          23d
kube-system   pod/kube-flannel-ds-xxdf6                   1/1       Running   0          23d
kube-system   pod/kube-proxy-5m95q                        1/1       Running   0          23d
kube-system   pod/kube-proxy-7sh7m                        1/1       Running   0          23d
kube-system   pod/kube-proxy-f7t9r                        1/1       Running   0          23d
kube-system   pod/kube-proxy-pkqvd                        1/1       Running   0          23d
kube-system   pod/kube-proxy-shrdr                        1/1       Running   0          23d
kube-system   pod/kube-scheduler-kubemaster-pi            1/1       Running   0          23d
kube-system   pod/kubernetes-dashboard-7fcc5cb979-8vbmp   1/1       Running   0          23d

NAMESPACE     NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kube-system   service/kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP   23d
kube-system   service/kubernetes-dashboard   NodePort    10.102.144.189   <none>        443:30383/TCP   23d

NAMESPACE     NAME                                   DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
kube-system   daemonset.extensions/kube-flannel-ds   5         5         5         5            5           beta.kubernetes.io/arch=arm   23d
kube-system   daemonset.extensions/kube-proxy        5         5         5         5            5           <none>                        23d

NAMESPACE     NAME                                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.extensions/kube-dns               1         1         1            1           23d
kube-system   deployment.extensions/kubernetes-dashboard   1         1         1            1           23d

NAMESPACE     NAME                                                    DESIRED   CURRENT   READY     AGE
kube-system   replicaset.extensions/kube-dns-7b6ff86f69               1         1         1         23d
kube-system   replicaset.extensions/kubernetes-dashboard-7fcc5cb979   1         1         1         23d

NAMESPACE     NAME                             DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
kube-system   daemonset.apps/kube-flannel-ds   5         5         5         5            5           beta.kubernetes.io/arch=arm   23d
kube-system   daemonset.apps/kube-proxy        5         5         5         5            5           <none>                        23d

NAMESPACE     NAME                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/kube-dns               1         1         1            1           23d
kube-system   deployment.apps/kubernetes-dashboard   1         1         1            1           23d

NAMESPACE     NAME                                              DESIRED   CURRENT   READY     AGE
kube-system   replicaset.apps/kube-dns-7b6ff86f69               1         1         1         23d
kube-system   replicaset.apps/kubernetes-dashboard-7fcc5cb979   1         1         1         23d
~~~

### Cleanup

Teardown cluster

```bash
sudo kubeadm reset
sudo docker rm $(sudo docker ps -qa)
sudo docker image rm $(sudo docker image list -qa)
sudo apt-get purge kubeadm kubectl kubelet
sudo apt-get autoremove
sudo rm -rf ~/.kube
```

### Reference Links

- [Kubeadm on Hypriot](https://blog.hypriot.com/post/setup-kubernetes-raspberry-pi-cluster/)