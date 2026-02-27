---
date: '2013-11-12'
layout: post
permalink: /archives/58-Building-a-NAS.html
title: Building a NAS
---


The HTPC box and various computers around the house use a mix of
internal drives, external USB and eSATA drives. It is quite a mess, and
backups are sporadic at best. The HTPC especially has grown organically
with USB and eSATA as it needed more and more space.  
  
![](/assets/images/building-a-nas/nas_mess1.jpg)

So it was finally time for a decent NAS.  

The hardware:  
**Motherboard**: $132 [SUPERMICRO MBD-X10SLM-F-O uATX Server
Motherboard](http://www.newegg.com/Product/Product.aspx?Item=N82E16813182820R)  
Key features:  
  - Haswell 1150
  - DDR3 1600 ECC ram
  - Dual GigE (in case I find a cheap LACP switch one day)
  - IPMI (since the NAS runs completely headless)
  - and I like the real USB port right on the motherboard

**CPU**: $205 [Intel Intel Xeon E3-1220V3
Haswell 3.1GHz](http://www.newegg.com/Product/Product.aspx?Item=N82E16819116907)  
**RAM**: $208 2 x [Kingston 8GB 240-Pin DDR3 SDRAM ECC
Unbuffered](http://www.newegg.com/Product/Product.aspx?Item=N82E16820239399)  
**Drives**: $800 8 x [TOSHIBA PH3300U-1I72 3TB 7200 RPM 64MB Cache
SATA 6.0Gb/s 3.5"](http://www.newegg.com/Product/Product.aspx?Item=N82E16822149396)  
**SATA Card**: $115 [IBM - SERVERAID M1015 8CHANNEL PCI-E X8
SAS/SATA](https://www.serversupply.com/CONTROLLERS/SAS-SATA/SERVERAID%20M1015/IBM/46M0831.htm)  
**SAS-\>SATA Breakout Cables**: $18.50 2 x [0.5m 30AWG Internal Mini
SAS 36pin (SFF-8087) Male w/ Latch to SATA 7pin Female (x4) Forward
Breakout
Cable](http://www.monoprice.com/Product?c_id=102&cp_id=10254&cs_id=1025406&p_id=8186)  
**Case**: $79 [Fractal Design Define R4 Black Pearl Silent ATX Mid Tower
Case](http://www.newegg.com/Product/Product.aspx?Item=N82E16811352021)  
**PSU**: $40 Corsair 80+ 750W (Don't need it that beefy, but I already
had it from a previous Newegg promotion)  
**Total**: $1597.50  
  
![](/assets/images/building-a-nas/nas_parts1.jpg)  
Most of these things were picked up during various deals so you will
probably not be able to find the exact items at the prices listed. But
Black Friday is coming up, so you should be able to do better\! Newegg
had a good deal on the drives for $99 each, but they would only let me
buy 5. Bought another 3 from Amazon for $105 each. Probably not a bad
idea to have drives from 2 different batches anyway. And yes, I know, a
windowed case for a NAS box makes no sense. Windowed cases in general
make no sense, but I like these Fractal cases with the internal sideways
disk trays and the windowed version was on sale.  
  
Carl helped me put the hardware together.  
  
![](/assets/images/building-a-nas/nas_carl1.jpg)  
  
The final build:  
  
![](/assets/images/building-a-nas/nas.jpg)  
  
I like how clean things are when you just have those two SAS cables that
go to the controller.  
  
**Software**  
  
I looked at a couple of different options. Rolling my own Linux-ZFS,
OpenMediaVault, OpenIndiana, NAS4Free, FreeNAS and a couple of others
and ended up using FreeNAS. Having a FreeBSD box back in the household
seems right and my initial FreeNAS test was completely smooth.  
  
I cross-flashed the IBM M1015 as per
[http://www.servethehome.com/ibm-serveraid-m1015-part-4/](http://www.servethehome.com/ibm-serveraid-m1015-part-4/ "http://www.servethehome.com/ibm-serveraid-m1015-part-4/")
to look like an LSI9211-8i in IT mode. This disables any hardware raid
and gives the OS direct access to the drives. I had to modify those
steps slightly to use the UEFI shell to do it. You can read more about
that at
[http://brycv.com/blog/2012/flashing-it-firmware-to-lsi-sas9211-8i/](http://brycv.com/blog/2012/flashing-it-firmware-to-lsi-sas9211-8i/ "UEFI Flash").
Then I dd'ed the FreeNAS 9 image onto a decent USB stick and stuck that
stick right into the USB port on the motherboard. I had a slight issue
with it not wanting to do a warm-boot from the stick, but that was
solved by turning off xHCI in the Bios. That, of course, means no USB3
speeds, but I don't need it for anything. It boots plenty fast from the
USB2 stick.  
  
I created a single RaidZ2 16TB ZFS Volume. FreeNAS complained a bit
about that not being an optimal configuration, but to keep things simple
I prefer just having one single big volume like that. I turned on NFS
and rsyncd. Performance-wise I am essentially maxing out the GigE.  
  

    sent 2047209500 bytes  received 38 bytes  110659975.03 bytes/sec
    sent 55 bytes  received 2047209509 bytes  110659976.43 bytes/sec

I was slightly surprised that I could write to it at \> 100MB/s. I had
read that RaidZ2 writes were a bit slow. But I guess I
hardware-overkilled it into submission.  
  
I also added a dataset specifically for TimeMachine backups and turned
on AFP just for that dataset. That worked surprisingly well. The ancient
kitchen iMac saw it right away as a valid TimeMachine drive as did my
Hackintosh. And of course the Linux-based HTPC and the various Linux
servers around the house just speak NFS to it.  
  
I do notice the link getting saturated, especially with 2 TimeMachine
backups going and me trying to rsync all the various drives to it all at
the same time. It makes me want to go find an LACP-capable switch on
EBay and turn on link-aggregation. But having 5 different boxes all
writing at the same time is a one-time setup event. In normal use that
will never happen, so that would be even more overkill than the hardware
this thing already has. I'll update this entry if anything else
interesting comes up, but so far it has been a rather uneventful
project. A bit boring really when everything just works like that.
