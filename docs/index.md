# TURING MACHINES

## Personal, Mobile Edge Computers and Lab

Turing Pi is a private mobile cloud in a mini ITX form factor. It’s a scale model of bare metal clusters like you see in data centers. Turing Pi cluster board can scale up to 7 compute nodes.

### Specs:

* 1 Gbps Ethernet
* 7 x 40 pin GPIO
* Multiple USB
* HDMI, Audio jack, ATX
* Up to 28 cores
* Up to 7 GB RAM

### Features:

* Flash mode - flash RPi compute modules through the board
* Boot mode - boot OS through eMMC or SD card or netboot
* Power management for each node (on/off/reboot)
* Real-time clock (RTC)
* I2C cluster management bus

### Turing Pi cluster board could be an excellent fit for the following use cases:

* Edge apps hosting
* Homelab
* Develop and learn cloud-native technologies like Kubernetes, Docker, Serverless, Microservices on bare metal
* Cloud-native apps testing environment
* Learn concepts of distributed Machine Learning apps
* Hosting and managing IoT apps

Prototype and learn cluster applications, parallel computing, and distributed computing concepts on bare metal
The Turing Pi top/master node can act as a NAT/Router for the rest of the nodes. The main advantage is that if
you move the cluster from one location to another, all the nodes IPs stay identical. The next thing is when you
set up the operating system, you immediately realize how unpractical it is to plug in each module into a master
node and connect display via HDMI, as well as connect USB mouse and keyboard to perform the first initialization.
However, the Hypriot OS and some other OS preconfigure nodes with SSH enabled Docker container and makes it super
easy to setup each node using SSH.
