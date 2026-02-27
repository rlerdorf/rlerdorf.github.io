---
date: '2018-01-22'
layout: post
permalink: /archives/61-Testing-VPS-solutions.html
title: Testing VPS solutions
---


I am trying to see if I should move from my own hardware sitting in a
data center in Milpitas to a VPS. My main criteria is that I need at
least 4 decently fast cores and at least 8G of memory. I also need about
500G of storage and low ping times from home in Silicon Valley. I was
originally just trying to figure out how much faster DigitalOcean's
optimized droplets were compared to the standard and posted that to
Twitter. Scope creep happened and I ended up testing 10 different
providers.  

For the lazy, my verdict for all 10 is right here (with a couple of
referral links). For the full details including CPU, disk and network
tests along with more detailed observations and screenshots, read on.

**DigitalOcean**         [referral - get $10
credit](https://m.do.co/c/a9a3856b1b80 "DigitalOcean referral")  
*$40/month 8 GB 4 core 160 GB SSD + $50/month 500GB Volume in SFO2  
make 34m38s 2nd try 4m39s, disk write 350MB/s, read 1.8 GB/s(\!), net 2
Gbits/s*  
  

The provisioning process is amazing. Fast and responsive. Support is
quick and effective. I was a bit disappointed by the performance of the
standard droplets, especially the first one I tested, but the *$80 8GB/4
core/50GB SSD* optimized droplets absolutely scream only being beat on
PHP compile time by a bare metal Vultr box.

-----

**Vultr**         [referral - get $10
credit](https://www.vultr.com/?ref=7314620 "Vultr referrral")  
*$40/month 8GB 4 core 100GB SSD in Los Angeles  
make 3m,16s, disk write 306MB/s, read 200MB/s, net 2.5 Gbits/s*  
  

A serious competitor to Digital Ocean. I would use this. Especially if
they brought block storage to the west coast. Price and performance is
great and the Web UI for provisioning and managing instances is clear
and easy to use. Even without the block storage, the bare metal instance
with the 2x240 GB SSDs has adequate space. Since it is bare-metal I
assume I would need to mirror the two drives for redundancy so it is
still not close enough to my 500GB target.

-----

**Linode**        
 [referral](https://www.linode.com/?r=2c964a203b1250c4e79673c2418b108ef70e734f "Linode referral link")  
*$40/month 8GB 4 core 96GB SSD + $50/month 500GB volume in Fremont,CA  
make 4m15s, disk write 634 MB/s, read 355 MB/s, net 1.1 Gbits/s*  
  

Everything just worked and performance was acceptable across the board
with the only exception being block volume reads. I found those to be a
bit too slow. The price/performance ratio is good. At the common
$80/month price point you get 12 GB of ram, 6 cores and 192 GB SSD. If
the block volume reads performance is improved, I could use this.

-----

**GCP (Google Cloud Platform)**  
*$88/month 8GB 4 core 10GB SSD + $20/month 500GB HDD in Oregon  
make 4m8s, disk write 159 MB/s, read 98 MB/s, net 1 Gbits/s*  
  

With the lower-cost HDD block volume storage, GCP is interesting. But I
had some performance confusion testing HDD vs. SDD and for $88 it would
be nice to get a larger SSD. On the wrong side of the price/performance
ratio for me.

-----

**Upcloud**         [referral - get $25
credit](https://www.upcloud.com/register/?promo=7QZ77S "Upcloud referral link")  
*$80/month 8GB 4 core 70GB SSD + $110/month for 500GB in Chicago  
make 2m22s, disk write 481 MB/s, read 420 MB/s, net 438 Mbits/s*  
  

Good price/performance ratio and if they would bring their cheaper class
of block volume service to the U.S. this would be an option for me. As
it is right now, I would have to pay $110/month for the extra 500GB of
space I need on top of the $80/month for the VPS and that puts it out of
my price range.

-----

**AWS Lightsail**  
*$80/month 8GB 2 core 80GB SSD + $50/month for 500GB in Oregon  
make 4m23s, disk write 249 MB/s, read 130 MB/s, net 140 Mbits/s*  
  

Decent performance for a 2-core VPS. I couldn't figure out how to
provision a 4-core one. Probably user error on my part, but I did try
for a while. I only have so much patience for large complex Web UIs.
Lightsail also didn't have Debian 9 as an option at the time. Debian 8
only. $80/month for a 2 core VPS with average performance is on the
expensive end of the spectrum, so not for me.

-----

**VMHaus**  
*$28/month 8GB 4 core 100GB nvme SSD in Los Angeles  
make 3m11s, disk write 286 MB/s, read 335 MB/s, net 830 Mbits/s*  
  

Excellent price/performance ratio and the provisioning process was fast
and efficient. But I can't use it since there is no way to add extra
storage.

-----

**OVH**  
*$35/month 8GB, 4 cores, 100GB SSD + $42/month for 500GB Volume in
Beauharnois, Canada  
make 4m18s (2 core only), disk write 550 MB/s, read 260 MB/s, net 100
Mbits/s*  
  

Terrible provisioning process. The Web UI is atrocious and some things
simply didn't work. When I finally did get my VPS, it worked well
though, so that one-time hassle shouldn't discourage you. Since I didn't
find a free trial, I tested a cheaper 2-core option with no block
storage, but it performed well. If they had a west coast POP it could be
an option, but without that it isn't viable for me.

-----

**Azure**  
*$133/month 4 core 60GB SSD in West US 2 (Oregon?)  
make 3m34s, disk write 23 MB/s, read 16 MB/s, net 1.23 Gbits/s*  
  

This one was painful. Yes, I have a bit of a Microsoft aversion, but I
tried to keep an open mind. Read the full description of my Azure
adventure. Expensive, apparently no IPv6, slow disk IO, and I couldn't
figure out block storage options. Definitely not for me.

-----

**Contabo**  
*$11/month 12GB 4 core 300GB SSD somewhere in Germany  
make 3m47s, disk write 144 MB/s, read 308 MB/s, net 88 Mbits/s*  
  

I like Contabo. It took perhaps 20 minutes to get my VPS and there was
packet loss on the network, but that was resolved. For $11/month this is
an outstanding deal. Being in Europe with no N.American POP I can't use
it as my primary VPS, but I will probably keep this one just to have a
personal box in Europe to play with.

-----

# Full Observations and test numbers

My preferred OS is Debian 9.3 so that is what I chose for all of these.
My steps after provisioning:  

    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    sudo reboot (if the above pulled in a new kernel or anything significant)
    
    sudo apt-get install git gcc make autoconf automake bison flex libxml2-dev
    git clone https://github.com/php/php-src.git
    cd php-src
    ./buildconf -f
    ./configure
    time make -j 4

For the disk tests, I cleared the cache with:

    /sbin/sysctl -w vm.drop_caches=3

between the write and the read test.

Network tests are biased towards providers with a point of presence near
Hurricane Electric in Fremont, CA. I am using their public iperf server
at iperf.he.net to test ping and bandwidth.

There is, of course, much more to VPS hosting than how fast you can
compile PHP on it, but for my heavy compile/valgrind/compile/valgrind
workload along with various low-volume friends+family west-coast
centric-hosting, and a few servers like irc, Minecraft and other random
things running, I figure if it compiles PHP quickly, chances are it will
do those others things pretty fast as well. Completely unscientific.
Standard disclaimers apply here. Never trust other peoples' tests. Do
your own tests for your typical workloads.

My base reference is my own box. It is an 8GB 4 core box with a ton of
disk from 2011. It is getting old and slow so the big decision is
whether to buy a kick-ass new box with probably at least 8 cores and
32GB of memory and fast mirrored SSDs for around $3000 and continue
paying $100/month to park that in the data center, or can one of these
providers convince me that the price-performance-hassle dance has
finally swung their way?

With hosting at $100 and spreading the $3000 cost of the server out over
the 7-year lifespan, that gives a budget of around $135. Anything in the
vicinity of $135/month will work out to about the same cost over the
next 7 years. And I have to keep in mind that my own hardware won't get
faster over those 7 years. Chances are I can upgrade my VPS during that
time and stay at the same monthly cost.

My current ancient box builds PHP (with ccache and all other caches
cleared) in 4m55s. Anything I move to has to beat that.

  

-----

-----

  

# Digital Ocean

*Standard Droplet $40/month extra 500G storage volume $50/month  
8 GB 4 vCPUs 160 GB SSD in SFO2  
Debian 9.3 x64  
*

This was super disappointing. As far as I am concerned this VPS was
unusable. It took over 34 minutes to compile PHP where no other VPS I
tested took over 5 minutes. And in general everything I did on it felt
sluggish. I even opened a support ticket on it because I didn't think
this level of crap performance could be normal. And yes, I tried it a
couple of times over 2 days, and the performance was bad the whole time.
Also the disk write performance of only 36 MB/s didn't seem healthy.
Perhaps I just got unlucky. Digital Ocean provides instances for the PHP
project and I haven't seen an instance quite this slow, but that is the
risk you take with a shared resource VPS. You could end up with a noisy
neighbour and/or a provider that over provisions the host you are on.

    $ make -j 4
    real   34m38.738s
    user   113m0.244s
    sys     7m42.592s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    1
    Core(s) per socket:    1
    Socket(s):             4
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 79
    Model name:            Intel(R) Xeon(R) CPU E5-2650 v4 @ 2.20GHz
    Stepping:              1
    CPU MHz:               2199.998
    BogoMIPS:              4399.99
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              256K
    L3 cache:              30720K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           799M   11M  789M   2% /run
    /dev/vda1       158G   11G  141G   7% /
    tmpfs           4.0G     0  4.0G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           4.0G     0  4.0G   0% /sys/fs/cgroup
    tmpfs           799M     0  799M   0% /run/user/1001

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 234.35 s, 36.7 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    vm.drop_caches = 3
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 44.3119 s, 194 MB/s
    
    $ sudo hdparm -Tt /dev/vda1
    /dev/vda1:
     Timing cached reads:   1680 MB in  2.00 seconds = 841.56 MB/sec
     Timing buffered disk reads: 342 MB in  3.01 seconds = 113.76 MB/sec

    Network
    Ping 4ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  1.90 GBytes  1.63 Gbits/sec  1749             sender
    [  4]   0.00-10.00  sec  1.90 GBytes  1.63 Gbits/sec                  receiver

-----

*Standard Droplet $40/month 2nd try  
8 GB 4 vCPUs 160 GB SSD in SFO2  
Debian 9.3 x64  
*

After receiving this response to my support ticket asking about the
performance.

![DigitalOcean Support
response](/assets/images/testing-vps-solutions/DO_support.png)  

I tore down that first attempt and tried creating a new identical
Droplet also in SFO2. The response is reasonable if they actually
checked and didn't find a hardware issue. Who knows what idiot customers
do to slow down their VMs... In this case I hadn't done anything weird
to my OS, of course, but they have no way of knowing that. This 2nd try
gave:

    $ make -j 4
    real    4m39.342s
    user    14m6.524s
    sys 0m51.288s

That's a vast improvement over the 34 minutes and more in line with what
I would expect.

Also, disk write performance magically jumped from 36 to 122 MB/s. And
buffered disk reads from 194 to 695 MB/s. So if there was no hardware
issue, then it was a very persistent noisy neighbour.

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 70.6552 s, 122 MB/s
    $ hdparm -Tt /dev/vda1
     Timing cached reads:   14878 MB in  1.99 seconds = 7458.18 MB/sec
     Timing buffered disk reads: 2086 MB in  3.00 seconds = 694.72 MB/sec

-----

*Optimized Droplet $80/month extra 500G storage volume $50/month  
8 GB 4 vCPUs 50 GB SSD in SFO2  
Debian 9.3 x64  
*

The experience here was the complete opposite of the standard droplet
experience. This VPS felt healthy and responsive and was a joy to use.

Adding a 500G storage volume is dead-simple. And they hold your hand:

![DigitalOcean Volume
Documentation](/assets/images/testing-vps-solutions/DO_volume1.png)  

But, as you can see from the disk perf test below, attached volumes
aren't as fast as the blazingly fast natively attached storage. Plenty
fast for random Web content though.

    $ time make -j 4
    real   1m57.448s
    user   5m49.412s
    sys    0m18.112s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    1
    Core(s) per socket:    1
    Socket(s):             4
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 85
    Model name:            Intel(R) Xeon(R) Platinum 8168 CPU @ 2.70GHz
    Stepping:              4
    CPU MHz:               2693.682
    BogoMIPS:              5387.36
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              1024K
    L3 cache:              33792K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           799M   11M  789M   2% /run
    /dev/vda1        50G  2.2G   46G   5% /
    tmpfs           4.0G     0  4.0G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           4.0G     0  4.0G   0% /sys/fs/cgroup
    tmpfs           799M     0  799M   0% /run/user/1001
    /dev/sda        492G   73M  467G   1% /mnt/volume-sfo2-01

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 24.5217 s, 350 MB/s
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 4.77441 s, 1.8 GB/s
    
    $ sync; dd if=/dev/zero of=/mnt/volume-sfo2-01/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 38.1517 s, 225 MB/s
    $ dd if=/mnt/volume-sfo2-01/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 45.3508 s, 189 MB/s
    
    $ sudo hdparm -Tt /dev/vda1
    /dev/vda1:
     Timing cached reads:   13046 MB in  2.00 seconds = 6534.63 MB/sec
     Timing buffered disk reads: 5364 MB in  3.00 seconds = 1787.94 MB/sec
    
    $ sudo hdparm -Tt /dev/sda
    /dev/sda:
     Timing cached reads:   13114 MB in  2.00 seconds = 6568.30 MB/sec
     Timing buffered disk reads: 588 MB in  3.00 seconds = 195.85 MB/sec

    Network
    Ping 3ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  2.30 GBytes  1.98 Gbits/sec  3323             sender
    [  4]   0.00-10.00  sec  2.30 GBytes  1.98 Gbits/sec                  receiver

  

-----

-----

  

# Vultr

The Vultr experience was good. The web UI is clear and easy to use. So
easy that I ended up testing 4 different setups. My target was 8GB of
ram and 4 cores, but since Vultr had an $80 option that gave 16/6, I had
to test that too. New instances, even the bare metal one, come up
quickly with my ssh key installed. IPv6 works, and the performance is
great across the board. CPU, disk io and network are all at the top of
what I tested. The only negative for me was that when I went to add my
500GB block storage to my Silicon Valley or Los Angeles instances, I
found it wasn't possible. Block storage is only available to New Jersey
hosted instances. For people on the east coast or people less sensitive
to higher ping latency that probably doesn't matter. Just put your
instances in New Jersey.

![Vulr VPS list](/assets/images/testing-vps-solutions/vultr.png)

And look further down at that performance on the bare metal instance. It
makes me want to buy myself a new colo box instead of going with a VPS
at all :)

It was also interesting that the shared instances in Los Angeles had a
fatter pipe than the Silicon Valley ones. Perhaps more to do with the
location than the type of instance? Not sure.

*Shared instance $40/month  
8GB 4 core 100GB SSD in Los Angeles  
Debian 9.3 x64  
*

    $ time make -j 4
    real    3m16.440s
    user    10m10.296s
    sys 0m27.888s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    1
    Core(s) per socket:    4
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 61
    Model name:            Virtual CPU a7769a6388d5
    Stepping:              2
    CPU MHz:               2394.454
    BogoMIPS:              4788.90
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              4096K
    L3 cache:              16384K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           799M  8.4M  791M   2% /run
    /dev/vda1        99G  1.2G   93G   2% /
    tmpfs           4.0G     0  4.0G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           4.0G     0  4.0G   0% /sys/fs/cgroup
    tmpfs           799M     0  799M   0% /run/user/0

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 28.1047 s, 306 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    vm.drop_caches = 3
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 43.4428 s, 198 MB/s
    
    $ sudo hdparm -Tt /dev/vda1
     Timing cached reads:   14174 MB in  2.00 seconds = 7103.81 MB/sec
     Timing buffered disk reads: 620 MB in  3.01 seconds = 206.17 MB/se

    Network
    Ping 9ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  2.92 GBytes  2.50 Gbits/sec    2             sender
    [  4]   0.00-10.00  sec  2.92 GBytes  2.50 Gbits/sec                  receiver

-----

*Shared instance $80/month  
16GB 6 core 200GB SSD in Los Angeles  
Debian 9.3 x64  
*

    $ time make -j 6
    real    2m53.018s
    user    11m22.284s
    sys 0m42.356s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                6
    On-line CPU(s) list:   0-5
    Thread(s) per core:    1
    Core(s) per socket:    6
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 61
    Model name:            Virtual CPU a7769a6388d5
    Stepping:              2
    CPU MHz:               2394.454
    BogoMIPS:              4788.90
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              4096K
    L3 cache:              16384K
    NUMA node0 CPU(s):     0-5

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            7.9G     0  7.9G   0% /dev
    tmpfs           1.6G  8.5M  1.6G   1% /run
    /dev/vda1       197G  2.3G  185G   2% /
    tmpfs           7.9G     0  7.9G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           7.9G     0  7.9G   0% /sys/fs/cgroup
    tmpfs           1.6G     0  1.6G   0% /run/user/0

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 26.5059 s, 324 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    vm.drop_caches = 3
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 45.6597 s, 188 MB/s
    
    $ sudo hdparm -Tt /dev/vda1
     Timing cached reads:   16254 MB in  1.99 seconds = 8169.12 MB/sec
     Timing buffered disk reads: 612 MB in  3.01 seconds = 203.42 MB/sec

    Network
    Ping 9ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  3.50 GBytes  3.01 Gbits/sec  780             sender
    [  4]   0.00-10.00  sec  3.50 GBytes  3.01 Gbits/sec                  receiver

-----

*Vultr dedicated instance $120/month  
16GB 4 core 2x120GB SSD in Silicon Valley  
Debian 9.3 x64  
*

    $ time make -j 4
    real    2m57.574s
    user    9m27.568s
    sys 0m23.828s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    1
    Core(s) per socket:    4
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 60
    Model name:            Virtual CPU 714389bda930
    Stepping:              1
    CPU MHz:               3600.010
    BogoMIPS:              7200.02
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              4096K
    L3 cache:              16384K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            7.9G     0  7.9G   0% /dev
    tmpfs           1.6G  8.5M  1.6G   1% /run
    /dev/vda1       109G  1.2G  102G   2% /
    tmpfs           7.9G     0  7.9G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           7.9G     0  7.9G   0% /sys/fs/cgroup
    tmpfs           1.6G     0  1.6G   0% /run/user/0

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 18.0482 s, 476 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 24.5393 s, 350 MB/s
    $ sudo hdparm -Tt /dev/vda1
     Timing cached reads:   24706 MB in  1.99 seconds = 12385.75 MB/sec
     Timing buffered disk reads: 1144 MB in  3.00 seconds = 381.07 MB/sec

    Network
    Ping 1ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec   957 MBytes   803 Mbits/sec   59             sender
    [  4]   0.00-10.00  sec   953 MBytes   800 Mbits/sec                  receiver

-----

*Vultr bare metal instance $120/month  
32GB 8 core 2x240GB SSD in Silicon Valley  
Debian 9.3 x64  
*

    $ time make -j 8
    real    1m25.129s
    user    8m4.512s
    sys 0m14.576s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                8
    On-line CPU(s) list:   0-7
    Thread(s) per core:    2
    Core(s) per socket:    4
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 158
    Model name:            Intel(R) Xeon(R) CPU E3-1270 v6 @ 3.80GHz
    Stepping:              9
    CPU MHz:               799.938
    CPU max MHz:           4200.0000
    CPU min MHz:           800.0000
    BogoMIPS:              7584.00
    Virtualization:        VT-x
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              256K
    L3 cache:              8192K
    NUMA node0 CPU(s):     0-7

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev             16G     0   16G   0% /dev
    tmpfs           3.2G  8.9M  3.2G   1% /run
    /dev/sda1       221G  1.2G  208G   1% /
    tmpfs            16G     0   16G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs            16G     0   16G   0% /sys/fs/cgroup
    /dev/sdb1       220G   61M  208G   1% /data
    tmpfs           3.2G     0  3.2G   0% /run/user/0

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 11.2637 s, 763 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 43.6689 s, 197 MB/s
    
    $ sudo hdparm -Tt /dev/sda1
     Timing cached reads:   35872 MB in  1.99 seconds = 17998.86 MB/sec
     Timing buffered disk reads: 640 MB in  3.01 seconds = 212.75 MB/sec
    
    $ sudo hdparm -Tt /dev/sdb1
     Timing cached reads:   35656 MB in  1.99 seconds = 17889.01 MB/sec
     Timing buffered disk reads: 1512 MB in  3.00 seconds = 503.51 MB/sec

    Network
    Ping 1ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  1.87 GBytes  1.60 Gbits/sec  2040             sender
    [  4]   0.00-10.00  sec  1.86 GBytes  1.60 Gbits/sec                  receiver

  

-----

-----

  

# Linode

Linode has been around forever. The Web UI kind of feels outdated but it
is functional and has everything you need. I also see they are working
on a new UI. The UI is the last thing I care about as long as it lets me
do what I need, and it did. It was also easy to add a 500G volume and
attach it to my instance.

-----

*Linode instance $40/month plus $50/month for a 500GB block storage
volume  
8GB 4 core 96GB disk in Fremont,CA  
Debian 9.3 x64  
*

    $ time make -j 4
    real    4m15.778s
    user   13m50.316s
    sys     0m43.592s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    1
    Core(s) per socket:    1
    Socket(s):             4
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 63
    Model name:            Intel(R) Xeon(R) CPU E5-2680 v3 @ 2.50GHz
    Stepping:              2
    CPU MHz:               2499.996
    BogoMIPS:              5001.32
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              4096K
    L3 cache:              16384K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/root        95G  1.3G   89G   2% /
    devtmpfs        3.9G     0  3.9G   0% /dev
    tmpfs           3.9G     0  3.9G   0% /dev/shm
    tmpfs           3.9G  9.5M  3.9G   1% /run
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
    tmpfs           798M     0  798M   0% /run/user/0
    /dev/sdc        492G   73M  467G   1% /mnt/data

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 13.5535 s, 634 MB/s
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 24.1752 s, 355 MB/s
    
    $ sudo hdparm -Tt /dev/sda
     Timing cached reads:   14488 MB in  1.99 seconds = 7263.31 MB/sec
     Timing buffered disk reads: 608 MB in  3.00 seconds = 202.50 MB/sec
    
    $ sync; dd if=/dev/zero of=/mnt/data/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 42.2463 s, 203 MB/s
    $ dd if=/mnt/data/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 133.053 s, 64.6 MB/s
    
    $ sudo hdparm -Tt /dev/sdc
     Timing cached reads:   14206 MB in  1.99 seconds = 7121.47 MB/sec
     Timing buffered disk reads: 164 MB in  3.02 seconds =  54.36 MB/sec

    Network
    Ping 2ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  1.28 GBytes  1.10 Gbits/sec  5731             sender
    [  4]   0.00-10.00  sec  1.28 GBytes  1.10 Gbits/sec                  receiver

  

-----

-----

  

# GCP

I find the GCP UI annoying. But again, the target audience here isn't
the enthusiast managing one instance. It is aimed at admins managing
hundreds of servers. But again, it got the job done after a bit of
fiddling so not an issue. I really liked the fact that there was a
low-cost block volume option. $0.04/GB per month for standard and $0.170
for SSD. And no per-IO ops charges like Azure has. Attaching it to an
instance via gcloud is a bit fiddly. Had to read the docs to get
    to:

    gcloud compute instances attach-disk instance-1 --disk data --zone us-west1-a

but again, if I was managing hundreds or thousands of these I would
definitely appreciate the cli approach. And I guessed my way to finding
it with:

    fdisk /dev/sdb

I am sure it is documented somewhere, but nobody holds your hand on GCP.
It also wasn't until I played with the disks that I realized that I had
created my instance with a "standard" boot disk. You can see in the disk
perf numbers that they aren't great.

-----

*GCP custom instance $86.38/month, $20 for 500GB HDD or $85 for 500GB
SDD  
8GB 4 vCPUs 10GB SSD in us-west-1a  
Debian 9.3 x64  
*

    $ time make -j 4
    real    4m8.294s
    user    13m37.076s
    sys     0m32.440s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    2
    Core(s) per socket:    2
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 79
    Model name:            Intel(R) Xeon(R) CPU @ 2.20GHz
    Stepping:              0
    CPU MHz:               2200.000
    BogoMIPS:              4400.00
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              256K
    L3 cache:              56320K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           799M   10M  789M   2% /run
    /dev/sda1       9.8G  2.2G  7.1G  24% /
    tmpfs           4.0G     0  4.0G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           4.0G     0  4.0G   0% /sys/fs/cgroup
    /dev/sdb1       492G   73M  467G   1% /mnt/data

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=4096; sync
    4294967296 bytes (4.3 GB, 4.0 GiB) copied, 26.9569 s, 159 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    $ dd if=/root/tempfile of=/dev/null bs=1M count=4096
    4294967296 bytes (4.3 GB, 4.0 GiB) copied, 44.0671 s, 97.5 MB/s
    
    $ sudo hdparm -Tt /dev/sda1
     Timing cached reads:   17362 MB in  1.99 seconds = 8703.42 MB/sec
     Timing buffered disk reads: 256 MB in  3.04 seconds =  84.13 MB/sec
    
    $ sync; dd if=/dev/zero of=/mnt/data/tempfile bs=1M count=4096; sync
    4294967296 bytes (4.3 GB, 4.0 GiB) copied, 24.469 s, 176 MB/s
    $ dd if=/mnt/data/tempfile of=/dev/null bs=1M count=4096
    4294967296 bytes (4.3 GB, 4.0 GiB) copied, 43.419 s, 98.9 MB/s
    
    $ sudo hdparm -Tt /dev/sda1
     Timing cached reads:   17862 MB in  1.99 seconds = 8957.35 MB/sec
     Timing buffered disk reads: 296 MB in  3.00 seconds =  98.59 MB/sec

    Network
    Ping 22ms
    [  4]   0.00-10.00  sec  1.19 GBytes  1.02 Gbits/sec    0             sender
    [  4]   0.00-10.00  sec  1.19 GBytes  1.02 Gbits/sec                  receiver

-----

*GCP custom instance $87.68/month, $20 for 500GB HDD or $85 for 500GB
SDD  
8GB 4 vCPUs 10GB SSD in us-west-1a  
Debian 9.3 x64  
*

Let's try this again, but this time with an SSD for the OS. I don't
think it should even be an option to not have an SSD boot disk, and it
definitely shouldn't be the default. It also only made it about $1 more
expensive per month to switch to an SSD for the OS disk. Of course, I do
this, only to have disk performance decrease. No idea what that is
about. Although, hdparm did show faster read times, so perhaps it was
something about the huge 'dd' I was doing that didn't sit well with this
disk type.

I also noticed they automatically run 3 daemons:

<https://toys.lerdorf.com/uploads/gcp_htop.png>  

Compile time did improve though:

    $ make -j 4
    real    3m50.495s
    user    12m29.808s
    sys 0m30.684s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    2
    Core(s) per socket:    2
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 79
    Model name:            Intel(R) Xeon(R) CPU @ 2.20GHz
    Stepping:              0
    CPU MHz:               2200.000
    BogoMIPS:              4400.00
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              256K
    L3 cache:              56320K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           799M   18M  781M   3% /run
    /dev/sda1       9.8G  2.2G  7.1G  24% /
    tmpfs           4.0G     0  4.0G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           4.0G     0  4.0G   0% /sys/fs/cgroup
    /dev/sdb1       492G   73M  467G   1% /mnt/data

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=4096; sync
    4294967296 bytes (4.3 GB, 4.0 GiB) copied, 98.9939 s, 43.4 MB/s
    $ dd if=/root/tempfile of=/dev/null bs=1M count=4096
    4294967296 bytes (4.3 GB, 4.0 GiB) copied, 128.055 s, 33.5 MB/s
    
    $ sudo hdparm -Tt /dev/sda1
     Timing cached reads:   14770 MB in  2.00 seconds = 7402.96 MB/sec
     Timing buffered disk reads: 740 MB in  3.00 seconds = 246.58 MB/sec

    Network
    Ping 22ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  1.18 GBytes  1.01 Gbits/sec    0             sender
    [  4]   0.00-10.00  sec  1.18 GBytes  1.01 Gbits/sec                  receiver

  

-----

-----

  

# Upcloud

Upcloud, like GCP, lets you provision a cheaper class of block volume.
Unfortunately not in the US. Only in London and Helsinki from what I
could tell. So for an instance in Chicago it looks like it would cost me
an extra $110/month to get a 500GB volume attached. The Web UI doesn't
let you know that HDD volumes aren't available in your region until you
are done filling everything out and try to deploy. An earlier hint would
have been appreciated.

Other than that, performance was great across the board for my Upcloud
instance. CPU and disk were fast. Network a bit on the slow side with
high latency because they don't have a west coast POP.

-----

*Upcloud instance $80/month + $110 for a 500GB disk  
8GB 4 core 70GB SSD in Chicago\#1  
Debian 9.3 x64  
*

    $ time make -j 4
    real    2m22.411s
    user    7m14.228s
    sys     0m21.164s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    1
    Core(s) per socket:    1
    Socket(s):             4
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 79
    Model name:            Intel(R) Xeon(R) CPU E5-2687W v4 @ 3.00GHz
    Stepping:              1
    CPU MHz:               2999.996
    BogoMIPS:              5999.99
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              4096K
    L3 cache:              16384K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           799M   11M  789M   2% /run
    /dev/vda1        69G  887M   68G   2% /
    tmpfs           4.0G     0  4.0G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           4.0G     0  4.0G   0% /sys/fs/cgroup
    tmpfs           799M     0  799M   0% /run/user/0

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 17.8492 s, 481 MB/s
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 20.4395 s, 420 MB/s
    $ sudo hdparm -Tt /dev/vda1
     Timing cached reads:   20812 MB in  1.99 seconds = 10435.19 MB/sec
     Timing buffered disk reads: 728 MB in  3.00 seconds = 242.55 MB/sec

    Network
    Ping 49ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec   522 MBytes   438 Mbits/sec    0             sender
    [  4]   0.00-10.00  sec   522 MBytes   438 Mbits/sec                  receiver

  

-----

-----

  

# AWS Lightsail

The main issue I had with AWS Lightsail is that I couldn't get a 4 core
instance. The result is a somewhat sluggish 4m23s PHP compile time. It
was also a minor annoyance that only Debian 8 was available. But
manually uprading to Debian 9.3 takes less than 5 minutes, so I can't
gripe too much about that. Other than that performance was ok for a
small VSP. Disk performance was ok, not great, and network was a bit on
the slow side. Creating and attaching a 500GB volume was simple.

<https://toys.lerdorf.com/uploads/lightsail.png>  

-----

*AWS Lightsail instance $80/month plus $50/month for 500GB Volume  
8GB 2 core 80GB SSD in Oregon-1  
Manually upgrade to Debian 9.3 x64  
*

    $ time make -j 4
    real    4m23.368s
    user    7m51.100s
    sys     0m20.968s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                2
    On-line CPU(s) list:   0-1
    Thread(s) per core:    1
    Core(s) per socket:    2
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 63
    Model name:            Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz
    Stepping:              2
    CPU MHz:               2400.186
    BogoMIPS:              4800.08
    Hypervisor vendor:     Xen
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              256K
    L3 cache:              30720K
    NUMA node0 CPU(s):     0,1

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           799M  8.4M  791M   2% /run
    /dev/xvda2       79G  1.6G   74G   3% /
    tmpfs           3.9G     0  3.9G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
    tmpfs           799M     0  799M   0% /run/user/1000
    /dev/xvdf1      492G   73M  467G   1% /mnt/data

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 34.4451 s, 249 MB/s
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 66.0235 s, 130 MB/s
    
    $ sudo hdparm -Tt /dev/xvda2
     Timing cached reads:   18310 MB in  1.99 seconds = 9182.22 MB/sec
     Timing buffered disk reads: 316 MB in  3.01 seconds = 104.84 MB/sec
    
    $ sync; dd if=/dev/zero of=/mnt/data/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 28.1742 s, 305 MB/s
    $ dd if=/mnt/data/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 66.0153 s, 130 MB/s
    
    $ sudo hdparm -Tt /dev/xvdf1
     Timing cached reads:   17318 MB in  1.99 seconds = 8683.93 MB/sec
     Timing buffered disk reads: 490 MB in  3.01 seconds = 162.94 MB/sec

    Network
    Ping 21ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec   171 MBytes   144 Mbits/sec  280             sender
    [  4]   0.00-10.00  sec   162 MBytes   136 Mbits/sec                  receiver

  

-----

-----

  

# VMHaus

VMHaus has a nice clean Web UI and and simple VPS offerings. I had a
slight problem booting one of my test instances and support got back to
me in under 4 minutes which was impressive. Apparently there was a
problem on a dhcp server. Performance was excellent across the board.
Not the absolute fastest I saw, but definitely the best
price/performance ratio of all the solutions I tested. At $28/month this
is a great deal. There are no bells and whistles, including no option
for attaching any sort of block storage. If you don't host a zillion
baby photos for friends and family you are probably fine with the
generous 100GB provided.

*VMHaus instance $28/month  
8GB 4 core 100GB nvme in Los Angeles  
Debian 9.3 x64  
*

    $ time make -j 4
    real    3m11.901s
    user    9m37.644s
    sys 0m37.420s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    1
    Core(s) per socket:    1
    Socket(s):             4
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 45
    Model name:            Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz
    Stepping:              7
    CPU MHz:               2599.998
    BogoMIPS:              5199.99
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              256K
    L3 cache:              20480K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           799M   11M  789M   2% /run
    /dev/vda1        99G  1.3G   94G   2% /
    tmpfs           4.0G     0  4.0G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           4.0G     0  4.0G   0% /sys/fs/cgroup
    tmpfs           799M     0  799M   0% /run/user/0

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 30.0619 s, 286 MB/s
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 5.88135 s, 1.5 GB/s
    $ sudo hdparm -Tt /dev/vda1
     Timing cached reads:   16404 MB in  1.99 seconds = 8223.77 MB/sec
     Timing buffered disk reads: 1006 MB in  3.00 seconds = 335.16 MB/sec

    Network
    Ping 9ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec   990 MBytes   830 Mbits/sec   54             sender
    [  4]   0.00-10.00  sec   987 MBytes   828 Mbits/sec                  receiver

  

-----

-----

  

# OVH

Since I couldn't find a free trial, I tested the cheaper
8GB/2core/40GBSDD version for $14.55 to stay under my $20 testing
budget. Their only North American POP is near Montreal, so ping times to
the west coast aren't great.

The actual provisioning was pretty painful. Their signup page asks for a
birthday, and no matter what you enter it determines you are under 18
and you can't continue. So you have to leave that field blank. And
sometimes things don't load. Overall the Web UI is terrible and getting
to the point where you have a working VPS was terrible compared to the
other providers I tested. After payment it took about 10 minutes for the
VPS to be ready, but when it came up it was a fully updated Debian 9.3
install, which was nice.

For a 2-core VPS, the 4m18s PHP build time was good. It should be close
to twice as fast on a 4 core, so in line with the fastest competitors.
Disk IO wasn't blazingly fast, but not bad, and I saw a steady 100
Mbit/sec connection to the west coast.

-----

*$35/month, additional 500GB of disk $42/month  
8GB, 4 cores, 100GB SSD in Beauharnois, Canada  
Debian 9.3  
*

    $ make -j 2
    real    4m18.397s
    user    7m40.360s
    sys 0m21.828s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                2
    On-line CPU(s) list:   0,1
    Thread(s) per core:    1
    Core(s) per socket:    1
    Socket(s):             2
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 60
    Model name:            Intel Core Processor (Haswell, no TSX)
    Stepping:              1
    CPU MHz:               2394.452
    BogoMIPS:              4788.90
    Virtualization:        VT-x
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              4096K
    NUMA node0 CPU(s):     0,1

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.8G     0  3.8G   0% /dev
    tmpfs           780M  8.4M  772M   2% /run
    /dev/sda1        40G  1.5G   37G   4% /
    tmpfs           3.9G     0  3.9G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 15.6188 s, 550 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 32.7707 s, 262 MB/s
    $ sudo hdparm -Tt /dev/sda1
     Timing cached reads:   17580 MB in  1.99 seconds = 8813.83 MB/sec
     Timing buffered disk reads: 774 MB in  3.00 seconds = 257.59 MB/sec

    Network
    Ping 67ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec   118 MBytes  98.7 Mbits/sec   79             sender
    [  4]   0.00-10.00  sec   115 MBytes  96.5 Mbits/sec                  receiver

  

-----

-----

  

# Azure VPS

Oh Azure\! This one is going to get a bit ranty. I Spent a good 20
minutes clicking around the provisioning Web UI. To be fair, it is more
geared to people needing to provision a lot of servers. Doing a single
one like this is not the target audience as far as I can tell. But
still, instead of presenting a couple of standard options and a way to
build your own custom config, Azure gives you 92 options (depending on
which region you select):

    A0: 1 Cores(s), 0.75 GB RAM, 20 GB Temporary storage, $0.020/hour
    A1: 1 Cores(s), 1.75 GB RAM, 70 GB Temporary storage, $0.060/hour
    A2: 2 Cores(s), 3.5 GB RAM, 135 GB Temporary storage, $0.120/hour
    A3: 4 Cores(s), 7 GB RAM, 285 GB Temporary storage, $0.240/hour
    A4: 8 Cores(s), 14 GB RAM, 605 GB Temporary storage, $0.480/hour
    A5: 2 Cores(s), 14 GB RAM, 135 GB Temporary storage, $0.250/hour
    A6: 4 Cores(s), 28 GB RAM, 285 GB Temporary storage, $0.500/hour
    A7: 8 Cores(s), 56 GB RAM, 605 GB Temporary storage, $1.000/hour
    A8: 8 Cores(s), 56 GB RAM, 382 GB Temporary storage, $0.975/hour
    A9: 16 Cores(s), 112 GB RAM, 382 GB Temporary storage, $1.950/hour
    A10: 8 Cores(s), 56 GB RAM, 382 GB Temporary storage, $0.780/hour
    A11: 16 Cores(s), 112 GB RAM, 382 GB Temporary storage, $1.560/hour
    B1S: 1 Cores(s), 1 GB RAM, 2 GB Temporary storage, $0.014/hour
    B2S: 2 Cores(s), 4 GB RAM, 8 GB Temporary storage, $0.055/hour
    B1MS: 1 Cores(s), 2 GB RAM, 4 GB Temporary storage, $0.028/hour
    B2MS: 2 Cores(s), 8 GB RAM, 16 GB Temporary storage, $0.110/hour
    B4MS: 4 Cores(s), 16 GB RAM, 32 GB Temporary storage, $0.220/hour
    B8MS: 8 Cores(s), 32 GB RAM, 64 GB Temporary storage, $0.439/hour
    A1 v2: 1 Cores(s), 2 GB RAM, 10 GB Temporary storage, $0.043/hour
    A2 v2: 2 Cores(s), 4 GB RAM, 20 GB Temporary storage, $0.091/hour
    A4 v2: 4 Cores(s), 8 GB RAM, 40 GB Temporary storage, $0.191/hour
    A8 v2: 8 Cores(s), 16 GB RAM, 80 GB Temporary storage, $0.400/hour
    A2m v2: 2 Cores(s), 16 GB RAM, 20 GB Temporary storage, $0.149/hour
    A4m v2: 4 Cores(s), 32 GB RAM, 40 GB Temporary storage, $0.297/hour
    A8m v2: 8 Cores(s), 64 GB RAM, 80 GB Temporary storage, $0.594/hour
    D1: 1 Cores(s), 3.5 GB RAM, 50 GB Temporary storage, $0.077/hour
    D2: 2 Cores(s), 7 GB RAM, 100 GB Temporary storage, $0.154/hour
    D3: 4 Cores(s), 14 GB RAM, 200 GB Temporary storage, $0.308/hour
    D4: 8 Cores(s), 28 GB RAM, 400 GB Temporary storage, $0.616/hour
    D11: 2 Cores(s), 14 GB RAM, 100 GB Temporary storage, $0.195/hour
    D12: 4 Cores(s), 28 GB RAM, 200 GB Temporary storage, $0.390/hour
    D13: 8 Cores(s), 56 GB RAM, 400 GB Temporary storage, $0.780/hour
    D14: 16 Cores(s), 112 GB RAM, 800 GB Temporary storage, $1.542/hour
    D1 v2: 1 Cores(s), 3.5 GB RAM, 50 GB Temporary storage, $0.070/hour
    D2 v2: 2 Cores(s), 7 GB RAM, 100 GB Temporary storage, $0.140/hour
    D3 v2: 4 Cores(s), 14 GB RAM, 200 GB Temporary storage, $0.279/hour
    D4 v2: 8 Cores(s), 28 GB RAM, 400 GB Temporary storage, $0.559/hour
    D5 v2: 16 Cores(s), 56 GB RAM, 800 GB Temporary storage, $1.117/hour
    D11 v2: 2 Cores(s), 14 GB RAM, 100 GB Temporary storage, $0.185/hour
    D12 v2: 4 Cores(s), 28 GB RAM, 200 GB Temporary storage, $0.371/hour
    D13 v2: 8 Cores(s), 56 GB RAM, 400 GB Temporary storage, $0.741/hour   
    DS14 v2: 16 Cores(s), 112 GB RAM, 224 GB Temporary storage, $1.482/hour
    DS15 v2: 20 Cores(s), 140 GB RAM, 280 GB Temporary storage, $1.853/hour
    D2 v3: 2 vCPU(s), 8 GB RAM, 50 GB Temporary storage, $0.117/hour
    D4 v3: 4 vCPU(s), 16 GB RAM, 100 GB Temporary storage, $0.234/hour
    D8 v3: 8 vCPU(s), 32 GB RAM, 200 GB Temporary storage, $0.468/hour
    D16 v3: 16 vCPU(s), 64 GB RAM, 400 GB Temporary storage, $0.936/hour
    D32 v3: 32 vCPU(s), 128 GB RAM, 800 GB Temporary storage, $1.872/hour
    D64 v3: 64 vCPU(s), 256 GB RAM, 1600 GB Temporary storage, $3.744/hour
    D2 v2 Promo: 2 Cores(s), 7 GB RAM, 100 GB Temporary storage, $0.117/hour
    D3 v2 Promo: 4 Cores(s), 14 GB RAM, 200 GB Temporary storage, $0.234/hour
    D4 v2 Promo: 8 Cores(s), 28 GB RAM, 400 GB Temporary storage, $0.468/hour
    D5 v2 Promo: 16 Cores(s), 56 GB RAM, 800 GB Temporary storage, $0.936/hour
    D11 v2 Promo: 2 Cores(s), 14 GB RAM, 100 GB Temporary storage, $0.148/hour
    D12 v2 Promo: 4 Cores(s), 28 GB RAM, 200 GB Temporary storage, $0.296/hour
    D13 v2 Promo: 8 Cores(s), 56 GB RAM, 400 GB Temporary storage, $0.593/hour
    D14 v2 Promo: 16 Cores(s), 112 GB RAM, 800 GB Temporary storage, $1.186/hour
    E2 v3: 2 vCPU(s), 16 GB RAM, 32 GB Temporary storage, $0.148/hour
    E4 v3: 4 vCPU(s), 32 GB RAM, 64 GB Temporary storage, $0.296/hour
    E8 v3: 8 vCPU(s), 64 GB RAM, 128 GB Temporary storage, $0.593/hour
    E16 v3: 16 vCPU(s), 128 GB RAM, 256 GB Temporary storage, $1.186/hour
    E32 v3: 32 vCPU(s), 256 GB RAM, 512 GB Temporary storage, $2.371/hour
    E64 v3: 64 vCPU(s), 432 GB RAM, 864 GB Temporary storage, $4.469/hour
    F1: 1 Cores(s), 2 GB RAM, 16 GB Temporary storage, $0.062/hour
    F2: 2 Cores(s), 4 GB RAM, 32 GB Temporary storage, $0.124/hour
    F4: 4 Cores(s), 8 GB RAM, 64 GB Temporary storage, $0.249/hour
    F8: 8 Cores(s), 16 GB RAM, 128 GB Temporary storage, $0.498/hour
    F16: 16 Cores(s), 32 GB RAM, 256 GB Temporary storage, $0.997/hour
    G1: 2 Cores(s), 28 GB RAM, 384 GB Temporary storage, $0.610/hour
    G2: 4 Cores(s), 56 GB RAM, 768 GB Temporary storage, $1.220/hour
    G3: 8 Cores(s), 112 GB RAM, 1536 GB Temporary storage, $2.440/hour
    G4: 16 Cores(s), 224 GB RAM, 3072 GB Temporary storage, $4.880/hour
    G5: 32 Cores(s), 448 GB RAM, 6144 GB Temporary storage, $8.690/hour
    H8: 8 Cores(s), 56 GB RAM, 1000 GB Temporary storage, $0.971/hour
    H16: 16 Cores(s), 112 GB RAM, 2000 GB Temporary storage, $1.941/hour
    H8m: 8 Cores(s), 112 GB RAM, 1000 GB Temporary storage, $1.301/hour
    H16m: 16 Cores(s), 224 GB RAM, 2000 GB Temporary storage, $2.601/hour
    H16mr: 16 Cores(s), 224 GB RAM, 2000 GB Temporary storage, $2.861/hour
    H16r: 16 Cores(s), 112 GB RAM, 2000 GB Temporary storage, $2.136/hour
    L4: 4 Cores(s), 32 GB RAM, 678 GB Temporary storage, $0.344/hour
    L8: 8 Cores(s), 64 GB RAM, 1388 GB Temporary storage, $0.688/hour  
    L16: 16 Cores(s), 128 GB RAM, 2807 GB Temporary storage, $1.376/hour
    L32: 32 Cores(s), 256 GB RAM, 5630 GB Temporary storage, $2.752/hour

I literally had to inspect the HTML select box element and copy the
contents out to a text file to sort it to find my options for my target
4 core, 8 GB VPS. There were only two:

    A4 v2: 4 Cores(s), 8 GB RAM, 40 GB Temporary storage, $0.191/hour
    F4: 4 Cores(s), 8 GB RAM, 64 GB Temporary storage, $0.249/hour

I also got super lost trying to figure out what it would cost to bring
the VPS up to 500 GB of persistent storage. And then to make things even
more confusing, when I started the virtual machine creation process and
came to the "Choose your virtual machine size" step, I got a bunch of
different options not included in the above list with most of them
listed as "Not Available" including both the A4 v2 and F4 options I had
so carefully located.

![Azure VPS options](/assets/images/testing-vps-solutions/azure.png) ![Azure
VPS Options](/assets/images/testing-vps-solutions/azure2.png)  

But after clicking around a bit and fiddling with the filter sliders, I
got this view.

![Azure VPS options](/assets/images/testing-vps-solutions/azure3.png)

which is where I got that $133/month number from.

Once provisioned, which took longer than most, this was the Debian 9.3
VPS came up completely current. As in \`apt-get update && apt-get
upgrade\` had nothing to do. That might explain why it took slightly
longer to provision.

The Azure VPS also had the heaviest provisioning agent of all the ones I
tested. It looks like it is doing a heartbeat once per second and doing
a (non-ssl) GET request to an IIS server upstream asking it for a
"GoalState". I listened and checked what the IIS server responded with.
The response from the management server is in the GoalState addendum
below. It is mostly self-explanatory, I think.

As far as I can tell I have no IPv6 connectivity on this VPS. The Azure
documentation talks about IPv6 so I am not sure what I missed during
provisioning. I went back over the steps to see if I ever had an IPv6
option and I didn't see it and the management dashboard has no mention
of IPv6 anywhere. This was the only VPS I tested that didn't come with
easy IPv6 support.

![Azure Dashboard](/assets/images/testing-vps-solutions/azure_dash1.png)
![Azure Dashboard](/assets/images/testing-vps-solutions/azure_dash2.png)  

-----

*$133/month  
8GB 4 vCPU 60GB SSD West US 2  
Debian 9.3  
*

    $ time make -j 4
    real    3m34.782s
    user    11m9.464s
    sys 0m30.872s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    2
    Core(s) per socket:    2
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 85
    Model name:            Intel(R) Xeon(R) Platinum 8168 CPU @ 2.70GHz
    Stepping:              4
    CPU MHz:               2693.682
    BogoMIPS:              5387.36
    Virtualization:        VT-x
    Hypervisor vendor:     Microsoft
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              1024K
    L3 cache:              33792K
    NUMA node0 CPU(s):     0-3

The disk layout got a bit interesting:

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            3.9G     0  3.9G   0% /dev
    tmpfs           797M  8.9M  789M   2% /run
    /dev/sda1        30G  1.1G   27G   4% /
    tmpfs           3.9G     0  3.9G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
    /dev/sdb1        32G   49M   30G   1% /mnt/resource
    tmpfs           797M     0  797M   0% /run/user/1000

And they have this disclaimer in
*/mnt/resource/DATALOSS\_WARNING\_README.txt*:

``` 
  WARNING: THIS IS A TEMPORARY DISK.

  Any data stored on this drive is SUBJECT TO LOSS and THERE IS NO WAY TO RECOVER IT.

  Please do not use this disk for storing any personal or application data.

  For additional details to please refer to the MSDN documentation at :
  http://msdn.microsoft.com/en-us/library/windowsazure/jj672979.aspx
```

    $ sync; dd if=/dev/zero of=/home/rasmus/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 372.933 s, 23.0 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    vm.drop_caches = 3
    $ dd if=/home/rasmus/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 128.466 s, 66.9 MB/s
    
    $ sudo hdparm -Tt /dev/sda1
    /dev/sda1:
     Timing cached reads:   15618 MB in  1.99 seconds = 7849.35 MB/sec
     Timing buffered disk reads:  50 MB in  3.07 seconds =  16.26 MB/sec
    
    $ sync; dd if=/dev/zero of=/mnt/resource/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 105.762 s, 81.2 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    vm.drop_caches = 3
    $ dd if=/mnt/resource/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 125.382 s, 68.5 MB/s
    
    $ sudo hdparm -Tt /dev/sdb1
    /dev/sdb1:
     Timing cached reads:   13242 MB in  1.99 seconds = 6649.23 MB/sec
     Timing buffered disk reads: 108 MB in  3.01 seconds =  35.89 MB/sec

    Network
    Ping 23ms
    Bandwidth:
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  1.43 GBytes  1.23 Gbits/sec    0             sender
    [  4]   0.00-10.00  sec  1.43 GBytes  1.23 Gbits/sec                  receiver

## GoalState Addendum

    <GoalState xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="goalstate10.xsd">                                      
      <Version>2012-11-30</Version>
      <Incarnation>1</Incarnation>
      <Machine>
        <ExpectedState>Started</ExpectedState>
        <StopRolesDeadlineHint>300000</StopRolesDeadlineHint>
        <LBProbePorts>
          <Port>16001</Port>
        </LBProbePorts>
        <ExpectHealthReport>FALSE</ExpectHealthReport>
      </Machine>
      <Container>
        <ContainerId>c17254d8-f744-4181-b569-8e19a15a49f9</ContainerId>
        <RoleInstanceList>
          <RoleInstance>
            <InstanceId>1e05ec1a-a437-4b96-ae4e-e97deaf7a219._test</InstanceId>
            <State>Started</State>
            <Configuration>
              <HostingEnvironmentConfig>http://168.63.129.16:80/machine/c17254d8-f744-4181-b569-8e19a15a49f9/1e05ec1a%2Da437%2D4b96%2Dae4e%2De97deaf7a219.%5Ftest?comp=config&amp;type=hostingEnvironmentConfig&amp;incarnation=1</HostingEnvironmentConfig>
              <SharedConfig>http://168.63.129.16:80/machine/c17254d8-f744-4181-b569-8e19a15a49f9/1e05ec1a%2Da437%2D4b96%2Dae4e%2De97deaf7a219.%5Ftest?comp=config&amp;type=sharedConfig&amp;incarnation=1</SharedConfig>
              <ExtensionsConfig>http://168.63.129.16:80/machine/c17254d8-f744-4181-b569-8e19a15a49f9/1e05ec1a%2Da437%2D4b96%2Dae4e%2De97deaf7a219.%5Ftest?comp=config&amp;type=extensionsConfig&amp;incarnation=1</ExtensionsConfig>
              <FullConfig>http://168.63.129.16:80/machine/c17254d8-f744-4181-b569-8e19a15a49f9/1e05ec1a%2Da437%2D4b96%2Dae4e%2De97deaf7a219.%5Ftest?comp=config&amp;type=fullConfig&amp;incarnation=1</FullConfig>
              <Certificates>http://168.63.129.16:80/machine/c17254d8-f744-4181-b569-8e19a15a49f9/1e05ec1a%2Da437%2D4b96%2Dae4e%2De97deaf7a219.%5Ftest?comp=certificates&amp;incarnation=1</Certificates>
              <ConfigName>1e05ec1a-a437-4b96-ae4e-e97deaf7a219.0.1e05ec1a-a437-4b96-ae4e-e97deaf7a219.0._test.1.xml</ConfigName>
            </Configuration>
          </RoleInstance>
        </RoleInstanceList>
      </Container>
    </GoalState>

  

-----

-----

  

# Contabo

Contabo is interesting. Extremely low-cost and the VPS didn't provision
right away. You can pay extra to have it provision "within a few hours".
It actually only took 20 minutes to get my VPS without paying the extra.
Their dedicated servers look interesting as well, but above my $20
testing budget. Ugliest web site you could imagine, but who cares. They
were also the only provider offering VNC access to the console. I tried
it. It works well. A bit slow, but VNC from California to Germany is not
going to be fast, and it is only needed when you mess up your boot image
somehow. But, wait, there is more... Your $11 also buys you a /64 IPv6
subnet :)

When I first created my VPS I was getting about 40% packet loss. But it
looks like I got unlucky. I contacted support and they got back to me in
about an hour apologizing for some technical issues that are all cleared
up now.

Contiuing the trend, the slightly longer provisioning resulted in a
fully up to date Debian 9.3 install.

Performance is ok across the board. Obviously ping times to California
aren't going to be great.

One interesting tidbit I noticed was that my VPS kept getting DNS
queries from an IP in India for "www.bollywoodbackless.com". I don't
even want to look up what that might be. Perhaps I got a recycled IP and
this used to be a DNS server.

![Wireshark
screenshot](/assets/images/testing-vps-solutions/contabo_tshark1.png)  

-----

*$11/month  
12GB, 4 cores, 300 GB SSD  
Debian 9.3  
*

    $ make -j 4
    real    3m47.022s
    user    11m39.176s
    sys 0m42.684s

    $ lscpu
    Architecture:          x86_64
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian
    CPU(s):                4
    On-line CPU(s) list:   0-3
    Thread(s) per core:    1
    Core(s) per socket:    4
    Socket(s):             1
    NUMA node(s):          1
    Vendor ID:             GenuineIntel
    CPU family:            6
    Model:                 85
    Model name:            Intel(R) Xeon(R) Silver 4114 CPU @ 2.20GHz
    Stepping:              4
    CPU MHz:               2200.000
    BogoMIPS:              4400.00
    Hypervisor vendor:     KVM
    Virtualization type:   full
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              4096K
    L3 cache:              16384K
    NUMA node0 CPU(s):     0-3

    $ df -kh
    Filesystem      Size  Used Avail Use% Mounted on
    udev            5.9G     0  5.9G   0% /dev
    tmpfs           1.2G   11M  1.2G   1% /run
    /dev/sda2       294G  2.1G  277G   1% /
    tmpfs           5.9G     0  5.9G   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           5.9G     0  5.9G   0% /sys/fs/cgroup
    /dev/sda1       922M   61M  799M   7% /boot
    tmpfs           1.2G     0  1.2G   0% /run/user/0

    $ sync; dd if=/dev/zero of=/root/tempfile bs=1M count=8192; sync
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 59.6787 s, 144 MB/s
    $ sudo /sbin/sysctl -w vm.drop_caches=3
    $ dd if=/root/tempfile of=/dev/null bs=1M count=8192
    8589934592 bytes (8.6 GB, 8.0 GiB) copied, 11.3284 s, 758 MB/s
    $ sudo hdparm -Tt /dev/sda2
     Timing cached reads:   10770 MB in  2.00 seconds = 5395.17 MB/sec
     Timing buffered disk reads: 926 MB in  3.00 seconds = 308.41 MB/sec

    Network
    Ping 152ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  54.2 MBytes  45.4 Mbits/sec  1411             sender
    [  4]   0.00-10.00  sec  50.9 MBytes  42.7 Mbits/sec                  receiver

Since this server is in Germany, testing network against
speedtest.serverius.net in the Netherlands as well:

    Ping 21ms
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec   105 MBytes  88.2 Mbits/sec  2418             sender
    [  4]   0.00-10.00  sec   101 MBytes  85.1 Mbits/sec                  receiver
