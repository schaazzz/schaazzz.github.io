---
layout: post
title: Setting Linux Environment Variables, Bash Init Scripts and Loading/Execution Order
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux development bash script etc environment pam
---

This post serves as a reference for various Linux startup, login and init scripts, environment variables and the order in which they are loaded or executed.

I use Arch Linux with Bash, but the general concepts should be applicable to any distro or shell. I also use Bash as my default login shell. You can confirm which login shell you're running (sh or bash) by using `echo $SHELL`. The default user shell can be changed using the `chsh` command. <!--more-->

## /etc/environment

- Loaded  by Linux PAM (Pluggable Authentication Modules)
- More specifically by __pam_env.so__ which is loaded by login shells as well as the Display Manager (for my setup - LXQT with SDDM - it is loaded by __/etc/pam.d/sddm-greeter__)
- Is NOT a shell script, i.e. existing variables or bash script commands can't be used (e.g. `echo $VAR` will not work)
- A good place for setting system wide environment variables --> also works for SSH
- Environment variables can be assigned, one per line, as: `FOO = Hi, I'm a variable & my name is FOO`
- But if Bash commands need to be executed (e.g. setting terminal colors or modification of `$PATH`), __/etc/bash.bashrc__ or __/etc/profile__ (login shells only) are good options for doing it for all users. For user specific script commands __~/.bashrc__ can be used for non-login shells or either of __.bash_profile__, __.bash_login__ or __.profile__ for login shells.

## ~/.bashrc

- Bash executes commands in __~/.bashrc__ (if it exists) for interactive non-login shells (i.e. it is not executed when using SSH)
- Good for user specific environment variables and commands when Bash is running locally

## $BASH_ENV

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

~/.bash_profile, ~/.bash_login, ~/.profile
---

For login shells, Bash searches for & executes whichever script it finds in the following order of priority:

- __.bash_profile__
- __.bash_login__
- __.profile__

## /etc/profile

- Executed by login shells and the Display Manager
- Sources __/etc/bash.bashrc__ and all *.sh<!--*--> files in the __/etc/profile.d/__ folder
- For login shells, __/etc/profile__ can be used for executing Bash commands or setting system-wide environment variables. But directly editing __etc/profile__ is generally not recommended as it is a configuration file for system packages; it is a better idea to create a new script with an .sh extension in the __/etc/profile.d/__ folder

## /etc/bash.bashrc

- Loaded by all interactive (login AND non-login shells)

## Execution/Loading Order

- __/etc/environment__: Environment variables added here are always available in all types of sessions

### Interactive/Non-login (e.g. local terminal):

- __/etc/bash.bashrc__
- __~/.bashrc__

### Interactive/Login (e.g. virtual terminals opened using Ctrl-Alt-Fn or `tmux`):

- __/etc/profile__
- __/etc/profile.d/*.sh__
- __/etc/bash.bashrc__
- __~/.bash_profile__: or ~/.bash_login or ~/.profile (whichever you prefer)

### For SSH:

- __$BASH_ENV__
- __/etc/profile__
- __/etc/profile.d/*.sh__
- __/etc/bash.bashrc__
- __~/.bash_profile__: or ~/.bash_login or ~/.profile (whichever you prefer)

## References

1. [https://help.ubuntu.com/community/EnvironmentVariables#System-wide_environment_variables](https://help.ubuntu.com/community/EnvironmentVariables#System-wide_environment_variables)
2. [https://wiki.archlinux.org/index.php/Bash](https://wiki.archlinux.org/index.php/Bash)
3. [https://bugs.launchpad.net/ubuntu/+source/manpages/+bug/18574/comments/13](https://bugs.launchpad.net/ubuntu/+source/manpages/+bug/18574/comments/13)
4. [http://askubuntu.com/questions/98433/run-a-script-on-login-using-bash-login](http://askubuntu.com/questions/98433/run-a-script-on-login-using-bash-login)
5. [http://linux.die.net/man/8/pam_env](http://linux.die.net/man/8/pam_env)
6. [http://serverfault.com/questions/593472/where-is-bash-env-usually-set](http://serverfault.com/questions/593472/where-is-bash-env-usually-set)
7. [http://stackoverflow.com/questions/3327013/how-to-determine-the-current-shell-im-working-on](http://stackoverflow.com/questions/3327013/how-to-determine-the-current-shell-im-working-on)
8. [http://www.chainsawonatireswing.com/2012/02/02/find-out-what-your-unix-shells-flags-are-then-change-them//?from=@#fn:curly-brace-expansion](http://www.chainsawonatireswing.com/2012/02/02/find-out-what-your-unix-shells-flags-are-then-change-them//?from=@#fn:curly-brace-expansion)
9. [http://capistranorb.com/documentation/faq/why-does-something-work-in-my-ssh-session-but-not-in-capistrano/](http://capistranorb.com/documentation/faq/why-does-something-work-in-my-ssh-session-but-not-in-capistrano/)
