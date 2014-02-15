---
layout: post
title: "Optimizing Linux Power Usage"
date: 2013-05-17 22:54
comments: true
categories: 
- Linux
- OpenSuSE 12.3
- Power Usage
---
A lot of Linux distro is not optimised for laptops, some of them could use up much power if you don't optimize it. By the end of this guide, you should be able to reduce your power consumption by 3-5 watts, I know it seems not much but it can give my machine 15-30 mins more power. By default, my machine used up more than 24 watts of power, as indicated by `powertop`. To find out the power usage of your machine:  
<!-- more -->

1\. Unplug or switch off your AC to your laptop  
2\. Enter `sudo /usr/sbin/powertop` in a terminal  
{% imgcap center /images/powertopOverview.png 'The left most column is my power consumption in watts' %}  

To reduce the power consumption, I have installed `laptop-mode-tools`, as its name suggest, it is a tool for laptops. Once installed, I found that my wireless driver (`ath5k`) does not support power saving mode yet, so I have to disable it. Edit `/etc/laptop-mode/` and change `WIRELESS_BATT_POWER_SAVING=1 to WIRELESS_BATT_POWER_SAVING=0`, this step is optional, nothing might happen if you don't do anything to it.  

To enable `laptop-mode`:

	$ sudo systemctl enable laptop-mode.service

Next, I created a custom script for `laptop-mode-tools` to enable certain power saving not included in `laptop-mode-tools` modules:  

1\. Edit `$HOME/bin/powersaving_on` and add the following lines:  

	#!/bin/sh

	# ATI Radeon power saving
	echo profile > /sys/class/drm/card0/device/power_method
	echo low > /sys/class/drm/card0/device/power_profile
	
	# Audio power saving
	echo 1 > /sys/module/snd_hda_intel/parameters/power_save
	echo Y > /sys/module/snd_hda_intel/parameters/power_save_controller
	
	# Writeback time
	echo 1500 > /proc/sys/vm/dirty_writeback_centisecs

2\. Edit `$HOME/bin/powersaving_off` and add the following lines:  

	#!/bin/sh
	
	# ATI Radeon power saving
	echo profile > /sys/class/drm/card0/device/power_method
	echo default > /sys/class/drm/card0/device/power_profile
	
	# Audio power saving
	echo 2 > /sys/module/snd_hda_intel/parameters/power_save
	echo N > /sys/module/snd_hda_intel/parameters/power_save_controller
	
	# Writeback time
	echo 500 > /proc/sys/vm/dirty_writeback_centisecs

3\. Add executable bit to both scripts:

	$ chmod +x $HOME/bin/powersaving_on; chmod +x $HOME/bin/powersaving_off

4\. Create symbolic links for laptop-mode-tools:

	$ sudo ln -s /home/<username>/bin/powersaving_on /etc/laptop-mode/batt-start/powersaving_on;\
	sudo ln -s /home/<username>/bin/powersaving_on /etc/laptop-mode/lm-ac-stop/powersaving_on;\
	sudo ln -s /home/<username>/bin/powersaving_off /etc/laptop-mode/batt-stop/powersaving_off;\
	sudo ln -s /home/<username>/bin/powersaving_off /etc/laptop-mode/lm-ac-start/powersaving_off;\
	sudo ln -s /home/<username>/bin/powersaving_off /etc/laptop-mode/nolm-ac-start/powersaving_off;\
	sudo ln -s /home/<username>/bin/powersaving_off /etc/laptop-mode/nolm-ac-stop/powersaving_off

__Explanation and Notes:__  
_Step 1_: Enable some powersaving features to reduce power usage (require root permission), see the script's comments. You can change `echo low > /sys/class/drm/card0/device/power_profile` to `echo mid > /sys/class/drm/card0/device/power_profile` if you need more power  
  
_Step 2_: Disable powersaving features by setting all its values to default  
  
_Step 3_: Make both scripts executable  
  
_Step 4_: I have wrote it in a way that you can cut and paste into your terminal emulator in one step, just replace `<username>` with your username. `laptop-mode-tools` provide a way for users to execute certain scripts when on AC or battery by placing your scripts in its corresponding directories:

*  /etc/laptop-mode/batt-start: Executed when laptop enters battery mode  
*  /etc/laptop-mode/batt-stop: Executed when laptop exits battery mode  
*  /etc/laptop-mode/lm-ac-start: Executed when `laptop-mode` is enabled AND laptop enters AC mode  
*  /etc/laptop-mode/lm-ac-stop: Executed when `laptop-mode` is enabled AND laptop exits AC mode  
*  /etc/laptop-mode/nolm-ac-start: Executed when `laptop-mode` is disabled through `/etc/laptop-mode/laptop-mode.conf` AND laptop enters AC mode  
*  /etc/laptop-mode/nolm-ac-stop: Executed when `laptop-mode` is disable through `/etc/laptop-mode/laptop-mode.conf` AND laptop exits AC mode  

__Other Tips:__  
1\. Disable bluetooth: `sudo rfkill block bluetooth`  
2\. It seems that monitor used up most power (11-18 watts depending on brightness on my machine), reduce brightness to save more power  
3\. Another power killer is WiFi, (more than 6 watts on my machine), so turn it off if you don't use it  

__REFERENCES:__  
1\. [Great ArchWiki Article on Power Saving](https://wiki.archlinux.org/index.php/Power_saving)  
2\. [ATI Radeon Power Management Guide](http://aubreypwd.com/blog/2012/09/14/howto-ubuntu-12-04-open-source-radeon-drivers-and-power-management/)  
3\. [Linux Journal Article on laptop-mode-tools](http://www.linuxjournal.com/article/7539?page=0,1)  
4\. [Using ATI Radeon Power Management with laptop-mode-tools](http://www.overclock.net/t/731469/how-to-power-saving-with-the-radeon-driver)  
