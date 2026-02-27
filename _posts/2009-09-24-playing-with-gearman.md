---
date: '2009-09-24'
layout: post
permalink: /archives/51-Playing-with-Gearman.html
title: Playing with Gearman
---


This was written in September 2009 when the current version of Gearman
was 0.9. Thanks to Eric Day for answering my dumb questions along the
way.  
  
To get started, install Gearman. I am on Debian, so this is what I
installed:

    % apt-get install gearman gearman-job-server gearman-tools libgearman1 libgearman-dev libdrizzle-dev

Enable Gearman in **/etc/default/gearman-server**  
Set up Gearman to use MySQL for its persistent queue store in
**/etc/default/gearman-job-server**

``` 
 PARAMS="-q libdrizzle --libdrizzle-host=127.0.0.1 --libdrizzle-user=gearman \
                       --libdrizzle-password=your_pw --libdrizzle-db=gearman \
                       --libdrizzle-table=gearman_queue --libdrizzle-mysql"

% mysqladmin create gearman

% mysql 
mysql> create USER gearman@localhost identified by 'your_pw';
mysql> GRANT ALL on gearman.* to gearman@localhost;
```

\*\* **Careful**, if you are running MySQL using **--old-passwords**
this won't work with libdrizzle. You will need to get the 41-char
password hash with a little snippet of PHP that does the double sha1
encoding:

    % php -r "echo '*'.strtoupper(sha1(sha1('your_pw',true)));"
    
    % mysql
    mysql> UPDATE mysql.user set Password='above_output' where User='gearman';
    
    % mysqladmin flush-privileges

Then start the server.

    % /etc/init.d/gearman-job-server start

Check to make sure gearmand is running. If it isn't, check for errors in
**/var/log/gearman-job-server/gearman.log**  
Also note that your password will be visible with '**ps**' with this
setup. The Gearman guys will be addressing this in gearmand-0.10  
  
Next, let's get the gearman PHP extension installed. Grab it from svn:

    % svn co http://svn.php.net/repository/pecl/gearman/trunk gearman
    % cd gearman
    % phpize
    % ./configure --with-php-config=/usr/local/bin/php-config

(If you have autoconf problems, **apt-get install autoconf2.59** and set
your **PHP\_AUTOCONF** env variable to "autoconf2.59")

    % make install

edit your **php.ini** file and add: **extension=gearman.so**  
  
Now we can add a worker. A worker is something that will process a
Gearman request. We can write it in almost any language, but here is one
in PHP (**worker.php**):
```php
    #!/usr/local/bin/php
    <?php
    $worker= new GearmanWorker();
    $worker->addServer('127.0.0.1');
    
    $worker->addFunction("reverse", "reverse_fn");
    
    while (1) {
      print "Waiting for job...\n";
      $ret= $worker->work();
      if ($worker->returnCode() != GEARMAN_SUCCESS) break;
    }
    function reverse_fn($job) {
      $workload= $job->workload();
      echo "Received job: " . $job->handle() . "\n";
      echo "Workload: $workload\n";
      $result= strrev($workload);
    
      for($i=1; $i<=10;  $i++) {
        $job->status($i,10);
        sleep(1);
      }
    
      echo "Result: $result\n";
      return $result;
    }
```

And start it up:
```shell
    % chmod +x worker.php
    % ./worker.php
    Waiting for job...
```
Now, in another terminal we can test it from the command line:  

    % gearman -f reverse Rasmus
    10% Complete
    20% Complete
    30% Complete
    40% Complete
    50% Complete
    60% Complete
    70% Complete
    80% Complete
    90% Complete
    100% Complete
    sumsaR

Or, better yet, from our Web application we can call it like this:

```php
    <?php
    // Set up connection to gearmand
    $client= new GearmanClient();
    $client->addServer('127.0.0.1');
    
    $task = $client->addTask("reverse", "ABC123");
    $task = $client->addTask("reverse", "DEF456");
    $result = $client->runTasks();
```

Here the two addTask() calls simply add the tasks to the queue, but
doesn't run them yet, so they return right away. runTasks() is blocking,
and since we only have one worker running, it will run them one after
the other and this call will take 20 seconds to return.  
  
So, as you might expect, if you start a second instance of the worker
program, these two tasks will run in parallel and runTasks() will return
in 10 seconds this time.  
  
