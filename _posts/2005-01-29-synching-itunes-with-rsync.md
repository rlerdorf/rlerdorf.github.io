---
date: '2005-01-29'
layout: post
title: Synching iTunes with rsync
---


![](/assets/images/synching-itunes-with-rsync/itunes.jpg) I have an XP box that
acts mostly as a uPNP media server for the Philips Streamium and also
serves as the server for digital cameras and the other mp3 players.
However, since getting the Powerbook and now the iShuffle I have been
using iTunes on my Powerbook to manage my mp3's. So the problem was that
I wanted to be able to easily update my XP box with the iTunes music
library from my Powerbook and let Christine update her iRiver player
from it. With the [Windows version of iTunes](http://apple.com/itunes)
installed and the [UMS
firmware](http://www.iriveramerica.com/support/ums.aspx) for Christine's
iRiver 390T it is easy to use iTunes to populate the iRiver. The tricky
part was synchronizing the libraries including the ratings and play
counts.  
  
I marked my Windows iTunes folder as shared along with my music folder.
I have these on separate drives. If your music folder is in your iTunes
directory you could simplify this script a bit:

No iframe support? You can see the code at
<http://lerdorf.com/synci.phps>

When you run this it makes your Windows iTunes look exactly like your
OSX iTunes by mounting the Windows shares, copying over the hdfm file,
munging the paths in the XML file and rsyncing the music folder which
means you will lose any library properties local to your Windows box. I
would also suggest exiting iTunes on both machines before running this
script and synching your clocks from an ntp server. The clock on my XP
box drifts badly so I had to add --modify-window to the rsync command.  
  
It should be simple to reverse the script if you want your Windows
iTunes to be the master, and with a bit of work you could probably make
it go in both directions. But this serves my needs nicely.
