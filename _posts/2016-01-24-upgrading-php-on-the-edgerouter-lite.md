---
date: '2016-01-24'
layout: post
title: Upgrading PHP on the EdgeRouter Lite
---

After nearly 7 years of service I retired my Asus RT-16 router, which wasn't really a router, but a re-purposed wifi access point running AdvancedTomato. In its place I got a Ubiquiti EdgeRouter Lite. It is Debian-based and has a dual-core 500MHz 64-Bit MIPS CPU (Cavium Octeon+), 512M of ram and a 4G removable onboard USB stick for less than $100. The router is completely open and, in fact, any advanced configuration has to be done from the command line. The Web UI has been improving, but there are still many things you can't do in it. In other words, exactly the type of device I prefer.

An open device with a Web UI means that the first thing I did was poke and prod at how it was implemented. Visually it looks like this:

![](/assets/images/upgrading-php-on-the-edgerouter-lite/erlite-ui.jpg)

Behind the scenes it is using lighttpd/1.4.35 with FastCGI PHP-5.4.42 and XCache 3.1.0.

![](/assets/images/upgrading-php-on-the-edgerouter-lite/erlite_phpinfo.png)

I obviously can't have a device in my house running PHP 5.4 now that we are in the age of PHP 7. So today's project was to get it updated.

## Step 1 - Getting a Big-Endian MIPS64 build of PHP 7

It would be nice if I could apt-get install a PHP 7 package, but unfortunately the Debian PHP 7 packages are for stretch/sid and not Debian 7.8 (wheezy) that the router is running. Typically I would install a cross-compiling environment on one of my fast build boxes and cross-compile. Unfortunately even though https://wiki.debian.org/CrossToolchains says it exists, I only found a little-endian MIPS64 cross-compilation environment, no big-endian. Oh well, this Cavium CPU isn't that slow and it is easy enough to apt-get install the gcc toolchain on it, so I compiled directly on the router. A make -j 2 took about 2 hours. I installed it into */opt/php7*. You can grab my build at [php_erlite_mips.tar.bz2](https://lerdorf.com/php_erlite_mips.tar.bz2). Expand it into */opt*. Test it:

```shell-session
root@router:~# /opt/php7/bin/php -v
PHP 7.0.4-dev (cli) (built: Jan 23 2016 17:06:26) ( NTS )
Copyright (c) 1997-2016 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
```

## Step 2 - Configuration

Here is the */etc/php7/cgi/php.ini* that I am using:

```ini
[PHP]
    short_open_tag = On
    error_reporting = -1
    display_errors = Off
    log_errors = On
    variables_order = "GPCS"    
    request_order = "GP"        
    register_argc_argv = Off  
    auto_globals_jit = On 
    file_uploads = On             
    upload_max_filesize = 128M    
    max_file_uploads = 20         
    default_socket_timeout = 60 
    cgi.fix_pathinfo=1

    [Session]                                     
    session.save_handler = files                  
    session.save_path = /var/run/php5
    session.use_cookies = 1                       
    session.use_only_cookies = 1                  
    session.name = PHPSESSID                      
    session.auto_start = 0                        
    session.cookie_lifetime = 0                   
    session.cookie_path = /                       
    session.cookie_domain =                       
    session.cookie_httponly = 0                   
    session.cookie_secure = 1                     
    session.serialize_handler = php               
    session.gc_probability = 0                 
    session.gc_divisor = 1000                  
    session.gc_maxlifetime = 1440              
    session.bug_compat_42 = Off                
    session.bug_compat_warn = Off              
    session.referer_check =                    
    session.entropy_length = 512               
    session.cache_limiter = nocache            
    session.cache_expire = 180                 
    session.use_trans_sid = 0                  
    session.hash_function = 1                  
    session.hash_bits_per_character = 5        
    session.lazy_write = 0

    [OpCache]
    zend_extension=opcache.so
    opcache.enable=1
    opcache.memory_consumption=20M
    opcache.interned_strings_buffer=4
    opcache.max_accelerated_files=10000
    opcache.revalidate_freq=60
    opcache.fast_shutdown=1
```

