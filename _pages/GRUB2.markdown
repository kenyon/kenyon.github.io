---
title: GRUB2
---
With GRUB 2 on Debian and Ubuntu, I like to edit `/etc/default/grub` and set `GRUB_CMDLINE_LINUX=""` or `GRUB_CMDLINE_LINUX_DEFAULT=""`. I'm removing "quiet" and "splash" because I want to see all the boot messages.

Then run `sudo update-grub` to make the change take effect in `/boot/grub/grub.cfg`.
