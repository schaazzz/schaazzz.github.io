---
layout: post
title: Setting Linux Environment Variables, Bash Init Scripts and Loading/Execution Order
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development bash script etc environment pam
---

This post serves as a reference for various Linux startup/login/init scripts, environment variables and the order in which they are loaded/executed.

I use Arch Linux with Bash, but the general concepts should be applicable to any distro or shell. I also use Bash as my default login shell. You can confirm which login shell you're running (sh or bash) by using `echo $SHELL`. The default user shell can be changed using the `chsh` command. <!--more-->

/etc/environment
---

- Loaded  by Linux PAM (Pluggable Authentication Modules)
- More specifically by __pam_env.so__ (called by __/etc/pam.d/sddm-greeter__)
- Is NOT a shell script, i.e:existing variables or bash script commands can't be used (e.g. `echo $VAR` will not work)
- Only contains assignments, one per line: `FOO = Hi, I'm a variable & my name is FOO`
- A good place for setting system wide environment variables --> also works for SSH

~/.bashrc
---

- Bash executes commands in __~/.bashrc__ (if it exists) for interactive non-login shells (i.e. it is not executed when using SSH)
- Good for user specific environment variables and commands when Bash is running locally

$BASH_ENV
---

- For non-interactive shells, if __$BASH_ENV__ is set, Bash executes the script specified in __$BASH_ENV__
- I usually set __$BASH_ENV__ in __/etc/environment__ to __~/.bash_env__, this way, each user can have their own __.bash_env__

__Note:__ SSH is an interactive login shell BUT it _starts_ non-interactively. This is how I confirmed this:

- If any `echo` statements present in my __$BASH_ENV__ script are executed while logging in via SSH, then this is proof enough that Bash is executing the __$BASH_ENV__ script, which is only done for non-interactive shells. But more specifically, I used `echo $-`.
- `$-` is a special environment variable containing current shell options, if it contains `i`, it is an interactive shell:

<pre>
$ echo $-
himBH
</pre>

- If you add `echo $-` to your __$BASH_ENV__ file, logging in via SSH will show something like `hBc` confirming script execution and its non-interactive nature. All other scripts are executed in interactive mode.

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

<pre>
# SSH loading order
.bash_profile
.bash_login
.profile
</pre>

<hr>

SSH:

<pre>
C:\t\m\pyconti_portable_python_full> ssh schaazzz@192.168.56.101
schaazzz@192.168.56.101's password:
Last login: Sat Aug 15 01:45:06 2015 from 192.168.56.1
LOG: Sourcing /home/schaazzz/.bash_env
LOG: Sourcing /etc/profile
LOG: Sourcing /etc/bash.bashrc
LOG: Sourcing /home/schaazzz/.bash_profile
schaazzz@archlinuxvm:~/$ logout
Connection to 192.168.56.101 closed.

C:\t\m\pyconti_portable_python_full> ssh root@192.168.56.101
root@192.168.56.101's password:
Last login: Sat Aug 15 02:36:57 2015 from 192.168.56.1
LOG: Sourcing /root/.bash_env
LOG: Sourcing /etc/profile
LOG: Sourcing /etc/bash.bashrc
LOG: Sourcing /root/.bash_profile
root@archlinuxvm:~/$
</pre>

Interactive:

<pre>
LOG: Sourcing /etc/bash.bashrc
LOG: Sourcing /home/schaazzz/.bashrc
</pre>


References
---

1. [https://help.ubuntu.com/community/EnvironmentVariables#System-wide_environment_variables](https://help.ubuntu.com/community/EnvironmentVariables#System-wide_environment_variables)
2. [https://bugs.launchpad.net/ubuntu/+source/manpages/+bug/18574/comments/13](https://bugs.launchpad.net/ubuntu/+source/manpages/+bug/18574/comments/13)
3. [http://askubuntu.com/questions/98433/run-a-script-on-login-using-bash-login](http://askubuntu.com/questions/98433/run-a-script-on-login-using-bash-login)
4. [http://linux.die.net/man/8/pam_env](http://linux.die.net/man/8/pam_env)
5. [http://serverfault.com/questions/593472/where-is-bash-env-usually-set](http://serverfault.com/questions/593472/where-is-bash-env-usually-set)
