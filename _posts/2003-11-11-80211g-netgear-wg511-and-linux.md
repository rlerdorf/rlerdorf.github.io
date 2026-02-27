---
date: '2003-11-11'
layout: post
permalink: /archives/15-802.11g-Netgear-WG511-and-Linux.html
title: 802.11g Netgear WG511 and Linux
---


[![](/assets/images/80211g-netgear-wg511-and-linux/wg511.Thumb.jpg)](http://www.amazon.com/exec/obidos/ASIN/B00008I9K8/lerdorf-20)
I picked up a cheap Netgear WG511 the other day. Got it for $35,
probably because they have recently released the WG511T which uses the
Atheros super-G chipset. The older WG511 uses the Prism Duette chipset
which isn't officially supported on Linux by anybody. I say officially,
because some code has snuck out and there is a new site out there
devoted to it. Have a look at <http://prism54.org/>. I haven't tried
that driver yet, but I will update this when I do. **\[Update -
Feb.18/2004\]** I am now using the driver from prism54.org compiled into
my 2.6.3 kernel on my Thinkpad and it works nicely.  
  
For now I wanted to give the [Linuxant
Driverloader](http://www.linuxant.com/driverloader/) a whirl to see if I
could use the native Windows XP drivers directly on my Thinkpad with a
very recent 2.4.22 kernel. It worked amazingly well. See the extended
entry for the step-by-step screenshots.  
  
Of course, the whole point of going with 802.11g over 802.11b is to go
faster. I haven't done any real performance tests yet with this Windows
driver running on Linux. Hopefully I will get some time to test it
against the native driver soon. Step 1 was to plug my Thinkpad into a
wired port. How old-fashioned\! And then plug the new Netgear PCMCIA
card in. My kernel obviously didn't know what to do with it at this
point. I then grabbed the
[driverloader-1.38.tar.gz](https://www.linuxant.com/driverloader/wlan/full/archive/driverloader-1.38/driverloader-1.38.tar.gz)
tarball, ran "**make install**" and then the dldrconfig command as
shown: ![](/assets/images/80211g-netgear-wg511-and-linux/dl0.png) From then on it was
a web-based install. Cool\! So the first hurdle was to find the [Windows
XP drivers for the
WG511](http://www.netgear.com/support/support_details.asp?dnldID=424)
and actually get the .inf, .sys and .arm files out of the annoying
executable Netgear provides. I cheated and used an XP box to install
them and just copied them over from the drivers directory. They are
probably also on the CD that came with the card, but I wanted the
latest. You then feed the web interface the .inf file.
![](/assets/images/80211g-netgear-wg511-and-linux/dl1.png) It figures out that I need
the .sys and .arm files as well.
![](/assets/images/80211g-netgear-wg511-and-linux/dl2.png) It has ingested the
Windows driver and reads the MAC off of my card.
![](/assets/images/80211g-netgear-wg511-and-linux/dl3.png) Ah, an **Advanced**
button. I like those. You always find all the essential settings that
the vendors think you are too dumb to understand there.
![](/assets/images/80211g-netgear-wg511-and-linux/dl4.png) Here we find that we can
enable the power saving features of the driver.
![](/assets/images/80211g-netgear-wg511-and-linux/dl5.png) Next I need a free trial
license to activate it. Clicking through (remember I have a wired
interface up still) is easy enough. Just enter the email address and
license string you get from the Linuxant site:
![](/assets/images/80211g-netgear-wg511-and-linux/dldone.png) And you are done\! Now
just use your standard iwconfig tool like with any other wireless driver
and it just works\! ![](/assets/images/80211g-netgear-wg511-and-linux/dl7.png)
