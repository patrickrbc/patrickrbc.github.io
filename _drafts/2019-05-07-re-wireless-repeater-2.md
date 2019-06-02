---
layout: post
title:  "Reverse engineering a wireless repeater - Part II"
date:   2019-05-07 15:34:00 -0300
comments: true
categories: 
---


This is the second part of a series of posts about reverse engineering a
wireless repeater. Before reading this you might want to take a look at the
[first part](/2019/04/01/re-wireless-repeater), where I introduced the subject
and explained how to gather some information without digging into the hardware.
If you still haven't downloaded the firmware of our target device you can get
it [here](http://en.intelbras.com.br/sites/default/files/downloads/fw_nplug_1_0_0_14.zip).

Firstly, make sure you have extracted the zip file. Using the **file** command we
can verify that the firmware was compiled for Linux MIPS.

```
$ file fw_NPLUG_1_0_0_14.bin
fw_NPLUG_1_0_0_14.bin: u-boot legacy uImage, Linux Kernel Image, Linux/MIPS, OS
Kernel Image (lzma), 1731916 bytes, Wed Oct 12 17:28:28 2016, Load Address:
0x80000000, Entry Point: 0x802CB000, Header CRC: 0xEC8AD091, Data CRC:
0x3A3E5EAC
```

There is not much you can do with this file as it is. In order to extract
useful stuff from it we need to find the offset of known file format signatures
inside the binary. Once you know the offset you could extract the chunks using
the Linux **dd** tool.

However, it is not necessary to do that manually, instead, we will use a tool
called [*Binwalk*](https://github.com/ReFirmLabs/binwalk). This tool was created
by [Craig Heffner](http://www.devttys0.com/) and it helps a lot in the process of extracting file systems
from firmware files.

# Using *Binwalk*

Running *Binwalk* against the firmware without specifying any options would
give us the following output:

```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             uImage header, header size: 64 bytes, header CRC: 0xEC8AD091, created: 2016-10-12 17:28:28, image size: 1731916 bytes, Data Address: 0x80000000, Entry Point: 0x802CB000, data CRC: 0x3A3E5EAC, OS: Linux, CPU: MIPS, image type: OS Kernel Image, compression type: lzma, image name: "Linux Kernel Image"
64            0x40            LZMA compressed data, properties: 0x5D, dictionary size: 33554432 bytes, uncompressed size: 3829956 bytes
```

That means the binary has a header and some compressed data starting at offset
0x40. This assumption might be true or not, the tool is just trying to
interpret the bytes in the file.
Using the **xxd** tool we can examine the first bytes from the binary which
refer to the header:


```
$ xxd -l 64 fw_NPLUG_1_0_0_14.bin
00000000: 2705 1956 ec8a d091 57fe 72bc 001a 6d4c  '..V....W.r...mL
00000010: 8000 0000 802c b000 3a3e 5eac 0505 0203  …..,..:>^.....
00000020: 4c69 6e75 7820 4b65 726e 656c 2049 6d61  Linux Kernel Ima
00000030: 6765 0000 0000 0000 0000 0000 0000 0000  ge..............
```


*Binwalk* assumed the following chunk was related to some LZMA compressed data
because of the signature `5d 00 00 00 02`. This signature might
[vary](lzma-file-format) depending
on the compression level used, but apparently it always start with `5d 00 00`.
Here is the start of the compressed data:

```
[...]
00000040: 5d00 0000 02c4 703a 0000 0000 0000 006f  ].....p:.......o
00000050: fdff ffa3 b7ff 473e 4815 7239 6151 b892  …...G>H.r9aQ..
00000060: 28e6 a386 07f9 eee4 1e82 d32f c53a 3c01  (........../.:<.
00000070: 4bb1 7ec9 8a8a 4d2f a30d d97f a6e3 8c23  K.~...M/.......#
00000080: 1153 e059 18c5 758a e277 f886 f31f 9269  .S.Y..u..w.....i
00000090: 516e f9a2 84d4 b228 94f8 6af5 1bf9 141e  Qn.....(..j.....
[...]
```



The ```-e (--extract)``` flag, will extract the files identified during a
signature scan and the ```-M (--matryoshka)``` flag will perform this
extraction recursively. There is also another really useful option in *Binwalk*
which allows an entropy analysis of the file. This can be helpful when
*Binwalk*
doesn't work properly on the target file and you have to do a manual analysis.

For example, taking a step back, if we used *Binwalk* extraction only once, we
would find a file named **40** (this number refers to its offset in hexadecimal).
Using *Binwalk* with the ```-E (--entropy)``` flag in this file will pop the
following line chart (generated with *pyqtgraph*).

![Entropy of file 40](/assets/nplug/40.png)
<br>

The Y axis is the entropy and the X axis refers to the offset in that file.
With this graph we can infer that there are some readable stuff in that file
but there are some other regions with high entropy. A high entropy in a file
probably means compressed or encrypted data, but it could also be just garbage.

The signature scan for this file would give us the following output:

```
$ binwalk 40
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
2658360       0x289038        Linux kernel version "2.6.21 (root@linux-qxix)
(gcc version 3.4.2) #1086 Thu Oct 13 01:28:23 CST 2016"
2659360       0x289420        CRC32 polynomial table, little endian
2686144       0x28FCC0        SHA256 hash constants, little endian
2728378       0x29A1BA        Unix path: /mru/rcvseq/sendseq/lns debug
reorderto
2736880       0x29C2F0        Unix path: /etc/Wireless/RT2860AP/RT2860AP.dat
2737968       0x29C730        XML document, version: "1.0"
2870768       0x2BCDF0        CRC32 polynomial table, little endian
3059712       0x2EB000        LZMA compressed data, properties: 0x5D,
dictionary size: 1048576 bytes, uncompressed size: 2943488 bytes
```

Performing another extraction with *Binwalk* in the file **40** will output
another file named **2EB000**. We can generate the entropy analysis line chart
for the new file too.

![Entropy of file 2EB000](/assets/nplug/2EB000.png)
<br>

This time we can see a lot of regions in the file with data that is probably
readable. Not surprisingly, we have found the file system of the firmware. Of
course, we could have simply extracted everything with just one command like
the following demonstration:


```
$ binwalk -eM fw_NPLUG_1_0_0_14.bin
$ cd fw_NPLUG_1_0_0_14.bin.extracted/_40.extracted/_2EB000.extracted/cpio-root/
$ tree
.
├── bin
│   ├── ash -> busybox
│   ├── ate
│   ├── ated
│   ├── busybox
│   ├── cat -> busybox
│   ├── chmod -> busybox
│   ├── cp -> busybox
│   ├── date -> busybox
│   ├── default.cfg
│   ├── dnrd
│   ├── echo -> busybox
│   ├── hostname -> busybox
│   ├── httpd
│   ├── inadyn
│   ├── iptables
│   ├── iwconfig
│   ├── iwpriv
│   ├── kill -> busybox
│   ├── ls -> busybox
│   ├── miniupnpd
│   ├── mkdir -> busybox
│   ├── mount -> busybox
│   ├── msg
│   ├── mtd_write
│   ├── netctrl
│   ├── nvram_clear -> ralink_init
│   ├── nvram_commit -> ralink_init
│   ├── nvram_get -> ralink_init
│   ├── nvram_set -> ralink_init
│   ├── nvram_show -> ralink_init
│   ├── ping -> busybox
│   ├── pppd
│   ├── ps -> busybox
│   ├── ralink_init
│   ├── rm -> busybox
│   ├── sh -> busybox
│   ├── sntp
│   ├── switch
│   ├── tenda_chck_conn
│   ├── tenda_deamon
│   ├── tquser
│   ├── udhcpc
│   └── udhcpd
├── dev
│   └── pts
├── etc
│   └── fstab
├── etc_ro
│   ├── inittab
│   ├── linuxigd
│   ├── miniupnpd.conf
│   ├── motd
│   ├── ppp
│   │   ├── 3g
│   │   ├── ip-up
│   │   ├── peers
│   │   └── plugins
│   │       └── rp-pppoe.so
│   ├── rcS
│   ├── usb
│   ├── web
│   │   ├── blank.htm
│   │   ├── css
│   │   ├── direct_reboot.asp
│   │   ├── direct_reboot_wizard.asp
│   │   ├── error.asp
│   │   ├── firewall_clientfilter.asp
│   │   ├── firewall_clientsetting.asp
│   │   ├── firewall_mac.asp
│   │   ├── firewall_macsetting.asp
│   │   ├── firewall_packagefilter.asp
│   │   ├── firewall_urlfilter.asp
│   │   ├── firewall_urlsetting.asp
│   │   ├── images
│   │   │   ├── bg_button.gif
│   │   │   ├── bg_button_over.gif
│   │   │   ├── bt_disable2.png
│   │   │   ├── bt_disable.png
│   │   │   ├── bt_in_long.PNG
│   │   │   ├── bt_long.PNG
│   │   │   ├── bt_mouse-in2.png
│   │   │   ├── bt_mouse-in.png
│   │   │   ├── bt_nomal_login.gif
│   │   │   ├── bt_normal2.png
│   │   │   ├── bt_normal.png
│   │   │   ├── bt_over_login.gif
│   │   │   ├── bt_press2.png
│   │   │   ├── bt_press.png
│   │   │   ├── load_bg.png
│   │   │   ├── loading.gif
│   │   │   ├── login_bg.jpg
│   │   │   ├── login_left.gif
│   │   │   ├── login_right.gif
│   │   │   ├── login_top.gif
│   │   │   ├── menubg.gif
│   │   │   ├── menu_bg.jpg
│   │   │   ├── nav_normal.gif
│   │   │   ├── ok.png
│   │   │   ├── product.png
│   │   │   ├── top.png
│   │   │   └── wan.png
│   │   ├── ipMacBind.asp
│   │   ├── js
│   │   │   ├── encode.js
│   │   │   ├── menus_list.js
│   │   │   └── system_tool.js
│   │   ├── lan.asp
│   │   ├── lan_dhcp_clients.asp
│   │   ├── lan_dhcps.asp
│   │   ├── login.asp
│   │   ├── mac_clone.asp
│   │   ├── main.htm
│   │   ├── mode_select.asp
│   │   ├── nat_dmz.asp
│   │   ├── nat_virtualportseg.asp
│   │   ├── net_tc.asp
│   │   ├── password_error.asp
│   │   ├── password_succeed.asp
│   │   ├── public
│   │   │   ├── base.css
│   │   │   ├── common.css
│   │   │   ├── gozila.js
│   │   │   ├── j.js
│   │   │   ├── menu.js
│   │   │   ├── page_div.css
│   │   │   ├── page_div.js
│   │   │   ├── style.css
│   │   │   └── table.js
│   │   ├── reboot.asp
│   │   ├── routing_static.asp
│   │   ├── routing_table.asp
│   │   ├── system_backup.asp
│   │   ├── system_hostname.asp
│   │   ├── system_log.asp
│   │   ├── system_password.asp
│   │   ├── system_reboot.asp
│   │   ├── system_remote.asp
│   │   ├── system_restore.asp
│   │   ├── system_status.asp
│   │   ├── system_upgrade
│   │   ├── upgrading.asp
│   │   ├── upnp_config.asp
│   │   ├── wan_connected.asp
│   │   ├── wan_dns.asp
│   │   ├── wan_portParam.asp
│   │   ├── wireless_basic.html
│   │   ├── wireless_filter.html
│   │   ├── wireless_scan.asp
│   │   ├── wireless_security.html
│   │   ├── wireless_state.html
│   │   └── wizard.asp
│   ├── Wireless
│   │   ├── iNIC
│   │   ├── RT2860AP
│   │   │   ├── RT2860_default_novlan
│   │   │   └── RT2860_default_vlan
│   │   └── RT61AP
│   ├── wlan
│   │   └── RT5350_AP_1T1R_V1_0.bin
│   └── xml
├── home
├── init -> bin/busybox
├── lib
│   ├── ipsec
│   ├── ld-uClibc-0.9.28.so
│   ├── ld-uClibc.so.0 -> ld-uClibc-0.9.28.so
│   ├── libcrypt-0.9.28.so
│   ├── libcrypt.so -> libcrypt-0.9.28.so
│   ├── libcrypt.so.0 -> libcrypt-0.9.28.so
│   ├── libc.so -> libuClibc-0.9.28.so
│   ├── libc.so.0 -> libuClibc-0.9.28.so
│   ├── libdl-0.9.28.so
│   ├── libdl.so -> libdl-0.9.28.so
│   ├── libdl.so.0 -> libdl-0.9.28.so
│   ├── libiw.so.29
│   ├── libnvram-0.9.28.so
│   ├── libnvram.so -> libnvram-0.9.28.so
│   ├── libnvram.so.0 -> libnvram-0.9.28.so
│   ├── libpthread-0.9.28.so
│   ├── libpthread.so -> libpthread-0.9.28.so
│   ├── libpthread.so.0 -> libpthread-0.9.28.so
│   ├── libresolv-0.9.28.so
│   ├── libresolv.so -> libresolv-0.9.28.so
│   ├── libresolv.so.0 -> libresolv-0.9.28.so
│   ├── libtenda_common.so
│   ├── libuClibc-0.9.28.so
│   ├── libutil-0.9.28.so
│   ├── libutil.so -> libutil-0.9.28.so
│   ├── libutil.so.0 -> libutil-0.9.28.so
│   └── modules
│       ├── 2.6.21
│       │   └── kernel
│       │       └── net
│       │           └── netfilter
│       │               └── xt_webstr.ko
│       ├── ipt_httpredirect.ko
│       └── xt_turbo_qos.ko
├── media
├── mnt
├── proc
├── sbin
│   ├── arp -> ../bin/busybox
│   ├── halt -> ../bin/busybox
│   ├── ifconfig -> ../bin/busybox
│   ├── init -> ../bin/busybox
│   ├── insmod -> ../bin/busybox
│   ├── lsmod -> ../bin/busybox
│   ├── poweroff -> ../bin/busybox
│   ├── pppoe.sh
│   ├── reboot -> ../bin/busybox
│   ├── rmmod -> ../bin/busybox
│   ├── route -> ../bin/busybox
│   └── vconfig -> ../bin/busybox
├── sys
├── tmp
├── usr
│   ├── bin
│   │   ├── expr -> ../../bin/busybox
│   │   ├── killall -> ../../bin/busybox
│   │   └── top -> ../../bin/busybox
│   ├── codepages
│   └── sbin
│       ├── brctl -> ../../bin/busybox
│       └── telnetd -> ../../bin/busybox
└── var

41 directories, 185 files
```
# Conclusion

In this post we learned how to extract the file system from a firmware file.
Sometimes this task can be much harder than what was demonstrated here. A
firmware file can be constructed in many different ways which makes it
difficult to analyze a bunch of them.

In the next part I am planning to go through the file system trying to find
interesting stuff, stay tuned!

# References
 - [Binwalk][binwalk]
 - [Notes on the LZMA format][lzma]
 - [/dev/ttys0](http://www.devttys0.com/)

[binwalk]: https://github.com/ReFirmLabs/binwalk
[lzma]: https://github.com/cscott/lzma-purejs/blob/master/FORMAT.md#lzma-file-format
