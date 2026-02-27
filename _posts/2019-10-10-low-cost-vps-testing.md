---
date: '2019-10-10'
layout: post
title: Low-Cost VPS Testing
---


Last time around I was looking for a large dedicated VPS in the $100/month price range. Since then
I have started deploying a lot more tiny cloud instances that cost less than $10/month to run just one or two apps.
So this time around I am looking for the best bang for the buck for less $10 per month. Unmanaged, of course.
Just spin it up with a recent OS and put my public key in the root account. Should be simple enough. Let's see how they do.


I tested the following providers. To understand why I rated the way I did, click through to
read the details:

- [Kamatera](#kamatera) Really fast disk, ok network, ok cpu, ok signup. Overall Good.
- [A2 Hosting](#a2hosting) Good disk, fast network, good cpu, bad signup. Overall ok.
- [Hostwinds](#hostwinds) ok disk, fast network, ok cpu, good signup. Overall ok.
- [1&1 Ionos](#ionos) Good disk, good network, good cpu, ok signup. Overall Good.
- [Clouding.io](#clouding) ok disk, good network, good cpu, good signup. Overall Good.
- [Hetzner](#hetzner) Fast disk, fast network, ok cpu, great signup. Overall really Good.
- [Upcloud](#upcloud) Good disk, good network, fast cpu, great signup. Overall really Good.
- [vpsdime](#vpsdime) Fast disk, fast network, fast cpu, bad signup. Overall ok.
- [Lunanode](#lunanode) Fast disk, good network, fast cpu, good signup. Overall really Good.
- [Vultr](#vultr) Good disk, good network, good cpu, good signup. Overall good.
- [Linode](#linode) Really fast disk, fast network, ok cpu, good signup. Overall Good.
- [OVH](#ovh) Good disk, ok network, good cpu, bad signup. Overall ok.
- [RamNode](#ramnode) Good disk, good network, ok cpu, good signup. Overall Good.
- [Contabo](#contabo) Good disk, good network, fast cpu, bad signup. Overall ok.
- [Scaleway](#scaleway) Good disk, fast network, cool cpu, good signup. Overall really Good.
- [DigitalOcean](#digitalocean) Good disk, really fast network, ok cpu, great signup. Overall really Good.
- [BuyVM](#buyvm) Good disk, fast network, fast cpu, bad signup. Overall Good.
- [Amazon Lightsail](#lightsail) Slow disk, slower network, fast cpu, ok signup. Overall ok.
- [Microsoft Azure](#azure) Really slow disk, fast network, good cpu, bad signup. Overall ok.
- [Skysilk](#skysilk) Ok disk, ok network, fast cpu, good signup. Overall ok.
- [VPSServer](#vpsserver) Fast disk, good network, fast cpu, good signup. Overall Good.
- [81VPS](#81vps) Ok disk, good network, really slow cpu, bad signup. Overall ok.

### Affiliate/Referral links

I have included some referral links. I have more than enough hosting available to me, so don't feel obligated to click them.
But some of them give you some free credit as well.

### The Tests

My very unscientific overall perf test which combines cpu and disk io is to time how long it takes to compile PHP.

```shell-session
git clone https://git.php.net/repository/php-src.git
cd php-src
./buildconf -f
./configure --disable-all
time make (make -j N for N vCPUs)
```

Disk IO testing for large-file read/write along with random buffered reads is done with dd and hdparm like this:

```shell-session
sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
# Then wipe the cache (except on OpenVZ virts since you don't really have your own kernel)
/sbin/sysctl -w vm.drop_caches=3
dd if=/root/tempfile of=/dev/null bs=1M count=8192
/sbin/sysctl -w vm.drop_caches=3
hdparm -Tt /dev/sdaN
```

Network perf to California and Netherlands:

```shell-session
  iperf3 -c iperf.he.net
  iperf3 -p 5002 -c speedtest.serverius.net
```

<a id="kamatera"></a>

---

## [Kamatera](https://www.kamatera.com/)

![](/assets/images/low-cost-vps-testing/kamatera-services.png)
Kamatera has a PHP service option. For $9/month you get a dedicated cpu thread, 1G of ram and 20G of SSD.
That's a nice little setup given the performance should be guaranteed. It is based on Ubuntu 18.04-LTS and comes with
PHP 7.3 and nginx-1.14 preconfigured for PHP-FPM.

You can spin up servers in Hong Kong, Toronto, NY, Santa Clara, Texas, Amsterdam, Frankfurt, London, and 5 different cities in Israel. I chose Santa Clara.

### Positives

- Good PHP 7.3 default config with opcache enabled and mysqlnd plus most common extensions compiled in.
- nginx/php-fpm works out of the box. You can drop a PHP app into /var/www/html and it will just work
- Free single-server 30-day trial. Really nice to be able to test out the VPS before committing!

### Negatives
- no way to upload a public key to your profile, so you have to do a password-root-login initially
- no IPv6. It's late 2019. Every service should provide IPv6 alongside IPv4
- Even though the PHP setup has MySQL/MariaDB support, you have to install your own DB server to use it locally.
  Of course, there is also a MySQL-specific service, so you could run two servers and separate your web from your DB,
  but at this pricepoint I don't see too many people doing that
- These days any PHP setup is going to need Composer installed. It wasn't.
- nginx setup could use some tuning. (switch to a unix-domain socket for php-fpm, enable sendfile, tune proxy buffers)

### PHP Notes
Since this was an nginx/php-fpm service, some nginx/php-fpm specific notes:
- Remember to turn off display_errors in `/etc/php/7.3/fpm/php.ini` before you go live
- if this is going to be a dedicated PHP server, I would tweak the following:
  * opcache.memory_consumption from 128 to 512 (but leave room for your DB - if you app is DB-heavy, go lower or get a bigger VPS)
  * opcache.interned_strings_buffer from 8 to 64
  * switch nginx to `server unix:/run/php/php7.3-fpm.sock;` and php-fpm to `listen = /run/php/php7.3-fpm.sock`
  * if you are not going to use a cdn for static files, enable `sendfile` in nginx

Specs |   
:--- | :---
OS | Ubuntu 18.04-LTS
Hypervisor | VMware
CPU (dedicated thread) | Intel(R) Xeon(R) CPU E5-2660 v3 @ 2.60GHz
Ram | 1G
Disk | 20G
Bandwidth | 5TB/month at 10 Gbit/sec
Cost | $9/month
Server location | Santa Clare, CA
Time to spin up new server | 13 minutes


Performance |
:--- | :---
PHP compile time | 3m46s
Disk IO | 890 MB/s write, 855 MB/s read, 911 MB/s buffered reads, 6165 MB/s cached reads
Network | 185 Mbits/sec down, 183 Mbits/sec up to Fremont, CA
|        |  62 Mbits/sec down,  52 Mbits/sec up to Netherlands
<br/>
I also tried the $4/month non-dedicated cpu thread service. Like the previous test, I spun it up in Santa Clara.
This time I just selected a regular VPS. It spun up much quicker than the previous service one. 3 minutes vs. 13.
Probably because it installed a lot less on it. One slight annoyance was that Debian 10 was not an option. It has
been out for a couple of months, so I would expect to see it by now. Performance-wise the $4/month non-dedicated
cpu VPS performed about the same as the $9/month dedicated. But, that extra $5 gives you some assurance that you
will always see this level of performance. With the cheaper option you have to hope a noisy neighbour doesn't show
up.

Specs |
:--- | :---
OS | Debian 9.4
Kernel | 4.9.0
Hypervisor | VMware
CPU | Intel(R) Xeon(R) CPU E5-2660 v3 @ 2.60GHz
Ram | 1G
Disk | 20G
Bandwidth | 5TB/month at 10 Gbit/sec
Cost | $4/month
Server location | Santa Clare, CA
Time to spin up new server | 3 minutes

Performance |
:--- | :---
PHP compile time | 3m58s
Disk IO | 997 MB/s write, 1.0 GB/s read, buffered reads 934 MB/s, cached reads 7358 MB/s
Network | 170 Mbits/sec down, 167 Mbits/sec up to Fremont, CA
|       |  44 Mbits/sec down,  41 Mbits/sec up to Netherlands

### Conclusion

I like Kamatera. The signup process and the free trial was smooth. I was able to try different configurations without
a bunch of billing hassle and the performance was decent. Initial public key server auth, IPv6, and Debian 10 would be
at the top of my wishlist for this service.

<a id="a2hosting"></a>

---

## [A2 Hosting](https://www.a2hosting.com/vps-hosting/unmanaged)

They have $5 single core, 512M ram and 20G SSD. For $10 you get 1G of ram and an extra 10G of SSD storage.
Sounds promising. Locations are limited to Michigan, Arizona, Amsterdam and Singapore. I chose the $5 VPS in
Arizona. Again, no Debian 10 and the signup process had a bunch of strange upsell options I have never heard of,
but whatever, at least they weren't added by default. Also, the "Complete Order" page spun forever on me, but I
got an email with an order confirmation so it went through.
![](/assets/images/low-cost-vps-testing/a2hosting.png)
![](/assets/images/low-cost-vps-testing/a2hosting1.png)

### Negatives
- No public key initial access
- No Ipv6
- No Debian 10
- I had to go look at the sshd config to see that the default sshd only listens on port 7822
- OpenVZ virt which means, amongst other things, that you are usually stuck sharing an ancient kernel with other virts on the box
- Not great disk IO performance

### Positives
- Pretty nice HTML5 serial console (which I needed to figure out how to ssh in)
- Good CPU and network performance

Specs |
:--- | :---
OS| Debian 9.11
Kernel| 2.6.32
Hypervisor| OpenVZ (not really a hypervisor, of course)
CPU| Intel(R) Xeon(R) CPU E5-2620 v3 @ 2.40GHz
Ram| 512M
Disk| 20G
Bandwidth| 2 TB/month
Cost| $5/month
Server location| Arizona
Time to spin up new server| 5 minutes

Performance |
:--- | :---
PHP compile time| 2m50s
Disk IO| 470 MB/s write (no way to clear disk cache on OpenVZ so skipped read tests)
Network| 701 Mbits/sec down, 701 Mbits/sec up to Fremont, CA
|      | 114 Mbits/sec down, 113 Mbits/sec up to Netherlands

### Conclusion
I am not a fan of OpenVZ VPS'. The performance tends to be quite good as long as your provider
doesn't oversell the host. OpenVZ doesn't isolate you very well from noisy neighbours, so the risk
for performance degradation is higher. My test VPS performed well and if it continues to perform at
the current level then it is good bang for the buck at $5/month.


<a id="hostwinds"></a>

---

## [Hostwinds](https://www.hostwinds.com/vps/unmanaged-linux)

![](/assets/images/low-cost-vps-testing/hostwinds1.png)
1G Ram with 30G of SSD for $4.49 and 2G/50G for $8.99. Locations in Dallas, Seattle and Amsterdam.
I chose the $4.49 ($4.99 recurring) plan in Seattle. Simple and transparent signup process.
I really like when providers give me my own IPv6 /64 subnet to play with. In this case the
management console didn't let you set the rDNS for arbitrary addresses in that subnet though.
I'd have to hit that "Add IPv6 Address" button *a lot* to get to `2607:5501:3000:43f:1:5ee:bad:c0de`

But adding it manually on the VM without worrying about the console worked ok. You just don't
get reverse dns for it.

```
PING 2607:5501:3000:43f:1:5ee:bad:c0de(2607:5501:3000:43f:1:5ee:bad:c0de) 56 data bytes
64 bytes from 2607:5501:3000:43f:1:5ee:bad:c0de: icmp_seq=1 ttl=53 time=27.1 ms
64 bytes from 2607:5501:3000:43f:1:5ee:bad:c0de: icmp_seq=2 ttl=53 time=28.0 ms
64 bytes from 2607:5501:3000:43f:1:5ee:bad:c0de: icmp_seq=3 ttl=53 time=26.0 ms
64 bytes from 2607:5501:3000:43f:1:5ee:bad:c0de: icmp_seq=4 ttl=53 time=25.5 ms
64 bytes from 2607:5501:3000:43f:1:5ee:bad:c0de: icmp_seq=5 ttl=53 time=25.8 ms
```

There is also a "Firewall Profile" thing, but I couldn't figure out how to create a
new profile. It just had a default profile that allowed all traffic.

![](/assets/images/low-cost-vps-testing/hostwinds3.png)
![](/assets/images/low-cost-vps-testing/hostwinds2.png)
![](/assets/images/low-cost-vps-testing/hostwinds4.png)

<div style="clear: both;"></div>

### Positives
- You get to see all your configuration options before paying
- IPv6 with a /64 Subnet!
- It installed the public key I added to my profile, so I could ssh right in!
- Fast network IO

### Negatives
- I had issues booting the server initially. Had to pop into the VNC console and Ctrl-Alt-Del it
- No Debian 10
- There was a key I didn't recognize in `~root/authorized_keys`
- Really slow disk writes! (perhaps I got unlucky?)

Specs |
:--- | :---
OS| Debian 9.1
Kernel| 4.9.0
Hypervisor| kvm
CPU| Intel Xeon E3-12xx v2 (Ivy Bridge, IBRS)
Ram| 1G
Disk| 30G
Bandwidth| 1 TB/month
Cost| $4.99/month
Server location| Seattle
Time to spin up new server| 10 minutes

Performance |
:--- | :---
PHP compile time| 5m45s
Disk IO| 59 MB/s write, 637 MB/s read, buffered reads 450 MB/s, cached reads 6828 MB/s
Network| 901 Mbits/sec down, 899 Mbits/sec up to Fremont, CA
|      | 131 Mbits/sec down, 130 Mbits/sec up to Netherlands

### Conclusion
Hostwinds gets almost everything right although my instance suffered from really slow disk writes.
I went back a couple of days later and re-ran the large-file write test and it was slow again.

<a id="ionos"></a>

---

## [1&1 IONOS](https://www.ionos.com/servers/vps)

$2/month! 512M ram, 10G SSD, single-core, of course. Locations United States (Kansas),
Germany, UK and Spain. They also have 2 vCores with 2G of ram and 80G SSD for $10/month which is a really good deal.
But I had to try the $2 one.

### Positives
- $2/month!
- Quick sign-up process

### Negatives
- Kind of annoying contract-based signup and billing system makes it hard to switch and try different services
- No public key initial auth
- No Debian 10
- No IPv6 (but it looks like they might support it for the larger instances?)

Specs |
:--- | :---
OS| Debian 9.11
Kernel| 4.9.0
Hypervisor| VMware
CPU| Intel(R) Xeon(R) Gold 5120 CPU @ 2.20GHz
Ram| 512M
Disk| 10G
Bandwidth| Unlimited
Cost| $2/month
Server location| Kansas
Time to spin up new server| 10 minutes

Performance |
:--- | :---
PHP compile time| 3m10s
Disk IO| 387 MB/s write, 443 MB/s read, buffered reads 121 MB/s, cached reads 7112 MB/s
Network| 272 Mbits/sec down, 270 Mbits/sec up to Fremont, CA
| |      130 Mbits/sec down, 129 Mbits/sec up to Netherlands

### Conclusion
Hard to beat $2/month for a capable VPS with supposedly unlimited bandwidth.

<a id="clouding"></a>

---

## [Clouding.io](https://clouding.io/en/)

0.5(!) vCores, 1G of ram and 5G of SSD for €3/month. Each additional 5G of SSD is €0.50. I was not given a choice of
where to host it, but the ip shows up as being in Barcelona, so I guess that is the only data center available.
The signup process was easy, and while they didn't let me upload my own public key, they created one for me and let me
create additional ones and had me download the private key. Then just `ssh -i ~/.ssh/clouding.pem root@<ip>` and I was off and
running.

![](/assets/images/low-cost-vps-testing/clouding.png)
![](/assets/images/low-cost-vps-testing/clouding2.png)
![](/assets/images/low-cost-vps-testing/clouding3.png)

One interesting note was that the marketing folks from Clouding were the only ones to catch on and reach out to me while I was writing this.
I suppose it helps being a smaller provider for this, but i thought it was pretty impressive that they connected the dots and figured out I was
probably writing a follow-up to my January 2018 VPS Testing article.

### Positives
- Easy signup with a free trial
- Public key initial login
- Nice default firewall setup and a good UI to customize it.
- Good performance on the PHP compile test

### Negatives
- No IPv6
- No Debian 10
- Somewhat slow disk read times
- Only Barcelona data center
- I didn't see a way to configure reverse dns

Specs |
:--- | :---
OS| Debian 9.3
Kernel| 4.9.0
Hypervisor| KVM
CPU| Intel Xeon Virtual CPU
Ram| 1G
Disk| 10G
Bandwidth| 2TB/month and €0.02/GB beyond that
Cost| €3.50/month
Server location| Barcelona
Time to spin up new server| 2 minutes

Performance |
:--- | :---
PHP compile time| 3m20s
Disk IO| 480 MB/s write, 189 MB/s read, buffered reads 173 MB/s, cached reads 8956 MB/s
Network| 89 Mbits/sec down, 87 Mbits/sec up to Fremont, CA
|      | 419 Mbits/sec down, 417 Mbits/sec up to Netherlands
<br />
I tested a 2nd server setup since they were nice enough to contact me. This time a slightly bigger
instance for €9/month with 1 core, 4G of ram and 15G of storage. And I went with the "LEMP" template
instead of Debian to see what their PHP setup looked like.

It was a standard Ubuntu nginx 1.14.0 PHP 7.2.19 setup. For an instance with 4G of ram like this,
assuming it will be a dedicated web server, you would want to edit `/etc/php/7.2/fpm/php.ini` and set:

```ini
[opcache]
opcache.enable=1
opcache.memory_consumption=1024
opcache.interned_strings_buffer=256
opcache.max_accelerated_files=100000
opcache.validate_timestamps=1
opcache.revalidate_freq=5
opcache.save_comments=1
opcache.use_cwd=1
opcache.enable_file_override=0
opcache.enable_cli=0
opcache.max_wasted_percentage=10
opcache.interned_strings_buffer=256
opcache.fast_shutdown=1
opcache.huge_code_pages=0
opcache.optimization_level=-1
opcache.log_verbosity_level=2
```

And then run `systemctl restart php7.2-fpm` to restart PHP-FPM. If you have a ton of PHP code, keep an eye on `<?php phpinfo();?>` and look for any restarts.
Especially the `OOM restarts` counter you will find the in the `Opcache` section of `phpinfo()`. If that is increasing more than once every couple of days as
you push new code, then you need to allocate more memory to Opcache.

This setup had `fastcgi_pass   unix:/run/php/php7.2-fpm.sock;` in the nginx config to have nginx talk to php over a unix domain socket, which is good.
And in general the nginx setup had good defaults.

Specs |
:--- | :---
OS| Ubuntu 18.04.1 LTS
Kernel| 4.15.0
Hypervisor| KVM
CPU| Intel Xeon Virtual CPU
Ram| 4G
Disk| 15G
Bandwidth| 2TB/month and €0.02/GB beyond that
Cost| €9/month
Server location| Barcelona
Time to spin up new server| 5 minutes

Performance |
:--- | :---
PHP compile time| 2m55s
Disk IO| 590 MB/s write, 184 MB/s read, buffered reads 181 MB/s, cached reads 7504 MB/s
Network| 63 Mbits/sec down, 61 Mbits/sec up to Fremont, CA
|      | 470 Mbits/sec down, 469 Mbits/sec up to Netherlands



### Conclusion
The slow network speed to California is to be expected from a server hosted in Barcelona. Disk reads
were a bit below average, but overall this was a pleasant experience.  The PHP compile time was near
the top of the list which is a good indication that the VPS performance is solid.


<a id="hetzner"></a>

---

## [Hetzner](https://www.hetzner.com/cloud)

Hetzner has been around forever. Locations in Germany (Nuremberg and Falkenstein) and Helskinki. For €2.49/month you
get 1 vCPU, 2G RAM and 20G SSD. Or for €4.90 2vCPU/4G/40G. That's quite a bit of power for a service just over $5/month.
I really like Hetzner's provisioning process. You get a lot of control over everything. You can even choose between ext4
and XFS for your filesystem.
![](/assets/images/low-cost-vps-testing/hetzner1.png)
![](/assets/images/low-cost-vps-testing/hetzner2.png)
![](/assets/images/low-cost-vps-testing/hetzner3.png)
![](/assets/images/low-cost-vps-testing/hetzner4.png)

### Positives
- Debian 10
- Public key initial login with your own uploaded key
- IPv6 with a /64 subnet and reverse DNS
- Floating IPs that can be assigned to any VPS
- Backchannel private network
- Choice between ext4 and XFS filesystems (certain workloads are better suited to XFS)
- The way you can create, attach, detach, re-attach and grow storage volumes is extremely nice

### Negatives
- Only European data centers

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| Intel Xeon Processor (Skylake, IBRS)
Ram| 2G
Disk| 20G
Bandwidth| 20TB/month
Cost| €2.49/month
Server location| Nuremberg
Time to spin up new server| 1 minute

Performance |
:--- | :---
PHP compile time| 4m30s
Disk IO| 525 MB/s write, 1.2 GB/s read, buffered reads 1.1 GB/s, cached reads 5662 MB/s
Network| 117 Mbits/sec down, 115 Mbits/sec up to Fremont, CA
| |      2.2 Gbits/sec down, 2.2 Gbits/sec up to Netherlands
<br />
I also tested the €4.90 2vCPU/4G/40G config.

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| 2 x Intel Xeon Processor (Skylake, IBRS)
Ram| 2G
Disk| 20G
Bandwidth| 20TB/month
Cost| €4.90/month
Server location| Nuremberg
Time to spin up new server| 1 minute

Performance |
:--- | :---
PHP compile time| 2m31s
Disk IO| 495 MB/s write, 710 MB/s read, buffered reads 481 MB/s, cached reads 6388 MB/s
Network| 153 Mbits/sec down, 151 Mbits/sec up to Fremont, CA
| |      1.5 Gbits/sec down, 1.5 Gbits/sec up to Netherlands

### Conclusion
Hetzner is a very polished service to use. New servers spin up almost instantly. It is up to date with the latest
OS versions and gives you all sorts of configuration options. IPv6 with your own /64 subnet and reverse DNS and 
the price vs. performance is awesome. If you need a place to host in Europe, you can't go wrong with Hetzner.

<a id="upcloud"></a>

---

## [Upcloud](https://upcloud.com/signup/?promo=7QZ77S)
*Disclaimer:* I really like Upcloud. Through their great referral program from my first article I got a lot of free
credits and have been using those to host small projects. If you sign up using my [referral link](https://upcloud.com/signup/?promo=7QZ77S)
you will get a $25 credit on the service.

$5/month gets you 1 core, 1G of ram and 25G of SSD. For $10 you get double the ram and storage. Locations are Frankfurt,
Helsinki, Amsterdam, Singapore, London, Chicaco and San Jose. You have plenty of configuration options, including the
type of network adapter (VirtIO, Intel E1000 or RealTek RTL8139 emulation) and a choice between standard VGA and Cirrus
Logic for your display adapter. The display adapter probably makes a difference if you are doing Windows hosting, which
they also support. For my Linux hosting I don't see how it comes into play.

![](/assets/images/low-cost-vps-testing/upcloud1.png)
![](/assets/images/low-cost-vps-testing/upcloud2.png)

### Positives
- Debian 10
- Public key initial login with your own uploaded key
- IPv6
- Easy to use firewall setup
- Custom images, templates and startup scripts
- Really fast raw CPU speed for a $5 VPS

### Negatives
- Disk IO and network performance is average
- No IPv6 /64 subnet. You can only request 5 IPv6 addresses

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| Intel(R) Xeon(R) Gold 6136 CPU @ 3.00GHz
Ram| 2G
Disk| 25G
Bandwidth| 1TB/month
Cost| $5/month
Server location| San Jose
Time to spin up new server| 1 minute

Performance |
:--- | :---
PHP compile time| 2m40s
Disk IO| 422 MB/s write, 408 MB/s read, buffered reads 363 MB/s, cached reads 8632 MB/s
Network| 295 Mbits/sec down, 295 Mbits/sec up to Fremont, CA
| |      126 Mbits/sec down, 126 Mbits/sec up to Netherlands

### Conclusion
It is impressive to see a single core VPS compile PHP in under 3 minutes! The provisioning
process is amazing and now that they have a San Jose location I get 10ms pings from my home
connection.

<a id="vpsdime"></a>

---

## [vpsdime](https://vpsdime.com/)

Locations Los Angeles (1Gbps), UK (1Gbps), Dallas (10Gbps) and Seattle (10Gbps). Their map
shows locations in New Jersey and Amsterdam as well, but I didn't get those as options for my
$7 VPS. For those $7/month you get 4 vCPUs, 6G of RAM and 30G of disk! That is almost unbelievable.
Let's see what the catch is.

Interesting note during their signup process:

    Anything related to ZenCash, Digital currency mining, Mass mailing, Torrents, TOR, Mining,
    IRC, Teamspeak, Starbound, Runescape, Minecraft and Multics are NOT ALLOWED.
    Please read our TOS before ordering.

The signup process was a bit confusing to me. You didn't really get to configure anything

This also happened to be the first provider to cleartext email me both the root and
management console passwords. I was a little hard on them in my instant cancellation. As it turns out
a number of providers I tried after them did the same thing. From a customer support angle I can
understand it. It is hard to explain SSH keys. It is much easier to just give out a password.
But, I still insist on the public key option, even if a provider wants to default to passwords.

### Positives
- Debian 10
- Amazing performance at this price but beware of load limits and less resource isolation because of OpenVZ

### Negatives
- No public key initial login
- Emailed me my vpsdime.com password
- Emailed me my VPS root password
- The signup process wasn't super clear
- No IPv6
- The management UI isn't great
- OpenVZ
- Load limits

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| OpenVZ
CPU| 4 x Intel(R) Xeon(R) CPU E5-2697 v2 @ 2.70GHz
Ram| 6G
Disk| 30G
Cost| $7/month
Bandwidth| 2TB/month
Server location| Dallas
Time to spin up new server| 10 minutes

Performance |
:--- | :---
PHP compile time| 1m8.153s
Disk IO| 818 MB/s write, 939 MB/s read, buffered reads 496 MB/s, cached reads 9047 MB/s
Network| 770 Mbits/sec down, 770 Mbits/sec up to Fremont, CA
| |      188 Mbits/sec down, 187 Mbits/sec up to Netherlands

### Conclusion
So, the catch was that it was an OpenVZ VPS. If you can get beyond the rather crappy UI,
the terrible password management and the somewhat harsh load limits, this can give you
amazing performance for the price. However, as they note:

    "Users are REQUIRED to control their 15-min system load to remain below 2.0 at all times."

If you don't do this, your VPS can get suspended on you. So really, this isn't actually a VPS.
This is more like shared hosting where you are really prone to a noisy neighbour affecting you.
They do have a more traditional VPS service called "Premium VPS" at $20/month where you get
1 dedicated vCPU, 4G or RAM and 60G of SSD. That has no load limits, of course. If you are
interested, I have an affiliate link you can use:  https://vpsdime.com/aff.php?aff=1986

<a id="lunanode"></a>

---

## [Lunanode](https://www.lunanode.com/?r=12095)

$3.50 fpr 1 vCPU, 1G of RAM and 15G of SSD. For $7 you can choose between either 2G of RAM or 2 vCPUs
along with 20G of storage. Very nice signup process. It is obvious where to add your SSH public key which
is always a good sign. Locations are Toronto, Montreal and Roubaix (France). I went with the $7/month 2vCPU
option in Toronto.

### Positives
- $20 free trial credit - affiliate link: https://www.lunanode.com/?r=12095
- IPv6
- Public key initial login (If I hadn't done an ISO install to get Debian 10)

### Negatives
- No Debian 10 template, but they did have a Debian 10 ISO option so it took a little longer
  to bring up because I had to do a manual install in a VNC window
<div style="clear: both;"></div>
    ![](/assets/images/low-cost-vps-testing/lunanode.png)
- Had a weird issue that I couldn't get to the iperf port (5201) on iperf.he.net's IPv6 address
  It worked ok when I switched to IPv4. It didn't appear to be a firewall issue, at least not
  any firewall I could see.
- Not completely obvious that you have to click your Keypair button to activate it. Since it is solid-filled blue
  it looks selected, but unless it has an orange border, it isn't.
<div style="clear: both;"></div>
    ![](/assets/images/low-cost-vps-testing/lunanode1c.png) ![](/assets/images/low-cost-vps-testing/lunanode2c.png) 

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| 2 x Intel(R) Xeon(R) CPU E5-2690 v2 @ 3.00GHz
Ram| 1G
Disk| 20G
Cost| $7/month
Bandwidth| 2TB/month
Server location| Toronto
Time to spin up new server| 15 minutes (because I did an ISO install)

Performance |
:--- | :---
PHP compile time| 1m59s
Disk IO| 330 MB/s write, 1.8 GB/s read, buffered reads 1.5 GB/s, cached reads 9559 MB/s
Network| 365 Mbits/sec down, 365 Mbits/sec up to Fremont, CA
| |      238 Mbits/sec down, 237 Mbits/sec up to Netherlands
<br />
I also tried the $3.50 VPS but this time used the Debian 9 template so I wouldn't have to go through the manual
ISO install again. Interestingly you log in as debian@ instead of root@ then you can `sudo bash` from there.

Specs |
:--- | :---
OS| Debian 9.11
Kernel| 4.9.0
Hypervisor| KVM
CPU| Intel(R) Xeon(R) CPU E5-2690 v2 @ 3.00GHz
Ram| 512M
Disk| 16G
Cost| $3.50/month
Bandwidth| 1TB/month
Server location| Toronto
Time to spin up new server| 1 minute

Performance |
:--- | :---
PHP compile time| 3m55s
Disk IO| 278 MB/s write, 878 MB/s read, buffered reads 626 MB/s, cached reads 9866 MB/s
Network| 397 Mbits/sec down, 397 Mbits/sec up to Fremont, CA
| |      241 Mbits/sec down, 240 Mbits/sec up to Netherlands

### Conclusion
I like Lunanode. I had some networking confusion, but they were my own doing. If you remove the default security
group in the management UI you can't get anywhere. Performance for the 2vCPU $7 VPS was great and if you are looking
for a Canadian presence or a non-US North American one, this is a great choice.
For a $20 credit you can use my [referral link](https://www.lunanode.com/?r=12095)

<a id="vultr"></a>

---

## [Vultr](https://www.vultr.com/?ref=7314620)

$2.50, $3.50 and $5 plans with increasing ram, storage and bandwidth. The $2.50 plan is IPv6-only! There are certainly
use-cases where you don't need IPv4. So, of course I had to try that one out. 1 vCPU, 512M of ram and 10G of SSD.
Lots of locations all over the world. I chose Atlanta because the $2.50 IPv6-only was only available there and in New Jersey.

![](/assets/images/low-cost-vps-testing/vultr.png)
![](/assets/images/low-cost-vps-testing/vultr2.png)

<div style="clear: both;"></div>

### Positives
- Public key initial login
- Debian 10
- IPv6 with a /64!
- $2.50!

### Negatives
- Very low bandwidth limit but the $5 option gives you 1TB

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| Virtual CPU 523cbcdd6ca4
Ram| 512M
Disk| 10G
Bandwidth| 0.5TB/month
Cost| $2.50/month
Server location| Atlanta
Time to spin up new server| 1 minute

Performance |
:--- | :---
PHP compile time| 3m36s
Disk IO| 357 MB/s write, 281 MB/s read, buffered reads 292 MB/s, cached reads 9604 MB/s
Network| 237 Mbits/sec down, 236 Mbits/sec up to Fremont, CA
| |      190 Mbits/sec down, 190 Mbits/sec up to Netherlands

### Conclusion
Interestingly I wasn't able to compile PHP without adding a 1G swap file:
```shell-session
    dd if=/dev/zero of=/swap.img bs=1M count=1024
    mkswap /swap.img
    chmod 0600 /swap.img
    swapon /swap.img
```
But with that, the performance was pretty good. 3m36 to compile and we know it went into swap doing so.
For $2.50/month this was a solid little server. Of course, you would have to keep the IPv6-only aspect in
mind for any use-cases. But if you need a server with 18446744073709551616 different possible IP addresses, or
perhaps just a server where you can make your IP `2001:19f0:5401:2117:1:5ee:bad:c0de`. You can even rdns it
easily in the management UI:
```shell-session
    $ host 2001:19f0:5401:2117:1:5ee:bad:c0de
    e.d.0.c.d.a.b.0.e.e.5.0.1.0.0.0.7.1.1.2.1.0.4.5.0.f.9.1.1.0.0.2.ip6.arpa domain name pointer iseebadcode.lerdorf.com.
```
You can add an IPv4 address to it for $3/month, but they have better $5 deals which include an IPv4. Overall
Vultr is a solid service. They get everything right. [Referral link](https://www.vultr.com/?ref=7314620)

<a id="linode"></a>

---

## [Linode](https://www.linode.com/?r=2c964a203b1250c4e79673c2418b108ef70e734f)

Plenty of locations world-wide.  They have a "Nanode" for $5/month with 1 vCPU, 1G of ram and 25G of storage.
I chose their Fremont location.

![](/assets/images/low-cost-vps-testing/linode1.png)
![](/assets/images/low-cost-vps-testing/linode3.png)
![](/assets/images/low-cost-vps-testing/linode4.png)

<div style="clear: both;"></div>

### Positives
- Public key initial login with my own key
- IPv6
- Debian 10
- Very nice provisioning process

### Negatives
- It would be nice if you could add an IPv6 network via the Web UI. It is only possible by contacting support.
    ![](/assets/images/low-cost-vps-testing/linode2.png)
- No cloud firewall

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| Intel(R) Xeon(R) CPU E5-2680 v2 @ 2.80GHz
Ram| 1G
Disk| 25G
Bandwidth| 1TB/month
Cost| $5/month
Server location| Fremont
Time to spin up new server| 1 minute

Performance |
:--- | :---
PHP compile time| 4m6s
Disk IO| 902 MB/s write, 1.2 GB/s read, buffered reads 1.2 GB/s, cached reads 8984 MB/s
Network| 1.1 Gbits/sec down, 1.1 Gbits/sec up to Fremont, CA
| |      126 Mbits/sec down, 126 Mbits/sec up to Netherlands

### Conclusion

Very nice network performance from Fremont to Fremont :)
The new Web UI is much much better than the old one, and the performance of the $5 Nanode was excellent.
[Referral link](https://www.linode.com/?r=2c964a203b1250c4e79673c2418b108ef70e734f)

<a id="ovh"></a>

---

## [OVH](https://www.ovh.com/world/vps/)

$3.35 for 1 vCPU, 2G ram and 20G SSD. Double ram and storage for $6.87. They have data centers all over, but
for some reason I was only able to choose the Beauharnois Canada location. And the ordering experience is terrible.
You select your VPS and provide billing info and you get a:

    "Within a maximum of 24 hours, you will receive an email confirming that your VPS has been delivered, along with your bill."

where other providers spin up your VPS within minutes at that point. In the end I got that email about 15 minutes later
on a Sunday evening.

![](/assets/images/low-cost-vps-testing/ovh1.png)
![](/assets/images/low-cost-vps-testing/ovh2.png)
![](/assets/images/low-cost-vps-testing/ovh3.png)

### Positives
- Debian 10
- IPv6
- Unlimited bandwidth (but at 100 Mbps)
- You can set the VM to "terminate on expiry date" to avoid getting billed for extra time you don't need

### Negatives
- The difference between the management consoles at us.ovhcloud.com and ca.ovh.com is super-confusing. They have separate logins and I can never figure out which is which.
- Emailed root password! Are SSH keys really that difficult to manage?
- The provided IPv6 wasn't configured on my VM but it worked ok after I manually added it to the `/etc/network/interfaces.d/50-cloud-init.cfg` file.
- Difficult to figure out how to delete the instance

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| Intel Core Processor (Haswell, no TSX, IBRS)
Ram| 2G
Disk| 20G
Bandwidth| Unlimited at 100 Mbps
Cost| $3.35/month
Server location| Beauharnois
Time to spin up new server| 15 minutes

Performance |
:--- | :---
PHP compile time| 3m42s
Disk IO| 186 MB/s write, 606 MB/s read, buffered reads 486 MB/s, cached reads 7981 MB/s
Network| 97 Mbits/sec down, 95 Mbits/sec up to Fremont, CA
| |      92 Mbits/sec down, 90 Mbits/sec up to Netherlands

### Conclusion
Disk writes and network performance were quite poor. The emailed root password just grates on my nerves. Having said that, I am actually an OVH US customer
because I have one of their dedicated bare metal servers and the provisioning experience wasn't great for that either. I got it at a great price through
some introductory offer when they opened a west coast data center and as bad as some of these provisioning and management interfaces are, once you are up and
running it doesn't matter that much.

Surprisingly the IPv6 I was assigned already had someone else's reverse dns. That is not uncommon for IPv4, but the IPv6 range of addresses is so vast that
it is really unusual to see that. And there was an odd thing about needing a number keypad for numbers in the console window. Which wasn't true, of course. When
I send it `123` it has no way of knowing if that was typed on number keypad or not.

![](/assets/images/low-cost-vps-testing/ovh4.png) ![](/assets/images/low-cost-vps-testing/ovh5.png)


<a id="ramnode"></a>

---

## [RamNode](https://www.ramnode.com/#pricing)

$3 1vCPU, 512M ram and 10G SSD. $5 for 2 vCPU, 1G and 25G SSD. I chose the $5 option in their LA data center. They also have Atlanta, NYC, Seattle and Netherlands.

### Positives
- Public key initial login with my own key
- Debian 10
- IPv6

### Negatives
- Management UI is a bit sparse. No cloud firewall or any bells and whistles
- They have a "Security Groups" thing but I couldn't figure out how to add my own rules to it

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| AMD EPYC Processor (with IBPB)
Ram| 1G
Disk| 25G
Bandwidth| 2TB/month
Cost| $5/month
Server location| Los Angeles
Time to spin up new server| 12 minutes

Performance |
:--- | :---
PHP compile time| 4m8s
Disk IO| 475 MB/s write, 212 MB/s read, buffered reads 133 MB/s, cached reads 2054 MB/s
Network| 340 Mbits/sec down, 338 Mbits/sec up to Fremont, CA
| |      132 Mbits/sec down, 131 Mbits/sec up to Netherlands

### Conclusion
For a 2-core VPS, I was hoping for a little faster PHP compile time. Disk reads were also a bit slow.
But overall for $5 this is a good VPS. Everything worked out of the box.

<a id="contabo"></a>

---

## [Contabo](https://contabo.com/?show=vps)

€4.99/month for 4 vCPU, 8G of ram and 200G SSD! And €8.99 gets you 6 cores, 16G of ram and 400G of storage.
They also offer "SSD-boosted" HDD VPS setups if you need lots of storage.  Another what's the catch offer.
I was a big fan of Contabo last time around, so maybe there is no catch. Let's see.  I chose the €4.99 Debian 10
option. It says it is on a 200 Mbit/s switch port, so I suspect that will be reflected in the bandwidth numbers.
The €8.99 option is on a 400 Mbit/s switch port.

![](/assets/images/low-cost-vps-testing/contabo1.png)
![](/assets/images/low-cost-vps-testing/contabo2.png)
![](/assets/images/low-cost-vps-testing/contabo3.png)

### Positives
- IPv6 with a /64 and rDNS (have to run `enable_ipv6 and reboot` to activate it)
```
$ ping 2a02:c207:2030:5849:1:5ee:dead:c0de
PING 2a02:c207:2030:5849:1:5ee:dead:c0de(2a02:c207:2030:5849:1:5ee:dead:c0de) 56 data bytes
64 bytes from 2a02:c207:2030:5849:1:5ee:dead:c0de: icmp_seq=1 ttl=56 time=167 ms
64 bytes from 2a02:c207:2030:5849:1:5ee:dead:c0de: icmp_seq=2 ttl=56 time=163 ms
64 bytes from 2a02:c207:2030:5849:1:5ee:dead:c0de: icmp_seq=3 ttl=56 time=162 ms
64 bytes from 2a02:c207:2030:5849:1:5ee:dead:c0de: icmp_seq=4 ttl=56 time=162 ms
64 bytes from 2a02:c207:2030:5849:1:5ee:dead:c0de: icmp_seq=5 ttl=56 time=161 ms
$ host 2a02:c207:2030:5849:1:5ee:dead:c0de
e.d.0.c.d.a.e.d.e.e.5.0.1.0.0.0.9.4.8.5.0.3.0.2.7.0.2.c.2.0.a.2.ip6.arpa domain name pointer iseedeadcode.lerdorf.com.
```
- Debian 10
- Unlimited bandwidth (at 200 Mbps)

### Negatives
- Setup fee if you don't commit to a 12-month contract
- No locations other than Nuremberg (they have a Munich one too, but that wasn't given as an option)
- Provisioning took a while (34 minutes)
- Emailed cleartext contabo.com account password
- Emailed cleartext VPS root password
- Emailed cleartext VNC root password

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| 4 x Intel(R) Xeon(R) CPU E5-2630 v4 @ 2.20GHz
Ram| 8G
Disk| 200G
Bandwidth| Unlimited at 200 Mbps
Cost| €4.99/month
Server location| Nuremberg
Time to spin up new server| 34 minutes

Performance |
:--- | :---
PHP compile time| 1m45s
Disk IO|  133 MB/s write,  973 MB/s read, buffered reads 435  MB/s, cached reads 6504 MB/s
Network|  139 Mbits/sec down, 139 Mbits/sec up to Fremont, CA
| |       202 Mbits/sec down, 189 Mbits/sec up to Netherlands

### Conclusion
The 200G of storage and raw performance at this pricepoint is hard to beat. Disk writes were
a little slow, but reads were fast and you are a bit limited by the 200 Mbit switch port on
the smaller VPS. Jump to the €8.99 option for more bandwidth. Those emailed passwords though!
I understand from a customer support angle that it is easier to just do that, but it wouldn't
hurt them to have an "advanced" option somewhere where people who know what they are doing could
add their public keys.

<a id="scaleway"></a>

---

## [Scaleway](https://www.scaleway.com/en/pricing/)

Locations in Paris and Amsterdam. €2.99/month gets you 2 vCPU with 2G of ram and 20G NVMe storage.
Also, cheap ARM instances! For the same €2.99/month you can get a 4 core ARM instance with 2G of ram
and 50G of SSD. Or a 6 core/4G instance for €5.99. Of course I had to try the ARM instance. I chose
the €2.99 in their Paris data center.

![](/assets/images/low-cost-vps-testing/scaleway1.png)
![](/assets/images/low-cost-vps-testing/scaleway2.png)
![](/assets/images/low-cost-vps-testing/scaleway3.png)

### Positives
- Public key initial login with my own key
- Debian 10
- IPv6 with a /64
- Firewall in the admin console
- ARM!
- Unlimited fast network

### Negatives
- I couldn't find the rDNS management UI for my nice /64, but I may have missed it

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| 4 x ThunderX 88XX
Ram| 2G
Disk| 50G
Bandwidth| Unlimited at up to 1 Gbps
Cost| €2.99/month
Server location| Paris
Time to spin up new server| 2 minutes

Performance |
:--- | :---
PHP compile time| 7m14
Disk IO|   91 MB/s write,  412 MB/s read, buffered reads 251 MB/s, cached reads 691 MB/s
Network|   136 Mbits/sec down, 135 Mbits/sec up to Fremont, CA
| |        848 Mbits/sec down, 846 Mbits/sec up to Netherlands

### Conclusion
Since this is an ARM VPS, you can't really compare the performance numbers
directly. You get generous storage, 4 cores, decent disk read io and a fast
unlimited network connection.  It should work well for a small web server that
can handle a large number of concurrent requests as long as each one isn't too
cpu intensive. And Scaleway gets everything right on the provisioning side, so
their regular non-Arm VPS options look like a good choice too.

<a id="digitalocean"></a>

---

## [Digital Ocean](https://m.do.co/c/a9a3856b1b80)

Everyone knows Digital Ocean. I probably don't need to include it, but just to compare their current $5/month droplet to the others, I have.
You get 1 vCPU (shared), 1G of ram and 25G of SSD. I chose the San Francisco location. They also have NY, Amsterdam, Singapore, London, Frankfurt, Toronto
and Bangalore.

![](/assets/images/low-cost-vps-testing/docean1.png)
![](/assets/images/low-cost-vps-testing/docean2.png)
![](/assets/images/low-cost-vps-testing/docean3.png)

### Positives
- Public key initial login with my own key
- Debian 10
- IPv6
- Great provisoning experience
- Great management UI
- Firewall in the admin console
- Local Linux distro mirrors so your package installs are lightning quick
- Great documentation and tutorials for how to do all sorts of things with your Droplet

### Negatives
- You only get a /124 IPv6 subnet (16 IPs - how do they even do that?)

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| KVM
CPU| Intel(R) Xeon(R) Gold 6140 CPU @ 2.30GHz
Ram| 1G
Disk| 25G
Bandwidth| 1TB/month
Cost| $5/month
Server location| San Francisco
Time to spin up new server| 1 minute

Performance |
:--- | :---
PHP compile time| 4m28s
Disk IO|    476 MB/s write, 535 MB/s read, buffered reads 495 MB/s, cached reads 6741 MB/s
Network|    2 Gbits/sec down, 2 Gbits/sec up to Fremont, CA
| |       147 Mbits/sec down, 147 Mbits/sec up to Netherlands

### Conclusion
Digital Ocean has been a solid choice for years. Not necessarily the absolute fastest nor cheapest, but everything just works.
The management UI is great. It is a bit annoying that they only provide a non-standard IPv6 /128 subnet. They own the entire
2604:a880:: block and they can fit 4,294,967,296 /64 subnets in that. Over half the people on the planet would need to sign up
for a Digital Ocean VPS and enable IPv6 for them to run out. It is a bit nit-picky, of course. I use Digital Ocean myself and
a bunch of php.net services are hosted on instances there so I nit-pick on their IPv6 setup because there is nothing else to criticize.
For a $50 credit, you can use my [referral link](https://m.do.co/c/a9a3856b1b80)

<a id="buyvm"></a>

---

## [BuyVM](https://my.frantech.ca/aff.php?aff=3233)

There are lots of options below $10 here on both OpenVZ and KVM. They state,

    "BuyVM has developed multiple in-house solutions to mitigate "noisy neighbor" issues.

which is interesting and I would love to know exactly what they have done. But I am still going to
test one of the KVM options instead. They call them KVM Slice servers and they come in $2, $3.50, $7
of the ones under $10. 1 vCPU with 512M/1G/2G of ram and 10G/20G/40G of SSD storage. I went with the
middle $3.50 one because 512M is just not enough memory for any of my use cases.

They have locations in Las Vegas, NY and Luxembourg. I went with Las Vegas.

![](/assets/images/low-cost-vps-testing/buyvm1.png)
![](/assets/images/low-cost-vps-testing/buyvm2.png)
![](/assets/images/low-cost-vps-testing/buyvm3.png)

### Positives
- Debian 10
- IPv6 with a /64 subnet and rDNS (not enabled by default)
- Unlimited fast network

### Negatives
- No public key initial login
- They emailed me a plaintext password for the management UI
- They ask you to choose a hostname, and apparently it has to be unique across all customers?
- Server didn't boot initially (No bootable device)
<div style="clear: both;"></div>
    ![](/assets/images/low-cost-vps-testing/buyvm.png)
- When I finally did get it up, the Debian 10 installed was a bit old. Took a while to dist-upgrade

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| Microsoft Hyper-V on top of Qemu
CPU| Intel(R) Xeon(R) CPU E3-1270 v3 @ 3.50GHz
Ram| 1G
Disk| 20G
Bandwidth| Unlimited at 1 Gbps
Cost| $3.50/month
Server location| Las Vegas
Time to spin up new server| 10 minutes

Performance |
:--- | :---
PHP compile time| 2m48s
Disk IO|    501 MB/s write, 861 MB/s read, buffered reads 917 MB/s, cached reads 12124 MB/s
Network|    919 Mbits/sec down, 919 Mbits/sec up to Fremont, CA
| |         153 Mbits/sec down, 153 Mbits/sec up to Netherlands

### Conclusion

The VM didn't boot initially. I went through the management console and
requested a reinstall and it came up after that. Performance was near the top
of all the providers tested and the management UI, called Stallion, is ok. It
looks like it is all written in PHP, by the way. Interestingly you can change
your CPU model between "Intel/AMD" and "Qemu". You can also toggle APCI, APIC
and PAE and choose your network and hard disk drivers. If you are running
Windows in your VM this would be handy.

The IPV6 part of the management console gave the impression that it could
enable it in the VM, but that didn't work for me. I had to do it manually. It
probably works on other operating systems. Easy enough, just modify
`/etc/network/interfaces` and add:
```
    iface eth0 inet6 static
        address 2605:6400:0020:016a:1:5ee:bad:c0de
        gateway 2605:6400:0020::1
```
It is a /64 with rDNS, so iseebadcode is back:
```shell-session
    $ host 2605:6400:0020:016a:1:5ee:bad:c0de
    e.d.0.c.d.a.b.0.e.e.5.0.1.0.0.0.a.6.1.0.0.2.0.0.0.0.4.6.5.0.6.2.ip6.arpa domain name pointer iseebadcode.lerdorf.com.
```
And speaking of rDNS, the previous owner's rDNS was left on my IPv4. It was hsiaokang.pigbaby.com.

Despite the bad first impression, this turned out to be a solid VM.
[Referral link](https://my.frantech.ca/aff.php?aff=3233)

<a id="lightsail"></a>

---

## [Amazon Lightsail](https://aws.amazon.com/lightsail/pricing/)

I find the AWS UI hard to navigate. But once you find your way to the Lightsail section it gets a bit easier.
Upload your public key, choose an OS and an instance plan. The $3.50/month was free for a month, so I picked
that one. 1 vCPU, 512M of ram and 20G SSD. For $5 you get 1G/40G which is probably the one you should get if
you are going to go with Lightsail.
<div style="clear: both;"></div>
![](/assets/images/low-cost-vps-testing/ls1.png)
![](/assets/images/low-cost-vps-testing/ls2.png)
![](/assets/images/low-cost-vps-testing/ls3.png)
![](/assets/images/low-cost-vps-testing/ls4.png)
<div style="clear: both;"></div>
![](/assets/images/low-cost-vps-testing/lsc2.png)
![](/assets/images/low-cost-vps-testing/lsc3.png)

### Positives
- Public key initial login with my own key

### Negatives
- No Debian 10 and took a while to dist-upgrade because the latest was 9.5
- No IPv6

Specs |
:--- | :---
OS| 9.11 (upgraded from 9.5)
Kernel| 4.9.0
Hypervisor| Xen
CPU| Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz
Ram| 512M
Disk| 20G
Bandwidth| 1TB/month
Cost| $3.50/month
Server location| Montreal
Time to spin up new server| 2 minutes

Performance |
:--- | :---
PHP compile time| 2m53s
Disk IO|    65 MB/s write, 65 MB/s read, buffered reads 81 MB/s, cached reads 9774 MB/s
Network|    27 Mbits/sec down, 24 Mbits/sec up to Fremont, CA
| |         42 Mbits/sec down, 40 Mbits/sec up to Netherlands

### Conclusion
The lack of IPv6 from a major provider is baffling to me. Disk IO performance was relatively poor and somehow
network from Montreal to Europe was faster than to California. But both were much slower than any other provider
tested here. Perhaps they have throttled both disk and network IO on this free mini instance. As tested, this
doesn't seem like a very serious offering.

<a id="azure"></a>

---

## [Azure](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/series/)

My last time around looking at Azure I got completely lost and frustrated in the UI. 
Again, after way too many clicks I got to a list of VM options which had 221 entries. At least I could sort
it by monthly cost. Only two options below $10. B1ls and B1s for $3.87 and $7.74. 512M vs. 1G of ram, so I chose the larger
of the two in the US-West-2 region.
<div style="clear: both;"></div>
![](/assets/images/low-cost-vps-testing/azure1.png)
![](/assets/images/low-cost-vps-testing/azure2.png)
![](/assets/images/low-cost-vps-testing/azure3.png)

### Positives
- Free trial worth $200
- Debian 10
- Public key initial login with my own key and to my own specified account

### Negatives
- It took a lot of clicking to get to the point of actually provisioning the VM

Specs |
:--- | :---
OS| Debian 10.1
Kernel| 4.19.0
Hypervisor| Microsoft Hyper-V
CPU| Intel(R) Xeon(R) CPU E5-2673 v4 @ 2.30GHz
Ram| 1G
Disk| 30G
Bandwidth| I gave up trying to figure out what the bandwidth limits/costs were
Cost| $7.74/month
Server location| US-West
Time to spin up new server| 3 minutes

Performance |
:--- | :---
PHP compile time| 3m25s
Disk IO|    24 MB/s write, 28 MB/s read, buffered reads 25 MB/s, cached reads 8339 MB/s
Network|    905 Mbits/sec down, 904 Mbits/sec up to Fremont, CA
| |         140 Mbits/sec down, 139 Mbits/sec up to Netherlands

### Conclusion
I didn't think it was possible to get slower than Amazon Lightsail's disk IO, but Azure managed to somehow.
CPU and network performance was good though. And I liked that I was able to specify my own initial account
to create. But for the price, there are better offerings out there.

<a id="skysilk"></a>

---

## [Skysilk](https://www.skysilk.com/ref/CDdqohO0Om)

Signup was smooth. I liked that it was possible to turn off password logins entirely before the VPS was even created and the process
to add an ssh key was trivial. Locations are Los Angeles and NY. The LA data center has more bandwidth and it is
closer to me, so I chose that for a $5/month VPS which gave me 1 vCPU, 2G of ram and 30G of SSD.
<div style="clear: both;"></div>
![](/assets/images/low-cost-vps-testing/ss1.png)
![](/assets/images/low-cost-vps-testing/ss2.png)
![](/assets/images/low-cost-vps-testing/ss3.png)

### Positives
- Public key initial login with my own key and password logins optionally disabled
- IPv6 with rDNS
- Interesting "Boost" mode where for 50 cents you can boost your VPS for 24 hours. In my case that meant doubling the ram from 2G to 4G.

### Negatives
- No Debian 10
- LXC, like OpenVZ, this is not hardware virtualization so resource isolation is not as good
- Only an IPv6 /124 and they don't make it obvious that you can have 16 IPv6 IPs
- rDNS UI only lets you set the rDNS for one IPv6 ip. They probably didn't intend to let users add IPs manually like I did?

Specs |
:--- | :---
OS| Debian 9.11 (Upgraded from the installed 9.1)
Kernel| 4.15.18
Hypervisor| LXC
CPU| Intel(R) Xeon(R) CPU E5-2678 v3 @ 2.50GHz
Ram| 2G
Disk| 30G
Bandwidth| 12TB/month at up to 500 Mbps
Cost| $5/month
Server location| Los Angeles
Time to spin up new server| 2 minutes

Performance |
:--- | :---
PHP compile time| 2m45s
Disk IO|    217 MB/s write, 131 MB/s read, buffered reads N/A MB/s, cached reads N/A MB/s
Network|    201 Mbits/sec down, 183 Mbits/sec up to Fremont, CA
| |         111 Mbits/sec down, 99 Mbits/sec up to Netherlands

### Conclusion
I prefer hardware virtualization for the better resource isolation. That said, this was a good LXC implementation with public key setup, password logins disabled
and IPv6 with a /124 (16 IPs) and it got under the 3-minute mark for the PHP compile.  This is not a bad choice if you are ok with a container instead of a hardware
virt. [Referral link](https://www.skysilk.com/ref/CDdqohO0Om)

<a id="vpsserver"></a>

---

## [vpsserver.com](https://www.vpsserver.com/?affcode=ee6c3a3a3688)
Locations in Silicon Valley, Toronto, Tokyo, London, Frankfurt, Amsterdam, NY, Singapore, Sydney, Hong Kong, Miami, Dallas and Chicago. For $4.99 you get 1 vCPU, 1G of ramand 25G of SSD. $9.99 doubles that, including the CPUs which is a pretty good deal. Since $9.99 just sneaks in under my < $10 rule for this test, I chose that one in the Silicon Valley location.
Smooth signup process. I hadn't added my SSH key before I created the VPS, but that was my mistake. I do prefer when that is part of the flow so you don't have to
go looking for it.

### Positives
- 1-day free trial
- 7-day free trial w/ credit card
- Public key initial login with my own key
- Firewall in the admin console
- Debian 10
- IPv6 available as an option in the admin console

### Negatives
- You don't get an IPv6 /64, but you can request multiple free IPv6 IPs from the console
- I noticed some network blips pinging the IPv6 interface
- There were two keys that weren't mine in `~root/.ssh/authorized_keys` presumably for the admin console (at least I hope that is what they were for)

Specs |
:--- | :---
OS| Debian 10.1 (upgraded from 10.0)
Kernel| 4.19.0
Hypervisor| KVM
CPU| 2 x QEMU Virtual CPU version 2.5+
Ram| 2G
Disk| 50G
Bandwidth| 2TB/month
Cost| $9.99/month
Server location| Silicon Valley
Time to spin up new server| 5 minutes

Performance |
:--- | :---
PHP compile time| 2m46s
Disk IO|    778 MB/s write, 664 MB/s read, buffered reads 292 MB/s, cached reads 5918 MB/s
Network|    517 Mbits/sec down, 517 Mbits/sec up to Fremont, CA
| |         141 Mbits/sec down, 141 Mbits/sec up to Netherlands

### Conclusion
I am not that enthusiastic about admin consoles that log into my VPS and make changes to my network config and potentially other things.
Once deployed, nobody and nothing should have access to my server except me. To allocate me another IP, just give me the IP info and if you
want to hold my hand give me a short snippet to add to my `/etc/network/interfaces` file.

I would have preferred a /64 IPv6 subnet. They own the full 2604:bc0:: block, so they literally have over 4 billion /64 subnets available to them.
Interestingly I noticed that 2604:bc0:1:: is assigned to their Chicago data center, 2604:bc0:2:: to NY, :3 to Silicon Valley, and :4 to Toronto.


That aside, this was a nice setup. Compile time under 3 minutes, which you would expect from a dual-core VM, and both disk and network IO were fast.
[Referral link](https://www.vpsserver.com/?affcode=ee6c3a3a3688)

<a id="81vps"></a>

---

## [81vps.com](https://81vps.com)

For $4/month you get 1 vCPU, 1G of RAM and 15G of storage. But they also have some really cheap 6 and 12-month deals. Interestingly they have an option
for an Asia-Pacific optimized network out of their Los Angeles data center location.

### Positives
- Provisioning was lightning quick
- Nearly unlimited fast network

### Negatives
- No public key initial login
- Emailed me my VPS root password
- Management console a bit slow and awkward to use
- no IPv6
- No Debian 10

Specs |
:--- | :---
OS| Debian 9.11 (upgraded from Debian 9.4)
Kernel| 4.9.0
Hypervisor| KVM
CPU| Intel(R) Xeon(R) CPU E5-2620 v2 @ 2.10GHz
Ram| 1G
Disk| 15G
Bandwidth| 100TB/month on a 1 Gbps port
Cost| $4/month
Server location| Los Angeles
Time to spin up new server| 1 minute

Performance |
:--- | :---
PHP compile time| 9m4s
Disk IO|   275 MB/s write, 157 MB/s read, buffered reads 205 MB/s, cached reads 1165 MB/s
Network|   458 Mbits/sec down, 456 Mbits/sec up to Fremont, CA
| |        141 Mbits/sec down, 140 Mbits/sec up to Netherlands

### Conclusion
Really slow PHP compile time. Disk IO wasn't super-fast either, but not slow enough to be causing such a slow
PHP compile time. There just wasn't enough CPU available. Network speeds were good with a really high transfer limit.

