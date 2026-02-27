---
date: '2004-09-11'
layout: post
permalink: /archives/23-Gallery-and-the-Coral-Distribution-Network.html
title: Gallery and the Coral Distribution Network
---


The [Coral Distribution Network](http://www.scs.cs.nyu.edu/coral/) (CDN)
is a very nice shiny toy for all of us who sometimes struggle with
limited bandwidth. Let the [NSF](http://www.nsf.gov) pay for it\!  
  
My photo album is what chews up the most bandwidth from my site, so it
made sense to rig it up first to optionally be available via CDN. CDN is
basically just a big Akamai-like network of servers that cache things
and serve them up to you from servers that are close to you
network-wise. It is has some fancy code behind it, but who cares how it
works, it just does. Let's get to the interesting part. As you have
probably figured out by now, you can access any site on the Web via
Coral by simply appending ".nyud.net:8090" to the domain part of the URL
and there are plugins available that add a menu item to automatically do
this for you client-side. The problem with this is that you don't want
to have to do this for every page on a site. It would be much nicer if
the site would adjust its links such that if you enter the site via
Coral, all the local links from that site would be Coral links. You may
of course want to make some exceptions for pages that are interactive in
some way, but for something like a photo album where 99% of people just
look at mostly static content it works well.  
  
You can see the patch at
[http://gallery.menalto.com](http://gallery.menalto.com/index.php?name=PNphpBB2&file=viewtopic&t=20810)  
  
Note again that this quick hack does not in any way handle links that
shouldn't be Coral'ed. Like the login link, comment stuff or all the
administrative tools. It turns your Gallery into a read-only site if you
come in using Coral. As the administrator of my album I can figure out
that I need to go directly to it in order to add photos. But a more
complete patch that doesn't Coralize stuff that shouldn't be would be
nice.  
  
You can have a look at it in action at
<http://www.phpics.com.nyud.net:8090/>  
  
I submitted it to the [Gallery
Folks](http://gallery.menalto.com/index.php?name=PNphpBB2&file=viewtopic&t=20810)
so let's see if they take it and run with it to integrate it
completely.  
  
**Update:** A variation of this that also supports people running their
Gallery on weird ports has been committed to Gallery's CVS, so you may
just want to update from there to get it.
