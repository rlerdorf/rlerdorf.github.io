---
date: '2004-05-20'
layout: post
permalink: /archives/22-IBM-Thinkpad-T42p.html
title: IBM Thinkpad T42p
---


![](/assets/images/ibm-thinkpad-t42p/101075.jpg) This should be a fun
toy when it shows up. I ordered a T42p today with the 15" 1600x1200
screen and a Dothan 1.8GHz CPU. Kept the RAM and HD low and will just
add more later. Crucial doesn't list it yet, but I figure it uses DDR
PC2700 non-parity RAM just like the T41p. Now to find a decent 80G 7200
rpm notebook drive to toss in it.  
  
I did consider a Powerbook for a while, but I am too used to Linux and I
really am more comfortable on a Thinkpad. One of the big draws of the
Powerbook was the DVI-out on it, but with the Thinkpad mini-dock which
is only $89, you get that anyway, and it's not like I will be tossing my
20" LCD into my backpack and bringing it with me, so the dock can just
live permanently by the LCD for a very nice dual-headed workstation when
I am working from home.  
  
By the way, ordering this thing was amazingly painful. The ordering web
site is/was completely messed up and there was no way to click your way
to it. Had to call and have a human do it for me. If you are looking for
one, I would suggest finding someone who works at IBM who will let you
use their friends+family EPP discount and go in via
[www.ibm.com/shop/epp](http://www.ibm.com/shop/epp) or if you own some
IBM stock you can use the Shareholder Purchase Program at
[www.ibm.com/shop/us/spp](http://www.ibm.com/shop/us/spp) to get a
couple of hundred dollars off your price.  
  
**June 23rd Update**: It finally arrived\! Of course I am out of town so
I can't play with it yet. Frustrating.  
  
**July 5th Update**: Finally back in the country and have started
playing with this beast. It's the same thickness as my old T20, about an
inch wider and a bit over half and inch taller. But that Flexview
display is amazing. And no, 1600x1200 looks just fine on it. I never
really understood the argument that a display could be too small for a
high resolution. Just set your font size to your liking. The higher
resolution means your anti-aliased fonts have that much more definition
to them making them clearer and easier to read which is exactly what you
need on a "small" display.  
  
I am waiting on another [512M of
RAM](http://www.newegg.com/app/viewProductDesc.asp?description=20-145-484&depa=0)
and a speedy [7200 RPM 7K60
drive](http://www.newegg.com/app/viewProductDesc.asp?description=22-146-020&depa=0)
to install Debian on. I'll keep the 5K80 that came with it as a
secondary XP drive that I can pop in the Ultrabay the one or two times a
year I actually use Windows.  
  
**July 10 Update**: Debian has gone onto this thing. It installed pretty
smoothly. I always use this [31M XFS boot
iso](http://people.debian.org/~blade/XFS-Install/download/bootbf2_4-xfs_iso.zip)
for installing Debian these days. To do a network install just remember
to specify "e1000" when you get to the part that asks you which extra
drivers to load. Here is my [.config](http://lerdorf.com/t42p.config) in
case you are curious. I ended up using ATI's drivers for the FireGL T2
(basically a Radeon 9600 card) that is in it. There are also open source
drivers ([here](http://people.debian.org/~daenzer/dri-trunk-unstable/))
which work nicely, but the 3D acceleration wasn't very good.  
![](/assets/images/ibm-thinkpad-t42p/fgl_glxgears.Thumb.png)If you
follow [these excellent
instructions](http://xoomer.virgilio.it/flavio.stanchina/debian/fglrx-installer.html)
it is easy to get the ATI drivers going, and you will have very fast
accelerated 3D. Make sure you build your own modules instead of trying
the precompiled ones he lists. I had a world of problems with those, but
as soon as I built my own against my 2.6.7 kernel everything started
working. I used the "fglrxconfig" program to generate my
[XF86Config-4](http://lerdorf.com/t42p_XF86Config-4) file for just
single-headed 1600x1200 for now. Need to play more with the port
replicator and dual-headed stuff and also come up with a way to reliably
connect to 1024x768 projectors. I am getting around 1785FPS from
glxgears and 380FPS from fgl\_glxgears (default window sizes). Offscreen
fgl\_glxgears runs at 1125FPS.  
  
Sound works fine with the snd\_intel8x0 driver, and the built-in a/b/g
wireless works nicely with the
[madwifi](http://sourceforge.net/projects/madwifi/) driver. I use
apt-get to grab it via this entry in my /etc/apt/sources.list file:

<div class="shell">

deb-src ftp://debian.marlow.dk/ sid madwifi

</div>

Someone in the comments mentioned problems with pcmcia stuff, but I
haven't seen any issues. Anything I plug in comes up right away.  
Dual-booting to WinXP sitting on the original drive in the ultrabay
worked on the first try. I added this to my /boot/grub/menu.lst file:

<div class="shell">

title Windows XP  
map (hd0) (hd1)  
map (hd1) (hd0)  
rootnoverify (hd1,0)  
chainloader +1

</div>

The big thing I still need to work more on is ACPI and getting it to
suspend and wake back up. It suspends perfectly right now, but it just
won't come back out of suspend which makes the fact that it can suspend
much less interesting.  
  
Another interesting problem I hit was that if I used the Radeon
Framebuffer to get a cool-looking console then the fglrx ATI driver
would crash the system on switching between X and the console. If I
don't use the framebuffer for the console everything is fine. Haven't
tracked down a solution to this one. For now I just use the vesa
framebuffer for the console which works well.  
  
**July 17 Update**: I spend half my life on planes and the other half
presenting. I haven't found any way to make the former easier on me as I
absolutely hate flying, but for the latter I trawled the Net and came up
with an idea by Klaus Weidner for running a vncserver and then a viewer
onto that server session both on the local lcd and on the external vga
port. That means that now when I present I can have the contents of the
projector in a window on my desktop. This will be very nice, especially
for my duller talks as I can read email or irc while presenting without
people seeing that.  
  
First, here is my [XF86Config-4](http://lerdorf.com/t42p_XF86Config-4)
file. Note the dual fglrx device sections and the dual screen sections
and finally the single and dual ServerLayout sections. Unfortunately X
is quite unhappy starting up with the dual layout if nothing is
connected, but you can check that with a tpctl call. I use this little
startx wrapper script:

<div class="shell">

\#\!/bin/sh  
if \[ \`tpctl --id | grep "monitor type" | cut -c41\` \!= 0 \] ; then  
startx -- -layout dual;  
else  
startx -- -layout single;  
fi

</div>

So I just need to restart X to have it automatically figure out if the
second display should be enabled or not. Next, to run vncserver and the
viewers along with a window manager (metacity) and a panel I use this
script:

<div class="shell">

\#\!/bin/sh  
PWFILE=$HOME/.vnc/passwd  
  
vncserver -geometry 1024x768 :3  
sleep 1  
xvncviewer -passwd $PWFILE -shared -fullscreen -display :0.1 :3 &  
x2vnc -passwdfile $PWFILE -shared -east localhost:3 &  
xvncviewer -passwd $PWFILE -shared :3 &  
DISPLAY=:3  
metacity &  
gnome-panel &

</div>

  
As far as my suspend problems go. The problem is the ATI fglrx driver. I
would have to switch back to the radeon driver but then I would lose
tv-out and some 3d-performance. Probably not a bad tradeoff actually.
