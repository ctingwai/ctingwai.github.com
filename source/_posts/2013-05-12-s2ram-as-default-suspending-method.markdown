---
layout: post
title: "S2ram as default suspending method"
date: 2013-05-12 23:45
comments: true
categories: 
- Suspend
- Linux
- OpenSuSE 12.3
---
When I installed OpenSuSE 12.3 on my VAIO laptop, suspend is not working (not even `suspend` command). Instead, I have to issue `s2ram` command in OpenSuSE to suspend my machine. To make it the default sleep module (`uswsusp`), here are the steps required:
<!-- more -->

1\. Edit `/etc/pm/config.d/module` and add the following line:

	SLEEP_MODULE=uswsusp

2\. Edit `/etc/pm/config.d/defaults` and add the following line:

	S2RAM_OPTS="-f"

3\. Reboot and it will go to sleep again.

__REFERENCES__  
1. [OpenSuSE Documentation](http://en.opensuse.org/SDB:Suspend_to_RAM)  
2. [Ask Ubuntu Thread](http://askubuntu.com/questions/54591/use-s2ram-when-closing-lid-with-kde)  
