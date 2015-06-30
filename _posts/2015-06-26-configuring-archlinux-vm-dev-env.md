---
layout: post
title: Configuring Your Arch Linux VM for Development
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development python ruby ssh
---

I am stuck on a Windows machine most of the time, but I prefer Linux. Also, Ruby on Rails on Windows is a real PITA. So, I configure a VM with all the necessary development tools and share my project/work folders with the VM and mount it using `mount.vboxsf`. Then I run the VM in headless mode and SSH into it. <!--more-->

Installing Python
---

- If Python is not already installed, install it using `$ pacman -S python2` (replace with python3 if you need Python 3).  
- Create a _python_ link for _python2_:

<pre>
$ sudo  ln  -s  /usr/bin/python2  /usr/bin/python
</pre>

- If PIP is not installed:

<pre>
$ wget  https://bootstrap.pypa.io/get-pip.py  
$ sudo  python  get-pip.py  
</pre>

- PIP can be invoked in one of the following ways:

<pre>
$ sudo  python  -m  pip  install  [package]  
$ sudo  pip  [package]
</pre>

Installing Ruby, Rails and Node.js:
---

<pre>
$ sudo  pacman  -S  ruby`  
$ gem  install  rails
$ gem  install  bundler
$ gem  update
$ sudo  install  nodejs         # Installing Node.js also fixes JavaScript issues while running Jekyll
</pre>

- To `~/.bashrc` add the following lines and use the `$ source ~/.bashrc` command to update the environment with the new values:

<pre>
PATH="$(ruby -e 'print Gem.user_dir')/bin:$PATH)"
export GEM_HOME=$(ruby -e 'print Gem.user_dir')
</pre>

- If you plan to SSH to this machine, the `/etc/environment` file is a better option than `~/.bashrc`:

<pre>
PATH="/home/schaazzz/.gem/ruby/2.2.0/bin":$PATH
GEM_HOME="/home/schaazzz/.gem/ruby/2.2.0"
</pre>

Installing and Configuring SSH:
---

<pre>
$ sudo  pacman  -S  openssh
$ nano  /etc/ssh/sshd_config         # Enable "AddressFamily any"
$ sudo  systemctl  enable  sshd.service
</pre>
