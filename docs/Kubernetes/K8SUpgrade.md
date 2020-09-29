# Upgrade tutorial


## Control plane node upgrade using kubeadm

```bash
# apt-mark unhold kubeadm && \
> apt-get update && apt-get install -y kubeadm && \
> apt-mark hold kubeadm
kubeadm was already not hold.
Hit:2 http://raspbian.raspberrypi.org/raspbian stretch InRelease
Hit:3 https://download.docker.com/linux/raspbian stretch InRelease
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Hit:5 http://archive.raspberrypi.org/debian stretch InRelease
Hit:4 https://packagecloud.io/Hypriot/rpi/debian stretch InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 38 not upgraded.
Need to get 8,095 kB of archives.
After this operation, 3,100 kB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main armhf kubeadm armhf 1.12.0-00 [8,095 kB]
Fetched 8,095 kB in 4s (1,962 kB/s)
(Reading database ... 30574 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.12.0-00_armhf.deb ...
Unpacking kubeadm (1.12.0-00) over (1.11.1-00) ...
Setting up kubeadm (1.12.0-00) ...
kubeadm set on hold.
```

```bash
sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.11.0
[upgrade/versions] kubeadm version: v1.12.0
[upgrade/versions] Latest stable version: v1.12.0
[upgrade/versions] Latest version in the v1.11 series: v1.11.3

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
Kubelet     3 x v1.11.1   v1.11.3

Upgrade to the latest version in the v1.11 series:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.11.0   v1.11.3
Controller Manager   v1.11.0   v1.11.3
Scheduler            v1.11.0   v1.11.3
Kube Proxy           v1.11.0   v1.11.3
CoreDNS              1.1.3     1.2.2
Etcd                 3.2.18    3.2.18

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.11.3

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
Kubelet     3 x v1.11.1   v1.12.0

Upgrade to the latest stable version:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.11.0   v1.12.0
Controller Manager   v1.11.0   v1.12.0
Scheduler            v1.11.0   v1.12.0
Kube Proxy           v1.11.0   v1.12.0
CoreDNS              1.1.3     1.2.2
Etcd                 3.2.18    3.2.24

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.12.0

_____________________________________________________________________
```

