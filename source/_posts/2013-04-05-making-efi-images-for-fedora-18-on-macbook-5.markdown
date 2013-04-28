---
layout: post
title: "Making EFI images for Fedora 18 on Macbook 5,1"
date: 2013-04-05 09:22
comments: true
categories:
- Macbook
- Fedora 18
- Fedora
- Linux
---

When I was enabling the theme features on my Macbook Fedora, I found that there is not much guides on how to enable it on an EFI systems such as Macbook, most of the guides I found are for non-EFI systems. So I think it's a good idea to create a simple guide for other people who encounter the same problem as me and for myself to refer in the future. 

Grub require certain modules to enable/disable some features such as the support for themes. On an EFI systems like Macbook, you are required to create an EFI image with modules that support the feature. The first step of creating an EFI image is to determine the names and features of modules. Some grub shell commands such as linuxefi and initrdefi are crucial to boot Linux, make sure you don't miss it when creating the image.

##1. Determining modules for the grub image##
When you issue `grub2-mkimage` command, it will look at the directory, `/usr/lib/grub/x86\_64-efi`, to find the modules specified in your command. A simple ls command will list all modules:
	# ls /usr/lib/grub/x86\_64-efi/\*.mod

Unfortunately, it does not tell us what features each of these modules provide. Luckily, within this directory, it also provide `command.lst` that list what commands each modules provide:
	# cat /usr/lib/grub/x86\_64-efi/command.lst

Other than commands, these modules also provide what filesystems and partition table that grub will support. The modules have a naming scheme, modules that start with "part\_" are for partition tables and modules with filesystem names are filesystems to support, such as `ext3.mod`.

Write down any features that you need on a text file with space separated for each modules and trailing `.mod` or `.module`, e.g. `part_msdos part_gpt ext3 ext4`, note that __partition tables come first, filesystem types come second, and any other modules come last__.

__NOTE:__ On some systems, you can include all modules the following command where ~/modules is the text file for the modules you need. This might not work on machines like mine:

	# ls /usr/lib/grub/x86_64-efi/*.mod | cut -d/ -f6 | cut -d. -f1 | tr '\n' ' ' > ~/modules

On my machine, including all modules does not work for me, but the following modules:

`part_gpt part_msdos fat ext2 hfs hfsplus chain boot configfile normal minicmd linuxefi reboot halt search gfxterm gfxmenu efi_gop efi_uga all_video loadbios gzio echo true loadenv bitmap_scale font cat help ls png jpeg tga test`

##2. Creating grub image##
Grub2 EFI image can be created using `grub2-mkimage`, some machine might use `grub-mkimage`, it's the same thing:

	# grub2-mkimage -p /boot/efi/EFI/fedora -o /boot/efi/EFI/fedora/grubx64.efi `cat ~/modules`

This command will create a grub image in `/boot/efi/EFI/fedora/grubx64.efi` and will look in directory `/boot/efi/EFI/fedora/` directory for config file, options are explained as follows:

_-p /boot/efi/EFI/fedora:_ This option will ask grub to look for the config file on /boot/efi/EFI/fedora directory<br />
_-o /boot/efi/EFI/fedora/grubx64.efi:_ This option will create the image as grubx64.efi in /boot/efi/EFI/fedora directory<br />
_\`cat ~/modules\`:_ This will print all modules listed in the text file ~/modules for grub2-mkimage to process, you can change this to the text file you store your modules

This configuration also assume that your efi images are stored in `/boot/efi/EFI/fedora` directory, it is a requirement to create an HFS partition and mount it in `/boot/efi` for my macbook to boot, if you use a different directory under /boot/efi/EFI, change it, e.g. `/boot/ef/EFI/redhat` if it is stored in `/boot/efi/EFI/redhat`.

It is best to create a config file in your `/boot/efi/EFI/fedora` directory if it is not present, by issuing `grub2-mkconfig` command:

	# grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg

This will create a config file in `/boot/efi/EFI/fedora/grub.cfg`, you may have to adjust the output file path if you specify a different path to grub2-mkimage's -p option.

##3. Blessing grubx64.efi image for pure EFI boot##

If you are using reFit or reFind, you can avoid this step as it will find your grub image dynamically, but if you want a pure EFI boot, you have to bless it using Mac OSX's `bless` command. You can either boot into recovery mode or Mac OSX to use this command (it is possible to bless it in your Linux machine by installing certain package, use `yum provides bless` on Fedora to search for the package that provides the `bless` command, but I have not personally tested it yet). Personally, I prefer to boot into recovery of Mac OSX Lion to use the bless command because it is much faster.

Once booted, you have to search for the full path of your grub image (grubx64.efi), on my system, it is in /Volumes/EFI\_Boot/EFI/, and issue the following command:

	$ sudo bless --folder /Volumes/EFI_Boot/EFI/fedora --file /Volumes/EFI_Boot/EFI/fedora/grubx64.efi

Once done, you can reboot your machine and hold option/alt key to find that your grub image is listed.
