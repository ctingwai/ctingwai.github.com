---
layout: post
title: "Fedora 18 LXDE Installation on Macbook"
date: 2013-04-05 11:36
comments: true
categories: 
- Macbook
- Linux
- Fedora 18
- Fedora
- Installation
---
The installation process for Fedora 18 is way easier than its previous versions, the process is not that different from other laptops. Even for creating media, all I have to do is to burn it to a DVD (or create bootable USB if you downloaded a Live CD) which, in its previous version require me to tweak it or use reFit/reFind to boot the installer. The only struggle that I had was when I am creating a DVD on my Macbook's superdrive, for some unknown reasons, it just couldn't read what it burnt, it means it can't boot into a newly burnt DVD. I have tried burning it using both Fedora 17 and Mac OSX 10.7, both failed, still can't read what it burnt. So, my only solution is to go for an external DVD drive, which I had lying around inside my now ancient PC, I connected it using a SATA to USB cable, burn it, booted it, and it works!

So let's go to the main point of this article, the installation process for Fedora 18 includes downloading media, making media, boot into the media, install and post-installation. You can also refer to [Fedora Documentation](http://docs.fedoraproject.org/) for a detailed installation process.

##1. Downloading media##
Depending on your preferences, the DVD or LXDE Live Media version. Whichever you download, remember that EFI systems require you to download X86\_64 (64-bit) architecture or it will not work (i386 is for 32-bit which you should avoid). DVD will neither boot from USB nor Live Media but DVD provides you with a wide range of package selection, you can choose to install Gnome, KDE etc. all from the DVD which is not present in Live CD installation.

##2. Making Media##
###a. Bootable DVD###
Once you have the DVD ISO, you can burn it using GUI CD burning program or cdrecord as follows:
	# cdrecord -v -dao speed=8 dev=/dev/sr0 /path/to/ISO

You can omit "speed=8" option, but lower speeds reduce write errors. Refer to [FOSSwire](fosswire.com/post/2007/11/burning-cd-and-dvd-iso-images-with-cdrecord) for detailed explanation of its options and cdrecord command.

