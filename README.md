# brotherscankeyd2: Scan Key Daemon for Brother Inc. Network Scanners, version 2

Copyright (C) 2016-2020 Frank Abelbeck <frank.abelbeck@googlemail.com>

License: GPL 3

## why i switched from brscan-skey (MaUwJo)

i enabled wireguard ( https://www.wireguard.com/ ) on my computer, it seems the brother brsan-skey was not able to use the right internal ip adress of my computer(192.168.0.X) . It was using the 2nd adress from WG interface (192.168.20.X) - And when pressing the scan key, the printer was waiting for a scan from 192.168.20.1, but the scan starts with 192.168.0.X ... And scanning wasn't possible anymore. 
Also I agree on having a binary isn't the best 

## Changes from MaUwJo

in script bskd2_scan2pdfsw

use of GraphicsMagick (http://www.graphicsmagick.org/) - it seems to have better compress ratio

use of https://github.com/ocrmypdf/OCRmyPDF - to get an PDF with text (OCR)

add sample systemd

## Overview

Years ago I bought a Brother multifunction printer and was impressed by the
Linux support Brother apparently is providing. The device just worked. In
addition, Brother offers a small tool brscan-skey which makes it possible to
initiate scans via the printer's scan key.

So in 2016 I create a small daemon to manage the printer's scan menu, the
Brother scan key daemon or short brotherscankeyd.

I created the program because...
 ...brscan-skey is just a binary and I didn't want to install it on my server;
 ...the tool intrigued me -- how does it work?

In 2020 I was forced to re-install my server. In the course of setting it up
I audited existing services and re-wrote brotherscankeyd to match my current
coding style, yielding brotherscankeyd2. Since I changed the CLI behaviour and
the init file syntax, I deemed it proper to start a clean repo and issue a new
version number.

brotherscankeyd2 is distributed in the hope that it will be of help. It comes
with absolutely no warranty (see license).

## Description

This program offers a network service to execute scripts/programs if a scan key
event occurs. It registers scan-to entries on one or more network printers and
listens to UDP packets coming from these devices. If a datagram of a known scan
key action is received, the corresponding script is called.

## Requirements

Which programs and libraries are needed?
(in parantheses: Gentoo Linux versions this program was created/tested with and
an URL for more information)

 * python3 (3.6.12, https://www.python.org/) (debian package python3)
 * net-snmp (5.9-r2, http://www.net-snmp.org/)  ( debian package snmp)
 * linuxfd  (9999, https://www.github.com/FrankAbelbeck/linuxfd)

Since brotherscankeyd2 is written in Python (currently my favourite
problem-solving programming language), you need a Python3K interpreter.
In addition, `/usr/bin/snmpset` is needed for issuing SNMP set packets.
Finally, you need linuxfd, my own python bindings for the signalfd, timerfd,
and eventfd system calls.

In order to execute the provided sample scripts you need the following programs:

 * sane-backends (1.0.30-r2, http://www.sane-project.org/)
 * sane-frontends (1.0.14-r5, http://www.sane-project.org/)
 * imagemagick (7.0.10.35, https://www.imagemagick.org/)
 * pdftk (3.0.0, https://gitlab.com/pdftk-java/pdftk)

Obviously, you'll need SANE to be able to scan anything, or more precisely,
`/usr/bin/scanimage` (backend) and `/usr/bin/scanadf` (frontend).
Image processing and pdf handling is done using `/usr/bin/convert` (imagemagick)
and `/usr/bin/pdftk` (pdftk).

In addition you need to check if `convert` is allowed to read/write pdf files.
Open `/etc/ImageMagick-7/policy.xml` and make sure that the following line is
defined. 

```
<policy domain="coder" rights="read|write" pattern="PDF" />
```

Per default this is disabled (`rights="none"`) in order to mitigate
ghostscript vulnerabilities (https://bugs.gentoo.org/664236).

## Installation: From Source

The following steps assume being run on a standard Linux system with OpenRC as user root.
Further it is assumed that $GITDIR equals your local ddmailer repo path.

0. Make sure all needed dependencies are installed
1. Place all files in `$GITDIR/bin` in `/usr/bin/`
2. Place `$GITDIR/openrc/brotherscankeyd2` in `/etc/init.d/`
3. Set permissions with `chmod 755 /etc/init.d/brotherscankeyd2 /usr/bin/brotherscankeyd2 /usr/bin/bskd2_*`
4. Create basic main configuration file with `/usr/bin/brotherscankeyd2 cfgMain > /etc/brotherscankeyd2.ini`
5. Set permission on main configuration file with `chmod 666 /etc/brotherscankeyd2.ini`
7. Edit this configuration file (add some menu entries!)
8. Add brotherscankeyd2 to the default runlevel with `rc-update add brotherscankeyd2 default`
9. Start brotherscankeyd2 with `/etc/init.d/brotherscankeyd2 start`

## Installation: Gentoo

I created two ebuilds which can be found in the `portage` subdirectory:

1. dev-python/linuxfd/linuxfd-9999.ebuild
2. media-gfx/brotherscankeyd2/brotherscankeyd2-9999.ebuild

Copy the contents of `portage` into your local portage repository. You can
find instructions for creating your own local portage repo in the [Gentoo Handbook](
https://wiki.gentoo.org/wiki/Handbook:AMD64/Portage/CustomTree#Defining_a_custom_ebuild_repository).

Run `repoman manifest` in every ebuild directory (builds the Manifest file).

Afterwards, emerge it with `emerge -va brotherscankeyd2`. This will pull in
dev-python/linuxfd, too.

## Usage

The service is started by calling `brotherscankeyd2 start`. Only root is allowed
to start or stop the program. During start-up, the program daemonises and drops
privileges to user nobody.

Calling `brotherscankeyd2 stop` will stop the daemon. Alternatively you can send
SIGTERM to the process. The process id can be found in the program's PID file
`/run/brotherscankeyd2.pid`.

Further steps are up to you and your printer setup. I provided examples for both
configuration and scripting.

## Changelog Version 2

 * **2020-12-01:** user setting now tweakable via configuration file
 
 * **2020-11-30:** initial commit, adapted to current Abelbeck coding standard

## Changelog Version 1

 * **2017-07-02:** fixed process management: replaced proc.kill() with proc.terminate()

 * **2017-06-24:** removed pysnmp dependencies; SNMP requests are now sent with
   /usr/bin/snmpset (http://net-snmp.sourceforge.net/).

 * **2016-08-06:** fixed a script calling race condition (scanner not immediately ready
   for connects of scanimage/scanadt after issuing a notification) by introducing a two second delay

 * **2016-07-25:** moved SNMP requests into subprocess to avoid blocking the main loop

 * **2016-07-17:** debugged program after first server trials; removed SEVERE scan
   script quoting errors, altered script calling procedure, introduced OpenRC init script

 * **2016-07-10:** switched to a unified output system (Python module logging and
   custom ConsoleHandler for coloured output); moved CONFIGFILE and PIDFILE to
   standard locations /etc and /var/run respectively (program tries both paths);
   scan2* scripts revisited, now seem to work; various bug fixes
 
 * **2016-07-05:** initial release as "works for me" version (except for the scan
   scripts scan2image.sh and scan2pdf.sh; they are still untested)
