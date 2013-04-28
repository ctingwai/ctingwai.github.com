---
layout: post
title: "Permanent DNS Settings for Network Manager"
date: 2013-04-23 16:39
comments: true
categories: 
- Fedora 18
- Fedora
- Linux
- Network Manager
- Networking
---
DNS settings in Linux are generally stored in `/etc/resolv.conf` file, you could basically just edit this file to change the DNS settings in any Linux systems. However, the change is not permanent, it will be overwritten by Network Manager when u reconnect or reboot. So, to make the change permanent, there are two methods, Network Manager's dispatcher script method and the immutable flag method.

##Using Network Manager's Dispatcher Method##
This solution is based on [ArchWiki](https://wiki.archlinux.org/index.php/NetworkManager#Use_OpenDNS_servers). Basically, the steps are:

1\. Become root and create a new file `/etc/resolv.conf.googledns`
2\. Edit `/etc/resolv.conf.googledns` and add the following lines:
	nameserver 8.8.8.8
	nameserver 8.8.4.4

3\. Add the following script to `/etc/NetworkManager/dispatcher.d/12-dns_server`:
	# cp -f /etc/resolv.conf.googledns /etc/resolv.conf

4\. Add executable bit to the script:
	# chmod +x /etc/NetworkManager/dispatcher.d/12-dns_server

5\. Restart Network Manager:
	# systemctl restart NetworkManager

What these steps do is just to replace `/etc/resolv.conf` with the file `/etc/resolv.conf.googledns` everytime you connect.

##Immutable Flag Method##
You can also try adding _immutable_ flag to `/etc/resolv.conf` once you have changed the DNS (`nameserver` field) in `/etc/resolv.conf` file:
	# chattr +i /etc/resolv.conf

Once it's set, not even the root can modify this file, so you have to remove _immutable_ flag first before editing this file:
	# chattr -i /etc/resolv.conf

I have not personally tested this method though, there might be some complaints by NetworkManager about not able to write to `/etc/resolv.conf`.
