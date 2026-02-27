---
date: '2003-12-16'
layout: post
permalink: /archives/1-Linksys-Wireless-Media-Appliance-updated.html
title: Linksys Wireless Media Appliance [updated]
---


**\[update\]** As Sean Lincolne pointed out, squishguava, the boot image
this thing loads over the network when it boots, is just a simple cramfs
image. The "COMPRESSED ROMFS" in my earlier 'strings' is a dead
give-away on this. I finally got some time to have a peek at it. It has
the following in
it:

|                  |                     |                    |             |
| ---------------- | ------------------- | ------------------ | ----------- |
| libc-2.1.3.so\*  | libm-2.1.3.so\*     | librms.so\*        | rio\*       |
| libdl-2.1.3.so\* | libmp3msp.so\*      | libthreadutil.so\* | scripts/    |
| libhttpmsp.so\*  | libmrdi.so\*        | libwmamsp.so\*     | version.txt |
| libiupnp.so\*    | liboalmsp.so\*      | mrd\*              | web/        |
| libixml.so\*     | libpil.so\*         | mrdDevice\*        |             |
| libllmsp.so\*    | libpthread-0.8.so\* | pilreg.dat\*       |             |

It is obviously not a full Linux filesystem, as can be seen by looking
at files in the scripts directory, this filesystem gets mounted on
`/guava`. I compiled a small statically linked busybox and stuck it in
`bb/` which among other things had `bb/sbin/telnetd` in it. Then I
hacked up `scripts/rio-script` and had it launch the `telnetd` right
before it launches the `rio`. Built a new cramfs squishguava image and
booted with it. It actually booted just fine. I was worried they might
have some sort of checksum check, or my added 600k would overflow
something, but it was fine. The bad news is that I couldn't get in via
telnet. So, either, my telnetd didn't start properly, or the port is
blocked.  
  
I did notice that the default shell is `/bin/ash` which also happens to
be the default shell in BusyBox and Linksys has used Busybox on other
devices in the past. I bet the rom firmware which has the root
filesystem for this thing is just a busybox image. So a bit more hacking
on that rio-script should let me either somehow get a message out to me
by trying various standard busybox commands, or I can run some stuff to
try to deblock the port. Any suggestions on what is likely to work right
away?  
  
More info. I replaced the rio binary with my arm cross-compiled telnetd
binary and it then doesn't get beyond the "Launching remote-IO" message
during boot. At least it tells me that what I am doing has some effect.
But I still can't get in via telnet. I also tried replacing it with a
script that tried to ping out, cat stuff to /dev/dsp and echo stuff to
various devices and none of that did anything I could
see/hear.  
**\[/update\]**  
  
