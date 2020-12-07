---
title: FreeBSD rebuild
---
Here is my condensed version of <http://www.freebsd.org/doc/en/books/handbook/makeworld.html>, or how to rebuild the FreeBSD world and kernel.

Once the `devel/subversion` port is installed, check out the source tree once:

```console
cd /usr && sudo svn co svn://svn.freebsd.org/base/stable/9/ src
```

Create a symlink to my custom kernel config, if necessary. For example:

```console
ln -s ~/MYCONFIG /usr/src/sys/amd64/conf
```

Update if the tree has already been checked out:

```console
cd /usr/src && sudo svn up
```

If `devel/subversion` is not available, update the source tree with csup:

```console
sudo csup -L 2 stable-supfile
```

Read updating notes and build:

```console
less /usr/src/UPDATING
cd /usr/obj && sudo chflags -vR noschg * && sudo rm -rvf *
cd /usr/src && sudo make cleandir; sudo make cleandir
sudo script /root/build-`date +%Y%m%dT%H%M%S`.log
make -j2 buildworld && make buildkernel
exit
sudo make installkernel
sudo shutdown -r now
# boot into single-user mode with boot -s at the loader prompt.
adjkerntz -i
mount -a # for a ZFS system: zfs mount -v -a && mount -u -w /
mergemaster -p -F -U
cd /usr/src
make installworld
mergemaster -i -F -U
shutdown -r now
(cd /etc && sudo git status; cd /usr/local/etc && sudo git status; cd /boot && sudo git status)
```

Commit changes to git repositories as necessary.
