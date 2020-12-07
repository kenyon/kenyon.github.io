---
title: Mac OS X Terminal.app
---
Terminal.app on Mac OS X is a little weird. It has some default settings that don't work right, at least for me. I'm logging in to all sorts of machines from Terminal.app: Linux, FreeBSD, Solaris, other Macs. I'm also often using screen and tmux. It's taken a while to get settings that work fairly nicely everywhere.

Here are my settings, grouped by preference pane.

## Startup
* Shells open with default login shell (/usr/bin/login).

## Settings
### Text
* Font: DejaVu Sans Mono 10 pt.

### Window
* Empty title.
* Active process name, dimensions, command key: checked. The rest are unchecked.
* Limit number of scrollback rows to 10000.

### Shell
* Run command: unchecked.
* When the shell exits, close if the shell exited cleanly.
* Prompt before closing: never.

### Keyboard
I used info from [this Mac OS X Hint](http://www.macosxhints.com/article.php?story=20040401033846410) to rebind home, end, pageup, and pagedown. By default, shift home and shift end had some incorrect strings. I wanted to correct them so that I could use them in mutt or vim. I also wanted to scroll Terminal's buffer using shift-pageup and shift-pagedown, like every other terminal program.

So, the result is this:

* home: \033[1~
* end: \033[4~
* page down: \033[6~
* page up: \033[5~
* shift page up: scroll to previous page in buffer
* shift page down: scroll to next page in buffer
* shift home: scroll to start of buffer
* shift end: scroll to end of buffer

* Use option as meta key: checked.

### Advanced
#### Emulation
* Declare terminal as: xterm-color.
* Delete sends Ctrl-H: unchecked.
* Escape non-ASCII input: unchecked.
* Paste newlines as carriage returns: checked.
* Strict VT-100 keypad behavior: unchecked.
* Scroll to bottom on input: checked.

#### Bell
* Audible bell: unchecked.
* Visible bell: checked.

#### International
* Character encoding: Unicode (UTF-8).
* Set LANG environment variable on startup: checked.

## See also
* [[Terminal setup]] describes my more general terminal and shell setup.
