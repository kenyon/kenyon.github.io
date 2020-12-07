---
title: Terminal setup
---
Here I'll describe my terminal and shell setup. See also [Mac OS X Terminal.app]({% link _pages/Mac_OS_X_Terminal.app.markdown %}) for info specific to my Mac OS X setup.

1. ToC
{:toc}

## Terminal emulator

The never-ending quest for the perfect terminal emulator.

Current best terminal: konsole, the version that comes with KDE 4 and later.

I tried mrxvt. Before mrxvt, I tried gnome-terminal. I didn't like its blinking cursor,
dependence on GNOME libraries (since I don't use much GNOME stuff), and how you can't wrap around
when switching tabs. For a long time before that, I used konsole. When I switched to Debian sid,
I had to use an older version of konsole which didn't have the ability to right-click on URLs and
launch them. Unfortunately, mrxvt doesn't have that either. Also, mrxvt doesn't support Unicode,
so manual pages look messed up in a UTF-8 locale (e.g., `LANG=en_US.UTF-8`).

rxvt-unicode supports Unicode, tabs, and URL launching. However, <https://bugs.debian.org/511377>
is quite annoying: you can't scroll using the keyboard when the mouse pointer is outside the
terminal window with tabbed enabled. rxvt-unicode has a cool fading feature, where the text can
fade when the terminal goes out of focus.

xfce4-terminal is decent. It does URL launching and tabs, and the tab switching wraps around. You
can't rearrange the tabs with the keyboard, but you can with the mouse. It's based on the same
terminal widget as gnome-terminal, so it has the same problem where it eats CPU time if you hold
down a cursor movement key in vim and the cursor is against a screen edge. It's also slow
graphically&mdash;it takes a lot of CPU to scroll, and it has a noticeable refresh delay when
switching among desktops containing xfce4-terminal windows (although I see this with rxvt-unicode
too, so it is probably just my video card (Intel G31 integrated on a Supermicro C2SBA+)). Also,
xfce4-terminal does not hide the mouse cursor when you're typing in the terminal, and doesn't
seem to have a way to configure it to do this.

Bug causing backspace to fail in screen under xfce4-terminal:

* <https://bugs.launchpad.net/bugs/29787>
* <http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=477455>

Command for setting the default browser for gnome-terminal and similar (as I reported in
<http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=487862#35>):

* `gconftool --type=string --set /desktop/gnome/url-handlers/http/command "x-www-browser %s"`

Plain xterm supports Unicode, but not URL launching or tabs.

Another idea is to use urxvt or xterm with GNU screen to get the same effect as tabbing.

### konsole

In the environment settings, I put `TERM=xterm-256color`. Verify: `tput colors`

Change the "open link" handlers with the KDE `systemsettings` program. Use the Default Applications and File Associations applets.

### mrxvtrc

Here is my ~/.mrxvtrc. I got this stuff by reading the mrxvt manual page and from [dotfiles.org](http://dotfiles.org/.mrxvtrc).

```conf
#Mrxvt.bottomTabbar: True
#Mrxvt.xftAntialias: 1 # antialiasing seems to be default on
Mrxvt.autohideTabbar: True
Mrxvt.background: black
Mrxvt.foreground: white
Mrxvt.hideButtons: True
Mrxvt.highlightTabOnBell: True
Mrxvt.loginShell: True
Mrxvt.pointerBlank: True
Mrxvt.saveLines: 10000
Mrxvt.sessionMgt: True
Mrxvt.syncTabTitle: True
Mrxvt.tabTitle: mrxvt
Mrxvt.xft: 1
Mrxvt.xftFont: DejaVu Sans Mono
Mrxvt.xftSize: 10
```

## screenrc

To enable 256 color support in screen, I used these settings, based on <http://www.frexx.de/xterm-256-notes/>.

```conf
startup_message off
deflogin on
altscreen on
shell -$SHELL
term screen-256color-bce

# set a big scrolling buffer
defscrollback 5000
# Set the caption on the bottom line
caption always "%{= kw}%-w%{= BW}%n %t%{-}%+w %-= @%H" # - %LD %d %LM - %c"

# terminfo and termcap for nice 256 color terminal
# allow bold colors - necessary for some reason
#attrcolor b ".I"
# tell screen how to set colors. AB = background, AF=foreground
#termcapinfo xterm 'Co#256:AB=\E[48;5;%dm:AF=\E[38;5;%dm'
# erase background with current bg color
defbce "on"
```

## tmux

I use tmux instead of screen now.

```conf
# Kenyon's tmux configuration.
# Created 2010-02-18.

set-option -g default-terminal screen-256color
set-option -g history-limit 6000
set-option -g status-left "[#H:#S]"
set-option -g status-left-length 25
set-option -g status-right ""
set-option -g visual-activity on
set-option -g visual-content on
```

## Shell

Since I started using Linux, I had been using bash. More recently (January 2009) I switched to zsh and I'm liking it.
