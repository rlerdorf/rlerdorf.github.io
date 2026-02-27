---
date: '2006-01-03'
layout: post
title: Updating my road warrior kit
---


My wife works for APC, so APC stuff is readily available. Here are a few
things that I am looking at for my [road warrior
kit](http://toys.lerdorf.com/archives/28-Whats-in-your-travel-bag.html).
I'd like to get to the point where I just have 1 plug and I am almost
there. The TravelPower adapter plugs into the wall/car/plane and it then
powers any USB devices like the mobile wifi router and the iShuffle (if
I had a cellphone - I hate cell phones, it would power that too), and
then the main power feeds into the universal battery which in turn
powers the Powerbook. When I unplug from the wall the battery takes over
and combined with the internal Powerbook battery I have about 130
Watt-hours, which for me translates into about 7 hours of laptop use.
This is what it looks like:

![](/assets/images/updating-my-road-warrior-kit/apc_gear.jpg)

The above description is what I'd like it to do, but unfortunately it
doesn't quite work. The stumbling block is the TravelPower adapter
thing.
[![](/assets/images/updating-my-road-warrior-kit/tpa90dc_sfl.jpg)](http://apc.com/resource/include/techspec_index.cfm?base_sku=TPA90DC&language=en&country_code=US&fnl=4265,3&fnl_basket=4265,4c)
My first big beef with it is that it weighs a ton. My second, and more
serious problem, is that it only goes to 20V and not 24V like the
universal battery. 24V happens to be what the Powerbook needs. I could
perhaps live with that if I could feed 20V to the universal battery and
have it output 24V to the Powerbook, but that doesn't work either. I can
charge the battery with it, and I can then power the Powerbook with the
battery, but I can't do both at the same time which defeats my goal of
being able to just plug in my bag in a single plug in the airport. My
smaller and much lighter Targus travel adapter is fine for
plane/train/car power, but since it doesn't plug into a regular plug as
well, I still need to bring something else. For now it will just have to
be my regular Powerbook power brick which can charge the universal
battery and the Powerbook at the same time and gets me most of the way
to the single plug in the airport, it just means I have to switch to the
Targus in the plane if I have power there. My product suggestion to APC
is a new version of the TravelPower adapter which is half the size and a
quarter of the weight and can go to 24V.

  

Next, the universal battery, or UPB80, is nice. It has been with me
around the world a few times now. It is small, light and cheap. You get
80 Watt-hours for about $150 when a 50 Watt-hour internal Powerbook
battery costs about the same.
[![](/assets/images/updating-my-road-warrior-kit/027C9B18-5056-9170-D3B5C34D52A80F74_pr.jpg)](http://apc.com/resource/include/techspec_index.cfm?base_sku=UPB80)
It can be a little bit flaky. If you don't set the voltage and plug
things in in the right order, it doesn't work. To fix it, just unplug
the main connector from it and plug it back in and it goes. It supports
any laptop I can think of with its 15V, 16V, 18V, 19V, 20V and 24V
settings along with a bag of various tips. You plug your existing laptop
power into one end of this silver cable that comes with it, plug the
middle into the battery and the other end into the laptop to charge the
battery and power your laptop at the same time. Then when you disconnect
from the wall the battery kicks in. Don't forget that as far as your
laptop is concerned it is still running in powered mode. It has no idea
you are using an external battery, so when running from this thing you
have to manually switch your laptop into low power consumption mode for
maximum battery life. Not much else to say about this one. It works. I
don't leave home without it.

  

I also recently picked up this very cool universal plug thing that
supports all the plug types in the world in a very small form factor.
Hard to describe. It looks like this
(unfolded):

[![](/assets/images/updating-my-road-warrior-kit/79D13042-5056-9170-D3EC4E3633A74B4E_pr.jpg)](http://www.apc.com/products/moreimages.cfm?partnum=INPA)

Try clicking on the above picture. It shows the various ways it can
unfold itself Optimus Prime-style.

  

The latest addition is the WMR1000G Wireles Mobile Router. This thing
can do AP, Router and Client mode just like my hacked WRT54G, but it is
tiny. I probably still won't give up the WRT since I use it to hook up
my huge antenna for the extreme situations where I am far from an access
point, but even then, I can put it back to back with the WRT and
rebroadcast the signal locally. I was doing that through the Powerbook
before, but it meant I had to be tethered to the WRT. Now I can be
wireless too and still use the big antenna to pick up remote networks.
[![](/assets/images/updating-my-road-warrior-kit/8F5182A6-5056-9170-D3845F56AED96B82_pr.jpg)](http://www.apc.com/resource/include/techspec_index.cfm?base_sku=WMR1000G)

It is interesting that it comes with a USB power cable along with a
regular small power block. I tried powering it from my Powerbook USB
port, but it didn't work, and there is also a big warning in the
instructions that it is only designed to be plugged into the TravelPower
Adapter thing (which doesn't help me much since that silly thing doesn't
support my Powerbook).

  

There is a little switch on the side that has 4 settings. AP, AP/Router,
Config and Client. To configure it, stick it in Config mode and connect
either via the wire or the wireless. ESSID is "default" in Config mode,
"APC\_Router" in AP/Router mode, and "APC\_AP" in AP mode. The
difference between AP and AP/Router mode seems to just be that in Router
mode you get DHCP/NAT, whereas in AP mode it is just passthrough. In
Config mode (login using Admin/APC) you have 4 main configuration
screens named, **`System Setup`**, **`AP Mode`**, **`AP/Router Mode`**,
and **`Client Mode`**. The config is pretty simple. System Setup has:

![](/assets/images/updating-my-road-warrior-kit/ss.png)

So you can set wired and wireless MAC addrs that can configure the thing
without needing a passport. Useful for the folks smart enough to figure
that out but dumb enough to forget their password?

  

On the AP Config screen you have:

![](/assets/images/updating-my-road-warrior-kit/ap.png)

No surprises here either. "Trusted Stations" means MAC filtering, and it
supports WEP and WPA-PSK.

  

On the AP/Router config screen, things get a bit more interesting.  

![](/assets/images/updating-my-road-warrior-kit/apr.png)

I am not sure what "Travel Mode (Hotel)" means under Connection type
there, but the other options are PPPoE, PPTP, L2TP and Static IP so I
guess Travel Mode means DHCP here. Weird. And under Advanced we see this
odd-looking menu:

![](/assets/images/updating-my-road-warrior-kit/adv.png)

I find it odd to see "Age of Empires" there. The last and only option
missing from the screenshot view is "Yahoo\! Messenger", so no other
games on the list. The next menu over, "Port Forwarding" lets you fully
configure it, thankfully.  

![](/assets/images/updating-my-road-warrior-kit/pf.png)

The DDNS screen has support for DynDNS, DtDNS and Cn99. The Network Diag
screen lets you do pings and dns lookups. Under Options you can
configure backup DNS servers and enabled UPnP (enabled by default). The
PC Database screen lets you hardcode static ips for the dhcp server -
Nice\! And the security screen looks like this:

![](/assets/images/updating-my-road-warrior-kit/sec.png)

Not entirely sure what they mean by a DoS firewall. What sort of DoS
attacks is it firewalling?

And finally the Client mode config screen.

![](/assets/images/updating-my-road-warrior-kit/cl.png)

This is the first place I think this router come up a bit short. There
doesn't appear to be any way to have it discover available access
points. The one you see on the screen there is one I configured manually
for my AP at home. Once you configure it to connect to a specific
wireless network, it works fine and basically just turns a wired
connection into a wireless one. I use this all the time with my WRT on
the road because the radio is stronger and more sensitive than my
Powerbook's wifi and with the big antenna connected there is no
comparison.

  

Without the ability to connect an external antenna and the lack of an AP
Scan feature in Client mode, I don't see it replacing my WRT, but I
still really like the features this tiny little thing has. And because
of its size and weight it has definitely earned itself a spot in my
travel bag.
