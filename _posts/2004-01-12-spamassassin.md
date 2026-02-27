---
date: '2004-01-12'
layout: post
permalink: /archives/18-Spamassassin.html
title: Spamassassin
---


...is normally a very good toy. I use it extensively with Razor2 and
Bayes hooked into it, it catches nearly everything. Today, however, I
was suddenly inundated with Viagra spam. Odd, since any such spam
usually sets off all sorts of SA alarms. And sure enough, the alarms
were there:
```
X-Spam-Status: No, hits=0.9 required=5.0
tests=BAYES\_99,BIZ\_TLD,CLICK\_BELOW,
HABEAS\_SWE,HTML\_50\_60,HTML\_LINK\_CLICK\_HERE,HTML\_MESSAGE,
MIME\_HTML\_ONLY,MIME\_HTML\_ONLY\_MULTI
```


So what gives? I have my BAYES\_99 rule cranked way up to 5.4, so that
alone should have put it over the 5.0 spam requirement, never mind all
the other ones it triggered. The answer is of course that there is a
large negative rule in there. In this case it was this HABEAS\_SWE
thing. The default SpamAssAss scores file has:
```
score HABEAS\_SWE -8.0 score HABEAS\_VIOLATOR 16.0
```


And it turns out that [Habeas](http://www.habeas.com) is some sort of
"good spam" company that you can pay to get yourself whitelisted if you
really need to spam people. I can see how that could be useful if you
have a newsletter or something that people subscribe to and then it
can't get through because of filters, but then these Habeas people damn
well better be on the ball and triple-check the intentions of everyone
and also run a tight ship security-wise. Given the fact that I received
at least 20 Viagra spams before I killed that -8 rule, they obviously
weren't quite on the ball and I don't particularly appreciate that this
rule was in the default SA config to begin with. I haven't tracked down
exactly who put that rule in and what sort of compensation changed
hands. If someone knows, I'd like to know. I would suggest that you find
your local.cf file. Mine is in /etc/spamassassin/local.cf and add:
```
score HABEAS\_SWE 0.0
```


Making it neutral.