```bash
$ sudo kubeadm upgrade apply v1.12.0
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade/apply] Respecting the --cri-socket flag that is set with higher priority than the config file.
[upgrade/version] You have chosen to change the cluster version to "v1.12.0"
[upgrade/versions] Cluster version: v1.11.0
[upgrade/versions] kubeadm version: v1.12.0
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler etcd]
[upgrade/prepull] Prepulling image for component kube-apiserver.
[upgrade/prepull] Prepulling image for component kube-controller-manager.
[upgrade/prepull] Prepulling image for component kube-scheduler.
[upgrade/prepull] Prepulling image for component etcd.
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-etcd
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-etcd
[upgrade/prepull] Prepulled image for component etcd.
[upgrade/prepull] Prepulled image for component kube-apiserver.
[upgrade/prepull] Prepulled image for component kube-controller-manager.
[upgrade/prepull] Prepulled image for component kube-scheduler.
[upgrade/prepull] Successfully prepulled the images for all the control plane components
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.12.0"...
Static pod: kube-apiserver-master-pi hash: c30b2fa49c49e091538b2ce8e4dae186
Static pod: kube-controller-manager-master-pi hash: 22f67939f8b1abea8ba99666b78b5c93
Static pod: kube-scheduler-master-pi hash: 0e545194d6b033abd681f02dfd11f4c8
Static pod: etcd-master-pi hash: 00575b778fb80d4e48241f80ceb2ac0f
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests040184315/etcd.yaml"
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2018-09-30-00-46-56/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s
Static pod: etcd-master-pi hash: 00575b778fb80d4e48241f80ceb2ac0f
Static pod: etcd-master-pi hash: 00575b778fb80d4e48241f80ceb2ac0f
Static pod: etcd-master-pi hash: 00575b778fb80d4e48241f80ceb2ac0f
Static pod: etcd-master-pi hash: 77c6076a4d6ee044b744b041125cf918
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[util/etcd] Waiting 0s for initial delay
[util/etcd] Attempting to see if all cluster endpoints are available 1/10
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests040184315"
[controlplane] wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests040184315/kube-apiserver.yaml"
[controlplane] wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests040184315/kube-controller-manager.yaml"
[controlplane] wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests040184315/kube-scheduler.yaml"
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2018-09-30-00-46-56/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s
Static pod: kube-apiserver-master-pi hash: c30b2fa49c49e091538b2ce8e4dae186
Static pod: kube-apiserver-master-pi hash: a92106d6db4c8b5835a47f5f56c33fdb
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2018-09-30-00-46-56/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s
Static pod: kube-controller-manager-master-pi hash: 22f67939f8b1abea8ba99666b78b5c93
Static pod: kube-controller-manager-master-pi hash: 980b4156606df8caafd0ad8abacc1485
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2018-09-30-00-46-56/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s
Static pod: kube-scheduler-master-pi hash: 0e545194d6b033abd681f02dfd11f4c8
Static pod: kube-scheduler-master-pi hash: 1b5ec5be325bf29f60be62789416a99e
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.12" in namespace kube-system with the configuration for the kubelets in the cluster
[kubelet] Downloading configuration for the kubelet from the "kubelet-config-1.12" ConfigMap in the kube-system namespace
[kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "master-pi" as an annotation
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.12.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

Kubernetes servers are running v1.12
```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.1", GitCommit:"b1b29978270dc22fecc592ac55d903350454310a", GitTreeState:"clean", BuildDate:"2018-07-17T18:53:20Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/arm"}
Server Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.0", GitCommit:"0ed33881dc4355495f623c6f22e7dd0b7632b7c0", GitTreeState:"clean", BuildDate:"2018-09-27T16:55:41Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/arm"}
```

my kubelets are still running kubernetes v1.11.1
```bash
$ kubectl get nodes
NAME        STATUS    ROLES     AGE       VERSION
home-pi     Ready     <none>    86d       v1.11.1
master-pi   Ready     master    86d       v1.11.1
nas-pi      Ready     <none>    85d       v1.11.1
```
## Upgrade kubectl

Install newer version of kubectl using apt-get

```bash
$ sudo apt-get install kubectl
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubectl
1 upgraded, 0 newly installed, 0 to remove and 37 not upgraded.
Need to get 8,639 kB of archives.
After this operation, 1,783 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main armhf kubectl armhf 1.12.0-00 [8,639 kB]
Fetched 8,639 kB in 6s (1,270 kB/s)
(Reading database ... 30574 files and directories currently installed.)
Preparing to unpack .../kubectl_1.12.0-00_armhf.deb ...
Unpacking kubectl (1.12.0-00) over (1.11.1-00) ...
Setting up kubectl (1.12.0-00) ...
```
Check that kubectl is now 1.12
```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.0", GitCommit:"0ed33881dc4355495f623c6f22e7dd0b7632b7c0", GitTreeState:"clean", BuildDate:"2018-09-27T17:05:32Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/arm"}
Server Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.0", GitCommit:"0ed33881dc4355495f623c6f22e7dd0b7632b7c0", GitTreeState:"clean", BuildDate:"2018-09-27T16:55:41Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/arm"}
```

## Upgrade kubelet on control plane node

```bash
$ sudo apt-get upgrade -y kubelet
Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
...
```

```bash
$ sudo systemctl restart kubelet
```

```bash
$ kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
home-pi     Ready    <none>   86d   v1.11.1
master-pi   Ready    master   86d   v1.12.0
nas-pi      Ready    <none>   85d   v1.11.1
```

Looks that as useual something is wrong with flannel CNI
```bash
$ kubectl get all --all-namespaces
NAMESPACE     NAME                                             READY   STATUS    RESTARTS   AGE
default       pod/helm-rpi-kubeplay-arm32v7-6cb66496c6-6gvf6   1/1     Running   0          60d
kube-system   pod/coredns-576cbf47c7-nzhfj                     1/1     Running   0          37m
kube-system   pod/coredns-576cbf47c7-wkmhr                     1/1     Running   0          37m
kube-system   pod/etcd-master-pi                               1/1     Running   0          3m43s
kube-system   pod/kube-apiserver-master-pi                     1/1     Running   2          3m43s
kube-system   pod/kube-controller-manager-master-pi            1/1     Running   1          3m43s
kube-system   pod/kube-flannel-ds-4495g                        1/1     Running   4          75d
kube-system   pod/kube-flannel-ds-52ssk                        0/1     Error     10         75d
kube-system   pod/kube-flannel-ds-gj65n                        1/1     Running   4          75d

Investigation shows:

```bash
$ kubectl logs pod/kube-flannel-ds-52ssk -n kube-system
I0930 01:34:13.768925       1 main.go:475] Determining IP address of default interface
I0930 01:34:13.778622       1 main.go:488] Using interface with name wlan0 and address 192.168.1.95
I0930 01:34:13.778716       1 main.go:505] Defaulting external address to interface address (192.168.1.95)
I0930 01:34:14.168318       1 kube.go:131] Waiting 10m0s for node controller to sync
I0930 01:34:14.168890       1 kube.go:294] Starting kube subnet manager
I0930 01:34:15.169929       1 kube.go:138] Node controller sync successful
I0930 01:34:15.170035       1 main.go:235] Created subnet manager: Kubernetes Subnet Manager - master-pi
I0930 01:34:15.170064       1 main.go:238] Installing signal handlers
I0930 01:34:15.170415       1 main.go:353] Found network config - Backend type: vxlan
I0930 01:34:15.170797       1 vxlan.go:120] VXLAN config: VNI=1 Port=0 GBP=false DirectRouting=false
E0930 01:34:15.256843       1 main.go:280] Error registering network: failed to configure interface flannel.1: failed to ensure address of interface flannel.1: link has incompatible addresses. Remove additional addresses and try again. &netlink.Vxlan{LinkAttrs:netlink.LinkAttrs{Index:5, MTU:1450, TxQLen:0, Name:"flannel.1", HardwareAddr:net.HardwareAddr{0x26, 0xd5, 0xd0, 0xdd, 0x56, 0x1a},
...
```

