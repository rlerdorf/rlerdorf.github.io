---
date: '2020-05-26'
layout: post
title: Remote Family Tech Support
---


![](/assets/images/remote-family-tech-support/letsencrypt-logo-horizontal.svg)
_Again, mostly a note to myself explaining how and why I set this up the way I did._

## The Project
Like just about every geek, I am tasked with helping family and friends with their computers. It is
easier to do this in person sitting in front of their computers, of course, but that's not possible
right now. So I needed a rock solid way to manage remote Windows and  Mac machines.

Why not just use something like Teamviewer or RemotePC? You definitely can. Both work well. In fact I use
them to bootstrap my setup. It is easier to ask someone to install the RemotePC or TeamViewer client than
it is to talk them through this setup. But, at least for me, TeamViewer on Linux is rather shaky, doesn't
support Wayland and the free version has a very short connection timeout. RemotePC doesn't have a Linux
version of the client software at all. 

My preference is simple shell access with the option to connect via VNC and also be able to SOCKS-proxy
onto the remote network to access anything there.

In the following instructions I am assuming you already have access to the remote machine through something
like the trial version of RemotePC or TeamViewer.

Note that on Windows with WSL 2 coming out shortly, you could just do this more easily from the Linux layer. I am
trying to keep this as lightweight as possible though, so I'll be doing it in native Windows without resorting to WSL.

## Making Windows Usable

