---
date: '2004-12-20'
layout: post
title: The Good, The Bad and The Ugly Powerbook
---


![](/assets/images/the-good-the-bad-and-the-ugly-powerbook/IMG_7327.JPG) After the rather
abrupt loss of my T42p I needed a new laptop quickly. The delay in
getting the Thinkpad originally combined with the headache of getting
Linux working on it nicely and also to some extent IBM dumping their PC
division all contributed to the decision to go with a 15" Powerbook. If
IBM would have supported a real OS on their hardware I would have bought
another one in a second. I have never been a Mac fan, but I needed a
working Unix laptop quickly. After years of having an absolute crap
operating system, they finally have a good one. This thing is a 1.5GHz
G4 with a half a Gig of ram and the 128M Radeon and then another 512M
from Crucial (http://www.crucial.com) added on. Would have been nice to
put 2G in it, but those 1G sticks are really expensive.  
  
After using it for a week and a half I am quite pleased with it overall.
Plenty of little gripes, but overall it is a useful little machine.  
  
<span style="font-weight: bold;">The Good (with a few reservations)  
</span><span style="font-weight: bold;"></span>

  - Stuff works. I didn't have to fight apm/acpi to get it to sleep
    properly when I shut the lid, nor did I have to massage my wireless
    drivers and give them a little tune-up every week.
  - It's a slick-looking block of aluminum, but it doesn't feel nearly
    as sturdy as the Thinkpad. I guess we will see if it holds up to my
    abuse and the many around-the-world trips this thing is going to
    have to endure.
  - The battery status indicator on the battery itself is nice.
  - The dock is cool-looking, but it quickly gets on your nerves. I
    found QuickSilver
    ([http://quicksilver.blacktree.com/](http://quicksilver.blacktree.com/ "QuickSilver"))
    to be a much quicker way to launch things without having to deal
    with the damn trackpad.  
  - Related to the last point, the very non-Unixy approach to installing
    applications simply by dragging them to the /Applications directory
    is very handy, especially the way everything is self-contained, but
    it does make it hard to run things from the command line. I tried
    the BBEdit demo, for example, and while I finally did figure out how
    to launch it from the shell, it wasn't very convenient. Not sure how
    you are supposed to use an editor that you can't quickly launch from
    the shell. I guess that's why QuickSilver exists. Vi/vim has worked
    fine for the past 20 years for me, and it will work fine for the
    next 20.  
  - Fink
    ([http://fink.sourceforge.net/](http://fink.sourceforge.net/ "Fink"))
    makes me feel somewhat at home giving me apt-get for most of the
    common packages I use. It was trivial getting my development
    environment set up with Xcode and all the various libraries and
    tools I need to build PHP. You also quickly become familiar with
    Versiontracker
    ([http://www.versiontracker.com/macosx/](http://www.versiontracker.com/macosx/ "Versiontracker"))
    for finding OSX applications.  
  - I like the fact that my ancient RG-1000 wireless gateways that I
    have a bunch of lying around in the garbage pile seem a lot more
    useful now since I can flash them with the Apple Airport firmware
    and make them look like real Airport AP's. It was always a massive
    pain trying to configure these things from Linux via the Java
    configurator thing.
  - Expose is just
    cool\!![](/assets/images/the-good-the-bad-and-the-ugly-powerbook/expose.gif)  
  - The backlit keyboard has a high coolness factor although it seems a
    bit too dim to really be useful. It also doesn't light up the row of
    function keys. A way to turn it on on demand as well as a way to
    crank up the brightness would be nice. It may be hiding in there
    somewhere, but I haven't run across it yet.  
  - The light around the power plug is cool too, except this one, unlike
    the keyboard backlight is really bright and doesn't turn off when
    the machine goes turns off the lcd so twice now my wife has
    repositioned the idle laptop sitting in the bedroom out of sight.  
  - The Skype ([http://www.skype.com](http://www.skype.com "Skype"))
    client is pretty and works very well.  
  - There is a plethora of decent web browsers. It came with Safari and
    IE. Add Mozilla, Firefox and Camino to that and you have 5 browsers
    that all seem to work well. I am used to Firefox from Linux, so I am
    sticking with that.

  
The Bad (with some workarounds)  
  

  - For some reason there is no way to do multiple virtual desktops in
    the standard GUI. Desktop
    ([http://wsmanager.sourceforge.net/](http://wsmanager.sourceforge.net/ "DesktopManager"))
    manager solves that annoyance. Almost anyway. It would be much more
    useful if Apple-Tab only cycled through the applications on the
    current desktop and not all of them. I am guessing they can't hook
    in and change this without being more integrated with the GUI.
    (boxes on the left in the menubar image above)  
  - iTunes is pretty nice, but why in the world doesn't it have a way to
    minimize it to the menu bar. If they are going to force me to keep
    that stupid menu bar on my screen at all times, the least they could
    do is make use of it. Luckily there is a very cheap little tool
    called Synergy
    ([http://wincent.com/a/products/synergy-classic/](http://wincent.com/a/products/synergy-classic/ "Synergy"))
    that fixes that problem. With Synergy installed using the "tabbed"
    (arrows in the menubar image above) look in the menu bar iTunes is
    great and would be in the "Good" section if I didn't have to buy
    this add-on Synergy thing to make it usable. It almost makes me want
    to pick up an ipod just to play with the integration.
  - I was also surprised how badly it reacted when I fed it an xvid AVI
    file. I thought this thing was a multimedia monster. VLC
    ([http://www.videolan.org/vlc/download-macosx.html](http://www.videolan.org/vlc/download-macosx.html "VLC"))
    took care of that.
  - I hate IM with a passion, but unfortunately I need to use it. Fire
    ([http://fire.sourceforge.net/](http://fire.sourceforge.net/ "Fire"))
    seems to do the trick, but I really haven't looked around much for
    anything better. My only real comment on this one is that the icon
    sure is ugly and that it hasn't crashed on me yet. Then again, it
    doesn't really understand Yahoo's status message stuff and Yahoo
    Messenger
    ([http://messenger.yahoo.com](http://messenger.yahoo.com "Yahoo Messenger"))
    isn't too bad so I have been switching between these two.
  - For irc I am used to using XChat. The Aqua version
    ([http://xchataqua.sourceforge.net/](http://xchataqua.sourceforge.net/ "XChat Aqua"))
    isn't great, but it works. It really could use an update to the
    current xchat code. The tabs are centered, which is odd, and when
    you get disconnected from an SSL'ed connection it gets an error
    trying to reconnect. I have to restart it in order to get it to
    connect again. I should probably just install X11 and run the real
    xchat instead, but I haven't gotten around to that yet.
  - Pine from fink works nicely. But I have been trying to join this
    century by using a graphical email client. I don't think I will be
    able to though. Mail.app is really slow at dealing with huge
    mailboxes over IMAP. Thunderbird is better and I am close to being
    able to use it, but why doesn't it let me map 'R' to Reply-All?
    Using
    ([http://mozilla.dorando.at/keyconfig.xpi](http://mozilla.dorando.at/keyconfig.xpi "keyconfig.xpi"))
    I can map it to Ctrl-R or Apple-R, but not simply R. And yes, I know
    this doesn't really have anything to do with the Powerbook or OSX,
    but since I am whining about stuff I figured I would throw it in. I
    also tried offlineimap
    ([http://offlineimap.sourceforge.net](http://offlineimap.sourceforge.net "Offline IMAP"))
    to try to speed up the IMAP and give me better disconnected support,
    but it uses Maildir and having 100,000 files in a directory
    apparently isn't something the OSX filesystem handles very well.
  - I have been using Kismet for years on Linux. Kismac
    ([http://binaervarianz.de/projekte/programmieren/kismac/](http://binaervarianz.de/projekte/programmieren/kismac/ "Kismac"))
    is the OSX version and it seems a bit flaky. It has hung on me a few
    times and it needs to do weird driver swaps because the native OSX
    drivers don't support promiscuous mode. The built-in wireless card
    doesn't seem to do promiscuous mode at all even with the driver
    swap, but my Cisco and Orinoco pcmcia cards both work ok with it.
  - Apple left/right to switch between terminal windows is a nice touch,
    especially since Apple-Tab only cycles through a single window of
    each running application. But Terminal seems to be the only
    application that supports this. The same thing for Firefox would be
    very handy.

  

![](/assets/images/the-good-the-bad-and-the-ugly-powerbook/IMG_7326.JPG) The Ugly

  - No dedicated Page-up/down keys\! I never realized how much I used
    those until I didn't have them anymore. Fn-up/down is the same
    thing, but you need two hands for that.
  - Too many modifiers\! Was Fn, Ctrl and Alt really not enough? Why an
    Apple key where the Alt key should be?
  - What's with the two Enter keys? Wouldn't it be more useful to have
    two Ctrl or two Alt keys instead? I definitely don't need two Apple
    and two Enter keys right next to each other. Obviously it is there
    for the 3 people left in the world that actually uses numlock and
    the keyboard as a numeric keypad replacement. So I have a proposal.
    Let's just deprecate numlock completely and give me a useful key
    there instead\!  
  - Inconsistency in keyboard access to various widgets. Some dropdown
    boxes don't have keyboard accelerators. 'u' to get you to "United
    States" in a long country dropdown, for example.
  - Not being able to hide the top menu bar is really hard to get used
    to. Coming from a 1600x1200 screen on the T42p down to this 1280x854
    screen I am already feeling quite cramped, especially vertically so
    losing another 16 pixels to a mostly useless menu bar is annoying.
    You can't even change the font in it or do anything to make it
    smaller as far as I can tell. Would it be so bad to provide a way to
    autohide it and move/resize it? Or even better, let me dock it.
  - The damn clock widget in the menu bar won't show me the day of the
    month. You can toggle showing the day of the week along with AM/PM
    and 12/24 hour displays, but you can't get it to say "Dec.20 11:00".
    I am usually with it enough to know what day of the week it is, but
    I am always forgetting the day of the month. I know it shows up in
    the menu when you click on it, but that means I have to move the
    mouse, click and then hit escape instead of just glancing up there.
    There are replacement things like wclock that will do this of
    course, but having to run yet another process just for that minor
    thing is dumb.
  - To keep complaining about the clock widget, why doesn't it show
    something useful, like the damn date, when I hover over it? Some
    applications, like Syngery, shows you something useful on hover from
    the menu bar, so I know it is technically possible. There are many
    other places where throwing in hover support would be nice.
  - ![](/assets/images/the-good-the-bad-and-the-ugly-powerbook/IMG_7330.JPG) I of course knew
    about the trackpad and single mouse button issue and I knew I would
    have problems with that, and I do. I was however under the
    impression that the GUI was designed in such a way that you really
    didn't need more than a single mouse button, but it turns out there
    are plenty of places where you really need to right-click
    (Ctrl-Click) on stuff. Like if you want to empty the trash without
    multiple clicks. A USB mouse mostly solves this annoyance, but that
    restricts me to having a surface nearby to use the mouse on.
  - I had heard iPhoto was slow. And it really is slow. It's slower than
    slow. I find myself pondering what it could thinking about while it
    is chewing on my photos. I hope it is doing something worthwhile
    with my cpu, like curing cancer, while it is grinding away. It is
    good for organizing photos as long as you aren't in a hurry and with
    the iPhotoToGallery
    ([http://zwily.com/iphoto/index.xsl](http://zwily.com/iphoto/index.xsl "iPhotoToGallery"))
    plugin it is really easy to keep the online photo album in synch as
    well.
  - Sticking with the iPhoto gripes. When I import photos from a fast
    Sandisk Ultra 512M CF card in a PCMCIA adapter iTunes skips. This is
    a 1.5GHz CPU with a Gig of RAM. Can it really not copy a file from a
    CF card and play an MP3 at the same time? Given all my other gripes,
    this is probably the most disappointing.  
  - Scrolling text input boxes in Firefox leave cursor artifact garbage
    on the screen. Probably something the Firefox folks are doing wrong
    and not Apple's fault. But still an annoyance I didn't have under
    Linux.
  - Why only a slow DVD-R drive? Every modern DVD burner out there these
    days doesn't care if you feed it DVD+R or DVD-R media. Seems like it
    is time to update that. And get one that isn't so noisy.
  - No amplified audio line-in. Means you need an amplified mic and all
    the cheap headsets out there designed for voice chat won't work. I
    suppose the answer is to use a USB headset instead.
