---
layout: post
title: Installing Arch Linux on VirtualBox
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development
---

Arch Linux has to be installed manually since the official [Arch Linux ISO](https://www.archlinux.org/download/) doesn't contain an installer, additionally, no packages are included in the ISO, so a working internet connection is required during installation. <!--more-->

Most of the information below can be found in the [official installation guide](https://wiki.archlinux.org/index.php/Installation_guide) but some other information requires some Googling and a little bit of digging around on the Arch Linux wiki.

<custom0>Virtual Machine Configuration</custom0>  

Configuration of the highlighted settings in the screenshot below are important and the rest are optional. Arch Linux can run on a pretty low memory configuration so you can set the size of the display and base memories according to your preferences.  

For storage, I configured a "dynamically allocated" 8.0 GB hard drive image since I wanted to upload the resulting VMs to Dropbox, but I would recommend using a "fixed size" hard drive if you have space to spare on your host machine.

![Virtual Machine Configuration]({{ site.baseurl }}/public/images/vm_config.png){: .center-image }  

The network settings are important for installation of Arch Linux. At a minimum, you need to bridge your host machine's network interface (which is connected to the internet) with your VM as show in the following image:  

![Virtual Machine Configuration - Network]({{ site.baseurl }}/public/images/vm_config_network.png){: .center-image }  

In case you would like any folders from your host machine to be accessible in your VM, the simplest way is to add them to the "Machine Folders" section under "Shared Folders" but do not selec the "Auto-mount" option otherwise they will only be accessible as root.

--------------------

...
... lsblk  
!['lsblk']({{ site.baseurl }}/public/images/lsblk_output.jpg){: .center-image }  
... parted /dev/sda print  
!['lsblk']({{ site.baseurl }}/public/images/parted_print_output.jpg){: .center-image }  
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
... pacman -Syu

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