On Windows, to make your life less painful install the latest PowerShell (currently PowerShell 7) from [PowerShell Releases](https://github.com/PowerShell/PowerShell/releases)
then run it as Administrator and install *choco*. See https://chocolatey.org/install

Then:

```
choco install -y vim grep unzip unrar 7zip ntop.portable
```

If you are stuck working locally on a Windows box, there is also a much better terminal available now. See https://github.com/microsoft/terminal.
In my case I will be ssh'ing into Windows from Linux which already has a decent terminal so that doesn't really apply here.


## NAT Gateway tunnel
I need some way to connect directly to the remote machine even though it has a non-routable IP behind a NAT gateway.
I could port-forward on the remote router, but this isn't always possible. Also, if the machine you are tasked with
managing is a laptop, that laptop can move around and might not always be behind the same router. Having the tunnel be
independent of the connection is more robust.

In order to create a tunnel I need somewhere with a routable IP to tunnel to. I use a $5/month [Upcloud VPS](https://upcloud.com/signup/?promo=7QZ77S)
(If you use that link to sign up we both get $50 credit)

A really interesting alternative is Cloudflare's Access product, and specifically the [Argo Tunnel](https://developers.cloudflare.com/access/setting-up-access/argo-tunnel/) part.
There is a section at the end of this post showing how to use that instead of a VPS.  Chances are Cloudflare's access points and overall network speed is going to be faster than
what you get with a low-cost VPS. In my case, I get about 300 Mbps in both directions from Upcloud which is faster than any of the connections of the remote machines I manage,
so the VPS isn't the limiting factor in my case plus I use the VPS for many other things. I still find Cloudflare Access super-interesting because of the other things it offers
like an option to use short-lived certs and the other authentication mechanisms it lets you easily put in front of any behind-NAT services you might want to expose to a limited
set of people.

### SSH Keys

On both Mac and Windows make a key for your tunnel with:

```
ssh-keygen -t ecdsa -b 521
```

Copy the contents of *~/.ssh/id_ecdsa.pub* to *~/.ssh/authorized_keys* on your cloud server and paste your own
public key into the Mac or Windows ~/.ssh/authorized_keys file.


### Creating the tunnel

Now I need the outbound tunnel. Check that you can ssh to your cloud server:

```
ssh -i C:\Users\rasmus\.ssh\id_ecdsa -p 5123 sshtunnel@tunnel.casa
```

I normally don't run my sshd on port 22 on any public server. In this example I have it on port 5123.
If that didn't work, try adding *-v -v -v* to the ssh command and see if you can spot why it didn't work.

Assuming you were able to ssh, you now need a persistent ssh tunnel. The full command is:

```
ssh -o ExitOnForwardFailure=yes -o ConnectTimeout=3 -o TCPKeepAlive=yes -o ServerAliveInterval=5 -o ServerAliveCountMax=5 -N -i C:\Users\rasmus\.ssh\id_ecdsa -p 5123 -R 2242:localhost:22 sshtunnel@tunnel.casa
```

You can run that from Powershell and it should establish a tunnel. However, I need to make it into a standalone service that starts up when Windows
starts and re-establishes the tunnel if it goes down. On Linux this is simple by wrapping autossh in a systemd service. On Mac and Windows it is a
bit more complicated.

### Mac Tunnel Service

First make a simple shell script and put it in */usr/local/bin*

```shell
#!/bin/bash
/bin/bash -c "while [ 1 ]; do ssh -o ExitOnForwardFailure=yes -o ConnectTimeout=2 -o TCPKeepAlive=yes -o ServerAliveInterval=2 -o ServerAliveCountMax=2 -N -i /home/rasmus/.ssh/id_ecdsa -p 5123 -R 2242:localhost:22 sshtunnel@tunnel.casa; sleep 1; done" >/dev/null 2>&1
```

Next create */Library/LaunchDaemons/com.apple.sshtunnel.plist*:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>  
    <key>Label</key>
    <string>com.apple.sshtunnel</string>
    <key>ProgramArguments</key>
    <array> 
        <string>/usr/local/bin/sshtunnel</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardErrorPath</key>
    <string>/dev/null</string>
    <key>StandardOutPath</key>
    <string>/dev/null</string>
</dict>
</plist>
```

Then start it with:
```
sudo launchctl load -w /Library/LaunchDaemons/com.apple.sshtunnel.plist
```

<a id="wintunnel"></a>
### Windows Tunnel Service

On Windows I needed to get a bit creative. I could write some C# code, but an easier way is to 
install WinSW from https://github.com/winsw/winsw/releases/

Rename the .exe to *ssh-tunnel.exe* and create *ssh-tunnel.xml*:

```xml
<service>
  <id>ssh-tunnel</id>
  <name>SSH Tunnel</name>
  <description>Establish an outbound ssh tunnel usually from behind NAT</description>
  <executable>C:\Windows\System32\OpenSSH\ssh.exe</executable>
  <arguments>-o ExitOnForwardFailure=yes -o ConnectTimeout=3 -o TCPKeepAlive=yes -o ServerAliveInterval=5 -o ServerAliveCountMax=5 -N -i C:\Users\rasmus\.ssh\id_ecdsa -p 5123 -R 2242:localhost:22 sshtunnel@tunnel.casa</arguments>
  <startmode>Automatic</startmode>
  <depend>Dnscache</depend>
  <serviceaccount>
    <user>rasmus</user>
    <password>login_password_for_rasmus</password>
    <allowservicelogon>true</allowservicelogon>
  </serviceaccount>
  <onfailure action="restart" delay="10 sec" />
  <logmode>rotate</logmode>
</service>
```

Then install and start the service with:
```shell
ssh-tunnel.exe install
ssh-tunnel.exe start
```

You should see it running:

```shell
PS C:\> Get-Service ssh-tunnel | Select-Object -Property Name, StartType, Status, RequiredServices

Name       StartType  Status RequiredServices
----       ---------  ------ ----------------
ssh-tunnel Automatic Running {Dnscache}
```

## GUI Access
Sometimes, especially on Windows, you need to see the GUI. Often it is to understand what the actual user problem is.

MacOS comes with a VNC server. You just have to enable it under *System-Preferences - Sharing*. Turn on *Screen Sharing*
and click on the *Computer settings* button and set a password for *VNC Viewers may control screen with password*.

On Windows install TigerVNC from http://tigervnc.bphinz.com/nightly/ configure it with a password and start the service.

Again, you can check if the service is running with:

```
PS C:\Users\rasmus> Get-Service TigerVNC | Select-Object -Property Name, StartType, Status

Name     StartType  Status
----     ---------  ------
TigerVNC Automatic Running
```

You'll need to configure command-line access to actually connect to this.


## Command-line Access
It is so much quicker to fix stuff from a command line. Chances are the remote connection doesn't have great
upstream bandwidth, so if you can avoid the GUI it makes things easier.

### SSHD

MacOS and Windows 10 both come with OpenSSH.

On Mac, go to *System-Preferences - Sharing* and turn on *Remote-login*.

On Windows run the following command to get the current OpenSSH version string.

```
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
```

Currently it is version *0.0.1.0*. The client is usually enabled by default, but the server isn't. To make sure both are
enabled, do (possibly substituting the version there):

```
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

In */ProgramData/ssh/sshd_config* comment out:

```
Match Group administrators
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

so it becomes:

```
#Match Group administrators
#       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

For added security you can also turn off password logins by setting *PasswordAuthentication* to *No* here.

Also, let's make Powershell 7 the login shell for incoming ssh connections:

```
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Program Files\PowerShell\7\pwsh.exe" -PropertyType String -Force
```

### Start SSHD

Now everything should be ready. Start the sshd service. On Windows do:

```
Set-Service -Name sshd -StartupType Automatic
Start-Service sshd
```

Check that it worked with:

```
PS C:\> Get-Service sshd

Status   Name               DisplayName
------   ----               -----------
Running  sshd               OpenSSH SSH Server
```

You can check if it will start automatically with:

```
PS C:\> Get-Service sshd | Select-Object -Property Name, StartType, Status

Name StartType  Status
---- ---------  ------
sshd Automatic Running
```

Now to actually ssh in, on your client machine create an entry in your *.ssh/config* file that looks something like this:
```
Host RemoteCompName
User rasmus
ProxyCommand ssh -p 5123 -e none tunnel.casa exec nc localhost 2242
LocalForward 2243 localhost:5900
DynamicForward 8081
```

Now you should be able to type:
```
ssh RemoteCompName
```
And you should get in.

The `LocalForward` rule there forwards `localhost:2243` to the remote machine's VNC port 5900. So in your VNC client
on you local machine, just connect to `localhost:2243` and enter the remote VNC password.

And finally, through the magic of SSH's `DynamicForward` mechanism, you can configure your browser's SOCKS v5 proxy to point to *127.0.0.1* port *8081*.
Once you have done that, you will be browsing as if you were connected directly to the remote network. This is very handy for configuring devices
on the remote network and much quicker than remotely controlling a browser.

<a id="cloudflare"></a>
## Cloudflare Access Tunnel Alternative
Cloudflare's [Argo Tunnel](https://developers.cloudflare.com/access/setting-up-access/argo-tunnel/) which is part of their
*Cloudflare Access* service can be used instead of a VPS. It is free right now (it says at least until September 2020) but
even after that the price is in line with a low-end VPS at $5/month.

At the high level you create a tunnel using their cloudflare daemon (cloudflared) and tunnel out to the Cloudflare network
from both the machine you want to manage and also from your client machine.

To configure it you need a Cloudflare managed domain. Mine is appropriately called **tunnel.casa**.
Follow Cloudflare's instructions for setting your nameservers, then sign the domain up for the *Access* product.
It is free, but they will ask for your credit card. It takes a little while for the DNS changes to catch up.
Then download *cloudflared* on both the remote machine and your local machine.

On the remote machine:

```
cloudflared login
```

This will bring up a browser window asking you to select your domain and then download a cert file. On Windows I had to manually put
that in *C:\Windows\system32\config\systemprofile\.cloudflared\*

Then I created *C:\Windows\system32\config\systemprofile\.cloudflared\config.yaml* containing:

```yaml
hostname: tunnel.casa
url: ssh://localhost:22
logfile: C:\Users\rasmus\.cloudflared\cloudflared.log
```

Then install it as a service:

```
cloudflared service install
```

Check it:

```
PS C:\Users\rasmus> Get-Service cloudflared | Select-Object -Property Name, StartType, Status

Name        StartType  Status
----        ---------  ------
cloudflared Automatic Running
```

If the StartType isn't set to `*Automatic*` you can fix that with:

```
Set-Service -Name cloudflared -StartupType Automatic
```

And if it isn't running start it with:

```
Start-Service cloudflared
```

On your local machine:

```
cloudflared login
```

Again, this will open up a browser and ask you to login and choose a domain, then download the cert.
On Linux it just worked for me. I didn't have to move anything. Then run:

```
cloudflared access ssh-config --hostname tunnel.casa
```

This will print out what you need to add to your *.ssh/config* file. For me it was:

```
Host tunnel.casa
  ProxyCommand /usr/local/bin/cloudflared access ssh --hostname %h
```

You can add a `User rasmus` line there to set the right username for the remote server.
Then try it:

```
ssh tunnel.casa
```

This should just work assuming you configured your remote sshd correctly as per the above instructions.

As per the VPS tunnel case, you can add:

```
LocalForward 2243 localhost:5900
DynamicForward 8081
```

to your *.ssh/config* entry for this machine and connect to VNC and proxy across this Cloudflare tunnel.

