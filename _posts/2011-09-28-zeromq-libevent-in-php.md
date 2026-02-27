---
date: '2011-09-28'
layout: post
permalink: /archives/57-ZeroMQ-+-libevent-in-PHP.html
title: ZeroMQ + libevent in PHP
---


While waiting for a connection in Frankfurt I had a quick look at what
it would take to make ZeroMQ and libevent co-exist in PHP and it was
actually quite easy. Well, easy after Mikko Koppanen added a way to get
the underlying socket fd from the ZeroMQ PHP extension. To get this
working, install the [PHP ZeroMQ
extension](http://www.zeromq.org/bindings:php) and the [PHP libevent
extension](http://pecl.php.net/package/libevent). First, a little
event-driven server that listens on loopback port 5555 and waits for 10
messages and then exits.  
  

Server.php
```php
<?php
    function print_line($fd, $events, $arg) {
        static $msgs = 1; 
        echo "CALLBACK FIRED" . PHP_EOL;
        if($arg[0]->getsockopt (ZMQ::SOCKOPT_EVENTS) & ZMQ::POLL_IN) {
            echo "Got incoming data" . PHP_EOL;
            var_dump ($arg[0]->recv());
            $arg[0]->send("Got msg $msgs");
        if($msgs++ >= 10) event_base_loopexit($arg[1]);
        }
    }
    
    // create base and event
    $base = event_base_new();
    $event = event_new();
    
    // Allocate a new context
    $context = new ZMQContext();
    
    // Create sockets
    $rep = $context->getSocket(ZMQ::SOCKET_REP);
    
    // Connect the socket
    $rep->bind("tcp://127.0.0.1:5555");
    
    // Get the stream descriptor
    $fd = $rep->getsockopt(ZMQ::SOCKOPT_FD);
    
    // set event flags
    event_set($event, $fd, EV_READ | EV_PERSIST, "print_line", array($rep, $base));
    
    // set event base
    event_base_set($event, $base);
    
    // enable event
    event_add($event);
    
    // start event loop
    event_base_loop($base);
```
  

Client.php

```php
<?php
    // Create new queue object
    $queue = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_REQ, "MySock1");
    $queue->connect("tcp://127.0.0.1:5555");
    
    // Assign socket 1 to the queue, send and receive
    var_dump($queue->send("hello there!")->recv());
```
  
You will notice when you run it that the server gets a couple of events
that are not actually incoming messages. Right now ZeroMQ doesn't expose
the nature of these events, but they are the socket initialization and
client connect. You will also get one for the client disconnect. A
future version of the ZeroMQ library will expose these so you can
properly catch when clients connect to your server.  
  
There really isn't much else to say. The code should be
self-explanatory. If not, see the [PHP
libevent](http://php.net/libevent) docs and the [PHP
ZeroMQ](http://php.zero.mq/) docs. And if you build something cool with
this, please let me know.
