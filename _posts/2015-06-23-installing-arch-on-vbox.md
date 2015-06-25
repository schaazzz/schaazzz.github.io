---
layout: post
title: Installing Arch Linux on VirtualBox
author: Shahzeb Ihsan
tags: virtualbox, virtualmachine, archlinux, linux, development
---

No installer bundled with the Arch Linux ISO, has to be installed manually. <!--more-->

--------------------

... no packages included, working internet connection required during installation  
... lsblk  
!['lsblk']({{ site.baseurl }}/public/images/lsblk_output.jpg)  
... parted /dev/sda print  
!['lsblk']({{ site.baseurl }}/public/images/parted_print_output.jpg)  
... parted /dev/sda  
... (parted) mklabel msdos  
... mkpart primary ext4 1MiB 200MiB  
... set 1 boot on  
... mkpart primary linux-swap 210MiB 1177MiB  
... mkpart primary ext4 1024MiB 100%  
... mkfs.ext4 /dev/sda1  
... mkfs.ext4 /dev/sda3  
... Activate swap  
...... mkswap /dev/sda2  
...... swapon /dev/sda2  
... mount /dev/sda3 /mnt  
... mkdir -p /mnt/boot  
... mount /dev/sda1 /mnt/boot  
... Install the base system  
...... pacstrap -i /mnt base base-devel  
... genfstab -U -p /mnt >> /mnt/etc/fstab  

--------------------

**chroot into your newly installed base system**  
...... arch-chroot /mnt /bin/bash  
... passwd  
... pacman -S grub os-prober  
... grub-install --recheck /dev/sda  
... grub-mkconfig -o /boot/grub/grub.cfg  
... exit  
... umount -R /mnt  
... reboot  

--------------------

**Network Configuration**  
... nano /etc/systemd/enp0s3.network  
... nano /etc/systemd/enp0s8.network  
... nano /etc/resolv.conf  
... systemctl enable dhcpcd@enp0s3.service  
... systemctl enable dhcpcd@enp0s8.service  
... systemctl restart systemd-networkd  
... systemctl status systemd-networkd  
... networkctl  

---------------------

**Installing VirtualBox Guest Addition**  
... pacman -S virtualbox-guest-modules virtualbox-guest-utils  
... modprobe -a vboxguest vboxsf vboxvideo  

---------------------

**Installing LXQT**  
... pacman -S lxqt  
... pacman -S openbox  
... pacman -S sddm  

---------------------

-------------> ???  
pacman -S lxqt openbox obconf xf86-video-fbdev xorg-server xorg-server-utils xorg-xinit mesa xf86-input-keyboard xf86-input-mouse xterm ttf-dejavu libxkbcommon libxkbcommon-x11
Then nano .xinitrc and type "exec startlxqt" save
startx and it's working.  
<-------------  

--------------------

... nano /etc/X11/xinit/xinitrc ==> add "exec startlxqt"  
... cp /etc/X11/xinit/xinitrc ~/.xinitrc  
... sddm --example-config > /etc/sddm.conf  
... systemctl enable sddm.service  

--------------------

**Installing XFCE**  
... pacman -S xfce4 xorg lxdm  
... systemctl enable lxdm.service  

--------------------

**Add user**  
adduser <username>  
passwd <username>  
visudo --> search for "# User privilege specification" and add "<username> ALL = (ALL) ALL"  
