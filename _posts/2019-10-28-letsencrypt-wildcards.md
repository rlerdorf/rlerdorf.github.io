---
layout: post
title: "Letsencrypt Wildcards"
date: 2019-10-28
---

![](/assets/images/letsencrypt-wildcards/letsencrypt-logo-horizontal.svg)
_This is mostly a note to myself explaining how and why I set this up the way I did._

## The Project
I have a number of domains spread across multiple registrars. Some are mine, others are friends and family domains
so they are spread out across multiple registrars and name service providers. This becomes a bit of a challenge to
manage when it comes to SSL certs for them, especially when someone wants to add a new host. Expanding the cert to
include the new hostname is a very manual process. It would be so much nicer if I could just use auto-renewing
wildcard certs everywhere. So that was the evening's project.

## Wildcard Certs
A wildcard cert is just what it sounds like. A certficate that matches a wildcard instead of a specific hostname.
Letsencrypt lets you add up to 100 hosts to a certificate. These 100 can actually be a mix of hostnames and wildcards.
For example, if we look at Wikipedia's cert:

```shell-session
$ nmap --script ssl-cert -p 443 wikipedia.com
Starting Nmap 7.70 ( https://nmap.org ) at 2019-10-13 07:45 PDT
Nmap scan report for wikipedia.com (208.80.153.232)
Host is up (0.051s latency).
Other addresses for wikipedia.com (not scanned): 2620:0:860:ed1a::9
rDNS record for 208.80.153.232: ncredir-lb.codfw.wikimedia.org

PORT    STATE SERVICE
443/tcp open  https
| ssl-cert: Subject: commonName=wikipedia.com
| Subject Alternative Name: DNS:*.en-wp.com, DNS:*.en-wp.org, DNS:*.mediawiki.com, DNS:*.voyagewiki.com,
                            DNS:*.voyagewiki.org, DNS:*.wiikipedia.com, DNS:*.wikibook.com,
                            DNS:*.wikibooks.com, DNS:*.wikiepdia.com, DNS:*.wikiepdia.org, DNS:*.wikiipedia.org,
                            DNS:*.wikijunior.com, DNS:*.wikijunior.net, DNS:*.wikijunior.org,
                            DNS:*.wikipedia.com, DNS:en-wp.com, DNS:en-wp.org, DNS:mediawiki.com,
                            DNS:voyagewiki.com, DNS:voyagewiki.org, DNS:wiikipedia.com, DNS:wikibook.com,
                            DNS:wikibooks.com, DNS:wikiepdia.com, DNS:wikiepdia.org, DNS:wikiipedia.org,
                            DNS:wikijunior.com, DNS:wikijunior.net, DNS:wikijunior.org, DNS:wikipedia.com
| Issuer: commonName=Let's Encrypt Authority X3/organizationName=Let's Encrypt/countryName=US
| Public Key type: ec
| Public Key bits: 256
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2019-09-29T08:00:26
| Not valid after:  2019-12-28T08:00:26
| MD5:   4663 84ce a1bd 7b44 4ea7 5dd2 96d5 0ade
|_SHA-1: b48c d95c 5240 d5ae 8b47 ec27 ca71 12f8 d276 c5d3
```

