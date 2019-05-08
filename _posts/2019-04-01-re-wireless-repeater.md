---
layout: post
title:  "Reverse engineering a wireless repeater - Part I"
date:   2019-04-01 17:21:00 -0300
comments: true
categories:
---

Do you have an old router left behind in your room? Ever wondered how it really
worked inside? Sit tight and get ready, because this is going to be a series of
blog posts about reverse engineering a wireless repeater. The methodology here
can be applied to other IoT devices.

For the first part, we are going to understand the basics and learn how to
collect information about our target. If you have any questions, please leave
a comment below so I can improve this article.

# What is a firmware?

The term "firmware" was firstly used in a magazine, published in 1967, and it
was referring to the CPU microcode existent between the hardware and the
software. Today, the term is more broadly used to refer to the BIOS,
bootloaders or embedded management systems.

There are many different kinds of firmwares files. It could be a complete
operation system with custom drivers and user applications, or it could contain
only the user applications. Sometimes, it could be composed only of partial
updates for an application, for example, the front-end of an embedded web
server.

# Why should we do this?

SOHO routers are the front door of your home network, therefore, a security
issue in this device might compromise all the connected clients. On the other
hand, SOHO routers are known for having security issues that are widely
exploited by malwares. In addition, the source code of the firmware is usually
not public. Unless someone does a reverse engineering on it, we won't be able
to know what is actually running inside it.

Another motivation is learning. Reversing SOHO routers is a good way to study
about common low-level vulnerabilities and reversing due to its simplicity
compared to modern systems.

# Information gathering

For our case study we are going to use an old wireless repeater named
**NPLUG**, previously offered by the Brazilian company **Intelbras**.
Unfortunately, I couldn't find this product on their website anymore, but this
is actually good for everyone. :)

Firstly, it is good to gather as much information about the device as you can.
This allows us to get a general idea of how it works and uncover some weakness
without the need to look inside. An easy way to do that is looking for the FCC
ID number of the device. The FCC ID is the registration number of that device in
Federal Communications Commission of United States. Using this number you can
find more information like user manuals, specifications and internal pictures.

Once you have that number you can search for it using sites like [FCC
ID.io][fccid-io] or the official [FCC][fccid-search] page. Depending on where are
your device distributed you might not find a FCC ID number on it, but another
certification number instead.

In Brazil, Anatel is the agency responsible for registering and regulating electronic devices.
Taking a look at the back of our device, between other information, we can find
a number near the Anatel logo. This is the Anatel's certification number and,
fortunately, [FCC ID.io][fccid-io] has a database for that too. 

![NPLUG wireless repeater](/assets/nplug/nplug.png)

I couldn't find much information (beyond what was on the vendor's website)
about the device using this number. However, if you are into embedded devices
you might have noticed similarities between the NPLUG and other wireless
repeaters. I had the same feeling and while looking at similar devices I
discovered a "cousin" of NPLUG, which is the [Tenda A5s][tenda-a5s].

Searching for "*fcc id tenda a5s*", I was able to find the [FCC
ID.io][fccid-a5s] page for the Tenda product and within it I could internal
pictures of the device. Looking at the information found I realized that both
devices are almost the same. They were just sold with a different name, by a
different company, in a different country. This is something that was
previously pointed out by other security researchers. The code reuse between
router manufacturers has propagated not only the same features but also the
same security holes between devices.

![Internal photos Tenda A5S](/assets/nplug/internal.png)
<center>
  <small>
    Source: https://fccid.io/V7TA5S/Internal-Photos/Internal-Photos-1889450
  </small>
</center>
<br>

# Firmware acquisition

The vendor website usually provides a download section where you can get
manuals and the firmware file. If this is not the case you might get lucky
searching for it in your search engine. For this series we are going to use the firmware of the NPLUG wireless repeater
which can be acquired [here](http://en.intelbras.com.br/sites/default/files/downloads/fw_nplug_1_0_0_14.zip).

When none of the above options is available you need to extract the firmware image from the hardware using one of the techniques that will be briefly described on the following posts.  

# Conclusion

This is all for now. I hope I had increased your appetite for what is coming
next.  Make sure you subscribe to the RSS feed and follow me on Twitter so you
can see when the Part 2 is released.


# References

 - [Zaddach, J., & Costin, A. (2013). Embedded devices security and firmware
   reverse engineering. Black-Hat USA.][zaddach-2013]
 - [FCC ID Search Form][fccid-search]
 - [FCC ID.io][fccid-io]

[tenda-a5s]: https://www.aliexpress.com/item/English-Firmware-Tenda-A5S-Mini-Router-Pocket-WiFi-Wireless-Router-Client-Universal-Repeater-WISP-150Mbps-Ethernet/32751330210.html
[fccid-io]: https://fccid.io/
[fccid-a5s]: https://fccid.io/V7TA5S
[fccid-search]: https://www.fcc.gov/oet/ea/fccid
[zaddach-2013]: http://s3.eurecom.fr/docs/bh13us_zaddach.pdfk
