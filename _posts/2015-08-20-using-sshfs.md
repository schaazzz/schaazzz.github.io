---
layout: post
title: Using SSHFS
author: Shahzeb Ihsan
tags: virtualbox virtualmachine archlinux linux sshfs vboxsf git
---

A while back I had issues with `git clone` on Windows host machine's folders mounted using __vboxsf__. Cloning, pushing or committing would result in Git complaining that it couldn't lock the config file because permission was denied. I uselessly Googled for a solution for about a week until I discovered SSHFS.
