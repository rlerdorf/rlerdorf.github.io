---
date: '2004-12-04'
layout: post
title: Philips Streamium SL300i
---


[![](/assets/images/philips-streamium-sl300i/sl300i.jpg)](http://www.streamium.com/products/sl300i/)
I seem to have this magic $200 pricepoint barrier below which I buy
anything that I can give an IP address on my LAN. A $50 Philips discount
plus a corporate partnership discount brought it under $200 for me. It
bugged me that my Linksys Media Applicance box only does Audio and
Images and even though it has network support it doesn't make use of
that to talk to the Internet in any way. Philips has thought things
through a bit better. The Streamium can be used in 2 modes. They call it
Internet and PC-Link. In PC-Link mode it is similar to the Linksys in
that it uses Intel's UPnP Media Server protocol to fetch content from a
local server and in Internet mode it connects to a number of online
content services including Yahoo, Radio Free Virgin, Playhouse Radio,
Andante and many others. You can also feed it your own streaming
sources. Note though that it doesn't support WMA nor ASF so it can't
stream from a number of Windows-centric streaming services.

  

It has both a 10/100 port and 802.11g support. I already have a WRT54G
sitting on top of my TV so I just plugged it into that, but I did try
the wireless config and it worked easily. Plugged in my WEP key and off
it went. The Philips Media Manager software that comes with it is
however horrible. It is this big Java mostrosity that chews up a whack
of RAM and as far as I can tell only runs on Windows. Go figure. Luckily
Philips didn't restrict the Streamium to their own media manager. You
can install any UPnP media server you want. I prefer a little tiny one
called [TwonkyVision](http://www.twonkyvision.de/UPnP/). It doesn't have
a GUI but instead takes the sane approach and has a web interface. And
it isn't written in Java so it is portable(\!?) It runs well on OSX,
Linux and Windows.

  

The TV interface seems a little clunky to me. It is somewhat hard to
navigate and it seems to flicker a bit. I don't think it is me or the TV
because the TiVo onscreen interface is rock-solid without any flicker at
all. The online part of this at [my.philips.com](http://my.philips.com)
is also not great. It's not bad, but it seems like they could do a lot
more with it. That's where you go to configure what the Internet portion
of the device should connect to. I like the fact that no local machine
needs to be available to use or configure the Internet side of it. Just
go to the web site and choose which streams, video and photo services to
use. Hook this thing up to an LCD hanging on a wall and configure it to
show your Yahoo Photos or galleries from Lex Fletcher's [Born to
Shoot](http://www.borntoshoot.com/page_4.htm) site and you have a pretty
nice dynamic picture frame.

  

Prodding it just a little bit shows the following:

``` shell
PORT     STATE    SERVICE
80/tcp   open     http
1720/tcp filtered H.323/Q.931
8080/tcp open     http-proxy
```

And the web server returns:

``` shell
200 OK
Cache-Control: no-cache
Connection: close
Server: Allegro-Software-RomPager/4.30
Content-Type: text/xml
Expires: Thu, 26 Oct 1995 00:00:00 GMT
Client-Date: Sat, 04 Dec 2004 16:11:14 GMT
Client-Peer: 192.168.1.106:80
Client-Response-Num: 1
```

followed by a bunch of XML which looks like a normal UPnP Media
response. Interestingly enough it looks like other devices can use this
one as their UPnP Media source. I am sure that is in the spec somewhere,
but I never realized that before that these things could be used as
media relays.

  
  

All in all it is a cool little device.
