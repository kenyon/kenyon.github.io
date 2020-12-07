---
title: Debian
---
This is my [Debian](http://debian.org/) page.

1. ToC
{:toc}

## Serial port card

It took a lot of time to figure out how to get this two-port serial interface card working, so I wanted to document it here for everyone's benefit.

Here is the card I got from Amazon: [Syba SD-PCI-2S PCI 32-Bit 2x Port Serial DB9 PCIe x1 Card](http://a.co/29bam5S)

I'm using it with Debian GNU/Linux 9.4 (stretch) and kernel 4.9.88-1+deb9u1 (2018-05-07). Do not use the driver from ASIX Corporation. There is no need for that because the driver is part of the standard Linux kernel these days.

Without any special configuration, the first port is detected by the kernel and works fine, but not the second port, even though you can see it in `lspci` output.

The key to getting both serial ports working is to use `setserial` from the setserial package to configure the serial ports. Here is my `/etc/serial.conf`:

```conf
/dev/ttyS0 uart 16550A port 0x03f8 irq 4 baud_base 115200 spd_normal skip_test
/dev/ttyS1 uart 16650V2 port 0xe000 irq 19 baud_base 115200 spd_normal skip_test
/dev/ttyS2 uart 16650V2 port 0xe010 irq 18 baud_base 115200 spd_normal skip_test
# have to also do autoconfig to get this second port working
/dev/ttyS2 autoconfig
```

`ttyS0` is some serial port on my motherboard that is not externally exposed, so it's useless. `ttyS1` and `ttyS2` are the interfaces of the Syba MosChip card. With this `setserial` configuration, both ports are detected and working on boot. For example, accessing the serial console of an Ubiquiti EdgeRouter: `screen /dev/ttyS1 115200`

Output of `lspci -vv`, for reference:

```console
02:00.0 Serial controller: Device 0310:9922 (prog-if 02 [16550])
        Subsystem: Device a000:1000
        Control: I/O+ Mem+ BusMaster- SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR+ FastB2B- DisINTx-
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Interrupt: pin A routed to IRQ 18
        Region 0: I/O ports at e010 [size=8]
        Region 1: Memory at f7c03000 (32-bit, non-prefetchable) [size=4K]
        Region 5: Memory at f7c02000 (32-bit, non-prefetchable) [size=4K]
        Capabilities: [50] MSI: Enable- Count=1/8 Maskable- 64bit+
                Address: 0000000000000000  Data: 0000
        Capabilities: [78] Power Management version 3
                Flags: PMEClk- DSI- D1- D2- AuxCurrent=320mA PME(D0-,D1-,D2-,D3hot+,D3cold+)
                Status: D0 NoSoftRst- PME-Enable- DSel=0 DScale=0 PME-
        Capabilities: [80] Express (v1) Legacy Endpoint, MSI 00
                DevCap: MaxPayload 512 bytes, PhantFunc 0, Latency L0s <1us, L1 <2us
                        ExtTag- AttnBtn- AttnInd- PwrInd- RBE+ FLReset-
                DevCtl: Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
                        RlxdOrd- ExtTag- PhantFunc- AuxPwr- NoSnoop-
                        MaxPayload 128 bytes, MaxReadReq 512 bytes
                DevSta: CorrErr+ UncorrErr- FatalErr- UnsuppReq+ AuxPwr- TransPend-
                LnkCap: Port #1, Speed 2.5GT/s, Width x1, ASPM L0s L1, Exit Latency L0s <64ns, L1 unlimited
                        ClockPM+ Surprise- LLActRep- BwNot- ASPMOptComp-
                LnkCtl: ASPM L0s Enabled; RCB 64 bytes Disabled- CommClk-
                        ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
                LnkSta: Speed 2.5GT/s, Width x1, TrErr- Train- SlotClk- DLActive- BWMgmt- ABWMgmt-
        Capabilities: [100 v1] Virtual Channel
                Caps:   LPEVC=0 RefClk=100ns PATEntryBits=1
                Arb:    Fixed- WRR32- WRR64- WRR128-
                Ctrl:   ArbSelect=Fixed
                Status: InProgress-
                VC0:    Caps:   PATOffset=00 MaxTimeSlots=1 RejSnoopTrans-
                        Arb:    Fixed- WRR32- WRR64- WRR128- TWRR128- WRR256-
                        Ctrl:   Enable+ ID=0 ArbSelect=Fixed TC/VC=01
                        Status: NegoPending- InProgress-
                VC1:    Caps:   PATOffset=00 MaxTimeSlots=1 RejSnoopTrans-
                        Arb:    Fixed- WRR32- WRR64- WRR128- TWRR128- WRR256-
                        Ctrl:   Enable- ID=1 ArbSelect=Fixed TC/VC=00
                        Status: NegoPending- InProgress-
        Capabilities: [800 v1] Advanced Error Reporting
                UESta:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
                UEMsk:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
                UESvrt: DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
                CESta:  RxErr+ BadTLP- BadDLLP- Rollover+ Timeout+ NonFatalErr+
                CEMsk:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
                AERCap: First Error Pointer: 00, GenCap- CGenEn- ChkCap- ChkEn-

02:00.1 Serial controller: MosChip Semiconductor Technology Ltd. MCS9922 PCIe Multi-I/O Controller (prog-if 02 [16550])
        Subsystem: Device a000:1000
        Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR+ FastB2B- DisINTx-
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0, Cache Line Size: 32 bytes
        Interrupt: pin B routed to IRQ 19
        Region 0: I/O ports at e000 [size=8]
        Region 1: Memory at f7c01000 (32-bit, non-prefetchable) [size=4K]
        Region 5: Memory at f7c00000 (32-bit, non-prefetchable) [size=4K]
        Capabilities: [50] MSI: Enable- Count=1/8 Maskable- 64bit+
                Address: 0000000000000000  Data: 0000
        Capabilities: [78] Power Management version 3
                Flags: PMEClk- DSI- D1- D2- AuxCurrent=375mA PME(D0-,D1-,D2-,D3hot+,D3cold+)
                Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
        Capabilities: [80] Express (v1) Legacy Endpoint, MSI 00
                DevCap: MaxPayload 512 bytes, PhantFunc 0, Latency L0s <1us, L1 <2us
                        ExtTag- AttnBtn- AttnInd- PwrInd- RBE+ FLReset-
                DevCtl: Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
                        RlxdOrd- ExtTag- PhantFunc- AuxPwr- NoSnoop-
                        MaxPayload 128 bytes, MaxReadReq 512 bytes
                DevSta: CorrErr+ UncorrErr- FatalErr- UnsuppReq+ AuxPwr- TransPend-
                LnkCap: Port #1, Speed 2.5GT/s, Width x1, ASPM L0s L1, Exit Latency L0s <64ns, L1 unlimited
                        ClockPM+ Surprise- LLActRep- BwNot- ASPMOptComp-
                LnkCtl: ASPM L0s Enabled; RCB 64 bytes Disabled- CommClk-
                        ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
                LnkSta: Speed 2.5GT/s, Width x1, TrErr- Train- SlotClk- DLActive- BWMgmt- ABWMgmt-
        Capabilities: [100 v1] Advanced Error Reporting
                UESta:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
                UEMsk:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
                UESvrt: DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
                CESta:  RxErr+ BadTLP- BadDLLP- Rollover+ Timeout+ NonFatalErr+
                CEMsk:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
                AERCap: First Error Pointer: 00, GenCap- CGenEn- ChkCap- ChkEn-
        Kernel driver in use: serial
```

## debconf

Make it so all questions are asked: `sudo dpkg-reconfigure debconf` and answer lowest priority.

## disable console beep

```sh
modprobe -r pcspkr
echo blacklist pcspkr >> /etc/modprobe.d/blacklist.conf
```

Set 'bell-style' to 'none' in `/etc/inputrc`.

`setterm -blength 0`

### See also

* [Removing annoying console beeps](http://www.debian-administration.org/articles/110)

## Rebuilding Debian packages

This is just my local, condensed version of [Raphaël Hertzog's](http://raphaelhertzog.com/) article [Howto to rebuild Debian packages](http://raphaelhertzog.com/2010/12/15/howto-to-rebuild-debian-packages/).

1. `apt-get source ntp`
1. `sudo apt-get build-dep ntp`
1. Make changes to package (or not). If changes were made, use `dch --local foo` to record a changelog entry and increase the version.
1. `debuild`
1. `dput local whatever.changes` or just install the resulting deb with `gdebi`.

## Local Debian package repository

Here is how I created a local Debian package repository, also called an archive, using mini-dinstall.

### ~/.mini-dinstall.conf

```ini
[DEFAULT]
architectures = all, amd64
archivedir = ~/repo
archive_style = flat
generate_release = 1
release_signscript = ~/bin/sign-release.sh
[squeeze]
alias = stable
```

Run `mini-dinstall --batch` to initialize the repository.

### ~/bin/sign-release.sh

```bash
#!/bin/bash
# -*- coding: utf-8 -*-
# Based on Sample script to GPG sign Release files
# Copyright © 2002 Colin Walters <walters@debian.org>
# License: GPLv2
set -e
KEYID=0x98FF3EF9C9B912D5
rm -f Release.gpg.tmp
gpg --default-key "$KEYID" --detach-sign -o Release.gpg.tmp "$1"
mv Release.gpg.tmp Release.gpg
```

Now `gpg --armor --export 0xc9b912d5 | sudo apt-key add -` to get APT to trust the local repository.

### ~/.dput.cf

```ini
[local]
method = local
run_dinstall = 0
post_upload_command = mini-dinstall --batch
incoming = ~/repo/mini-dinstall/incoming
```

Run `dput local pkg.changes` to upload the package to the repository.

### ~/.devscripts

```sh
DEBUILD_DPKG_BUILDPACKAGE_OPTS="-k0x98FF3EF9C9B912D5 -sa"
DSCVERIFY_KEYRINGS="~/.gnupg/pubring.gpg"
```

### sources.list entries

```conf
deb file:///home/kenyon/repo squeeze/
deb-src file:///home/kenyon/repo squeeze/
```

Now after a `sudo aptitude update` you should be able to install packages from the local repository.

### References

* manual pages and `/usr/share/doc` (read with `debmany` from debian-goodies): sources.list, mini-dinstall, dput
* <http://wiki.freegeek.org/index.php/Debian_Package_Repositories>
* <http://upsilon.cc/~zack/blog/posts/2009/04/howto:_uploading_to_people.d.o_using_dput/>
* <http://wiki.debian.org/HowToSetupADebianRepository>

## External links

* <http://debian.org/>
