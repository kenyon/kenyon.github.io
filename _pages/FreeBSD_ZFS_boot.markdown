---
title: FreeBSD ZFS boot
---
In 2009, I set up FreeBSD to boot from a ZFS mirrored dataset. This page documents how I did it.

1. ToC
{:toc}

## Hardware

Pentium 4 (i386) system with three 80-GB hard disks: ad2, ad4, and ad6. 3 GB RAM.

## Goals

* Have only ZFS.
* Have a mirror setup in the zpool.
* Have the system bootable from any of the disks in the mirror.

## Procedure

First I did a standard UFS2 minimal install of FreeBSD 8.0-BETA2 to `ad2`, csup to `RELENG_8`
(also known as `8-STABLE`), and [rebuilt world and kernel]({% link
_pages/FreeBSD_rebuild.markdown %}) with ZFS loader support. I also built a custom kernel with
`options KVA_PAGES=512` (necessary for i386, see the ZFS tuning guide) and a more specific
configuration for my hardware. These settings are on the `ad2` system (they also need to be on
the resulting ZFS system):

```console
echo 'KERNCONF=KRZFSOHM GENERIC' >> /etc/make.conf
echo 'LOADER_ZFS_SUPPORT=YES' >> /etc/make.conf
echo 'daily_status_zfs_enable="YES"' >> /etc/periodic.conf
```

These need to be only on the resulting ZFS system, not on the UFS system:

```console
echo 'zfs_load="YES"' >> /ohm/boot/loader.conf
echo 'vfs.root.mountfrom="zfs:ohm"' >> /ohm/boot/loader.conf
echo 'zfs_enable="YES"' >> /ohm/etc/rc.conf
```

ohm is the name of my zpool. Here we go:

```console
# Force the zpool creation due to the disks being slightly different sizes,
# since they are from different manufacturers. ZFS should make use of the
# smaller disk's size for the mirror (according to the ZFS cheat sheet, see
# the references section).
zpool create -f ohm mirror ad4 ad6
zpool set bootfs=ohm ohm
sh -c 'for fs in tmp usr/home var/log; do zfs create -p ohm/$fs; done'
chmod -v 1777 /ohm/tmp
mkdir -v /ohm/dev
cd /ohm && ln -s -w -v usr/home home
cd /ohm/var && ln -s -w -v ../tmp tmp && cd
zfs create -V 6gb ohm/swap
zfs set org.freebsd:swap=on ohm/swap
zfs set checksum=off ohm/swap

# Go to single user mode for safety and speed.
shutdown now

find -d / -not -path /ohm\* -not -path /etc/fstab -not -path /dev\* -not -path /tmp\* -not -path /var/tmp\* -not -path /usr/obj\* | cpio -pmuvd /ohm

echo 'vfs.root.mountfrom="zfs:ohm"' >> /ohm/boot/loader.conf
echo 'zfs_enable="YES"' >> /ohm/etc/rc.conf
echo -e "#Device\t\tMountpoint\tFStype\tOptions\t\tDump\tPass#" > /ohm/etc/fstab

zpool export ohm && zpool import ohm
cp -vi /boot/zfs/zpool.cache /ohm/boot/zfs
zpool export ohm
dd if=/boot/zfsboot of=/dev/ad4 bs=512 count=1
dd if=/boot/zfsboot of=/dev/ad6 bs=512 count=1
dd if=/boot/zfsboot of=/dev/ad4 bs=512 skip=1 seek=1024
dd if=/boot/zfsboot of=/dev/ad6 bs=512 skip=1 seek=1024

zpool import ohm
zfs set mountpoint=legacy ohm
sh -c 'for fs in tmp usr var; do zfs set mountpoint=/$fs ohm/$fs; done'

reboot
# Remember to change the BIOS hard drive boot priority setting.

# Add ad2 to the zpool to make it a 3-way mirror.
gpart delete -i 1 ad2
gpart destroy ad2
dd if=/boot/zfsboot of=/dev/ad2 bs=512 count=1
dd if=/boot/zfsboot of=/dev/ad2 bs=512 skip=1 seek=1024
zpool attach ohm ad4 ad2
```

## zfs internal error: failed to initialize ZFS library

This error happens when first trying to use ZFS commands in the FreeBSD livefs environment.

Solution:

```console
kldload /dist/boot/kernel/opensolaris.ko
kldload /dist/boot/kernel/zfs.ko
```

Reference: <https://bugs.freebsd.org/118855>

## References

None of these references contained precise instructions on how to do exactly what I wanted to do, but they all helped me figure this out.

* <https://wiki.freebsd.org/ZFS>
* <https://wiki.freebsd.org/ZFSTuningGuide>
* <http://www.waishi.jp/~yosimoto/diary/?date=20080909> (Japanese)
* <https://www.freebsd.org/doc/en/books/handbook/zfs.html>
* <https://lildude.co.uk/zfs-cheatsheet>
* <https://outpost.h3q.com/patches/manageBE/create-FreeBSD-ZFS-bootfs.txt>
* <https://web.archive.org/web/20100824022058/http://lulf.geeknest.org:80/blog/freebsd/Setting_up_a_zfs-only_system/>
* <http://unix.derkeiler.com/Mailing-Lists/FreeBSD/stable/2009-05/msg00539.html>
* <http://unix.derkeiler.com/Mailing-Lists/FreeBSD/stable/2009-05/msg00546.html>
* <https://bugs.freebsd.org/118855>
* The FreeBSD manual pages, including [zpool](https://www.freebsd.org/cgi/man.cgi?query=zpool&apropos=0&sektion=0&format=html) and [zfs](https://www.freebsd.org/cgi/man.cgi?query=zfs&apropos=0&sektion=0&format=html)
