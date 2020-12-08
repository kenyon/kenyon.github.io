---
title: Time server
---
This page documents my project to build a stratum 1 NTP time server at my house. Its hostname is einstein.kenyonralph.com.

1. ToC
{:toc}

## GPS selection

I selected the [Garmin GPS 18x LVC](https://www.amazon.com/gp/product/B0016O3T7A) for the following reasons:

* Plenty of documentation exists by people who have used this unit as an NTP timing source.
* The receiver and antenna are integrated into one enclosure.
* Minimal assembly and soldering necessary, compared to bare-board receivers.
* Available on [Amazon](https://amazon.com/) for a decent price.

I ordered the 18x (and a [serial cable](https://www.amazon.com/gp/product/B000ZKNANO)) on Amazon on 2011-09-05. UPS delivered it on 2011-09-09.

## Cable Construction

I simply spliced the GPS 18x LVC wires onto serial and USB wires. This matches what's documented in Garmin's GPS 18x LVC Technical Specifications.

## GPS preparation

I used a Windows Vista computer with Garmin's SNSRXCFG_270.exe to upgrade the firmware from version 3.60 to the latest, 3.70. I also used SNSRXCFG_270.exe to set the PPS pulse width to 200 ms, and disable all NMEA sentences except GPGGA.

## Debian GNU/Linux setup

Here are the steps in summary (written for Debian wheezy in February 2015):

* Install pps-tools
* Create udev rules
* Create systemd service unit for ldattach

Some details:

When you do `sudo ldattach PPS /dev/ttyS0`, the PPS modules will be loaded automatically and the device `/dev/pps0` will be created. Place some udev rules in `/etc/udev/rules.d/77-local.rules` to create device symlinks and run ldattach automatically:

```conf
SUBSYSTEM=="pps", MODE="0664" GROUP="dialout"
KERNEL=="ttyS2", RUN+="/bin/stty --file=/dev/ttyS2 9600"
KERNEL=="ttyS2" SYMLINK+="gps0"
KERNEL=="pps0" SYMLINK+="gpspps0"
```

systemd unit to run `ldattach` (enable with `systemctl enable ldattach@ttyS2`):

```systemd
[Unit]
Description=PPS Line Discipline for GPS Timekeeping for %i
Before=ntpd.service

[Service]
ExecStart=/usr/sbin/ldattach pps /dev/%i
Type=forking

[Install]
WantedBy=multi-user.target
```

Here is my working ntp.conf:

```conf
rlimit memlock -1
driftfile /var/lib/ntp/ntp.drift
restrict localhost
restrict default limited noquery
server darwin.kenyonralph.com iburst
pool 2.pool.ntp.org iburst
server 127.127.20.0 mode 16 minpoll 3 iburst
fudge 127.127.20.0 flag1 1 flag2 0 flag3 1 time2 0.545
leapfile /etc/ntp/leap-seconds.list
```

Example output after letting ntpd run for about 8 hours:

```console
kenyon@einstein ~ % ntpq -ccv -p -crv -ckern -csysinfo
associd=0 status=0000 no events, clk_unspec,
device="NMEA GPS Clock",
timecode="$GPGGA,172617,1111.3839,N,11111.4417,W,2,11,0.8,126.3,M,-35.5,M,,*71",
poll=3418, noreply=0, badformat=0, baddata=0, fudgetime2=545.000,
stratum=0, refid=GPS, flags=5
        remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
    2.us.pool.ntp.o .POOL.          16 p    -   64    0    0.000    0.000   0.000
oGPS_NMEA(0)     .GPS.            0 l    -    8  377    0.000    0.000   0.004
+darwin.kenyonra 127.67.113.92    2 u   50   64  377   28.478    4.232   0.953
-2001:470:c:4a1: 131.188.3.220    2 u   45   64  377  200.027   11.259   0.586
-2604:a880:800:1 184.105.182.7    3 u   39   64  377   82.776    4.196   1.805
-ntp.jtsage.com  216.218.254.202  2 u   24   64  377   69.561   -1.286   0.490
-2604:a880:800:1 184.105.182.7    3 u   46   64  377   82.421    3.827   0.603
*yurizoku.tk     209.51.161.238   2 u   59   64  377   80.916    1.554   0.619
+198.110.48.12 ( 204.9.54.119     2 u   35   64  377  100.437    1.796   1.330
-108.61.194.85 ( 201.198.247.252  2 u   53   64  377   45.230   -5.521   1.041
associd=0 status=041d leap_none, sync_uhf_radio, 1 event, kern,
version="ntpd 4.2.8p1@1.3265-o Mon Feb  9 09:59:02 UTC 2015 (1)",
processor="x86_64", system="Linux/3.16.7-ckt2", leap=00, stratum=1,
precision=-23, rootdelay=0.000, rootdisp=1.015, refid=GPS,
reftime=d884bdb9.bc1f69e3  Tue, Feb 10 2015  9:26:17.734,
clock=d884bdba.e1639551  Tue, Feb 10 2015  9:26:18.880, peer=27173, tc=3,
mintc=3, offset=-0.000130, frequency=-13.457, sys_jitter=0.004033,
clk_jitter=0.051, clk_wander=0.031, tai=35, leapsec=201507010000,
expire=201512280000
associd=0 status=041d leap_none, sync_uhf_radio, 1 event, kern,
pll offset:            0
pll frequency:         -13.4565
maximum error:         0.0015
estimated error:       3.4e-05
kernel status:         pll ppsfreq ppstime ppssignal nano
pll time constant:     3
precision:             1e-06
frequency tolerance:   500
pps frequency:         -13.4874
pps stability:         0.280609
pps jitter:            0.050
calibration interval   256
calibration cycles:    152
jitter exceeded:       45
stability exceeded:    2
calibration errors:    0
associd=0 status=041d leap_none, sync_uhf_radio, 1 event, kern,
system peer:        GPS_NMEA(0)
system peer mode:   client
leap indicator:     00
stratum:            1
log2 precision:     -23
root delay:         0.000
root dispersion:    1.015
reference ID:       GPS
reference time:     d884bdb9.bc1f69e3  Tue, Feb 10 2015  9:26:17.734
system jitter:      0.004033
clock jitter:       0.051
clock wander:       0.031
broadcast delay:    0.000
symm. auth. delay:  0.000
```

I also tried setting up a time server with FreeBSD to see if it's any easier or better.

## FreeBSD setup

This section was written in October 2011. I no longer run this FreeBSD system. I'm running the NTP server on the Linux machine described above.

I am using [ntp-devel](https://www.freshports.org/net/ntp-devel/) from ports.

### /etc/rc.conf

```conf
ntpd_enable="YES"
ntpd_flags="-N -p /var/run/ntpd.pid -f /var/db/ntpd.drift"
ntpd_program="/usr/local/sbin/ntpd"
ntpd_sync_on_start="YES"
```

### /etc/devfs.conf

```conf
link cuau0 gps0
```

### /etc/ttys

Commented out ttyu*.

### Kernel configuration

Put this in `/usr/src/sys/amd64/conf/PPS-GENERIC`:

```conf
#
# PPS -- Generic kernel configuration file for FreeBSD/amd64 PPS
#
include GENERIC
ident PPS-GENERIC
options PPS_SYNC
```

### /etc/make.conf

```conf
KERNCONF= PPS-GENERIC GENERIC
```

### /etc/ntp.conf

```conf
server 127.127.20.0 minpoll 3
fudge 127.127.20.0 flag1 1 flag2 0 flag3 1 time2 0.600
server darwin.kenyonralph.com iburst
server voodoo.kenyonralph.com iburst
server grunt.kenyonralph.com iburst
pool 2.us.pool.ntp.org iburst
```

### Example output

Example output after running for a few minutes and while doing buildworld and buildkernel:

```console
kenyon@gauss ~ % ntpq -p -c clockvar -c readvar; ntpdc -c kerninfo
        remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
oGPS_NMEA(0)     .GPS.            0 l   38   64  377    0.000    0.648   0.249
    2.us.pool.ntp.o .POOL.          16 p    -   64    0    0.000    0.000   0.002
*darwin.kenyonra 127.67.113.92    2 u   63   64  377   20.431    3.263   3.354
-voodoo.kenyonra 106.61.18.129    3 u   61   64  377    0.243    4.485   0.124
+grunt.kenyonral 106.61.18.129    3 u   64   64  377    0.182    3.521   0.225
-conquest.kjsl.c 69.36.224.15     2 u   62   64  377   26.374    5.768   2.522
-bindcat.fhsu.ed 128.138.140.44   2 u   63   64  377   75.219    4.115   1.846
-peachesgeldof.k 204.9.54.119     2 u   64   64  377   94.141    3.787   2.844
-ip-173-201-38-8 198.153.152.52   2 u   58   64  377   24.752    6.104   1.801
-druid.storyinme 130.207.244.240  2 u   56   64  377   65.657   -2.773   4.542
+cheezum.mattnor 129.7.1.66       2 u   65   64  377   45.797    2.972   2.286
associd=0 status=0000 no events, clk_unspec,
device="NMEA GPS Clock",
timecode="$GPGGA,085740,1111.1111,N,11111.1111,W,1,09,0.9,103.2,M,-35.3,M,,*77",
poll=14, noreply=0, badformat=0, baddata=0, fudgetime2=600.000,
stratum=0, refid=GPS, flags=5
associd=0 status=04ad leap_none, sync_uhf_radio, 10 events, kern,
version="ntpd 4.2.7p225@1.2483-o Fri Oct 21 04:59:50 UTC 2011 (1)",
processor="amd64", system="FreeBSD/9.0-RC1", leap=00, stratum=1,
precision=-19, rootdelay=0.000, rootdisp=2.380, refid=GPS,
reftime=d24d03df.edabdc26  Sat, Oct 22 2011  1:57:03.928,
clock=d24d0405.3f696dad  Sat, Oct 22 2011  1:57:41.247, peer=38673, tc=6,
mintc=3, offset=0.648, frequency=-4.048, sys_jitter=0.249,
clk_jitter=0.000, clk_wander=0.005
pll offset:           0.000624065 s
pll frequency:        -4.048 ppm
maximum error:        2.0824e-05 s
estimated error:      1.7e-08 s
status:               2007  pll ppsfreq ppstime nano
pll time constant:    6
precision:            1e-09 s
frequency tolerance:  496 ppm
kenyon@gauss ~ % ntpq -c "rv 38673"
associd=38673 status=973a conf, reach, sel_pps.peer, 3 events, sys_peer,
srcadr=GPS_NMEA(0), srcport=123, dstadr=127.0.0.1, dstport=123, leap=00,
stratum=0, precision=-20, rootdelay=0.000, rootdisp=0.000, refid=GPS,
reftime=d24d03de.ffd6d0b6  Sat, Oct 22 2011  1:57:02.999,
rec=d24d03df.edabdc26  Sat, Oct 22 2011  1:57:03.928, reach=377,
unreach=0, hmode=3, pmode=4, hpoll=6, ppoll=6, headway=0, flash=00 ok,
keyid=0, offset=0.648, delay=0.000, dispersion=0.928, jitter=0.249,
filtdelay=     0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00,
filtoffset=    0.65    0.70    0.76    0.80    0.86    0.91    0.99    1.06,
filtdisp=      0.00    0.96    1.92    2.88    3.84    4.80    5.76    6.72
```

Here is what kerninfo looks like with the PPS_SYNC kernel option:

```console
pll offset:           0.000123674 s
pll frequency:        -3.918 ppm
maximum error:        0.000111415 s
estimated error:      1.51e-07 s
status:               2107  pll ppsfreq ppstime ppssignal nano
pll time constant:    6
precision:            1e-09 s
frequency tolerance:  496 ppm
pps frequency:        -3.918 ppm
pps stability:        0.033 ppm
pps jitter:           4.729e-06 s
calibration interval: 64 s
calibration cycles:   27
jitter exceeded:      5
stability exceeded:   0
calibration errors:   10
```

## Conclusion

FreeBSD is much nicer than Linux (as of late 2011) at being a stratum 1 NTP server using a NMEA GPS with PPS reference clock.

As of early 2015, I would say it's about an equal amount of effort between FreeBSD and Debian
Linux. Debian Linux is a little more effort if you want to create a proper Debian package of NTP.

## Notes

* Show serial port settings:
  * Linux: `stty --all --file=/dev/ttyS0`
  * FreeBSD: `stty -a -f /dev/cuau0`
* Set serial port baud rate to 4800:
  * Linux: `stty --file=/dev/ttyS0 4800`
  * FreeBSD (but not really necessary since `cu` can do it, and doesn't seem to take effect anyway): `stty -f /dev/cuau0 4800`
* Observe NMEA output:
  * Linux: `cat /dev/ttyS0`
  * FreeBSD: `cu -l /dev/cuau0 -s 4800`

## References

* NTP documentation: [Reference Clock Support](https://www.eecis.udel.edu/~mills/ntp/html/refclock.html)
* NTP support wiki: [Configuring Garmin Refclocks](https://web.archive.org/web/20200919195309/http://support.ntp.org/bin/view/Support/ConfiguringGarminRefclocks)
* NTP support wiki: [Configuring NMEA Refclocks](https://web.archive.org/web/20200815214455/https://support.ntp.org/bin/view/Support/ConfiguringNMEARefclocks)
* NTP support wiki: [Garmin Refclock Users](https://web.archive.org/web/20201026102407/http://support.ntp.org/bin/view/Support/GarminRefclockUsers)
* [Adding a FreeBSD NTP server based on an GPS 18 LVC device](https://www.satsignal.eu/ntp/FreeBSD-GPS-PPS.htm) by David Taylor
* [Enabling ntpd PPS support for Debian Lenny Linux](https://www.worldtimesolutions.com/support/ntp/Debian_Lenny_Linux_PPS_support_for_ntpd.html) by World Time Solutions
* [Garmin GPS 18x OEM](https://buy.garmin.com/en-US/US/p/27594/pn/010-00321-31)
* [NTP server using PC gnu/linux and freebsd](https://www.wraith.sf.ca.us/ntp/) by Steven Bjork
* [Stratum 1 NTP, Garmin GPS 18 LVC on FreeBSD 8.0](http://ryandoyle.net/posts/stratum-1-ntp-garmin-gps-18-lvc-on-freebsd-80/) by Ryan Doyle
* [Synchronising to a Garmin GPS 18 LVC](http://www.sput.nl/time/garmin.html) by R.J. van der Putten
* [Synchronizing an NTP server to GPS/PPS](https://linlog.blogspot.com/2009/07/synchronizing-ntp-server-to-gpspps.html) by Pela-Suros
* [Synchronizing ntpd to a Garmin GPS 18 LVC via gpsd](http://www.rjsystems.nl/en/2100-ntpd-garmin-gps-18-lvc-gpsd.php) by Jaap Winius