The existing XCache config creates a 30M shm cache, but I found it only needs around 16M with PHP 7/OpCache so we can save a bit of ram by setting it to 20M instead. And revalidate_freq is set to 60s. Keep that in mind if you are editing web files. It will take up to a minute before your changes will be reflected.

Next, edit */etc/lighttpd/conf-enabled/15-fastcgi-php.conf* and change:

```
"bin-path" => "/usr/bin/php-cgi",
```

to:

```
"bin-path" => "/opt/php7/bin/php-cgi",
```

Then restart lighttpd:

```shell-session
root@router:~# ps aux | grep light
www-data 23335  1.0  0.6   7456  3348 ?        S    02:49   0:08 /usr/sbin/lighttpd -f /etc/lighttpd/lighttpd.conf
root     23875  0.0  0.1   2384   592 pts/1    S+   03:03   0:00 /bin/busybox grep light
root@router:~# kill 23335
root@router:~# /usr/sbin/lighttpd -f /etc/lighttpd/lighttpd.conf
```

## Step 3 - Fixing broken stuff

If you are following along on your own router, you will notice that the web UI is rather broken at this point. I had two issues. First the json-based api calls weren't working at all. This was due to **HTTP_RAW_POST_DATA** being removed in PHP 7. To fix it, you need to modify */var/www/php/core/classes/router.php*. Apply this patch:

```diff
    --- router.php.orig
    +++ router.php
    @@ -171,6 +171,10 @@

                    case 'post':
                        $this->variables = $this->mixedPost ? array_merge($_GET, $_POST) : $_POST;
    +                   if (empty($GLOBALS['HTTP_RAW_POST_DATA']) &&
    +                       strpos($_SERVER['CONTENT_TYPE'], 'www-form-urlencoded') === false) {
    +                       $GLOBALS['HTTP_RAW_POST_DATA'] = file_get_contents("php://input");
    +                   }   
                        if (!empty($GLOBALS['HTTP_RAW_POST_DATA']) && empty($this->variables['noraw'])) {
                            $this->rawData =& $GLOBALS['HTTP_RAW_POST_DATA'];
```

The second problem I hit was that the web socket wouldn't connect. Looking at */var/log/ubnt-daemon.log* I was seeing:

```
2016-01-24 18:59:43 ubnt-statsd: Session error: Invalid session ID in request: Success
```

This had me stumped for a while, especially because when I straced the ubnt-util binary which is the daemon process reading/writing to the web socket end point, everything worked. Very annoying, when your debugging mechanism makes the problem go away. But that in itself was a clue. strace slows down the process it is tracing by quite a bit. So the fact that it worked under strace pointed to a race condition/timing problem. And the error message talks about an invalid session id. PHP is writing a session file and my strace revealed that this ubnt-util process was reading from this session file. A new performance feature in PHP 7 is that session file writes are only done when the session changes and it is written later. This is normally perfectly fine, but in this case with a web socket firing mid-request and an out-of-band daemon trying to read the session file, the timing didn't work unless I slowed down the daemon. A hard to find bug, but a trivial fix:

```ini
session.lazy_write = 0
```

## Conclusion

So, did it actually make a noticable difference? Not really. Measuring it with siege it shows that pages load about 10% faster. The PHP part was never the bottleneck in this implementation. SSL negotiation and the IO related to the Javascript web socket magic are the main factors making the UI somewhat sluggish. Making the PHP part of the puzzle twice as fast didn't speed up the overall system by a factor of 2, unfortunately.

The new php-cgi binary is a bit smaller even though I actually added mysqli, mysqlnd, PDO, PDO_mysql and PDO_sqlite extensions (just in case I need it):

```
-rwxr-xr-x    1 root     root       6705944 Jan 23 18:32 /opt/php7/bin/php-cgi
-rwxr-xr-x    1 root     root        203180 Jan 25 05:47 /opt/php7/lib/php/20151012/opcache.so
-rwxr-xr-x    1 root     root       8436004 Jun 22  2015 /usr/bin/php-cgi
```

The memory use on a per-request basis looks to be cut by about 25% plus I saved 10M on the shm segment for the opcode cache which does make a difference on a small device like this.

Mostly this was just an exercise that taught me a lot about how the router works.

