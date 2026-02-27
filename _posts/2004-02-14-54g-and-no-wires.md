---
date: '2004-02-14'
layout: post
title: 54G and No Wires!
---


My home network has 3 annoying problems:  
**\#1** - The house has an odd shape and Christine's office is a long
way from the spot the cable comes into the house and where the wireless
gateway is. The signal strength in her office is barely adequate.  
**\#2** - I have a nice Raid5 server which I used to keep in the garage
at the previous house, but in this one the Internet connection
terminates inside the house and the machine is too noisy and ugly to
have sitting in the dining room. I'd like to get it back into the
garage, but there is no connection there currently.  
**\#3** - There is no phone jack anywhere near the TV, so I am
periodically running a long phoneline to the Tivo so it can make its
daily call. This is one of old series-1 Tivos which has one of the first
TivoNet cards in it, so if I could get network connectivity to the TV I
could make it update via that instead.  
  
The solution is of course to **buy more toys**\! In this case a couple
more WRT54G wireless gateways. At [$79.99 from
Amazon](http://www.amazon.com/exec/obidos/ASIN/B00007KDVI/lerdorf-20)
they are relatively cheap and Linksys has released all the source to the
Linux-based firmware. You can get it from their [GPL Code
Center](http://www.linksys.com/support/gpl.asp). People have taken this
code and created their own customized firmware. The best right now is
from [SveaSoft](http://www.sveasoft.com/forum6.html).  
Here is a picture of what I'd like to build:  
![](/assets/images/54g-and-no-wires/home-net.png)  
The main AP (192.168.1.1) is the one that is there now. The secondary
(192.168.1.2) is the one in the garage that I will plug my RAID5 box
into solving problem \#2. Christine's office is also above the garage,
so this secondary AP will be the one she will associate with and
hopefully get a stronger signal. That means it will have to talk to the
main AP via WDS (Wireless Distribution System) and then turn around and
talk to any wireless clients. The radio will have to flip back and forth
and I unfortunately lose half the bandwidth that way. This is 802.11g
though and currently trying to go directly from that office to the main
AP on the weak signal is causing the connection to drop back to 1Mbps as
it is. I am hoping to see close to 10Mbps in this new setup from this AP
in repeater mode.  
  
To solve problem \#3 I am going to have a third wrt54g near the TV. This
one doesn't need to repeat since it isn't that far from the main AP. All
I need from it is to connect as a regular wireless client to the main AP
and then act as a switch where I can plug the Tivo and probably the WMA
into. The WMA has wireless, but it is only 802.11b and this way I can
get more bandwidth to it by connection it to this 802.11g connected
switch.  
  
At this point I have AP \#2 installed in the garage and it is working
well. I ran some 30-second
[iperf](http://dast.nlanr.net/Projects/Iperf/) tests on it:  
  
- AP2 = WRT54G v2.0 Satori-pre1 (AP mode w/ 40-bit WEP)
- AP1 = WRT54G v1.0 Satori-pre1 (AP mode w/ 40-bit WEP)
- LAN = Various Linux servers and an XP box all with 100M NICs
- WAN = Thinkpad T20, Linux 2.6.3-rc2, Netgear WG511 802.11g card with the prism54 driver. 

```
LAN-AP2-WDS-AP1-LAN   9.2 Mbits/sec
LAN-AP1-LAN          93.8 Mbits/sec
LAN-AP2-LAN          93.9 Mbits/sec
WAN-AP2-LAN          19.5 Mbits/sec
WAN-AP1-LAN          19.9 Mbits/sec
WAN-AP2-WDS-AP1-LAN   5.1 Mbits/sec
WAN-AP1-WDS-AP2-LAN   5.8 Mbits/sec
```
A note on the above performance numbers. There are quite a few walls
between AP1 and AP2 for these measurements, so the WDS speeds are not
what they could be. I tested them next to each other as well and was
able to get it up to about 14 Mbits/sec. By locking it down to g-only,
turning off WEP and fixing the speed to 36Mb/s I was able to get it up
to 17 Mbits/sec. It's still not as fast as I would like and I think I
may just get a better antenna for the main AP and have Christine connect
directly to that while running the AP in the garage in client mode. This
should be quite a bit faster.  
  
I have ordered another WRT54G to sit by the TV, although I am also
tempted to get a decent antenna and have one sit around in monitor mode
just to keep track of what other traffic is flowing by the house here.  
  
This [Dual Diversity Flat Patch Antenna from
HyperLink](http://www.hyperlinktech.com/web/hg2411dp.php) looks like it
would be suitable to get a stronger signal to the far end of the house.
Remember that the Linksys boxes come with this weird RP-TNC connector,
so any antenna you buy for it should have a [Female Reverse Polarity
TNC](http://www.hyperlinktech.com/web/copyrighted_images/connector_rp-tnc.jpg)
on it.  
  
**\[Update March 5, 2004\]** I picked up a couple of those Hyperlink
antennas and they didn't boost things as much as I had expected. I did
hook up the WRT behind the TV as per the network diagram above and am
running it in client mode and it works well. It does seem like there is
an issue with plugging multiple clients into it in client mode, so
probably a driver bug somewhere. Nothing really to getting client mode
working. Simply set it to client mode on the wireless tab of the
firmware. However, I find that for some reason mine isn't associating
automatically. Logging in and manually doing:

``` shell
wl join Canada key 3132333435
```

seems to do the trick. (Not my real key obviously) You can then check
the status with:

``` shell
# wl assoc
SSID: "Canada"
Mode: Managed   RSSI: -40 dBm   noise: -85 dBm  Channel: 3
BSSID: 00:06:25:C5:32:21        Capability: ESS WEP ShortSlot
Supported Rates: [ 1(b) 2(b) 5.5(b) 6 9 11(b) 12 18 24 36 48 54 ]
```

One very useful feature of client mode would be to bring it on trips.
With its superior radio combined with one of the Hyperlink antennas it
should be able to pick up open gateways at quite a distance. See the
Kismet post for further details on using it for finding gateways and
with client mode you could then associate the wrt with a gateway and
plug yourself into one of the wired ports. No more messing around with
wireless drivers on your laptop.  
  
WDS mode between AP1 and AP2 is all done through the web interface as
well. Each WDS endpoint needs an ip. 10.0.0.1 and 10.0.0.2 in my case.
Then specify the MAC of the other AP in each web interface. Make sure
both are set to the same channel and same ESSID and turn off any
firewalling. There is one step the current firmware doesn't handle, so
you need to do that manually after each reboot. Either via the
Administration-\>Diagnostics tab in the web interface or by logging in,
issue this:

``` shell
brctl addif br0 wds0.2
```

to add the wds link to the LAN bridge. After doing these steps you
should be able to ping the other side.