###b. Bootable USB###
If you wanted to save a DVD, you can make a bootable USB drive, these steps are based on [Fedora's Documentation](http://docs.fedoraproject.org/en-US/Fedora/18/html/Installation_Guide/Making_USB_Media-UNIX_Linux.html#Making_USB_Media-UNIX_Linux-RHEL_Fedora-dd):

1\. Clear the first blocks of the USB drive, replace `/dev/sdX` with the correct device:
	# dd if=/dev/zero of=/dev/sdX bs=1M count=100
2\. Copy ISO to the device, replace `/dev/sdX` with the correct device and `path/to/fedora/dvd.iso` with the correct ISO file path:
	# dd if=path/to/fedora/dvd.iso of=/dev/sdX
3\. On my X86\_64 DVD ISO, I realised that the ID field of the partition table entry is incorrect, so I used `fdisk` to correct the problem.

##3. Booting the installer##
1\. Reboot Macbook  
2\. Hold option/alt key until boot medium selection menu appears  
3\. Insert the newly burnt DVD  
4\. Select "EFI Boot"  
After this step, it will drop you into a grub prompt, don't worry, it is a known bug listed [here](http://fedoraproject.org/wiki/Common_F18_bugs#UEFI_boot_of_Fedora_18_CD_or_DVD_disc_on_Mac_hardware_falls_to_a_prompt). So, enter the following command and it will boot normally:  
	grub> configfile (cd0,apple3)/EFI/BOOT/grub.cfg
5\. Select either one from the grub menu

##4. Installation##
The installation for Fedora 18 is rather straightforward, there is a major UI rework for Fedora 18. One problem that I encountered during installation is the difficulties to create a custom partition. The process for creating and using LVM is very confusing , in traditional installation for creating and use LVM, the process is as follows:

1\. Mark/Format partition as LVM  
2\. Create Volume Group from the LVM  
3\. Create Physical Volume from Volume Group  

In the new installer, step 1-2 is not needed anymore, you go directly into creating Physical Volumes. When you selected the HDD to partition, a dialog will appear, and it let you select either BTRFS, LVM or standard partition type. If you selected LVM, you might found that there is no way to create Volume Group. In fact, it is created automatically if you create a filesystem such as /home. You can select which Volume Group this newly created filesystem/mount point goes into, which the installer should create for you if you don't have one. You can modify this Volume Group's name by clicking the modify button.

I also found that there is a problem when you want to use all remaining free space for a Volume Group. If you didn't allocate all free space for newly created mount points/filesystems, it means that if your total free space is 100GB, and you want your Volume Group to use all 100GB, you can't just allocate a total of 50GB for Fedora installation, it will result in a Volume Group of 50GB and not 100GB. This is also listed as known bugs [here](http://fedoraproject.org/wiki/Common_F18_bugs#Difficult_to_install_to_free_space_within_an_existing_LVM_volume_group_.28VG.29).

There are times where your installer will freeze/stuck when partitioning, I can move the mouse pointer, but everything you click will not respond, it is actually a focus problem, you just have to press ESC to remove the focus. It is listed [here](http://fedoraproject.org/wiki/Common_F18_bugs#Installer_can_become_apparently_.27stuck.27_in_custom_partitioning_mode_due_to_window_focus_problems) as a known bug.

##5. Post installation##
###Package Updates###
The first and most important step in any fresh installation is to update packages to latest versions. On my Macbook, wireless isn't working so I have to pop in my extra wireless USB for package updates, you can also use a wired connection which works out of the box.

###Nouveau Problems###
After update to kernel 3.8 from 3.6, I found that the boot splash is corrupted, part of the mouse pointer disappeared, what I mean is that the mouse pointer is working but the middle of the mouse becomes transparent. There is also a sudden freeze/stuck when booted into desktop environment. This is a problem applied to nouveau's Kernel Mode Setting (KMS) on newer kernels (kernel 3.7 not tested) which is not yet fixed, so my solution is to install NVidia proprietary driver. Instructions can be found at <http://www.if-not-true-then-false.com/2013/fedora-18-nvidia-guide/>, be sure to install packages for 3D acceleration packages if you plan on using it.

###Screen brightness not working on NVidia proprietary driver###
The directory `/sys/class/backlight` is now empty, this means you can't control monitor brightness. You have to install nvidiabl driver for it to work.

	$ su -
	# cd
	# git clone https://github.com/guillaumezin/nvidiabl.git
	# cd nvidiabl
	# yum install dkms
	# make dkms-install
	# echo "blacklist apple\_bl >> /etc/modprobe.d/blacklist.conf"
	# modprobe -r apple\_bl
	# modprobe nvidiabl min=0 max=1050
	# touch /etc/modprobe.d/nvidabl.conf && echo "options nvidiabl min=0 max=1050" > /etc/modprobe.d/nvidiabl.conf

You can adjust min and max options for nvidiabl, `max=1050` works best for me, it might different for you, `min=0` is able to turn off monitor, change to larger than 0 if you don't want that.

References:  
1\. <https://mknowles.com.au/wordpress/2012/01/30/fedora-16-fixing-the-macbook-51-nvidia-lcd-brightness/>  
2\. <https://github.com/guillaumezin/nvidiabl>  

###Configuring sudo###
Personally, I prefer `sudo` over `su -c` command, it is also required for keyboard and monitor brightness control. It is actually up to your own preferences.

1\. `$ su -c 'visudo'`  
2\. Add `ALL=(ALL) ALL` (change user with your ) line below `root    ALL=(ALL) ALL` line without quotes  
3a. Add the following lines to the end if you want keyboard brightness control as well:  
	## Allow user to execute without tty
	Defaults:<username> !requiretty
	
	## Allow user to execute without password
	%<username>        ALL = NOPASSWD: /bin/chmod 666 /sys/class/backlight/nvidia_backlight/brightness, /bin/chmod 666 /sys/class/leds/smc\:\:kbd_backlight/brightness
3b. Add the following lines to the end if you don't want keyboard brightness control:  
	## Allow user to execute without tty
	Defaults:<username> !requiretty
	
	## Allow user to execute without password
	%<username>        ALL = NOPASSWD: /bin/chmod 666 /sys/class/backlight/nvidia_backlight/brightness
4\. Save and quit

Step 3 is optional to allow monitor and keyboard brightness control scripts to work. Skip this step if you don't want it or have other methods for controls.

__NOTE:__ visudo uses vi editor, it starts in command mode, you can enter insert mode by pressing 'i' and exit insert mode to go back to command mode by pressing ESC. Saving require you to enter :wq in command mode or by pressing ZZ in command mode.

References: <http://fedorasolved.org/post-install-solutions/sudo>

###Screen brightness shortcuts###
1\. Add the first script to `~/bin/backlightInc`:
	#!/bin/bash
	
	#Brightness File
	backlight="/sys/class/backlight/nvidia_backlight/brightness"
	max=`cat "/sys/class/backlight/nvidia_backlight/max_brightness"`
	
	current=$(cat $backlight)
	new=$((current +10))
	if [ $new -gt "$max" ]; then
	        new=$max
	fi
	
	# check the permissions of the `brightness_file'
	# file and change them only if necessary
	#
	# for this to work you should have a line like
	# %<username> ALL = NOPASSWD: /bin/chmod 666 <path to the brightness file>
	# in /etc/sudoers
	#
	# (add this line using `visudo' as root)
	#
	# You might need to add the following line if you get an error about tty
	# Defaults:<username> !requiretty
	
	
	if [ `stat -c %a "$backlight"` != 666 ]; then
	        sudo /bin/chmod 666 "$backlight"
	fi
	
	echo $new > $backlight
2\. Add the second script to `~/bin/backlightDec`:
	#!/bin/bash
	
	backlight="/sys/class/backlight/nvidia_backlight/brightness"
	
	current=$(cat $backlight)
	new=$((current - 10))
	if [ $new -lt 0 ]; then
	        new=0
	fi
	# check the permissions of the `brightness_file'
	# file and change them only if necessary
	#
	# for this to work you should have a line like
	# %<username> ALL = NOPASSWD: /bin/chmod 666 <path to the brightness file>
	# in /etc/sudoers
	#
	# (add this line using `visudo' as root)
	#
	# You might need to add the following line if you get an error about tty
	# Defaults:<username> !requiretty
	
	if [ `stat -c %a "$backlight"` != 666 ]
	then
	        sudo /bin/chmod 666 "$backlight"
	fi
	
	echo $new > $backlight
3\. Add executable bit to both scripts:
	$ chmod +x ~/bin/backlight\*

4\. Edit `~/.config/openbox/lxde-rc.xml` with your favourite editor  
5\. Add the following lines after `<keyboard>` but before `</keyboard>` tag:
		<!-- Monitor Brightness control -->
		<keybind key="XF86MonBrightnessDown">
		  <action name="Execute">
		    <command>backlightDec</command>
		  </action>
		</keybind>
		<keybind key="XF86MonBrightnessUp">
		  <action name="Execute">
		    <command>backlightInc</command>
		  </action>
		</keybind>
6\. Reconfigure openbox:
	$ openbox --reconfigure

###Keyboard brightness shortcut###
If your Macbook does not come with keyboard brightness, skip this step.  
1\. Add the first script to `~/bin/kbdBacklightInc`:
	#!/bin/bash
	
	#Brightness File
	backlight="/sys/class/leds/smc::kbd_backlight/brightness"
	max=`cat "/sys/class/leds/smc::kbd_backlight/max_brightness"`
	
	current=$(cat $backlight)
	new=$((current +5))
	if [ $new -gt "$max" ]; then
	        new=$max
	fi
	
	# check the permissions of the `brightness_file'
	# file and change them only if necessary
	#
	# for this to work you should have a line like
	# %<username> ALL = NOPASSWD: /bin/chmod 666 <path to the brightness file>
	# in /etc/sudoers
	#
	# (add this line using `visudo' as root)
	#
	# You might need to add the following line if you get an error about tty
	# Defaults:<username> !requiretty
	
	
	if [ `stat -c %a "$backlight"` != 666 ]; then
	        `sudo /bin/chmod 666 "$backlight"`
	fi
	
	echo $new > $backlight
2\. Add the second script to `~/bin/kbdBrightnessDec`:  
	#!/bin/bash
	
	backlight="/sys/class/leds/smc::kbd_backlight/brightness"
	
	current=$(cat $backlight)
	new=$((current - 5))
	if [ $new -lt 0 ]; then
	        new=0
	fi
	# check the permissions of the `brightness_file'
	# file and change them only if necessary
	#
	# for this to work you should have a line like
	# %<username> ALL = NOPASSWD: /bin/chmod 666 <path to the brightness file>
	# in /etc/sudoers
	#
	# (add this line using `visudo' as root)
	#
	# You might need to add the following line if you get an error about tty
	# Defaults:<username> !requiretty
	
	
	if [ `stat -c %a "$backlight"` != 666 ]
	then
	        `sudo /bin/chmod 666 "$backlight"`
	fi
	
	echo $new > $backlight
3\. Add execute bit to both scripts:
	$ chmod +x ~/bin/kbdBacklight*

4\. Edit `~/.config/openbox/lxde-rc.xml` with your favourite editor  
5\. Add the following lines after `<keyboard>` but before `</keyboard>` tag:
		<!-- Keyboard brightness control -->
		<keybind key="XF86KbdBrightnessDown">
		  <action name="Execute">
		    <command>kbdBacklightDec</command>
		  </action>
		</keybind>
		<keybind key="XF86KbdBrightnessUp">
		  <action name="Execute">
		    <command>kbdBacklightInc</command>
		  </action>
		</keybind>
6\. Reconfigure openbox:
	$ openbox --reconfigure

###Audio volume controls###
Edit `~/openbox/lxde-rc.xml` in your favourite editor and add the following lines between `<keyboard>` and `</keyboard>` tag:
		<!-- Audio volume control -->
		<keybind key="XF86AudioLowerVolume">
		  <action name="Execute">
		    <startupnotify>
		      <enabled>true</enabled>
		      <name>amixer</name>
		    </startupnotify>
		    <command>amixer -c 0 set Master 5- unmute</command>
		  </action>
		</keybind>
		<keybind key="XF86AudioRaiseVolume">
		  <action name="Execute">
		    <startupnotify>
		      <enabled>true</enabled>
		      <name>amixer</name>
		    </startupnotify>
		    <command>amixer -c 0 set Master 5+ unmute</command>
		  </action>
		</keybind>
		<keybind key="XF86AudioMute">
		  <action name="Execute">
		    <startupnotify>
		      <enabled>true</enabled>
		      <name>amixer</name>
		    </startupnotify>
		    <command>amixer set Master toggle</command>
		  </action>
		</keybind>
References: <http://wiki.lxde.org/en/LXDE:Questions#How_do_I_make_my_keyboard_volume_buttons_work.3F>

###Wireless Driver###
For broadcom cards, you have two driver options, B43 or Broadcom-STA. From my experience Broadcom-STA bug fixes are rather slow, you often have to wait weeks or even months for a bug to be fixed, and sometimes, they won't be fixed for versions/forever, so, I recommend that you only use Broadcom-STA only when you encounter problems with B43 driver. Please refer to B43 wireless driver page for B43 driver installation instructions and check driver compatibility.

###If you share data between Mac OSX and Fedora using soft/symbolic links AKA symlinks...###
You have to change your UID in Fedora to 501 or those directories will not be writable:
	$ sudo usermod -u 501

###Additional packages and configurations###
Once everything is working, you can install any additional packages, I included some package installation and configuration:

####Google Chome####
Refer to [Google Chrome Installation Fedora 18](http://www.if-not-true-then-false.com/2010/install-google-chrome-with-yum-on-fedora-red-hat-rhel/)

####Conky####
Coming soon...

####XTerm####
Coming soon...

####ClamTK####
Coming soon...
