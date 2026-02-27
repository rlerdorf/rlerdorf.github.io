---
date: '2011-05-21'
layout: post
title: ASRock Sandy Bridge Motherboard notes
---


I have pieced together two Sandy Bridge machines. This entry contains my
notes on the two machines. Mostly for myself to refer back to later, but
it might come in handy for others along the way.  

### Machine 1 - Overkill HTPC

  - Mythbuntu 10.10 initially but upgraded to full 11.04 when it was
    released
  - i5-2500k CPU
  - ASRock H67M LGA 1155 Intel H67 HDMI SATA 6Gb/s USB 3.0 Micro ATX
    Intel Motherboard
  - Seasonic PSU
  - G.SKILL Ripjaws X Series 8GB (2 x 4GB) 240-Pin DDR3 SDRAM DDR3 1333
    (PC3 10666) Model F3-10666CL9D-8GBXL
  - Crucial RealSSD C300 CTFDDAC064MAG-1G1 2.5" 64GB SATA III MLC SSD
  - Western Digital Caviar Green WD20EARS 2TB SATA 3.0Gb/s 3.5" HD
  - ASUS ENGT430/DI/1GD3(LP) GeForce GT 430 (Fermi) 1GB 128-bit DDR3 PCI
    Express 2.0 x16 HDCP Graphics card
  - AVS Gear GP-IR01BK Windows Vista Infrared MCE Black Remote Control
  - SilverStone Aluminum/Steel Micro ATX HTPC Computer Case GD05B
    (Black)
  - SiliconDust HDHomeRun HDHR-US Dual Tuner
  - RCA ANT751 Outdoor Antenna (installed in attic - see
    <http://flic.kr/p/9iFKer>)

  

### Machine 2 - Dev Box for the office

  - Ubuntu 11.04
  - i7-2600k CPU
  - ASRock Z68 Extreme4 LGA 1155 Intel Z68 HDMI SATA 6Gb/s USB 3.0 ATX
    Intel Motherboard
  - G.SKILL Ripjaws X Series 8GB (2 x 4GB) 240-Pin DDR3 SDRAM DDR3 1333
    (PC3 10666) Model F3-10666CL9D-8GBXL
  - G.SKILL Ripjaws Series 8GB (2 x 4GB) 240-Pin DDR3 SDRAM DDR3 1600
    (PC3 12800) Model F3-12800CL9D-8GBRL
  - Crucial M4 CT128M4SSD2 2.5" 128GB SATA III MLC Internal Solid State
    Drive (SSD)
  - 2 x SAMSUNG Spinpoint F4 HD204UI 2TB 5400 RPM SATA 3.0Gb/s 3.5" HD
  - CORSAIR Builder Series CX430 CMPSU-430CX 430W ATX12V Active PFC PSU
  - Old Antec case I had lying around

I went scouring slickdeals and other deal sites for most of these
components, so there are some mismatches. Like the slightly mismatched
ram in the second machine, and the fact that I am using a 2500k in an
H67 (B2\!) board. No real point in an unlocked cpu in a locked board,
but the k was cheaper than the non-k at the time, and who knows, I could
swap the motherboard. And yes, it is a B2-stepping board, so the SATA2
ports are iffy. But since I am not using them it doesn't bother me.

## Linux Installation

Using 11.04 and following the instructions for building a bootable USB
stick from <http://www.ubuntu.com/download/ubuntu/download> installation
was trivial.

### Installation notes

  - On the Extreme4 make sure your boot drive is plugged into one of the
    first two SATA-3 ports. The second 2 ports are handled by a
    different controller (Marvell) and aren't bootable by default. This
    can be toggled in the BIOS, but it is simpler to just use the right
    port.
  - Don't use the "raid" mode for your drives in the BIOS. This is also
    known as FakeRAID and it is annoying. Just set it to AHCI.

  

## Linux Configuration - RAID0

On the i7 box I am using the M4 SSD as my boot and OS drive and have
/var on the two 2TB drives in Raid0. It wasn't super obvious how to tell
the Ubuntu installer how to do that during the install, so I just did it
afterwards
    with:

    mdadm --create /dev/md0 --chunk=128 --level=0 --raid-devices=2 /dev/sdb1 /dev/sdc1

![](/assets/images/asrock-sandy-bridge-motherboard-notes/DiskUtility_scaled.png) I tend to
be rather anti-GUI for OS admin stuff, but I have to confess I used
"Disk Utility" for everything other than creating the actual */dev/md0*.

  

## Linux Configuration - SSD

Modern Linux kernels have TRIM support, but you still need to tell it to
actually trim. In my */etc/fstab* I
    have:

    UUID=65da1033-347d-4b9a-a660-312cb2f33ac0 /               ext4    discard,noatime,errors=remount-ro 0       1
    UUID=6f473b40-3ec0-419d-81b3-f77e7cc6dc08 /var            ext4    noatime,errors=remount-ro,user_xattr 0       2

And no, I don't actually have a swap partition. Swap seems like an
outdated concept to me. I have 16G of ram in this box. If it runs out
I'd rather have it die dramatically than start swapping on me. It's a
dev box, not a production server.

## Linux Configuration - lm-sensors

This gets a bit more interesting. After running *sensors-detect*
coretemp works out of the box on Ubuntu 11.04 which gets me:

    coretemp-isa-0000
    Adapter: ISA adapter
    Core 0:      +37.0°C  (high = +80.0°C, crit = +98.0°C)  
    
    coretemp-isa-0001
    Adapter: ISA adapter
    Core 1:      +37.0°C  (high = +80.0°C, crit = +98.0°C)  
    
    coretemp-isa-0002
    Adapter: ISA adapter
    Core 2:      +34.0°C  (high = +80.0°C, crit = +98.0°C)  
    
    coretemp-isa-0003
    Adapter: ISA adapter
    Core 3:      +39.0°C  (high = +80.0°C, crit = +98.0°C) 

but nothing else. I want fan speeds, voltages and the other temperature
sensors, but the kernel driver doesn't detect the nct6776 chip
correctly. However, there is a fixed version here:
<http://roeck-us.net/linux/drivers/w83627ehf/>. Grab that, make install
and modprobe it and I now get:

    nct6776-isa-0290
    Adapter: ISA adapter
    Vcore:        +0.96 V  (min =  +0.94 V, max =  +1.32 V)   
    in1:          +1.85 V  (min =  +0.90 V, max =  +1.15 V)   ALARM
    AVCC:         +3.26 V  (min =  +2.98 V, max =  +3.63 V)   
    +3.3V:        +3.26 V  (min =  +2.98 V, max =  +3.63 V)   
    in4:          +1.01 V  (min =  +0.90 V, max =  +1.10 V)   
    in5:          +1.70 V  (min =  +0.00 V, max =  +0.00 V)   ALARM
    3VSB:         +3.42 V  (min =  +2.98 V, max =  +3.63 V)   
    Vbat:         +3.28 V  (min =  +2.70 V, max =  +3.30 V)   
    fan1:        1724 RPM  (min =  200 RPM)
    fan2:        1945 RPM  (min =  400 RPM)
    SYSTIN:       +31.0°C  (high = +38.0°C, hyst = +35.0°C)  sensor = thermistor
    CPUTIN:       +30.0°C  (high = +80.0°C, hyst = +75.0°C)  sensor = thermistor
    AUXTIN:       +22.0°C  (high = +80.0°C, hyst = +75.0°C)  sensor = thermistor
    PECI Agent 0: +33.0°C                                    
    cpu0_vid:    +2.050 V

In addition to the previous coretemp sensors output. Of course, the
default limits and labels aren't going to match, and it is only showing
2 of the fans which looks to be a chassis fan (fan1) and the cpu fan
(fan2). Alternatively you can install the newer 2.6.39 kernel which has
this updated nct6776 support in it:

    sudo add-apt-repository ppa:kernel-ppa/ppa 
    sudo apt-get update
    sudo apt-get dist-upgrade

But you still don't get access to all 5 fans. A really hackish way to
get all 5 fans is to do:

    sudo rmmod w83627ehf
    sudo isaset -y 0x2e 0x2f 0x1c 0x3
    sudo isaset -y 0x2e 0x2f 0x24 0x1c
    modprobe w83627ehf

but it is a bit dangerous poking around in your sensor chip like that. I
am hoping a future ASRock bios will address this since it seems to be a
mistake in how they are setting up the nct6776 chip. I also need to
write an */etc/sensors.d/nct6776* file to set the labels and limits
properly once I figure out what these are. I will update this when I do
so.

## USB 3.0

Here come the really bad
    news.

    [ 9393.599237] xhci_hcd 0000:04:00.0: Unknown completion code 192 for reset device command.
    [ 9393.599243] usb 5-2: Cannot reset HCD device state
    [ 9393.599246] hub 5-0:1.0: Cannot enable port 2.  Maybe the USB cable is bad?
    [ 9393.599278] hub 5-0:1.0: unable to enumerate USB device on port 2

Both the H67M and the Extreme4 Z68 ASRock motherboards use the Etron
EJ168A USB 3.0 host controller. There is no support for this controller
in the Linux kernel as of kernel version 2.6.39. I assume it will
eventually be supported, but this was a definite lack of research on my
part. Currently for the best Linux support you want a board with the NEC
USB 3.0 controller on it. ASRock and Gigabyte boards all use the Etron.
Asus has switched to ASM for their latest z68 boards. The only z68 board
I can find with a NEC controller is the MSI.

## Overall

Despite the USB 3.0 disappointment, the machines are incredibly fast.
Especially the i7 with the z68 board. It runs Handbrake like you
wouldn't believe, although even without overclocking it (yet) the CPU
can easily hit 50°C when I am hitting it hard, so something other than
the stock Intel CPU cooler might be needed. And the M4 SSD makes
rebooting something I don't even think about anymore.

I also like the new Unity interface in Ubuntu 11.04 along with the
top-bar indicators.
