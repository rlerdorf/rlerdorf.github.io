---
date: '2003-11-09'
layout: post
title: 802.11g PCI card options
---


[![](/assets/images/80211g-pci-card-options/airstation.Thumb.gif)](http://www.amazon.com/exec/obidos/ASIN/B00009967Z/lerdorf-20)
Buffalo WLI-PCI-G54 uses the Broadcom chipset and has a cool-looking
external antenna. At this point I think the only hope of getting this to
work with Linux would be through [Linuxant's driverloader](http://www.linuxant.com/driverloader/).  

<div style="clear: both;"></div>

[![](/assets/images/80211g-pci-card-options/dwl.Thumb.jpg)](http://www.amazon.com/exec/obidos/ASIN/B00007LTBI/lerdorf-20)
D-Link DWL-G520 uses the Atheros 5002 chipset. Should work with the
madwifi driver, or with the Linuxant driverloader. It also claims to
support 108 Mbps Extreme-G.  

<div style="clear: both;"></div>

[![](/assets/images/80211g-pci-card-options/wg311.Thumb.jpg)](http://www.amazon.com/exec/obidos/ASIN/B00009YW8B/lerdorf-20)
Netgear WG311 uses the Intersil Prism GT chipset which has no native
Linux driver that I know of. But the Linuxant driverloader says it
supports it.  

<div style="clear: both;"></div>

[![](/assets/images/80211g-pci-card-options/wmp54g.Thumb.jpg)](http://www.amazon.com/exec/obidos/ASIN/B000085BD8/lerdorf-20)
Linksys WMP54G uses the Broadcom chipset and should work with the
Linuxant driverloader.  

<div style="clear: both;"></div>

[![](/assets/images/80211g-pci-card-options/wmp55ag.Thumb.jpg)](http://www.amazon.com/exec/obidos/ASIN/B00008RUKX/lerdorf-20)
The Linksys WMP55AG is an a/b/g card whereas all the previous were just
b/g. This one uses the Atheros 5212a chipset and should work with both
the madwifi driver and the Linuxant driverloader. This one is actually
just a PCI card with a mini-PCI adapter on it with a mini-PCI card plugged into it.  

<div style="clear: both;"></div>

[![](/assets/images/80211g-pci-card-options/dwl-ag520.Thumb.jpg)](http://www.amazon.com/exec/obidos/ASIN/B00008YGMS/lerdorf-20)
D-Link DWL-AG520 uses an Atheros chipset and should be supported by both
the madwifi and the Linuxant driverloader. Like the Buffalo, it has a
nice beefy external antenna.  

<div style="clear: both;"></div>

[![](/assets/images/80211g-pci-card-options/wag311.Thumb.jpg)](http://www.amazon.com/exec/obidos/ASIN/B0000C120S/lerdorf-20)
Netgear WAG311 like the other a/b/g cards is Atheros-based so it should
work with both the madwifi and the Linuxant driverloader.  
It probably would be a good idea to get an Atheros Super A-G based board
so I can go 108Mbps when the drivers support it and when I get a gateway
that can go that fast. I don't think my Broadcom-based WRT54G is going
to be able to support Super-G. I think the Linksys, D-Link and Netgear
a/b/g cards are all based on the same Atheros chipset, so the only
deciding factor is likely to be price between these. If anybody has one
of these and can inject a bit more data it would be appreciated. I will
update and bump this up as I learn more.