Let's apply usual receipe
``bash
sudo ip link delete flannel.1
```

Brute force fix seems to have done the fix....This is lucky we don't have production traffic on that node.
```bash
$ kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-576cbf47c7-nzhfj                1/1     Running   0          51m
coredns-576cbf47c7-wkmhr                1/1     Running   0          51m
etcd-master-pi                          1/1     Running   0          17m
kube-apiserver-master-pi                1/1     Running   2          17m
kube-controller-manager-master-pi       1/1     Running   1          17m
kube-flannel-ds-4495g                   1/1     Running   4          75d
kube-flannel-ds-52ssk                   1/1     Running   13         75d
kube-flannel-ds-gj65n                   1/1     Running   4          75d
kube-proxy-c2264                        1/1     Running   0          51m
kube-proxy-snjsg                        1/1     Running   0          49m
kube-proxy-zgqjb                        1/1     Running   1          50m
kube-scheduler-master-pi                1/1     Running   1          17m
kubernetes-dashboard-7d59788d44-rchkk   1/1     Running   25         83d
tiller-deploy-b59fcc885-dbvlv           1/1     Running   0          60d
```

## Update first worker node

```bash
sudo apt-get clean
sudo apt-get update
sudo apt-get install kubeadm
sudo apt-get install kubelet
sudo kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
sudo systemctl restart kubelet
```

Same flannel issue
```bash
$ kubectl get pods -n kube-system
NAME                                    READY   STATUS             RESTARTS   AGE
coredns-576cbf47c7-nzhfj                1/1     Running            0          87m
coredns-576cbf47c7-wkmhr                1/1     Running            1          87m
etcd-master-pi                          1/1     Running            0          53m
kube-apiserver-master-pi                1/1     Running            2          53m
kube-controller-manager-master-pi       1/1     Running            1          53m
kube-flannel-ds-4495g                   0/1     CrashLoopBackOff   10         75d
...
```

same hack to fix the issue
```bash
sudo ip link delete flannel.1
```

same hack seems to be effective
```bash
$ kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-576cbf47c7-nzhfj                1/1     Running   0          111m
coredns-576cbf47c7-wkmhr                1/1     Running   1          111m
etcd-master-pi                          1/1     Running   0          77m
kube-apiserver-master-pi                1/1     Running   2          77m
kube-controller-manager-master-pi       1/1     Running   1          77m
kube-flannel-ds-4495g                   1/1     Running   11         75d
...
```

### Other worker node

```
sudo apt-get clean
sudo apt-get update
sudo apt-get install kubeadm
sudo apt-get install kubelet
sudo kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
sudo systemctl restart kubelet
sudo ip link delete flannel.1
```

## Check consistency of the cluster

```bash
$ kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-576cbf47c7-nzhfj                1/1     Running   1          143m
coredns-576cbf47c7-wkmhr                1/1     Running   1          143m
etcd-master-pi                          1/1     Running   0          109m
kube-apiserver-master-pi                1/1     Running   2          109m
kube-controller-manager-master-pi       1/1     Running   1          109m
kube-flannel-ds-4495g                   1/1     Running   11         76d
kube-flannel-ds-52ssk                   1/1     Running   13         76d
kube-flannel-ds-gj65n                   1/1     Running   13         76d
kube-proxy-c2264                        1/1     Running   1          143m
kube-proxy-snjsg                        1/1     Running   1          141m
kube-proxy-zgqjb                        1/1     Running   1          142m
kube-scheduler-master-pi                1/1     Running   1          109m
kubernetes-dashboard-7d59788d44-rchkk   1/1     Running   25         83d
tiller-deploy-b59fcc885-dbvlv           1/1     Running   0          60d
```

```bash
$ kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
home-pi     Ready    <none>   86d   v1.12.0
master-pi   Ready    master   86d   v1.12.0
nas-pi      Ready    <none>   86d   v1.12.0
```

## Conclusion

- At a glance, the cluster seems to be healthy
- I still need to find sometool like sonobuoy to validate that the cluster is healthy

## Reference Links

- [Official Upgrade documentation](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-12/)