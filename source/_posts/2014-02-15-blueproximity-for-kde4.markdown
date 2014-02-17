---
layout: post
title: "Blueproximity for KDE4 Configuration"
date: 2014-02-15 14:35
comments: true
categories:
- Linux
- Bluetooth
- KDE
---
BlueProximity is a great addition to Linux, it adds a little of extra security
for you Linux machine. For those who don't know what it is, here's an excerpt
from BlueProximity:

{% blockquote BlueProximity http://blueproximity.sourceforge.net %}
This software helps you add a little more security to your desktop. It does so
by detecting one of your bluetooth devices, most likely your mobile phone, and
keeping track of its distance. If you move away from your computer and the
distance is above a certain level (no measurement in meters is possible) for a
given time, it automatically locks your desktop (or starts any other shell
command you want).

Once away your computer awaits its master back - if you are nearer than a given
level for a set time your computer unlocks magically without any interaction
(or starts any other shell command you want).
{% endblockquote %}
<!-- more -->

The problem I found when configuring BlueProximity on my Linux machine is the
commands, if you are on Gnome, nearly no configuration is needed, but I am on
KDE 4.x. So, here's the command I used to get it working on my machine:

- Lock command: `qdbus-qt4 org.kde.screensaver /ScreenSaver Lock`
- Unlock command: `qdbus-qt4 | grep kscreenlocker_greet | xargs -I {} qdbus-qt4 {} /MainApplication quit`
- Proximity command: `qdbus-qt4 org.freedesktop.ScreenSaver /ScreenSaver SimulateUserActivity`

If you are not using `qdbus-qt4`, substitute it with `qdbus`.
