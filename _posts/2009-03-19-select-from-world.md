---
date: '2009-03-19'
layout: post
permalink: /archives/49-Select-from-World.html
title: Select * from World
---


I have been having a lot of fun with two Yahoo\! technologies that have
been evolving quickly. [YQL](http://developer.yahoo.com/yql) and
[GeoPlanet](http://developer.yahoo.com/geo). The first, YQL, puts an
SQL-like interface on top of all the data on the Internet. And the
second, GeoPlanet, introduces the concept of a WOEID (Where-On-Earth ID)
that you can think of as a foreign key for your geo-related SQL
expressions.  
  
First some example YQL queries to get you used to this concept of
treating the Internet like a database. Go to [the YQL
Console](http://developer.yahoo.com/yql/console/) and paste these
queries into the console to follow along.  
  
```sql
select * from geo.places where text="SJC"
 ```
 
This looks up "SJC" in GeoPlanet and returns an XML result containing
this information:

```xml
        <woeid>12521722</woeid>
        <placeTypeName code="14">Airport</placeTypeName>
        <name>Norman Y Mineta San Jose International Airport</name>
        <country code="US" type="Country">United States</country>
        <admin1 code="US-CA" type="State">California</admin1>
        <admin2 code="" type="County">Santa Clara</admin2>
        <admin3/>
        <locality1 type="Town">Downtown San Jose</locality1>
        <locality2/>
        <postal type="Zip Code">95110</postal>
        <centroid>
            <latitude>37.364079</latitude>
            <longitude>-121.920662</longitude>
        </centroid>
        <boundingBox>
            <southWest>
                <latitude>37.35495</latitude>
                <longitude>-121.932152</longitude>
            </southWest>
            <northEast>
                <latitude>37.373211</latitude>
                <longitude>-121.909172</longitude>
            </northEast>
        </boundingBox>
```

The first thing to note is the **woeid**. It is just an integer, but it
uniquely identifies San Jose Airport. If you were to search for "San
Jose Airport" instead of "SJC" you would find that one of the places
returned has the exact same woeid. So, the woeid is a way to normalize
placenames. The other thing to note here is that you get an approximate
bounding box. This is what makes the woeid special. A place is more than
just a lat/lon. If I told you that I would meet you in Paris next week,
that doesn't tell you as much as if I told you that I would meet you at
the Eiffel Tower next week. If we pretend that the Eiffel tower is in
the center of Paris, those two locations might actually have the same
lat/lon, but the concept of the Eiffel Tower is much more precise than
the concept of Paris. The difference is the bounding box. And yes,
landmarks like the Eiffel Tower or Central Park also have unique woeids.
Try it:
```sql
    select * from geo.places where text="Eiffel Tower"
```
Note that the YQL console also gives you a direct URL for the results.
This last one is at
<http://query.yahooapis.com/v1/public/yql?q=select%20*%20from%20geo.places%20where%20text%3D%22Eiffel%20Tower%22&format=xml>.
Not the prettiest URL in the world, but you can feed that to a simple
little PHP program to integrate these YQL queries in your PHP code.
Something like this:
```php
    <?php
    $url = "http://query.yahooapis.com/v1/public/yql?q=";
    $q   = "select * from geo.places where text='Eiffel Tower'";
    $fmt = "xml";
    $x = simplexml_load_file($url.urlencode($q)."&format=$fmt");
    ?>
```
For a higher daily limit on your YQL queries you can grab an OAuth
consumer key and use the OAuth-authenticated YQL entry point. There is
an example of how to use [pecl/oauth](http://pecl.php.net/oauth) with
YQL at <http://paul.slowgeek.com/hacku/examples/yql-oath.php>. Take a
close look at the YQL query in that example. It
    is:  
  
```sql
    select * from html where xpath= '//tr//a[@href="https://toys.lerdorf.com/wiki/Capital_(political)"]/../../../td[2]/a/text()'  
     and url in (select url from search.web where url like '%wikipedia%' and query='Denmark' limit 1) 
```
Sub-selects\! So, we do a web search for urls containing the string
'wikipedia' whose contents contains 'Denmark'. That is going to get us
the Wikipedia page for Denmark. We then perform an xpath query on that
page to extract the text of the link containing the name of the capital
of Denmark. Change 'Denmark' in that query to any country and the query
will magically return the capital of that country. So, YQL is also a
general-purpose page scraper.  
  
But back to the woeid. Places belong to other places, and they are next
to other places and they contain even more places. That is, a place has
a parent, siblings and children. You can query all of these. Here is a
woeid explorer application written entirely in Javascript:  
  
<http://paul.slowgeek.com/hacku/examples/geoBoundingBoxTabs.html>  
  
Try entering some places or points of interest around the world and
click on the various radio buttons and then the "Geo It" button to see
the relationship between the places and the bounding boxes for all these
various places. If you look at the source for this application you can
see that it uses YQL's callback-json output, so there is no server-side
component required to get this to work. Try doing a search for "Eiffel
Tower" and turn on the "Sat" version of the map. You can see that the
bounding box is pretty damn good. Try it for other landmarks. Then walk
up the parent tree, or across the siblings. Or check out the Belongs-To
data.  
  
Once you have a woeid for a place, you can start using it on other
services such as Upcoming:  
```sql
    select * from upcoming.events where woeid=2487956
```
and Flickr:  
```sql
    select * from flickr.photos.search where woe_id=2487956
```
(yes, I know, it would have been nice if the column names were
consistent there)  
  
And finally you can also add YQL support for any open API out there.
There is a long list of them here:  
  
<http://github.com/spullara/yql-tables/tree/master>  
  
To use one of these, try something like
    this:
```sql
    use 'http://github.com/spullara/yql-tables/raw/master/yelp/yelp.review.search.xml' as yelp; 
    select * from yelp where term='pizza' and location='sunnyvale, ca' and ywsid='6L0Lc-yn1OKMkCKeXLD4lg'
```
As a bit of Geo and API nerd, this is super cool to me. I hope you can
find some interesting things to do with this as well. If you build
something cool with it, please let me know.
