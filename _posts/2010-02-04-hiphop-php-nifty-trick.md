---
date: '2010-02-04'
layout: post
permalink: /archives/53-HipHop-PHP-Nifty-Trick.html
title: HipHop PHP - Nifty Trick?
---


In a response to a question from ReadWriteWeb, among other things, I
wrote:

> My main worry here is that people think this is some kind of magic
> bullet that will solve their site performance problems. Generating C++
> code from PHP code is a nifty trick and people seem to have gotten
> quite excited about it. I'd love to see those same people get excited
> about basic profiling and identifying the most costly areas of an
> application. Speeding up one of the faster parts of your system isn't
> going to give you anywhere near as much of a benefit as speeding up,
> or eliminating, one of the slower parts of your overall system.

The "nifty trick" part of that seems to have become the story, and them
injecting a "just" in front it of it makes it sound more derogatory.
Anyone who knows me knows that I am a big fan of nifty tricks that solve
the problem. When I first heard about the Facebook effort I was assuming
they were writing a JIT based on LLVM V8 or something along those lines.
Writing a good JIT is hard. Doing static code analysis and generating
compilable C++ from it is indeed a nifty trick. It's not "just" a nifty
trick, it is a cool trick that takes advantage of a number of
characteristics of PHP. The main one being that you can't overload PHP
functions. strlen() is always strlen, for example. In Python, this would
be harder because you can overload everything.  
  
I also noted that most sites on the Web have a lot of lower hanging
fruit that would provide a much bigger performance improvement, if
fixed, than doubling the speed of the PHP execution phase. The
ReadWriteWeb site, for example, needs 160 separate HTTP requests and 41
distinct DNS lookups to load the front page. And once you get beyond the
frontend inefficiencies you usually find Database issues, inefficient
system call issues and general architecture problems that again aren't
solved by speeding up PHP execution.  
  
If you have done your homework and find that your web servers are
cpu-bound, you are already using an opcode cache like
[APC](http://pecl.php.net/apc) and your
[Callgrind](http://valgrind.org/info/tools.html#callgrind) callgraph
shows you that the PHP executor is a significant bottleneck, then HipHop
PHP is definitely something you should be looking at.
