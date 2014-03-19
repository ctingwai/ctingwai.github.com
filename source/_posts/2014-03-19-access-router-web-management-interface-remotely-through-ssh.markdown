---
layout: post
title: "Access Router Web Management Interface Remotely Through SSH"
date: 2014-03-19 14:01
comments: true
categories: 
- Networking
- Linux
- Unix
- SSH
---
As an administrator for multiple Linux servers in various location, I find it
hard to manage port forwarding in routers, of course you can configure a
remote GUI login such as RDP and VNC but all these servers only enable
services they need with SSH as their main method of remote administration
to these servers. Then, I found out about SSH tunneling, which is a lifesaver.
There are two main method to tunnel traffic through SSH:

1. [Forward a single port through SSH](http://superuser.com/questions/330131/ssh-tunnel-to-home-network-and-access-router-web-interface)
2. [Tunneling through SOCKS](https://wiki.archlinux.org/index.php/Secure_Shell#Encrypted_SOCKS_tunnel)
<!-- more -->

The first method is good for temporary tunneling and the second method is good
for when you want to encrypt your traffic. So, I think first method is more
suitable to my case, but you are free to use the second method as well.

## Method 1: Forward a single port through SSH
Make sure you know your router IP before you run SSH command, you can just SSH
into your remote machine to get the default using `route` command. After you
know your remote router IP, you can configure your tunnel using `ssh` command:

    ssh -p [remote ssh port if changed] -L8080:[remote router IP]:80 [username]@[host]

Make sure that you fill in those variables in square brackets. After entering
your user password, you can open your browser and access your router web management
through `127.0.0.1:8080`. Make sure that your firewall/iptables are not blocking port 8080.

## Method 2: Tunneling through SOCKS
This method forward ports according to the port you specified in your browser
or application. However, it is more complex and usually used to encrypt your
traffic. You create a SOCKS tunnel using `ssh` command:

    ssh -TND 4711 [username]@[host]

Make sure that you fill in those variables in square brackets. After entering
your user password, you need to configure your browser to use SOCKS proxy, you
can find more information on how to configure web browser for SOCKS proxy
[here](https://wiki.archlinux.org/index.php/Secure_Shell#Encrypted_SOCKS_tunnel).
Once configured, you need to enter the router's IP to access the router's web
management interface.
