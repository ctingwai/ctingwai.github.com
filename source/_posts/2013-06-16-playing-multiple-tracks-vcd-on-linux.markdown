---
layout: post
title: "Playing Multiple Tracks VCD on Linux"
date: 2013-06-16 13:43
comments: true
categories: 
- Linux
- VCD
- Multimedia
---
I have bought a VCD movie recently to find that it was not playable on my OpenSUSE, CD2 works fine but not CD1. Then I go back for a replacement disc on the shop and came back to watch it but unfortunately, the problem persist. The problem is that CD1 comtains multiple tracks that most Mediaplayer doesn't support (tried VLC and Kaffeine). So I started to dive into the problems and found a thread that provides useful information on playing VCD on Linux (Refer to references).
<!-- more -->

Basically there are two ways to work around this problem:  
1. Use MPlayer to play your disc  
2. Rip your VCD using vcdxrip  

##1. Use MPlayer to play your dics##
MPlayer was quite new to me actually, it is launched through the console, and it is does not appear in the menu. Here is the command to play VCD using MPlayer (install if you haven't already):

	mplayer vcd://<track number> -cdrom-device /dev/<your cd rom device>

Of course, you have to substitute `<track number>` and `<your cd rom device>` with the track number and your cd rom device. On my machine, it's on track 3 and my cd rom device is either dvd or sr0 (dvd is a symlink to sr0). So my complete command is:

	mplayer vcd://3 -cdrom-device /dev/sr0

Refer to mplayer's manpage for controls and details: `man mplayer`.

##2. Rip your VCD using vcdxrip##
This has not personally worked for me because it keeps stopping at certain points, it just doesn't copy the whole VCD. But it is still good to know if MPlayer does not work. The command is very easy and straight forward. Remember to change to a directory for your ripped files before executing this command:

	vcdxrip -C

And there will be multiple files present on your currect directory, where you could play using VLC or whatever media player you may wish.

##REFERENCES##
1. [MPlayer with VCD](http://www.mplayerhq.hu/DOCS/HTML/en/vcd.html)  
2. [Forums discussing issues with VCDs](http://forums.linuxmint.com/viewtopic.php?f=48&t=43106)