[![Linksys WMA11B](https://m.media-amazon.com/images/I/319XP4X37CL._AC_.jpg)](http://www.amazon.com/exec/obidos/ASIN/B00009PUM0/lerdorf-20)
This looks like a nifty little box that will make it easy to access
mp3's and photos directly from a remote-control TV-displayed interface.
Much nicer than needing to stick a PC next to the TV/Stereo in the
living room.  
  
![](/assets/images/linksys-wireless-media-appliance-updated/tv1.Thumb.jpg) This little device
showed up today. Had no trouble configuring it and hooking it up once I
shuffled the various cables around a bit on the back of the TV and
stereo. The music navigator is really nice on it and I like that you can
play mp3's while a photo album is cycling through. Will have to try this
thing against some Samba shares later on.  
  
No luck on the Samba shares, or any sort of shares at all actually. I
did a bit of sniffing of the datastream between the WMA and XP. When it
starts up the first thing it does after getting an IP via DHCP is to
grab its OS image from the XP box. That image is clearly a Linux 2.4.17
kernel and all communications appear to be via a UPnP A/V Media Renderer
SOAP thing. As far as I can tell, when you designate a directory via the
Media tool on the XP box, it creates a regular .m3u playlist out of that
and serves it up to the WMA when requested. There doesn't appear to be
any encryption involved, so getting this thing to work with a Linux box
as the server would involve creating a UPnP SOAP server that understood
the requests from the WMA. Not that this is a trivial effort, but
certainly not impossible and once done this thing would be able to serve
files up from anywhere a Linux box could access files from. Frankly I
don't see why the heck the SOAP server they provide for XP can't serve
up its playlists from a network share. There doesn't appear to be any
technical reason for this restriction. I bet that with a bit of hacking
and with the help of [libupnp](http://upnp.sourceforge.net/) this is
quite feasible.  
  
Or, alternatively, create a custom image from the sources Linksys is
supposed to provide. They have their [GPL
Page](http://www.linksys.com/support/gpl.asp) but it doesn't list the
WMA11B (yet?). As George notes, SOAP isn't exactly ideal for something
as simple as moving mp3s and image files around. An alternate image that
was able to mount shares directly, would be cool. It might require
sticking a .m3u playlist file in each directory so you wouldn't need to
do that on the WMA, but that wouldn't bug me either.  
  
For more info on the technology in this device, have a read through
[Intel's Digital Home
Site](http://developer.intel.com/technology/digitalhome/) or see the
extended entry for some nitty-gritty protocol details.  
Don't ask me why the boot image is called squishguava, but it is. Can't
gleam too much out of it other than the fact that it is very likely to
be a Linux image.
```
    % strings squishguava
    Compressed ROMFS
    6i8V
    Compressed
    version.txt
    libhttpmsp.so
    libiupnp.so
    libixml.so
    libllmsp.so
    libmp3msp.so
    libmrdi.so
    liboalmsp.so
    libpil.so
    librms.so
    libthreadutil.so
    libwmamsp.so
    mrdDevice
    pilreg.dat
    libc-2.1.3.so
    libdl-2.1.3.so
    libm-2.1.3.so
    libpthread-0.8.so
    scripts
    channelmgrscpd.xml
    remoteinputscpd.xml
    riodevicedesc.xml
    rioscpd.xml
    ConnectionManager.xml
    MediaRendererDevDesc.xml
    RendererControl.xml
    TCxml.xml
    Transport.xml
    rio-script
    mrd-script
```

The data stream when you fire this thing up is much more telling. Near
the start we see a, "Hey I am Awake" message. 172.16.10.100 is the WinXP
box and 172.16.10.105 is the WMA.

```
    NOTIFY /AdapterInfoService/event HTTP/1.1
    HOST: 172.16.10.100:8037
    Content-Type: text/xml
    NT: upnp:event
    NTS: upnp:propchange
    SID: uuid:1
    SEQ: 0
    Content-Length: 324
    <?xml version="1.0" encoding="utf-8"?><e:propertyset xmlns:e="urn:schemas-upnp-org:event-1-0"><e:property><BootImageVersion>&lt;firmware&gt;
     &lt;version&gt;Ver. 11 R06&lt;/version&gt;
     &lt;time&gt;09:50:28 AM&lt;/time&gt;
     &lt;date&gt;08/01/03&lt;/date&gt;
    &lt;/firmware&gt;
    </BootImageVersion></e:property></e:propertyset>
```

It also sends a NOT\_STARTED message:
```
    NOTIFY /ApplicationTransferService/event HTTP/1.1
    HOST: 172.16.10.100:8037
    Content-Type: text/xml
    NT: upnp:event
    NTS: upnp:propchange
    SID: uuid:2
    SEQ: 0
    Content-Length: 177
    <?xml version="1.0" encoding="utf-8"?><e:propertyset xmlns:e="urn:schemas-upnp-org:event-1-0"><e:property>
    <TransferState>NOT_STARTED</TransferState>
    </e:property></e:propertyset>
```

And the response back from the WinXP box is:
```
    POST /AdapterInfoService/control HTTP/1.1
    HOST: 172.16.10.105:62063
    SOAPACTION: "urn:schemas-upnp-org:service:AdapterInfoService:1#GetExtDeviceDescription"
    CONTENT-TYPE: text/xml ; charset="utf-8"
    Content-Length: 303
    <?xml version="1.0" encoding="utf-8"?>
    <s:Envelope s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" xmlns:s="http://schemas.xmlsoap.org/soap/envelope/"^gt;
       <s:Body>
          <u: xmlns:u="urn:schemas-upnp-org:service:AdapterInfoService:1" />
       </s:Body>
    </s:Envelope>
```

These go back and forth a few times with the same questions and answers
just to get synched up, I guess. Then after a bunch of other exchanges
where the WinXP box is asking the WMA what sort of box it is, it finally
tells the WMA where to go and fetch its brain:
```
    POST /ApplicationTransferService/control HTTP/1.1
    HOST: 172.16.10.105:62063
    SOAPACTION: "urn:schemas-upnp-org:service:ApplicationTransferService:1#SetApplicationPackageURI"
    CONTENT-TYPE: text/xml ; charset="utf-8"
    Content-Length: 473
    <?xml version="1.0" encoding="utf-8"?>
    <s:Envelope s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
       <s:Body>
          <u:SetApplicationPackageURI xmlns:u="urn:schemas-upnp-org:service:ApplicationTransferService:1">
             <ApplicationURI>uri://172.16.10.100:49156/squishguava</ApplicationURI>
             <ImageLength>2260992</ImageLength>
          </u:SetApplicationPackageURI>
       </s:Body>
    </s:Envelope>
```

SquishGuava\!\! The WMA sends back a SERVER\_WAIT while it adjusts to
its new brain, and then a STARTED followed by a COMPLETED when it is
ready. I also see some errors popping up in the stream:
```
    SERVER: Linux/2.4.17-rmk3-cot1, UPnP/1.0, Intel SDK for UPnP devices /1.2
    <s:Envelope
    xmlns:s="http://schemas.xmlsoap.org/soap/envelope/"
    s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
    <s:Fault>
    <faultcode>s:Client</faultcode>
    <faultstring>UPnPError</faultstring>
    <detail>
    <UPnPError xmlns="urn:schemas-upnp-org:control-1-0">
    <errorCode>-1</errorCode>
    <errorDescription>Invalid Action</errorDescription>
    </UPnPError>
    </detail>
    </s:Fault>
    </s:Body>
    </s:Envelope>
```

Nice to see that they aren't filling up my wireless bandwidth with any
useless traffic\! But eventually the WMA tells the server all about
itself:
```
    <root xmlns="urn:schemas-upnp-org:device-1-0">
    <specVersion>
    <major>1</major>
    <minor>0</minor>
    </specVersion>
    <URLBase>http://172.16.10.105:49152/</URLBase>
    <device>
    <deviceType>urn:schemas-upnp-org:device:RemoteIO:1</deviceType>
    <friendlyName>Linksys Wireless-B Media Adapter</friendlyName>
    <manufacturer>Linksys Corporation</manufacturer>
    <manufacturerURL>http://www.linksys.com</manufacturerURL>
    <modelDescription>UPnP Remote I/O Device 2.0</modelDescription>
    <modelName>UPnP Remote I/O Device</modelName>
    <modelNumber>1.0</modelNumber>
    <modelURL>http://www.linksys.com</modelURL>
    <serialNumber>1234567890012</serialNumber>
    <UDN>uuid:0006250DBAA7RemoteIO</UDN>
    <UPC>1234567891</UPC>
    <serviceList>
    <service>
    <serviceType>urn:schemas-upnp-org:service:RemoteIO:1</serviceType>
    <serviceId>urn:upnp-org:serviceId:RemoteIO1</serviceId>
    <controlURL>/upnp/control/RemoteIO1</controlURL>
    <eventSubURL>/upnp/event/RemoteIO1</eventSubURL>
    <SCPDURL>/rioscpd.xml</SCPDURL>
    </service>
    <service>
    <serviceType>urn:schemas-upnp-org:service:RemoteInput:1</serviceType>
    <serviceId>urn:upnp-org:serviceId:RemoteInput1</serviceId>
    <controlURL>/upnp/control/RemoteInput1</controlURL>
    <eventSubURL>/upnp/event/RemoteInput1</eventSubURL>
    <SCPDURL>/remoteinputscpd.xml</SCPDURL>
    </service>
    </serviceList>
    <presentationURL></presentationURL>
    </device>
    </root>
```

I haven't touched the serial number in that, by the way. Looks like they
aren't using it. Again, nice to see them going out of their way to save
bandwidth. After this comes a bunch more crap describing the services.
Stuff like this sort of makes me wonder:
```
    <PeerConnection>xrt2://172.16.10.100:52135/jdfkdjfkjdkfjdkjfkdjfkdjfkjdfkjdkfjdkfjiwi18
    </PeerConnection>
```

And a clue as to what file types the WMA can handle:
```
    <Sink>http-get:*:audio/mpeg:*,http-get:*:audio/x-mpeg:*,
    http-get:*:audio/mp3:*,http-get:*:audio/x-ms-wma:*,
    http-get:*:audio/mpegurl:*,http-get:*:audio/x-mpegurl:*,
    http-get:*:audio/m3u:*</Sink>
```

And some more interesting bits:
```
    <Event xmlns="urn:schemas-upnp-org:metadata-1-0/AVT/">
    <InstanceID val="1">
    <TransportState val="NO_MEDIA_PRESENT"></TransportState>
    <TransportStatus val="OK"></TransportStatus>
    <TransportPlaySpeed val="1"></TransportPlaySpeed>
    <CurrentPlayMode val="NORMAL"></CurrentPlayMode>
    <CurrentRecordQualityMode val="NOT_IMPLEMENTED"></CurrentRecordQualityMode>
    <PossiblePlaybackStorageMedia val="NETWORK"></PossiblePlaybackStorageMedia>
    <PossibleRecordStorageMedia val="NOT_IMPLEMENTED"></PossibleRecordStorageMedia>
    <PossibleRecordQualityModes val="NOT_IMPLEMENTED"></PossibleRecordQualityModes>
    <NumberOfTracks val="0"></NumberOfTracks>
    <CurrentMediaDuration val="0:00:00"></CurrentMediaDuration>
    <AVTransportURI></AVTransportURI>
    <AVTransportURIMetaData></AVTransportURIMetaData>
    <NextAVTransportURI></NextAVTransportURI>
    <NextAVTransportURIMetaData></NextAVTransportURIMetaData>
    <PlaybackStorageMedium val="NETWORK"></PlaybackStorageMedium>
    <RecordStorageMedium val="NOT_IMPLEMENTED"></RecordStorageMedium>
    <RecordMediumWri
    teStatus val="NOT_IMPLEMENTED"></RecordMediumWriteStatus>
    <CurrentTrack val="0"></CurrentTrack>
    <CurrentTrackDuration val="0:00:00"></CurrentTrackDuration>
    <CurrentTrackMetaData val="NOT_IMPLEMENTED"></CurrentTrackMetaData>
    <CurrentTrackURI></CurrentTrackURI>
    <CurrentTransportActions val="Play,Stop,Pause,Seek,Next,Previous"></CurrentTransportActions>
    </InstanceID>
    </Event>
```

Fast-forwarding through oodles of more SOAP stuff and we finally get to
the server sending a playlist:
```
    HTTP/1.1 206 Partial Content
    CONTENT-RANGE: bytes 0-2559/245115
    CONTENT-TYPE: audio/mpegurl
    Content-Length: 2560
    #EXTM3U
    #EXTINF:154,Stone Temple Pilots - "Unglued"
    http://172.16.10.100:54455/-1562803410/Stone%20Temple%20Pilots%20%20-%20%20%22Unglued%22.mp3
    #EXTINF:171,Enya - (1)The Celtz.MP3 Track 1 of 15
    http://172.16.10.100:54455/1695573734/Enya%20%20-%20%20%281%29The%20Celtz.MP3%20Track%201%20of%2015.mp3
```

This is in standard [M3U format](http://hanna.pyxidis.org/tech/m3u.html). And finally to actually
play a song, the WMA issues this request:
```
    GET /1695573734/Enya%20%20-%20%20%281%29The%20Celtz.MP3%20Track%201%20of%2015.mp3 HTTP/1.0
    Host: 172.16.10.100
    1
```

and gets back:
```
    HTTP/1.0 200 OK
    Content-Type: audio/mpeg
    Server: Intel CEL / CLR MiniWebServer
    [mp3 data]
```

So, that's the WMA. When I get back from Comdex/Apachecon I'll try
messing with libupnp. For now this toy works pretty well and Carl likes
it: ![](/assets/images/linksys-wireless-media-appliance-updated/tv1.jpg)
