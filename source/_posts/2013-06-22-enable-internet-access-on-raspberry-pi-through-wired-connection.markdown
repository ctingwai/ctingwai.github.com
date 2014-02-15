---
layout: post
title: "Enable Internet Access on Raspberry Pi Through Wired Connection"
date: 2013-06-22 17:00
comments: true
categories: 
- Raspberry Pi
- SSH
- Networking
---
The first I got my hands on a raspberry pi, I wanted to update its OS to the latest, but the problem is that my RPi is connected to my laptop (using OpenSuse) and I am using SSH to access it (I don't have access to any displays).
<!-- more -->

##Preparation##
First, prepare your SDCard for RPi, there are plenty of resources on how to prepare your SDCard. Then, follow [this guide](http://www.raspberrypi.org/archives/3760) to configure your RPi for SSH access from your machine.

###IP Addressing###
For this guide, I am using the following addressing scheme:  
- RPi eth0 => 192.168.10.2/24  
- Laptop eth0 (connected to RPi) => 192.168.10.1/24  
- Laptop wlan0 (connected to router, default gateway) => 192.168.1.6/24  
- Router (Internet access) => 192.168.1.254/24  

##Fixing Default Gateway on my Laptop##
When I connected to my RPi, Network Manager replaced my default gateway to `eth0`, so I am not able to access the Internet after connected to RPi. You verify it by using `route` command without parameters, if the default line points to eth0, it means your default gateway is overwritten, if it's not, it means you can skip this section, you can verify it by ping-ing `www.google.com`. To fix this problem, I deleted the default gateway and added a new default gateway that points to my router:

	$ sudo /sbin/route del default
	$ sudo /sbin/route add default gw 192.168.1.254 dev wlan0

Verify that your default gateway now points to wlan0 using `route` command without parameters. Try to ping `www.google.com` to make sure that your laptop have access to the Internet, your RPi won't be able to access the Internet if your laptop does not.

##Adding Default Gateway to RPi##
Using the `route command` add a default that points to eth0 interface:

	$ sudo route add default gw 192.168.10.1 dev eth0

##The Problem##
Out of the box without masquerading, once a packet sourced from 192.168.10.0/24 network (my RPi <==> laptop subnet) reaches the router on subnet 192.168.1.0/24, depends on the configuration, it will route or drop the packet. Even if the router successfully routed the packet to the Internet, it will definitely drop the packet, because it does not know where to route the packet for network 192.168.10.0/24 network. So, masquerading seems to be the only solution here.

##Masquerading and Configuration##
Masquerading is like that Linux version of NAT, it translates your internal network to external network (e.g. for Internet access). What it does is that any packets bound for any network from 192.168.10.0/24 subnet will be translated to 192.168.1.0/24 subnet, so my router knows where to route my 192.168.10.0/24 packet.

Since I am using OpenSUSE, I will configure masquerading through its own firewall using YaST.  
1. Open up firewall configuration in YaST and select `Interfaces`  
2. Double click eth0 interface and change it to `External Zone`  
3. Select `Masquerading` on the left and click `Masquerade Networks`  
4. Add `80` to `requested port`  
5. Add `192.168.1.6` (or your wlan0 IP) to `Redirect to Masqueraded IP`  
6. Add `81` to `Redirect to Port` or any ports you want your traffic to be translated to, but make sure you are not using any servers on that port  
7. Click `Add`  
8. Select `Startup` on the left and click `Save Settings and Restart Firewall Now`  
9. Try to ping `www.google.com` from your RPi, you should get be able to at this point  

##REFERENCES##
1. [Using your desktop or laptop screen and keyboard with your Pi](http://www.raspberrypi.org/archives/3760)
