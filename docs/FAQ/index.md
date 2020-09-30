# FAQ

## Here you find a list with the most frequent asked questions

### What can I do with the board?

* Home server (homelab) and application hosting
* Develop and learn cloud-native technologies (Kubernetes, Docker Swarm, Serverless, Microservices) on bare metal
* Cloud-native apps testing environment
* Learn concepts of distributed Machine Learning apps
* Hosting and managing IoT apps
* Prototype and learn cluster applications, parallel computing, and distributed computing concepts on bare metal
* Host K8S, K3S, Minecraft, Plex, Owncloud, Nextcloud, Seafile, Minio, Tensorflow, …

### Which Raspberry Pi models are compatible?

Turing Pi supports the following models with and without eMMC

* Raspberry Pi Compute Module 1
* Raspberry Pi Compute Module 3
* Raspberry Pi Compute Module 3+

### Does the board support the new Rasberry pi 4?

There is no Compute Model for the Raspberry pi 4th generation yet

### How to the compute modules communicate with each other?

The nodes interconnected with the onboard 1 Gbps switch. However, each node is limited with 100 Mbps USB speed. Also, there is an I2C bus to exchange some technical information between nodes, including Real-Time Clock (RTC).

### From where the cluster board boots OS?
You can boot the OS either from eMMC, SD card or netboot

### Does each node get its own IP address?

Yes

### Can I flash compute modules through the board?

Yes, you can flash a compute module using a top/master node.

### How do the Ethernet, USB, HDMI, and audio ports work? Is there a way to switch them between nodes?

There are 8 USB on the board. Each pair of USB connected to a particular node.
2x USB routed to the top/master node, 2x to the second node, 2x to the fourth node, 2x to the 6th node.
HDMI and audio connected with a top/master node.

### There are ATX and DC 12V power ports on the cluster board. Does this mean it can function from either an ATX power supply or 12V?

Yes