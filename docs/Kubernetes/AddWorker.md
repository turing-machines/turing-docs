# Add a worker

During some of the manipulation of the partition table of
my SD card, I ended up screwing up both my SD card and my backup Win32DiskImage backup.
Moreover if your SD card is 32G, it takes around 30 minute to restore from backup.
Hence the idea to come up with a way to build more resiliency in the cluster.
Recreating a node from scratch should not take more than 10 mn. The propose procedure
is still rather long because I did not push enough yet what the HypriotOS team, aka
build a default SD image where cloud-init does 100% of the initialization work.


## Base OS

Flash HypriotOS to SD and reboot Pi.

### OS

~~~
Flash the SD Card with HypriotOS
Connect Pi through to the cluster switch
~~~

### Freeze the new node IP

Access the Master PI, run `arp -a` and find the new IP.
Freeze the new IP in the dhcpcd.conf

```bash
ssh <masterip>
arp -a
vi /etc/dhcp/dhcpd.conf
```

### Ssh to the new node

Access the node from the Master PI:

```bash
ssh 192.168.2.xxx
docker ps
```

### Basic hostname

Freeze your configuration

```bash
sudo apt-get remove --purge cloud-init
sudo apt-get autoremove
```

Set the host name

```bash
sudo vi /etc/hosts
sudo hostnamectl set-hostname <xxx>
```

## Install kubeadm

### On Master PI

Firt access the kubemaster node and regenerate a token (if you did not use ttl=0 when using kubeadm init on the master):

```bash
kubeadm token create
```

### On New Worker PI

```bash
sudo -i

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubeadm kubectl kubelet
```

To help during the initialization phase, get kubeadm to download the images onto docker

```bash
kubeadm config images pull
```

Get the node to join the cluster

```bash
kubeadm join 192.168.2.1:6443 --token yyyyyy.xxxx --discovery-token-ca-cert-hash sha256:zzzz
```

## Conclusion

- Will have to come back later and use cloud-init, create a clean & small SD image for Win32DiskImage
- Will have to create more advanced partition on the SD card.