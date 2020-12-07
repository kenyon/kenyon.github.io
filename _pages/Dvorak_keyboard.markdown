---
title: Dvorak keyboard
---
Notes on how to switch between Dvorak and QWERTY keyboard layouts in various environments.

Also notes on my progress toward becoming a proficient Dvorak touch typist.

1. ToC
{:toc}

## Mac OS X

1. System Preferences -> International -> Input Menu.
1. Check Dvorak and US Extended.
1. Check "Show input menu in menu bar".
1. Set keyboard shortcut to switch between the two to Option-Command-Space.

## Xorg

Temporarily switch in Xorg:

* To Dvorak: `setxkbmap dvorak`
* To QWERTY: `setxkbmap us`

## Linux

### Debian, Ubuntu

Permanent switch in Xorg: `sudo dpkg-reconfigure xserver-xorg`

and say dvorak for keyboard layout.

Some HAL thing involving `cp /usr/share/hal/fdi/policy/10osvendor/10-keymap.fdi /etc/hal/fdi/policy/`(?) <http://jeromeandrieux.blogspot.com/2009/02/setup-dvorak-on-debian-with-xorg.html>

Console: `sudo dpkg-reconfigure console-setup`

Linux kernel keymap: `sudo dpkg-reconfigure console-data`

## FreeBSD

* Same as Xorg.
* 'sysinstall'
* `kbdmap`
* `kbdcontrol -l "us.dvorak"`
* <http://www.bobulous.org.uk/misc/usingDvorak.html>
* For the console, in `/etc/rc.conf`: `keymap="us.dvorak"`
* On FreeBSD 8: `sudo cp -vi /usr/local/share/hal/fdi/policy/10osvendor/10-x11-input.fdi /usr/local/etc/hal/fdi/policy ;` edit `/usr/local/etc/hal/fdi/policy/10-x11-input.fdi` and add `<merge key="input.xkb.layout" type="string">dvorak</merge>`.

## Solaris

* Same as Xorg. (?)
* `xmodmap /usr/share/xmodmap/xmodmap.dvorak`

## Progress

* 20090228: I started my Dvorak quest this week. I'm averaging 6 wpm in gtypist, and probably slower in general. After scratching off my keyboard's labels, I realized I don't properly touch type QWERTY, but I could type 80 wpm. I should try to fix my QWERTY technique too.
* 20090229: Up to 10 wpm in gtypist.
* 20090308: Up to 17 wpm in gtypist.
* 20090411: Still not as fast as I used to be in QWERTY, but now I can't touch type at all in QWERTY. I'm all Dvorak.
* 20100315: Probably faster in Dvorak than I was in QWERTY.

## See also

* [GNU Typist (gtypist)](http://www.gnu.org/software/gtypist/)
