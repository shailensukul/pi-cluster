# Pi Micro-server Cluster

![micro server image](/images/micro-server.jpg)

## Overview

While learning Docker Swarm concepts, I decided to create a microserver, composed of 4 Raspberry Pi 4B computers, so that I could better experience a real world scenario (and also coz it is pretty damn cool!)

## Items

* [4 X Raspberry Pi 4B 4G](https://www.littlebird.com.au/products/raspberry-pi-4-model-b-4-gb)
* [4 X Raspberry Pi PoE Hats](https://core-electronics.com.au/raspberry-pi-poe-hat-official.html)
* [4 X 32G SD card](https://www.amazon.com.au/gp/product/B073S49S8M)
* [Stackable Case](https://www.amazon.com.au/gp/product/B07MBXMSQX)
* [4 X 1 foot CAT 6 cables](https://www.amazon.com.au/gp/product/B008I8A0TW)
* [4 port power over ethernet switch](https://www.amazon.com.au/gp/product/B076PRM2C5)

## Installing the OS

* [Download the Raspian Buster Lite image](https://www.raspberrypi.org/downloads/raspbian/)
* [Download the Wind32DiskImager tool](https://raspberry-projects.com/pi/pi-operating-systems/win32diskimager) - I am flashing the SD card from Windows 10
* Insert the SD cards and name the volume PI01, PI02, PI03 and PI04
* In your Windows 10 computer, launch Wind32DiskImager, choose your SD card drive and the Raspian Buster Lite image and choose Write. Do it for all the 4 SD cards
* In the boot volume, create a empty file and call it ssh
* In the boot volume, create a file called wpa_supplicant.conf and add the following (replace with your own WIFI details):
```
country=AU # Your 2-digit country code
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="WIFI1"
    psk="password1"
    key_mgmt=WPA-PSK
	id_str="identifier1"
	priority=1
}

network={
    ssid="WIFI2"
    psk="password2"
    id_str="identifier2"
	priority=2
}
```
* Insert the Raspberry PI and turn on

## Discovering the IP Addresses of the Pis

* Connect the Pi devices to your network

* On a PC connected to the same network, discover your PC ip address

(On Windows 10)
```
ipconfig 
```

* Install nmap on WSL (Windows Subsystem for Linux)
```
sudo apt install nmap
```

* Use nmap to poll the devices on your network, using your ip address from above
```
nmap -sn 192.168.86.41/24 
```

* You will be able to then view the raspberry pi's IP addresses alongside the pi's hostname

![NMap Results](/images/nmap-scan.JPG)

## Changing the default password of the Pi & Set Hostname

* SSH into the Pi
```
ssh pi@<pi-ipaddress>
default password is: raspberry
```
* Change the default password
```
passwd
```

* Get to config
```
sudo aspi-config
```

* Select 2. Network Options

* Select N1 Hostname

* Set the hostname to pi-1, pi-2, pi-3 and pi-4

* Save and reboot 

* Now you will be able to:
```
ping pi-1

ssh pi@pi-1
```


## Install Docker and Docker Swarm

* First, in each Pi, create another user (do not use root) and give it admin privileges:
```
sudo adduser dockeradmin
sudo usermod -aG sudo dockeradmin
logout
```

* Now, you can SSH as the docker admin user
```
ssh dockeradmin@pi-1
```

* Install Docker
```
curl -fsSL https://get.docker.com | sh
```
[![install docker](https://asciinema.org/a/GNBxJGp0R9vnwul06a3qyUI7Y.svg)](https://asciinema.org/a/GNBxJGp0R9vnwul06a3qyUI7Y)


*  Create Swarm cluster
Initialize the Swarm cluster on the first node (e.g., node0 with ip 192.168.0.20)

```
sudo docker swarm init --advertise-addr 192.168.0.20
```

 * Add manager nodes
 To add a manager to swarm cluster, run the following command on manager Leader node:
 ```
 docker swarm join-token manager
 ```

 * Add worker nodes

 To add a worker node to swarm cluster, run the following command on each new worker node:

```
 docker swarm join --token SWMTKN-1-4ayyu7shasjzry6j6q007nd3ltal6ett0dxny8nrm0gr21z6be-c6b69sxqjgcpbeetqt4ww4vyg 192.168.0.20:2377
```

* Change roles
You can change the role of a node running:

```docker node promote <node>``` to promote a worker node to be a manager.

```docker node demote <node>``` to demote a manager node to a worker node.

* Listing nodes of swarm

```
docker node ls
```

* Running Docker Swarm Visualizer service

```
docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  alexellis2/visualizer-arm:latest
```

After the service creation, check the status by typing:

```
docker service ls
```

If REPLICAS value is 1/1, so the service is ready.
You can visit any node on port 8080 to see the service running, e.g.: http://pi-1:8080/

![Pi Visualizer](/images/pi-visualizer.JPG)