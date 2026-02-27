---
date: '2006-02-26'
layout: post
permalink: /archives/39-Apache-1.3.34-Debian-Package-w-mod_deflate-and-no-pthreads.html
title: Apache 1.3.34 Debian Package w/ mod_deflate and no pthreads
---


Debian's Apache1 package doesn't quite do what I need. I have been
building my own and overwriting the files from the Debian package, but
that can get annoying. So I hacked my changes into the Debian source
package and built real [.debs](http://lerdorf.com/debian). I figure they
might be useful to others.  
The main changes are to get rid of -lpthread (needed by mod\_perl) and
-lexpat and to add mod\_deflate. To enable mod\_deflate make sure your
/etc/apache/modules.conf file has:

    AddModule mod_deflate.c

And in your httpd.conf add:

    <IfModule mod_deflate.c>
        DeflateEnable           on
        DeflateMinLength        1024
        DeflateCompLevel        8
        DeflateProxied          on
        DeflateDisableRange     "MSIE 4."
        DeflateVary             on
        DeflateTypes            text/css
        DeflateTypes            text/plain
        DeflateTypes            text/rtf
        DeflateTypes            text/xml
        DeflateTypes            text/javascript
        DeflateTypes            image/vnd.dwg
        DeflateTypes            image/vnd.dxf
        DeflateTypes            application/msword
        DeflateTypes            application/vnd.hp-HPGL
        DeflateTypes            application/vnd.ms-access
        DeflateTypes            application/vnd.ms-excel
        DeflateTypes            application/vnd.ms-powerpoint
        DeflateTypes            application/vnd.ms-project
        DeflateTypes            application/vnd.visio
        DeflateTypes            application/x-javascript
    </IfModule>
