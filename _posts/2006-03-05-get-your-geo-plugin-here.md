---
date: '2006-03-05'
layout: post
permalink: /archives/40-Get-your-geo-plugin-here.html
title: Get your geo plugin here
---


PHP 5, JSON from pecl cvs, [Map Tile
API](http://developer.yahoo.net/maps/rest/V1/mapImage.html) and the [Yui
Connection Manager](http://developer.yahoo.net/yui/connection). Toss
them in a bag and shake and you get something like this. Enter a city
name, or even a pseudo-name like Philly or SFO and hit return. Each time
you enter a new name you will get a map tile for that city.

  
  
It's not all that fancy, but it sure is easy to do:  
  

  
  
33 lines total which includes about 6 lines of PHP which consists mostly
of building the URL to the map tile service. Then the Javascript which
has a callback function to read the form values and make the backend GET
request and a simple function to take the JSON response and add the map
tile to the document. And finally the actual HTML form.  
  
With a slight tweak you can change it to make a nice geocode lookup
field.  

  
  

  
  
So, fire up your text editors and start writing some plugins for blog
and forum packages and perhaps image gallery applications as well.
