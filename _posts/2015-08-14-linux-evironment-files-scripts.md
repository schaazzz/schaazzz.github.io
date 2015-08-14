---
layout: post
title: Linux Startup/Login/Init Scripts, Environment Variables and Loading/Execution Order
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development bash script etc environment pam
---

This is a live post (so-to-speak) which serves as a reference for various Linux startup/login/init scripts, environment and the order in which they are loaded/executed. I will update/revise this post as my knowledge in this area increases. <!--more-->

/etc/environment
---

- Loaded  by Linux PAM (Pluggable Authentication Modules)
- More specifically by __pam_env.so__ (called by __/etc/pam.d/sddm-greeter__)
- Is NOT a shell script, i.e:existing variables or bash script commands can't be used (e.g. echo $VAR will not work)
- Only contains assignments, one pair line: `FOO = Hi, I'm a variable & my name is FOO`


~/.bashrc
---

~/.bash_env
---

~/.profile
---

~/.bash_profile
---

~/.bash_login
---

/etc/profile
---

/etc/bash.bashrc
---

Execution/Loading Order
---

References
---

1. [https://help.ubuntu.com/community/EnvironmentVariables#System-wide_environment_variables](https://help.ubuntu.com/community/EnvironmentVariables#System-wide_environment_variables)
2. [https://bugs.launchpad.net/ubuntu/+source/manpages/+bug/18574/comments/13](https://bugs.launchpad.net/ubuntu/+source/manpages/+bug/18574/comments/13)
3. [http://askubuntu.com/questions/98433/run-a-script-on-login-using-bash-login](http://askubuntu.com/questions/98433/run-a-script-on-login-using-bash-login)
4. [http://linux.die.net/man/8/pam_env](http://linux.die.net/man/8/pam_env)
5. [http://serverfault.com/questions/593472/where-is-bash-env-usually-set](http://serverfault.com/questions/593472/where-is-bash-env-usually-set)
