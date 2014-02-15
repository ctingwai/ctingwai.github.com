---
layout: post
title: "Using Juniper VPN on 64 bit Linux"
date: 2013-05-18 18:21
comments: true
categories: 
- Juniper VPN
- Linux
- 64 bit
- Networking
---
When I was supporting a company that uses Juniper VPN with my colleague, I found that Juniper VPN is only supported in 32-bit version of Linux (though it was supported in 64-bit Windows and Mac machine, should ask them why they don't compile it for 64-bit Linux). I have spent hours finding solution to this situation and found one particular solution that just works. Tried [Mad Scientist's JNC (Juniper Network Connect)](http://mad-scientist.net/juniper.html) but it didn't work for some unknown reasons.
<!-- more -->

This solution is based on [Dominique Leuenberger's blog on 'Juniper VPN on openSUSE x86\_64'](http://dominique.leuenberger.net/blog/2010/07/juniper-vpn-on-opensuse-x86_64/), all credits goes to him/her.

##Requirements##
To use Juniper VPN, JRE or JDK with web plugins is a must ([Download Here](http://www.oracle.com/technetwork/java/javase/downloads/index.html)), it does not work with IcedTea and openJDK. We are not using any third party solution, so we have to comply to the Juniper VPN's system requirements.

##Steps##
1\. Download Juniper VPN through the software provided by the company. Once the applet is loaded, it should ask you for your root/su password, just press [Enter] twice. It will create `.juniper_networks` in your home directory.  
{% imgcap center /images/juniper-loading.png 'Juniper VPN Loading Screen' %}
{% imgcap center /images/juniper-pwdprompt.png 'Juniper VPN Password Prompt' %}  

2\. Change directory to `$HOME/.juniper_networks`

	cd $HOME/.juniper_networks

3\. Remove `network_connect` directory

	rm -rf network_connect

4\. Extract `ncLinuxApp.jar`

	unzip ncLinuxApp.jar

5\. Use `ldd` to find out required libraries for `network_connect/libncui.so` and `zypper wp <library>` or `yum provides <library>` to find out the libraries.

6\. Make a binary out of the library

	gcc -m32 -Wl,-rpath,`pwd` -o network_connect/ncui network_connect/libncui.so

7\. Set permission and owner/group

	sudo chown root:root network_connect/ncui
	sudo chmod 6711 network_connect/ncui

8\. Get the certificate

	sh network_connect/ncui <your Juniper VPN host> <certificatename>.cer

9\. Make sure that you are still logged into you VPN host, and find your DSID by browsing though your browser's cookie of your VPN site. Search for the cookie named DSID

10\. Connect to Juniper VPN

	network_connect/ncui -h <you Juniper VPN host> -c DSID=<value obtained in step 9> -f <certificate obtained in step 8>.cer

11\. (Optional) To ease future VPN connections, copy and paste the following script to `$HOME/bin/vpnConnect`

	#!/bin/bash
	
	if [ $# -lt 1 ]; then
	        echo -e "Usage:\t$0 <DSID>"
	        echo -e "\n\tNOTE: DSID can be found in the cookie after you logged into your VPN site"
	        exit 0
	fi

	# Connect to your VPN
	~/.juniper_networks/network_connect/ncui -h <your vpn host> -c DSID=$1 -f ~/.juniper_networks/<cert from step 8>.cer

12\. (Continue step 11) Add executable bit `chmod +x $HOME/bin/vpnConnect`

13\. (To connect after step 12) Use `vpnConnect <your DSID as in step 9>` to connect

##Alternative ways for shortening##
Personally I prefer to use a script to shorten my commands, because it allow me to specify usage notes and comments when the usage is not right, but if you are not like me, you can use Linux aliases to shorten it, refer to `man alias` for usage or Google it =)

##REFERENCES##
1\. [Mad Scientist's JNC (Juniper Network Connect)](http://mad-scientist.net/juniper.html)  
2\. [Dominique Leuenberger's blog on 'Juniper VPN on openSUSE x86\_64'](http://dominique.leuenberger.net/blog/2010/07/juniper-vpn-on-opensuse-x86_64/)
