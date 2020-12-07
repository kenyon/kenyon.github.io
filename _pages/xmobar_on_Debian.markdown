---
title: xmobar on Debian
---
* 20090105 update: I'm running [Debian]({% link _pages/Debian.markdown %}) on my main desktop machine now. Here's what I did this time. I was running Ubuntu, and the notes I wrote for that still exist below.
* 20100315 update: Probably none of these instructions are necessary anymore, since a recent version of xmobar is in the Debian repositories. Just `sudo aptitude install xmobar` should be sufficient.

How to install xmobar on Debian.

I already had zlib1g-dev and libxft-dev installed. zlib1g-dev would be pulled in by libghc6-zlib-dev anyway. Also already used aptitude to install xmonad, so that might have pulled in some dependencies necessary for the following steps.

1. ToC
{:toc}

## Debian instructions

### stm

1. `sudo aptitude install libghc6-stm-dev` # dependency of xmobar

### cabal

Since Debian sid doesn't have the Haskell X11-xft package, I want to install it with cabal-install, which Debian also does not have.

1. `darcs get --partial http://darcs.haskell.org/cabal-branches/cabal-1.6/` # (Per <http://www.haskell.org/cabal/code.html>.) Due to [cabal-install's](http://hackage.haskell.org/trac/hackage/wiki/CabalInstall) dependence on Cabal >=1.6 && <1.7, but Debian sid has Cabal-1.2.3.0 at the moment, which is part of ghc6.
1. `cd cabal-1.6`
1. `ghc --make Setup && ./Setup configure --user && ./Setup build && ./Setup install`
1. `cd ..`

### HTTP and zlib

HTTP and zlib are dependencies of cabal-install.

1. `sudo aptitude install libghc6-http-dev libghc6-zlib-dev` # pulls in libghc6-network-dev and libghc6-parsec-dev

Debian squeeze currently doesn't have the required version of libghc6-http-dev, so following <http://projects.haskell.org/http/> (or what used to be there):

1. `git clone git://code.galois.com/HTTPbis.git`
1. `runhaskell Setup configure --user && runhaskell Setup build && runhaskell Setup install`

### cabal-install

1. `darcs get --partial http://darcs.haskell.org/cabal-install/`
1. `cd cabal-install`
1. `runhaskell Setup configure --user && runhaskell Setup build && runhaskell Setup install`
1. `cd ..`

cabal binary is now installed as `~/.cabal/bin/cabal`. I've added `~/.cabal/bin` to my `PATH`.

### X11-xft

1. `cabal update`
1. `cabal install X11-xft`

### xmobar

1. `darcs get --partial http://code.haskell.org/xmobar/`
1. `cd xmobar`
1. `runhaskell Setup configure --user --flags="with_xft" && runhaskell Setup build && runhaskell Setup install`

Now the xmobar binary is installed as `~/.cabal/bin/xmobar`.

At first, `Setup configure` could not find X11-xft, even though ghc-pkg list showed X11-xft as being installed in my home directory. This was because I forgot the "`--user`" option to `Setup configure`.

### Notes

* PROTIP: After doing any `runhaskell Setup configure --user && runhaskell Setup build && runhaskell Setup install`, that package is registered under ~/.ghc and visible to ghc-pkg. If you `rm -rf ~/.cabal` without ghc-pkg unregistering the package, then subsequent builds will fail.
* runhaskell is equivalent to runghc. I suppose runhaskell is more generic, so that compilers other than ghc can be run.
* [Haskell Package Version Tracker](http://people.debian.org/~nomeata/hackagevsdebian.html) - Debian versus Hackage
* I had to add `lowerOnStart = True` to my xmobarrc to get it to work this time. Otherwise it's the same as shown at the bottom of this page.

----------

## Old Ubuntu instructions

(December 2008) I use [xmonad](http://xmonad.org/) on [Ubuntu]({% link _pages/Ubuntu.markdown %}). I use the xmobar status bar with it. Ubuntu doesn't have a package for xmobar, nor for many of its dependencies. So to build xmobar I had to build and install a bunch of dependencies. Also, in order to make installing Haskell packages easier, I installed cabal-install, which had its own dependencies.

### Dependencies with Ubuntu packages

First some dependencies can be installed using aptitude. These are libxft-dev (for xft support in xmobar), and zlib1g-dev for building cabal-install.

### Building cabal-install dependencies

Building parsec, stm, network, http, and zlib should now work with the command `runhaskell Setup configure --user && runhaskell Setup build && runhaskell Setup install` for each. Or you could use the bootstrap.sh included with cabal-install.

I think those are all of the dependencies.

### Building cabal-install

Now cabal-install should install into your home directory with the standard Haskell configure, build, and install commands:

```console
runhaskell Setup configure --user && runhaskell Setup build && runhaskell Setup install
```

### Installing X11-xft

With cabal installed, a simple `cabal install X11-xft` should work.

### Building xmobar

Finally, I was able to build xmobar. I'm using the version pulled directly from the darcs repository.

### xmobar.config

Here is my `~/.xmonad/xmobar.config`:

```haskell
Config { font = "-*-fixed-medium-r-normal-*-13-*-*-*-*-*-*-*"
        , bgColor = "black"
        , fgColor = "grey"
        , position = TopW L 90
        , commands = [ Run Weather "KNKX" ["-t","<station>: <tempC>C","-L","18","-H","25","--normal","green","--high","red","--low","lightblue"] 36000
                    , Run Network "eth0" ["-L","0","-H","32","--normal","green","--high","red"] 10
                    , Run Cpu ["-L","3","-H","50","--normal","green","--high","red"] 10
                    , Run Memory ["-t","Mem: <usedratio>%"] 10
                    , Run Swap [] 10
                    , Run Com "uname" ["-s","-r"] "" 36000
                    , Run Date "%a %b %_d %Y %H:%M:%S" "date" 10
                    , Run StdinReader
                    ]
        , sepChar = "%"
        , alignSep = "}{"
        , template = "%StdinReader% }{ %cpu% | %memory% * %swap% | %eth0% <fc=#ee9a00>%date%</fc>| %KNKX% | %uname%"
        }
```

### External links

* This blog post was helpful: <http://gimbo.org.uk/blog/2007/04/27/haskell-packages-gotcha-global-vs-per-user-package-databases/>
* Help make Ubuntu packages for Haskell stuff: <https://wiki.ubuntu.com/MOTU/Teams/UncommonProgrammingLanguages/Haskell>
* This thread helped figure out why zlib wasn't building: <http://www.nabble.com/zlib,-missing-zlib.h-td17547823.html>
