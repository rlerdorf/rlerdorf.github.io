---
date: '2006-02-27'
layout: post
permalink: /archives/38-The-no-framework-PHP-MVC-framework.html
title: The no-framework PHP MVC framework
---


**March 1, 2006 - Disclaimer**: Since a lot of people seem to me
misunderstanding this article. It isn't about OOP vs. Procedural
programming styles. I happen to lean more towards procedural, but could
easily have gone more OOP. I simplified the code a bit for brevity, but
have added a light OO layer back in the model now. Not that it makes a
difference. What I was hoping to get across here is a simple example of
how you can use PHP as-is, without additional complex external layers,
to apply an MVC approach with clean and simple views and still have all
the goodness of fancy Web 2.0 features. If you think I am out to
personally offend you and your favourite framework, then you have the
wrong idea. I just happen find most of them too complex for my needs and
this is a proposed alternative. If you have found a framework that works
for you, great.  
  

-----

  
So you want to build the next fancy Web 2.0 site? You'll need some gear.
Most likely in the form of a big complex MVC framework with plenty of
layers that abstracts away your database, your HTML, your Javascript and
in the end your application itself. If it is a really good framework it
will provide a dozen things you'll never need.  
I am obviously not a fan of such frameworks. I like stuff I can
understand in an instant. Both because it lets me be productive right
away and because 6 months from now when I come back to fix something,
again I will only need an instant to figure out what is going on. So,
here is my current approach to building rich web applications. The main
pieces are:  

  - [PHP 5](http://php.net)
  - [Yahoo\! User Interface Library](http://developer.yahoo.net/yui/)
  - [JSON](http://json.org)

## MVC?

I don't have much of a problem with MVC itself. It's the framework
baggage that usually comes along with it that I avoid. Parts of
frameworks can be useful as long as you can separate the parts out that
you need. As for MVC, if you use it carefully, it can be useful in a web
application. Just make sure you avoid the temptation of creating a
single monolithic controller. A web application by its very nature is a
series of small discrete requests. If you send all of your requests
through a single controller on a single machine you have just defeated
this very important architecture. Discreteness gives you scalability and
modularity. You can break large problems up into a series of very small
and modular solutions and you can deploy these across as many servers as
you like. You need to tie them together to some extent most likely
through some backend datastore, but keep them as separate as possible.
This means you want your views and controllers very close to each other
and you want to keep your controllers as small as possible.

## Goals for this approach

Clean and simple design

  - HTML should look like HTML
  - Keep the PHP code in the views extremely simple: function calls,
    simple loops and variable substitutions should be all you need

Secure

  - Input validation using pecl/filter as a data firewall
  - When possible, avoid layers and other complexities to make code
    easier to audit

Fast

  - Avoid include\_once and require\_once
  - Use APC and apc\_store/apc\_fetch for caching data that rarely
    changes
  - Stay with procedural style unless something is truly an object
  - Avoid locks at all costs

![](/assets/images/the-no-framework-php-mvc-framework/scr_mvc.png)

## Example Application

It is a form entry page with a bit of Javascript magic along with an
sqlite backend. Click around a bit. Try to add an entry, then modify it.
You will see the server-\>client JSON traffic displayed at the bottom
for debug purposes.  

## The Code

This is the code layout. It uses AJAX (with JSON instead of XML over the
wire) for data validation. It also uses a couple of components from the
[Yahoo\! user interface library](http://developer.yahoo.net/yui/) and
PHP's [PDO](http://php.net/pdo) mechanism in the model.

  
![](/assets/images/the-no-framework-php-mvc-framework/example.gif)  

The presentation layer is above the line and the business logic below.
In this simple example I have just one view, represented by the
**add.html** file. It is actually called add.php on the live server, but
I was too lazy to update the diagram and it really doesn't matter. The
controller for that view is called **add\_c.inc**. I tend to name files
that the user loads directly as *something.html* or *something.php* and
included files as *something.inc*. The rest of the files in the
presentation layer are common files that all views in my application
would share.

  

**ui.inc** has the common user interface components, **common.js**
contains Javascript helper functions that mostly call into the
presentation platform libraries, and **styles.css** provides the
stylesheet.

  

A common **db.inc** file implements the model. I tend to use separate
include files for each table in my database. In this case there is a
just single table called "items", so I have a single **items.inc** file.

## Input Filtering

You will notice a distinct lack of input filtering yet if you try to
inject any sort of XSS it won't work. This is because I am using the
[pecl/filter](http://pecl.php.net/filter) extension to automagically
sanitize all user data for me.

## View - add.html

Let's start with the View in **add.html**:

<div class="code">

</div>

  

The main thing to note here is that the majority of this file is very
basic HTML. No styles, or javascript and no complicated PHP. It contains
only simple presentation-level PHP logic. A modulus operation toggles
the colours for the rows of items, and a loop around a [heredoc
(\<\<\<)](http://php.net/manual/en/language.types.string.php#language.types.string.syntax.heredoc)
block performs variable substitutions. **head()** and **foot()**
function calls add the common template headers and footers.

  

If you wanted to make it even cleaner you could use an
[auto\_prepend\_file](http://php.net/manual/en/ini.core.php#ini.auto-prepend-file)
configuration setting which tells PHP to always include a certain file
at the top of your script. Then you could take out the include calls and
the initial **head()** function call. I tend to prefer less magic and to
control my template dependencies right in my templates with a very clean
and simple include structure. Try to avoid using *include\_once* and
*require\_once* if possible. You are much better off using a straight
include or require call, because the *\*\_once()* calls are very slow
under an opcode cache. Sometimes there is no way around using these
calls, but recognize that each one costs you an extra *open()* syscall
and hash look up.

## ui.inc

Here is the UI helper code from **ui.inc**:

<div class="code">

</div>

  

This file just contains the head() and foot() functions that contain
mostly plain HTML. I tend to drop out of PHP mode if I have big blocks
of HTML with minimal variable substitutions. You could also use
[heredoc](http://php.net/manual/en/language.types.string.php#language.types.string.syntax.heredoc)
blocks here, as we saw in **add.html**.

## Controller - add\_c.inc

Our Controller is in **add\_c.inc**:

<div class="code">

</div>

  

Our controller is going to manipulate the model, so it first includes
the model files. The controller then determines whether the request is a
POST request, which means a backend request to deal with. (You could do
further checks to allow an empty POST to work like a GET, but I am
trying to keep the example simple.) The controller also sets the
Content-Type to *application/json* before sending back JSON data.
Although this mime-type is not yet official so you might want to use
*application/x-json* instead. As far as the browser is concerned, it
doesn't care either way.

  

The controller then performs the appropriate action in the model
according to the specified command. A *load\_item*, for example, ends up
calling the **load()** method in the data model for the items table and
sends back a JSON-encoded response to the browser.

The important piece here is that the controller is specific to a
particular view. In some cases you may have a controller that can handle
multiple very similar views. Often the controller for a view is only a
couple of lines and can easily be placed directly at the top of the view
file itself.

## common.js

Next I need to catch these JSON replies, which I do in **common.js**:

<div class="code">

</div>

  

The **postForm()** and **postData()** functions demonstrate the genius
of the [Yahoo user interface
libraries](http://developer.yahoo.net/yui/): they provide us with
single-line functions to do our backend requests. The **fN** function in
the callback object does the bulk of the work, taking the JSON replies
generated by our controller and manipulating the DOM in the browser in
some way. There are also **fade()** and **unfade()** functions that are
called on status messages, and on validate errors to produce flashing
red field effects.

  

Note the bottom half of this file where **fancyItems()** and
**fancyForm()** implement all the client-side magic to animate the forms
by attaching handlers to various events. Often you will see server-side
business logic nicely separated from the templates, but then there are
big blocks of complicated client-side Javascript mixed into the template
which in my opinion defeats the clean separation goal. By going through
and attaching appropriate mouseover, mouseout, focus, blur and click
handlers after the fact I can keep my templates extremely clean and
still get a very dynamic experience. Here I am using the [event
library](http://developer.yahoo.net/yui/event/) from the [Yahoo\! user
interface libraries](http://developer.yahoo.net/yui/) to add the
handlers.

## Model - db.inc

Now for the model. First the generic **db.inc** which applies to all our
model components:

<div class="code">

</div>

  

I am using sqlite via [PDO](http://php.net/pdo) for this example, so the
**connect()** function is quite simple. The example also uses a fatal
error function that provides a helpful backtrace for any fatal database
error. The backtrace includes all the arguments passed to the functions
along the trace.

  

The **load\_list()** function uses an interesting trick: it uses APC's
[apc\_fetch()](http://php.net/apc_fetch) function to fetch an array
containing the list of item categories. If the list isn't in shared
memory, I read the file from disk and generate the array. I have made it
generic by using a [variable
variable](http://php.net/manual/en/language.variables.variable.php). If
you call it with **load\_list('categories')**, it automatically loads
**categories.txt** from the disk and creates a global array called
**$categories**.

## Model - items.inc

Finally, I have the model code for the items table, **items.inc**:

<div class="code">

</div>

  

At the top of each model file, I like to use a comment to record the
schema of any associated tables. I then provide a simple class with a
couple of methods to manipulate the table: in this case, **insert()**,
**modify()** and **load()**. Each function checks the database handle
property to avoid reconnecting in case I have multiple calls on each
request. You could also handle this directly in your **connect()**
method.

  

To avoid an extra time syscall, I use **$\_SERVER\["REQUEST\_TIME"\]**
to retrieve the request time. I am also using PDO's named parameters
mechanism, which is cleaner than trying to use question mark
placeholders.

## Conclusion

Clean separation of your views, controller logic and backend model logic
is easy to do with PHP. Using these ideas, you should be able to build a
clean framework aimed specifically at your requirements instead of
trying to refactor a much larger and more complex external framework.

  

Many frameworks may look very appealing at first glance because they
seem to reduce web application development to a couple of trivial steps
leading to some code generation and often automatic schema detection,
but these same shortcuts are likely to be your bottlenecks as well since
they achieve this simplicity by sacrifizing flexibility and performance.
Nothing is going to build your application for you, no matter what it
promises. You are going to have to build it yourself. Instead of
starting by fixing the mistakes in some foreign framework and
refactoring all the things that don't apply to your environment spend
your time building a lean and reusable pattern that fits your
requirements directly. In the end I think you will find that your
homegrown small framework has saved you time and aggravation and you end
up with a better product.
