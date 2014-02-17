---
layout: post
title: "Bluetoothd crash on connect"
date: 2014-02-17 21:47
comments: true
categories: 
- Bluetooth
- Linux
- Fix
---
I updated my Linux machine recently and the next thing I know is that bluetooth
keeps crashing with SIGSEGV. By following
[this Archlinux wiki's](https://wiki.archlinux.org/index.php/Step_By_Step_Debugging_Guide)
strace method, I am able to find out that the crash is caused by nothing more
than some missing files.
<!-- more -->

All I had to do is removing my device in `/var/lib/bluetooth/` folder,
but for safety, I renamed it instead:

    mv /var/lib/bluetooth/00\:11\:22\:33\:44\:55 /var/lib/bluetooth/00\:11\:22\:33\:44\:55.bak
    systemctl restart bluetooth

Make sure that you replace 00:11:22:33:44:55 with your bluetooth device
address, which should be in the directory `/var/lib/bluetooth/`.
