---
layout: post
title: Installing Arch Linux on VirtualBox
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development
---

Arch Linux has to be installed manually since the official [Arch Linux ISO](https://www.archlinux.org/download/) doesn't contain an installer, additionally, no packages are included in the ISO, so a working internet connection is required during installation. <!--more-->

Most of the information below can be found in the [official installation guide](https://wiki.archlinux.org/index.php/Installation_guide) but some other information requires some Googling and a little bit of digging around on the Arch Linux wiki.

Virtual Machine Configuration
---

Create a new virtual machine in VirtualBox for Arch Linux (sample configuration shown below). Once the VM is created add the Arch Linux ISO to the CD/DVD drive.

![Virtual Machine Configuration]({{ site.baseurl }}/public/images/vm_config.png){: .center-image }  

Configuration of the highlighted settings in the screenshot below are important and the rest are optional. Arch Linux can run on a pretty low memory configuration so you can set the size of the display and base memories according to your preferences.  

For storage, I configured a "dynamically allocated" 8.0 GB hard drive image since I wanted to upload the resulting VMs to Dropbox, but I would recommend using a "fixed size" hard drive if you have space to spare on your host machine.  

The network settings are important for installation of Arch Linux. At a minimum, you need to bridge your host machine's network interface (which is connected to the internet) with your VM as show in the following image:  

![Virtual Machine Configuration - Network]({{ site.baseurl }}/public/images/vm_config_network.png){: .center-image }  

In case you would like any folders from your host machine to be accessible in your VM, the simplest way is to add them to the "Machine Folders" section under "Shared Folders" but do not selec the "Auto-mount" option otherwise they will only be accessible as root.

Installing Arch Linux
---

Now that the VM has been created with the ISO already _inserted_, start the virtual machine <del>and login with user ID and password "root"</del>.  

<custom0>Partitioning the VM's Hard Drive</custom0>  

You can [partition the hard drive](https://wiki.archlinux.org/index.php/Partitioning) using either a [single root partition](https://wiki.archlinux.org/index.php/Partitioning#Single_root_partition) or [discrete partitions](https://wiki.archlinux.org/index.php/Partitioning#Discrete_partitions). I've chosen a mixture of the 2 partitioning schemes, 200MB as /boot for installing GRUB, 1024MB swap file and the rest allocated as a single root partition.  

- Let's use the `lsblk` command to take a look at all the available storage devices:<br>

<pre>
root@archiso ~ # lsblk
NAME             MAJ:MIN RM    SIZE  RO  TYPE  MOUNTPOINT
sda                8:0    0      8G   0  disk
sr0               11:0    1    637M   0  rom   /run/archiso/bootmnt
loop0              7:0    0  278.2M   1  loop  /run/archiso/sfs/airootfs
loop1              7:1    0     32G   1  loop
└─arch_airootfs  254:0    0     32G   0  dm    /
loop2              7:2    0    256M   0  loop
└─arch_airootfs  254:0    0     32G   0  dm    /  
</pre>

- As you can seen in the example above, in my case, _"sda"_ was the VM's hard drive image. We can find out a bit more about "sda" using `parted  /dev/sda  print`:<br>  

<pre>
root@archiso ~ # parted  /dev/sda  print
Error:  /dev/sda:  unrecognised  disk  label
Model:  ATA  VBOX HHRDDISK  (scsi)
Disk  /dev/sdai  8590MB
Sector  size  (logical/physical):  512B/512B
Partition  Table:  unknown
Disk Flags:
</pre>

- The following set of `parted` commands will create the desired partitions (200MB as /boot for installing GRUB, 1024MB swap file and the rest allocated as a single root partition):

<pre>
root@archiso ~ # parted  /dev/sda  
(parted)  mklabel  msdos  
(parted)  mkpart  primary  ext4  1MiB  200MiB  
(parted)  set  1  boot  on  
(parted)  mkpart  primary  linux-swap  210MiB  1177MiB  
(parted)  mkpart  primary  ext4  1024MiB  100%  
(parted)  print
Model:  ATA  VBOX  HARDDISK  (scsi)
Disk  /dev/sda:  8590MB
Sector  size  (logical/physical):  512B/512B
Partition  Table:  msdos
Disk  Flags:

Number   Start    End      Size     Type      File system   Flags
 1       1049kB   210MB    209MB    primary                 boot
 2       220MB    1234MB   1014MB   primary
 3       1234MB   8590MB   7356MB   primary
</pre>

- Using `lsblk` again, we can see the new partition table, note that _"sda"_ now has 3 subentries: _"sda1"_ (/boot), _"sda2"_ (swap) and _"sda3"_ (root partition).

<pre>
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda               8:0    0     8G  0 disk
├─sda1            8:1    0   199M  0 part
├─sda2            8:2    0   967M  0 part
└─sda3            8:3    0   6.9G  0 part
sr0              11:0    1   637M  0 rom  /run/archiso/bootmnt
loop0             7:0    0 278.2M  1 loop /run/archiso/sfs/airootfs
loop1             7:1    0    32G  1 loop
└─arch_airootfs 254:0    0    32G  0 dm   /
loop2             7:2    0   256M  0 loop
└─arch_airootfs 254:0    0    32G  0 dm   /
</pre>

- Let's format the boot and root partitions to Ext4. For this we will use the `mkfs.ext4` command:

<pre>
root@archiso ~ # mkfs.ext4  /dev/sda1  
root@archiso ~ # mkfs.ext4  /dev/sda3  
</pre>

- Now we'll format and activate the swap partition. `mkswap` will format _"/dev/sda2"_ as swap and `swapon` will tell the kernel that this is our swap partition.

<pre>
root@archiso ~ # mkswap  /dev/sda2  
root@archiso ~ # swapon  /dev/sda2  
</pre>

<custom0>Installing the Base System</custom0>  

Mount the root and boot partitions:

<pre>
root@archiso ~ # mount  /dev/sda3  /mnt  
root@archiso ~ # mkdir  -p  /mnt/boot  
root@archiso ~ # mount  /dev/sda1  /mnt/boot  
</pre>

Install the base system using `pacstrap`:  

<pre>
root@archiso ~ # pacstrap  -i  /mnt  base  base-devel  
</pre>

And finally, generate _"fstab"_:

<pre>
root@archiso ~ # genfstab  -U  -p  /mnt  >>  /mnt/etc/fstab  
</pre>

<custom0>Chroot into Your Newly Installed Base System</custom0>  

Since our Arch Linux system is now installed we can _chroot_ into and configure it...

<pre>
root@archiso ~ # arch-chroot  /mnt  /bin/bash  
</pre>

Set the root password...  

<pre>
root@archiso ~ # passwd
</pre>

If you want, you can now create a user (recommended):

<pre>
root@archiso ~ # useradd  <username>  
root@archiso ~ # passwd  <username>
</pre>

I would also recommend adding the user you created above to the SUDOers list by using the `visudo` command to open _sudo's_ configuration file. First we'll have to install _Vim_ using `pacman -S vim`. Once _Vim_ is installed, search for "## User privilege specification" and below the configuration for _root_ add your username: `<username> ALL = (ALL) ALL`

Download, install and configure grub...

<pre>
root@archiso ~ # pacman  -S  grub  os-prober  
root@archiso ~ # grub-install  --recheck  /dev/sda  
</pre>
root@archiso ~ # grub-mkconfig  -o  /boot/grub/grub.cfg  

Almost, done --- exit from _chroot_, unmount the mounted partitions and reboot (or shutdown). You can also remove the Arch Linux ISO from the VM's CD/DVD drive...

<pre>
root@archiso ~ # exit  
root@archiso ~ # umount  -R  /mnt  
root@archiso ~ # reboot  
</pre>

<custom0>Network Configuration</custom0>

Once you restart your VM and boot into your shiny new Arch Linux installation, you'll realize that the network doesn't work. Even though the network out-of-the-box in the installer, for some reason, it has to be configured from scratch now :angry:. I find this _feature_ <del>very strange</del> extremely annoying and I've seen this in other distros as well, even in some with fancy Live CDs. I wonder why the network configuration is not copied during installation or is it a step that I've missed during installation?

The following network configuration is based on the VM's network configuration shown above. `enp0s3` is the bridged network adapter and `ensp0s8` is the host only network. You need to create network files for both interfaces, e.g. `sudo nano /etc/systemd/network/enp0s3.network` and `sudo nano /etc/systemd/network/enp0s8.network`. The contents of these files for my installation are shown below:

<pre>
root@archiso ~ # cat  /etc/systemd/network/enp0s3.network  
</pre>

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
