---
date: '2016-03-03'
layout: post
title: megasync for Debian 9 Stretch
---


![](/assets/images/megasync-for-debian-9-stretch/Screenshot_from_2016-03-03_01-13-02.png)
Like most of my posts here, this is mostly a note to myself so I don't
forget how I did it.

I Moved to Debian 9 on my desktop box at home and everything works great
except I occasionally use Mega.nz and they don't provide a Debian 9
build. It would be great if they just provided a statically linked
generic Linux binary, but they don't. So, to make it work, grab their
Debian 8 .deb file.

Then pull it apart and fix the dependencies in it:

    $ ar x megasync-Debian_8.0_amd64.deb
    $ tar xzf control.tar.gz

edit the *control* file and change **libcrypto++9** to
**libcrypto++9v5** and put it back together:

    $ tar c postrm postinst control md5sums | gzip -c > control.tar.gz
    $ ar rcs megasync-Debian_9.0.deb debian-binary control.tar.gz data.tar.xz

Now you should have an installable package as long as you install the
dependencies:

    $ sudo dpkg -i megasync-Debian_9.0.deb

However, it won't work yet because it does actually need *libcrypto++9*
and not *libcrypto++9v5*. The super-quick and super-ugly way to handle
that is to grab *libcrypto++.so.9.0.0* from a Debian 8 box and put it in
*/opt/lib/libcrypto++.so.9.0.0* and then create a */usr/bin/megasync.sh*
script with this:

    #!/bin/sh
    LD_PRELOAD=/opt/lib/libcrypto++.so.9.0.0 /usr/bin/megasync

Don't forget to **sudo chmod +x /usr/bin/megasync.sh** and then edit
*/usr/share/applications/megasync.desktop* and set **TryExec** and
**Exec** to point to your *megasync.sh* script:

    [Desktop Entry]
    Type=Application
    Version=1.0
    GenericName=File Synchronizer
    Name=MEGASync
    Comment=Easy automated syncing between your computers and your MEGA cloud drive.
    TryExec=megasync.sh
    Exec=megasync.sh
    Icon=mega
    Terminal=false
    Categories=Network;System;
    StartupNotify=false
    X-GNOME-Autostart-Delay=60

That should do it.
