---
title: Ubuntu
---
Some of my notes on using Ubuntu Linux.

1. ToC
{:toc}

## Disabling touchpad tapping

For some annoying reason, it's very difficult to disable touchpad tap clicking, which is on by
default. TODO: Figure out how to disable this. Maybe try making a karmic package of
[kcm_touchpad](https://www.linux-apps.com/p/1127859).

## Right Alt produces wrong keysym, ISO_Level3_Shift

I had this problem on my [Debian]({% link _pages/Debian.markdown %}) machine too. It should
produce Alt_R. The solution is to run `sudo dpkg-reconfigure console-setup` and select No AltGr
key. I suspect this is related to selecting Dvorak as my keyboard layout.
