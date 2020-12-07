---
title: Encrypted backup
---
Documentation on my encrypted backup hard drive.

1. ToC
{:toc}

## Hardware

* [Seagate Backup Plus 4TB Portable External Hard Drive USB 3.0, Silver (STDR4000900)](https://smile.amazon.com/dp/B0196J43TE)

```console
kenyon@einstein ~ % sudo smartctl --all --device=sat /dev/disk/by-id/ata-ST4000LM024-2AN17V_WCK0T8TG
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-4.9.0-3-amd64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     ST4000LM024-2AN17V
Serial Number:    WCK0T8TG
LU WWN Device Id: 5 000c50 0a9957340
Firmware Version: 0001
User Capacity:    4,000,787,030,016 bytes [4.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    5526 rpm
Form Factor:      2.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-3 T13/2161-D revision 5
SATA Version is:  SATA 3.1, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Sun Jul 16 13:43:48 2017 PDT
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

## Software

* Linux 4.9.0-3-amd64 #1 SMP Debian 4.9.30-2+deb9u1 (2017-06-18) x86_64 GNU/Linux
* Debian GNU/Linux 9.0 stretch
* Important packages: dmsetup, cryptsetup

### Encryption

* cryptsetup 1.7.3

#### Creation

```console
sudo cryptsetup --verbose --verify-passphrase --use-random luksFormat /dev/disk/by-id/ata-ST4000LM024-2AN17V_WCK0T8TG-part1
sudo cryptsetup --verbose open --type luks /dev/disk/by-id/ata-ST4000LM024-2AN17V_WCK0T8TG-part1 bak
```

#### Use

Added to `/etc/fstab`:

```conf
/dev/mapper/bak       /bak            btrfs    user,noatime,noauto,x-systemd.automount 0   0
```

```console
sudo cryptsetup --verbose open --type luks /dev/disk/by-id/ata-ST4000LM024-2AN17V_WCK0T8TG-part1 bak
sudo mount /bak
```

Add entry to `/etc/crypttab`:

```conf
bak UUID=c54be6ba-9f10-41b3-a95f-6115d6933df9 none luks,noauto
```

After making the `crypttab` entry: `sudo cryptdisks_start bak && sudo mount /bak`

Or: `sudo systemctl start /bak`

### File system

#### Creation

After `sudo cryptsetup --verbose open --type luks
/dev/disk/by-id/ata-ST4000LM024-2AN17V_WCK0T8TG-part1 bak`, I did `sudo mkfs.btrfs --label bak
/dev/mapper/bak`.

#### Disconnecting

Before disconnecting the drive from the system, do this: `sudo umount /bak && sudo cryptdisks_stop bak`

Or: `sudo systemctl stop /bak && sudo systemctl stop systemd-cryptsetup@bak`

## References

* cryptsetup, luks: <http://code.google.com/p/cryptsetup/>
* <http://www.debian-administration.org/article/Encrypting_an_existing_Debian_lenny_installation>
* <http://madduck.net/docs/cryptdisk/>