We can see it was issued by Letsencrypt on Sept.29 and it expires on Dec.28 (all Letsencrypt certs expire after 90 days)
and that they have a whole mess of wildcards on the SAN (Subject Alternative Name) line. Even some misspelled versions
of their domain. So you can go to [https://wikiepdia.com](https://wikiepdia.com) and it will properly establish an SSL
connection and redirect you to the correct spelling of the site.

And yes, I use _nmap_ to check certs mostly because I can never remember the proper _openssl_ incantation to do it.

## Getting a wildcard cert
The tricky part about getting a wildcard cert from Letsencrypt is that you have to use their _DNS-01_ challenge type.
It is described at [https://letsencrypt.org/docs/challenge-types/](https://letsencrypt.org/docs/challenge-types/) and they
list pros and cons:

Pros:
- You can use this challenge to issue certificates containing wildcard domain names.
- It works well even if you have multiple web servers.

Cons:
- Keeping API credentials on your web server is risky.
- Your DNS provider might not offer an API.
- Your DNS API may not provide information on propagation times.

They talk about your DNS provider's API in the cons because this challenge requires adding a _TXT_ record to your
DNS. Every provider has a different way of doing that and _certbot_ has plugins for many of them. I have been using
the Cloudflare plugin for a while for a couple of domains. But like they say, some DNS providers don't offer an API,
but even if they do, I prefer to centralize this to a single mechanism for all my domains.

So, the way to detach yourself from your DNS providers is to become your own DNS provider. Running your own
name service seems a bit daunting, but it really isn't that bad. There are even some simplified solutions
dedicated to just this letsencrypt task. [ACME-DNS](https://github.com/joohoi/acme-dns) looks cool.
I am not using that, though.  I am just using the regular Debian _bind9_ setup. I am not going to describe
how to set up _bind9_ here. There are many guides out there showing how to do that. You can start with https://www.linuxbabe.com/debian/authoritative-dns-server-debian-10-buster-bind9.

## Making it work

I completely stole this from combining
[Sebastian Korotkiewicz's excellent post](https://sebastian.korotkiewicz.eu/techlog/automated-lets-encrypt-wildcard-certificates-with-local-bind/) with [Patrick Terlisten's](https://www.vcloudnine.de/using-lets-encrypt-dns-01-challenge-validation-with-local-bind-instance/).

The first thing you need to do is add an _NS_ record. It should look something like this in whatever UI your
DNS provider provides for you to add records:

```
_acme-challenge.example.com. 120 IN NS	ns.yourdomain.com.
```
_ns.yourdomain.com_ is your name server and the *_acme-challenge* host name is a special host
used by letsencrypt that will let you get a wildcard cert for _*.example.com_. In the _namecheap_ UI,
for example, you click on "Advanced DNS" then on the red "Add new record" link and scroll down and select
"NS Record" in the dropdown. Then you would enter *_acme-challenge* in the Host field and _ns.yourdomain.com_
in the Nameserver field.

Next, make yourself a */etc/bind/dyn* directory and create the file *_acme-challenge.example.com.zone*
containing something like:
```
$ORIGIN .
$TTL 604800     ; 1 week
_acme-challenge.example.com IN SOA _acme-challenge.example.com. root._acme-challenge.example.com. (
                                2019101107 ; serial
                                10800      ; refresh (3 hours)
                                7200       ; retry (2 hours)
                                604800     ; expire (1 week)
                                86400      ; minimum (1 day)
                                )
                        NS      ns.yourdomain.com.
                        A       12.34.56.78
```

And then in your */etc/bind/named.conf.local* file add something close to this:
```
zone "_acme-challenge.example.com" {
    type master;
    file "/etc/bind/dyn/_acme-challenge.example.com.zone";
    allow-query { any; };
    update-policy {
        grant letsencrypt_wildcard  name _acme-challenge.example.com. txt;
    };
    notify yes;
    allow-transfer { "trusted-transfer"; };
    check-names ignore;
};
```

The tricky part here is the _update-policy_ part. That says that anyone with the *letsencrypt_wildcard*
key is allowed to update *TXT* records for the *_acme-challenge.example.com* sub-domain.

To create this key you will need the dnssec-keygen tool from the _bind9utils_ package to generate a
Transaction Signature (TSIG) key like this:
```shell-session
$ dnssec-keygen -a HMAC-SHA512 -b 512 -n HOST letsencrypt_wildcard
Kletsencrypt_wildcard.+165+23392
```
That will have created a _.key_ and a _.private_ file in your current directory. The _.private_ file looks
like this:
```
Private-key-format: v1.3
Algorithm: 165 (HMAC_SHA512)
Key: MXTghKhtViRKznund212DYgSdhA7vuVJbqzWcDjfgZzLckVUt3P88etyiJejZ4plpFPnSanLRdG5pa/+sUJ1VQ==
Bits: AAA=
Created: 20191013151405
Publish: 20191013151405
Activate: 20191013151405
```
Grab the Key string from this file and create a _/etc/letsencrypt/super-secret-key_ file with:
```
key "letsencrypt_wildcard" {
    algorithm hmac-sha512;
    secret "MXTghKhtViRKznund212DYgSdhA7vuVJbqzWcDjfgZzLckVUt3P88etyiJejZ4plpFPnSanLRdG5pa/+sUJ1VQ==";
};
```
You can include this file from your */etc/bind/named.conf.local* or just paste the contents at the top.

Reload your bind config and on to the next step of teaching *certbot* how to update these *TXT* records.

Create the script */etc/letsencrypt/scripts/dns-auth.sh* containing:
```bash
#!/bin/bash

if [ -z "$CERTBOT_DOMAIN" ] || [ -z "$CERTBOT_VALIDATION" ]
then
echo "EMPTY DOMAIN OR VALIDATION"
exit -1
fi

HOST="_acme-challenge"

/usr/bin/nsupdate -k /etc/letsencrypt/super-secret-key << EOM
server ns.yourdomain.com
zone ${HOST}.${CERTBOT_DOMAIN}
update delete ${HOST}.${CERTBOT_DOMAIN} TXT
update add ${HOST}.${CERTBOT_DOMAIN} 300 TXT "${CERTBOT_VALIDATION}"
send
EOM
echo ""
sleep 3
```
and make it executable.

And you are basically done. Now test it. Use the *--dry-run* switch on certbot like this:

```shell-session
$ certbot certonly -n --dry-run --manual-public-ip-logging-ok --server https://acme-v02.api.letsencrypt.org/directory --agree-tos --manual --preferred-challenges=dns --manual-auth-hook /etc/letsencrypt/scripts/dns-auth.sh -d "*.example.com" -d example.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Obtaining a new certificate

IMPORTANT NOTES:
 - The dry run was successful.
```

If you get a successful dry run on your first try you are a lot better at this than I am. It took me a dozen attempts
before it worked.

One thing that can help is to skip the TSIG key initially and instead of the *update_policy* just do:
`allow-update { localhost; };` to allow all updates from localhost.

The next step after you get it working is to copy that _/etc/letsencrypt/scripts/dns-auth.sh_ script to your other
machines along with your *super-secret-key* file. This way you can have multiple machines all generate their
own _*.example.com_ wildcard cert.

## Making it automatic

Once you have successful dry-runs you can remove the *--dry-run* switch and let it actually do it. It should create an
*example.com.conf* file in */etc/letsencrypt/renewal*. Review that file and make sure it looks like that is what you
want to do on a renewal.

These days if you are using your Linux distro's *certbot* package, it has already installed a cron job or a *systemd* timer which will
call `certbot renew` for you at regular intervals. Make sure it will work by doing `sudo certbot renew --dry-run`. In my case I prefer
not to use any certbot plugins and just have certbot install the cert and nothing else. So I edit */etc/letsencrypt/cli.ini* and add:

```ini
renew-hook = systemctl reload nginx;service reload some_sysv_thing
```

If you have been messing around a lot with certbot, like I did, you might have a bunch of extra stuff happening when you run that renew
dry run. You can clean it up by deleting every *.conf* you don't need from */etc/letsencrypt/renewal*.

And finally check that your cron/timer is actually active. On modern *systemd* systems you should see a *certbot.timer* when you do `systemctl list-timers --all`

### References
- https://sebastian.korotkiewicz.eu/techlog/automated-lets-encrypt-wildcard-certificates-with-local-bind/
- https://www.vcloudnine.de/using-lets-encrypt-dns-01-challenge-validation-with-local-bind-instance/
- https://linux.m2osw.com/setting-bind-get-letsencrypt-wildcards-work-your-system-using-rfc-2136
- https://github.com/joohoi/acme-dns
