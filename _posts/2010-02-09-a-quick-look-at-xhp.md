---
date: '2010-02-09'
layout: post
title: A quick look at XHP
---


Facebook released a new PHP extension today that supports inlining XML.
This is a feature known as [XML Literals in Visual
Basic](http://msdn.microsoft.com/en-us/library/bb384832.aspx). Go read
their description here:
<http://www.facebook.com/notes/facebook-engineering/xhp-a-new-way-to-write-php/294003943919>  
It adds an extra parsing step which maps inlined XML elements to PHP
classes. These classes are
[core.php](http://github.com/facebook/xhp/blob/master/php-lib/core.php)
and
[html.php](http://github.com/facebook/xhp/blob/master/php-lib/html.php)
which covers all the main HTML elements. The syntax of those class
definitions is a bit odd. That oddness is explained in the [How It
Works](http://wiki.github.com/facebook/xhp/how-it-works) document.  
  
Essentially, it lets you turn:

    <?php
    if ($_POST['name']) {
        echo "<span>Hello, {$_POST['name']}.</span>";
    } else {
    ?>
        <form method="post">
        What is your name?<br>
        <input type="text" name="name">
        <input type="submit">
        </form>
    <?php
    }

into:

    <?php
    require './core.php';
    require './html.php';
    if ($_POST['name']) {
      echo <span>Hello, {$_POST['name']}.</span>;
    } else {
      echo
        <form method="post">
          What is your name?<br />
          <input type="text" name="name" />
          <input type="submit" />
        </form>;
    }

The main interest, at least to me, is that because PHP now understands
the XML it is outputting, filtering can be done in a context-sensitive
manner. The [input filtering](http://php.net/filter) built into PHP can
not know which context a string is going to be used in. If you use a
string inside an on-handler or a style attribute, for example, you need
radically different filtering from it being used as regular XML PCDATA
in the html body. Some will say this form is more readable as well, but
that isn't something that concerns me very much.  
  
The real question here is what is this runtime xml validation going to
cost you. I have given talks in the past where I have used "class br
extends html { ... }" as a classic example of something you should never
do. A br tag is just a br tag. When you need one, stick a \<br\> in your
page, don't instantiate a class and call a render() method. So, when I
looked at
[html.php](http://github.com/facebook/xhp/blob/master/php-lib/html.php)
and saw:

    class :br extends :xhp:html-singleton {
      category %flow, %phrase;
      protected $tagName = 'br';
    }

I got a bit skeptical. Another thing I have been known to tell people
is, "Friend don't let friends use Singletons." Which isn't something I
came up with. Someone, a friend, I guess, told me that years ago. Ok ok,
as Marcel points out in the comments, this isn't a real singleton, just
in name.  
  
The "singleton" looks like this:

    abstract class :xhp:html-singleton extends :xhp:html-element {
      children empty;
     
      protected function stringify() {
        return $this->renderBaseAttrs() . ' />';
      }
    }

which extends html-element which in turn extends primitive. You can go
read all the code for those yourself.  
  
Note that to build XHP you will need flex 2.5.35 which most distros
won't have installed by default. Grab the [flex
tarball](http://prdownloads.sourceforge.net/flex/flex-2.5.35.tar.gz?download)
and ./configure && make install it. Then you are ready to go.  
  
I pointed [Siege](http://www.joedog.org/index/siege-home) at my rather
underpowered AS1410 SU2300 with the above trivial form examples. The
plain PHP one and the XHP version. Ran each one 5 times benchmarking for
30s each time. The plain PHP one averaged around 1300 requests/sec. Here
is a representative sample:

    acer:~> siege -c 3 -b -t30s http://xhp.localhost/1.php
    ** SIEGE 2.68
    ** Preparing 3 concurrent users for battle.
    The server is now under siege...
    Lifting the server siege...      done.
    Transactions:              38239 hits
    Availability:             100.00 %
    Elapsed time:              29.60 secs
    Data transferred:           3.97 MB
    Response time:              0.00 secs
    Transaction rate:        1291.86 trans/sec
    Throughput:             0.13 MB/sec
    Concurrency:                2.93
    Successful transactions:       38239
    Failed transactions:               0
    Longest transaction:            0.05
    Shortest transaction:           0.00

And the XHP version:

    Transactions:                868 hits
    Availability:             100.00 %
    Elapsed time:              29.28 secs
    Data transferred:           0.08 MB
    Response time:              0.10 secs
    Transaction rate:          29.64 trans/sec
    Throughput:             0.00 MB/sec
    Concurrency:                2.99
    Successful transactions:         868
    Failed transactions:               0
    Longest transaction:            0.21
    Shortest transaction:           0.05

So, a drop from 1300 to around 30 requests per second and latency from
less than 10ms to 100ms. Running XHP on plain PHP is definitely out of
the question. But, knowing that Facebook uses APC heavily and looking
through the code (see the MINIT function in
[ext.cpp](http://github.com/facebook/xhp/blob/master/ext.cpp)) we can
see that it should play nicely with APC. So, re-running our PHP version
of the form, now with APC enabled, that goes from 1300 to around 1460
requests per second, and no measurable latency:

    Transactions:              43773 hits
    Availability:             100.00 %
    Elapsed time:              29.88 secs
    Data transferred:           4.55 MB
    Response time:              0.00 secs
    Transaction rate:        1464.96 trans/sec
    Throughput:             0.15 MB/sec
    Concurrency:                2.93
    Successful transactions:       43773
    Failed transactions:               0
    Longest transaction:            0.07
    Shortest transaction:           0.00

The XHP version of the form now with APC enabled:

    Transactions:               9707 hits
    Availability:             100.00 %
    Elapsed time:              29.45 secs
    Data transferred:           0.94 MB
    Response time:              0.01 secs
    Transaction rate:         329.61 trans/sec
    Throughput:             0.03 MB/sec
    Concurrency:                2.97
    Successful transactions:        9707
    Failed transactions:               0
    Longest transaction:            0.21
    Shortest transaction:           0.00

Much better. But it is still around a 75% performance drop from 1460 to
330 and a \~10ms latency penalty. And yes, I did have a default filter
enabled for these tests, so there was basic XSS filtering in place for
the naked $\_POST\['name'\] variable in the plain PHP version. Of
course, the default filtering would likely fail if the user data was
used in a different context. And this 75% is obviously going to depend
on what else is going on during the request. If you are spending most of
your time calculating a fractal or waiting on MySQL, you may not notice
XHP very much at all.  
  
The bulk of the time is spent in all the tag to class interaction. If
the core.php and html.php code was all baked into the XHP extension, it
would be a lot quicker, of course. So, when you combine XHP with HipHop
PHP you can start to imagine that the performance penalty would be a lot
less than 75% and it becomes a viable approach. Of course, this also
means that if you are unable to run HipHop you probably want to think a
bit and run some tests before adopting this. If you are already doing
some sort of external templating, XHP could very well be a faster
approach.  
**Update:** Here are the callgraphs. [![Without
XHP](/assets/images/a-quick-look-at-xhp/callgraph1.Thumb.png)](callgraph1.png?classes=float-center)
The first is the plain PHP+APC version without XHP. And the second is
the PHP+APC+XHP version. In the first you see all sorts of bits and
pieces all across the stack getting cpu time.  
In the second graph we see the effect of needing to copy and instantiate
those core and html classes on every request. They are cached in APC, of
course, but because of PHP's perfect sandbox, they cannot persist.
[![With
XHP](/assets/images/a-quick-look-at-xhp/callgraph2.Thumb.png)](callgraph2.png?classes=float-center)
So we went from spending around 1% of our time in the executor to over
80%.  
This isn't an entirely fair comparison, of course, since the plain
version has close to no PHP to execute while the XHP version has 93
userspace classes to deal with. I would guess that XHP could get quite a
boost if at least the primitives in core.php could be baked into the
extension. Ideally all 93 basic html classes would be in C++ in the
extension itself, but that would be a bit of a tedious undertaking.