If you don't want to hang around waiting for the results, you can do:
```php
    $task = $client->addTaskBackground("reverse", "ABC123");
```
This will cause runTasks() to be non-blocking. Useful for starting a
background jobs in one web request and then later coming back to check
the status of them.  
  
Speaking of checking the status. This isn't completely intuitive on the
first glance. Because of the way tasks are managed, you have to tell the
client library not to free the tasks as soon as they have been sent. So,
we need to do something like this:
```php
    $client= new GearmanClient();
    $client->addServer();
    $client->setOptions(GEARMAN_CLIENT_FREE_TASKS, 0);
```
Then we add and run 2 background tasks:
```php
    $task1 = $client->addTaskBackground("reverse", "ABC123");
    $task2 = $client->addTaskBackground("reverse", "DEF456");
    $result = $client->runTasks();
```
Now that the jobs are running, or in the queue, we can get the job
handles with:
```php
    $job1 = $task1->jobHandle();
    $job2 = $task2->jobHandle();
```
A job handle is just a string that looks like **"H:colo:70"**  
colo is the name of my machine.  
  
We can sleep(3) and then check the status.
```php
    var_dump($client->jobStatus($job1));
    var_dump($client->jobStatus($job2));
```
Each jobStatus() returns an array that looks like this:
```php
    array(4) {
      [0]=>
      bool(true)
      [1]=>
      bool(true)
      [2]=>
      int(6)
      [3]=>
      int(10)
    }
```
The first two elements are isKnown and isRunning respectively. These
flags do what they sound like. isKnown tells you if gearman knows about
the job. This will be true if the job is in the queue or it is running.
And isRunning obviously will be true if the job is currently running.
The next two fields are the numerator and denominator set by our worker.
In this case it is showing 6 out of 10, so 60% complete.  
  
Of course, these job handles, since they are just strings, could also be
stored in a user's session and checked from a separate request or from
an Ajax call. This is where the power of gearman starts to become
apparent. Out-of-band managed processing. And, of course, the job server
or servers don't need to run on localhost. You can set up a dedicated
farm of backend processing servers.  
  
In my example so far I started two background tasks, then checked each
of them in sequence by calling $client-\>jobStatus() on each job handle.
If you just have 2 jobs, that's not a big deal, but if you have a lot
and you want a snapshot of what they are doing, like if you want to know
which job is running faster, polling them in sequence will throw things
off. So, you can create a separate task for checking each status and run
those status checks just like you ran the jobs initially with
runTasks(). Like this:
```php
    $status1 = $client->addTaskStatus($job1);
    $status2 = $client->addTaskStatus($job2);
    $result = $client->runTasks();
```
And you can get those same four fields with:
```php
    echo $status1->isKnown(), $status1->isRunning(), 
         $status1->taskNumerator(), $status1->taskDenominator();
```
Obviously if either isKnown() or isRunning() returns false, there is no
point in checking the numerator and denominator.  
  
Your client and worker can also send data back and forth between each
other, but only for foreground tasks. First, from your worker you can
send data like this:
```php
    $job->data(serialize(posix_times()));
```
We can put that in our for-loop in the above example and have it update
once a second. In order to get this data from a foreground task, we have
to define callback functions because the runTasks() call is blocking. It
is done like this:
```php
    $client->setCompleteCallback("done");
    $client->setClientCallback("data");
    $task1 = $client->addTask("reverse", "ABC123");
    $result = $client->runTasks();
    
    function done($task) {
      echo "done() called\n";
      print_r($task->data());
      flush();
    }
    
    function data($task) {
      echo "data() called\n";
      print_r(unserialize($task->data()));
      flush();
    }
```
The data() function will be called whenever the worker calls
$job-\>data() and the done() function will be called when the task is
done. $task-\>data() inside the done() function will be set to the
returned result from the worker. In this example that is the reversed
string.  
  
Server-side callbacks in a Web app aren't terribly useful to me, so I
don't really see myself using this data-passing feature. Having my
frontend code sit around in a blocking runTasks() call isn't very
efficient. I'd rather be collecting other data and start sending out my
response to get my 1st-byte latency down. Therefore I only use
non-blocking background tasks and it currently isn't possible to use
this data passing mechanism from a background task. Right now if you
call $job-\>data() from a background worker, the data is simply
dropped.  
  
You will have to write it to memcache or some other data store from your
worker manually and have your clients check for data there. I think it
would be a nice enhancement to Gearman if it could support a data
storage plugin similar to the persistent queue storage plugin that would
simplify this case.
