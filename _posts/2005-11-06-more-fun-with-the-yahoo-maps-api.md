---
date: '2005-11-06'
layout: post
permalink: /archives/36-More-fun-with-the-Yahoo!-Maps-API.html
title: More fun with the Yahoo! Maps API
---


[![](/assets/images/more-fun-with-the-yahoo-maps-api/screenshot27.Thumb.png)](http://lerdorf.com/map?location=San%20Francisco,CA&what=4*%20Mexican%20Restaurants)
I spent a few hours with the Javascript-Flash Yahoo\! Maps API today.
The result is here:  
  
     
[http://lerdorf.com/map](http://lerdorf.com/map?location=San%20Francisco,CA&what=4*%20Mexican%20Restaurants)  
  

Have a look at the source. There is a source link at the top-right on
the page. It is about 90 lines of HTML and Javascript and virtually no
server-side scripting.  
I usually throw PHP code at all sorts of problems, but in this case I
could do pretty much everything I wanted to directly from Javascript via
the excellent API. The initial zooming from way out is a bit distracting
and doesn't work too well, but I wanted to play with that a bit. Once
you are zoomed in and searching for things, the search is updated as you
move around the map and everything is event-based so the page doesn't
need to be redrawn from scratch.

  

Both input fields are free-form. You can put a city name, a zip code or
a full address in the Location field and in the What field you put what
you are looking for. There is a special hack that checks for something
like 4\* and the filters the results to only show you those entries that
were rated 4\* or higher on Yahoo\! Local. You can of course also just
put something like "pizza" or "mexican" in that field.

  

Most of the magic here is done by 2 things. The LocalSearchOverlay and
the event handling. Note how Map.EVENT\_MOVE and Map.EVENT\_ZOOM\_END
are registered events in the onInitialize() function. When you scroll
the map or zoom it the onOverlayInit function will get called and the
LocalSearchOverlay will be recalculated for the new map coordinates.
Same thing happens when you change something in the input fields. The
updateMap() function is called which will center the map at the new
location and update the LocalSearchOverlay appropriately. There are a
few more Javascript tricks here and there in it, like updating the link
at the top so you always have a way to grab a link to your current
search and send it to someone, but other than that there really isn't
all that much to figure out here. Once you understand which events
happen when and which methods are available where, you can do some
really powerful things with this. It is all documented here:  
  
<http://developer.yahoo.net/maps/flash/V2/flashReference.html>  
  
and people are discussing it here:  
  
<http://groups.yahoo.com/group/yws-maps/messages>

  

In my last entry I showed how to parse a geocoded XML file and put
markers for each entry on the map. Someone asked me how I would do the
using the JS-DHTML API instead of the JS-Flash API. It's a whole lot
harder in DHTML, but it works. You can see it here:  
  
<http://lerdorf.com/php/ymap/dquakes.php>  
  
And I talk a bit about it here:  
  
<http://groups.yahoo.com/group/yws-maps/message/612>
