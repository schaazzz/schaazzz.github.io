---
layout: post
title: Configuring Your Arch Linux VM for Development
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development python ruby ssh
---

sudo ln -s /usr/bin/python2 /usr/bin/python
wget https://bootstrap.pypa.io/get-pip.py  
sudo python /mnt/schaazzz/projects/sandbox.local/get-pip.py  
sudo python -m pip install <package>  

sudo pacman -S ruby
~/.bashrc
.........+  PATH="$(ruby -e 'print Gem.user_dir')/bin:$PATH)"
.........+  export GEM_HOME=$(ruby -e 'print Gem.user_dir')
source ~/.bashrc

note: /etc/environment is a better option: works with ssh as well

... for SSH:
sudo pacman -S openssh
nano /etc/ssh/sshd_config => enable "AddressFamily any"
sudo systemctl enable sshd.service
