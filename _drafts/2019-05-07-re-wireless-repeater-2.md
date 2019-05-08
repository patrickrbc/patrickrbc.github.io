---
layout: post
title:  "Reverse engineering a wireless repeater - Part II"
date:   2019-05-07 15:34:00 -0300
comments: true
categories: 
---


This is the second part of a series of posts about reverse engineering a
wireless repeater. Before reading this you might want to check the [first
part](/2019/04/01/re-wireless-repeater) where I introduced the subject and
explained how to gather some information without digging into the hardware. If
you still haven't downloaded the firmware of our target device you can get it
[here](http://en.intelbras.com.br/sites/default/files/downloads/fw_nplug_1_0_0_14.zip).

Firstly, make sure you have extracted the zip file.

```
$ file fw_NPLUG_1_0_0_14.bin
fw_NPLUG_1_0_0_14.bin: u-boot legacy uImage, Linux Kernel Image, Linux/MIPS, OS
Kernel Image (lzma), 1731916 bytes, Wed Oct 12 17:28:28 2016, Load Address:
0x80000000, Entry Point: 0x802CB000, Header CRC: 0xEC8AD091, Data CRC:
0x3A3E5EAC
```

Binwalk is a tool that analyses a binary searching for known file format
signatures. Also, it can do a visual entropy analysis.

TODO:

- binwalk entropy
- binwalk extraction



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

In this post we learned how to extract the file system from a firmware. This
taks can be much harder than what was demonstrated here. A firmware file can be
constructed in many different ways which makes it difficult to analyze a bunch
of them.

# References

[zaddach-2013]: http://s3.eurecom.fr/docs/bh13us_zaddach.pdfk
