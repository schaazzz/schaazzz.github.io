---
layout: post
title: Download Arch Linux VMs
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development
---

I have uploaded 3 Arch Linux Virtual Machines to Dropbox. One without any desktop environment (command line only), another with LXQT and a third with XFCE. You will need to create a new user since these VMs only contain the default "root" account with password "root".

Download Links
---
- **[Arch Linux VM - No Desktop Environment](https://dl.dropboxusercontent.com/u/102452388/schaazzz.archlinux.64.vm.no.desktop.7z)**  
- **[Arch Linux VM - XFCE](https://dl.dropboxusercontent.com/u/102452388/schaazzz.archlinux.64.vm.xfce.7z)**  
_Login as root from the XFCE login prompt, open a command prompt and create a new user._  
- **[Arch Linux VM - LXQT](https://dl.dropboxusercontent.com/u/102452388/schaazzz.archlinux.64.vm.lxqt.7z)**  
_The login prompt you'll see doesn't allow root login by default, switch to a terminal using 'Ctrl+Alt+F2' and login as root and create a new user._  

Creating a New User:
---

<pre>
$ useradd [username]
$ passwd [username]
$ mkdir /home/[username]
$ chown -R [username]:[username] /home/[username]
</pre>

...for example:

<pre>
$ useradd tux
$ passwd tux
$ mkdir /home/tux
$ chown -R tux:tux /home/tux
</pre>

Adding a User to SUDOers
---

- I would also recommend adding the user you created above to the SUDOers list by using the `visudo` command to open _sudo's_ configuration file.
- Search for _"## User privilege specification"_ and below the configuration for _root_ add your username: `[username] ALL = (ALL) ALL`
