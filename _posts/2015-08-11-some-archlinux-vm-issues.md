---
layout: post
title: Some Arch Linux VM Issues & Their Solutions
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development bash script
---

Some recurring issues I've had with VirtualBox and Arch Linux and their solutions. [This bash script](https://github.com/schaazzz/script_magic/blob/master/reset_settings.sh) implements all these solutions. <!--more-->

## VM's Date/Time Out of Sync

Quite often when the Arch Linux VM is started from a previously saved machine state or the VM resumes after the Windows host machine wakes-up from sleep, the VM's time doesn't synchronize with the current host time. I tried setting the timesync threshold as follows but that didn't help.

<pre>
VBoxManage  guestproperty  set  {VM name}  /VirtualBox/GuestAdd/VBoxService/timesync-set-threshold  {threshold}
</pre>

I didn't invest too much time into investigating the root cause of this problem and the easiest solution I found was to use __ntpd__ (Network Time Protocol daemon).

- Install the __ntp__ package: `sudo pacman -S ntp`
- Run `sudo  ntpd  -qg` to manually synchronize time and date with the network
- `sudo  hwclock  --systohc` will set the VM's hardware clock to the system clock set by __ntpd__

## Network Issues After Switching to a Different Wi-Fi Connection

Sometimes, when you change your Wi-Fi connection (e.g. work Wi-Fi to home Wi-Fi), the network/internet doesn't work.

The following seems to fix this:

<pre>
$ sudo  systemctl  restart  systemd-networkd    # Restart the network daemon
$ sudo  resolvconf  -u                          # Regenerate DNS configuration & inform all subscribers
</pre>

## LXQT Desktop Doesn't Resize if the VM's Window Size Changes

The Window Manager (OpenBox in my case) resizes and the panel moves to the bottom of the screen but it looks like the wallpaper doesn't resize, where infact, it is the LXQT desktop module that hasn't resized. This failure happens in spite of the following:

- `vboxvideo` and `vboxguest` already loaded
- `VBoxClient-all` running
- VirtualBox's 'Auto-resize Guest Display' option clicked

Restarting the LXQT desktop module fixes this. You can do this from the __'LXQT settings -> Session Settings'__ window but I prefer the command line version:

<pre>
$ sudo  pcmanfm-qt  --desktop-off  --profile  lxqt      # Turn off the desktop manager
$ sudo  pcmanfm-qt  --desktop  --profile  lxqt      # Turn off the desktop manager
</pre>
