# Setting up PHP & Hello PHP site

[PHP][] is one most versatile web languages around. It is also very simple to
setup. There are various other web toolkits out there, but for the sake of
simplicity of this tutorial we'll stick with PHP for now

## Enable PHP on Apache

Let's update our apache manifest to include the Apache PHP plugin:

```puppet
include apache
include apache::mod::php
```

and as usual, apply it via Puppet:

```
[root@my-first-app ~]# puppet apply apache-via-mod.pp
Notice: Compiled catalog for my-first-app.c.graphite-playground.google.com.internal in environment production in 1.36 seconds
Notice: /Stage[main]/Apache::Mod::Php/Apache::Mod[php5]/Package[php]/ensure: created
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.d/php.conf]/ensure: removed
Notice: /Stage[main]/Apache::Mod::Php/File[php5.conf]/ensure: defined content as '{md5}24420bffd2939b1fe3ff0ca36dbad419'
Notice: /Stage[main]/Apache::Mod::Php/Apache::Mod[php5]/File[php5.load]/ensure: defined content as '{md5}5aabed26d29e38135b5157acff80a2a6'
Notice: /Stage[main]/Apache/File[/etc/httpd/conf.modules.d/10-php.conf]/ensure: removed
Notice: /Stage[main]/Apache::Service/Service[httpd]: Triggered 'refresh' from 1 events
Notice: Applied catalog in 4.51 seconds
[root@my-first-app ~]# 
```

Puppet did all the necessary work to setup the necessary modules on your
machine, including the necessary packages from the Operating System. Apache is
now *ready* to take PHP content, so let's do it.


## Update main website page

Let's replace our main "coming soon" page with a page that tells the time:

```puppet
include apache
include apache::mod::php

# Let's get rid of the coming soon file
file { '/var/www/html/index.html':
  ensure => absent,
}

# Define our main website as a PHP page
file { '/var/www/html/index.php':
  ensure  => file,
  content => '<?php echo "Now it is " . date("r"); ?>', # echoes the time
}
```

... and apply ....

```
[root@my-first-app ~]# puppet apply php-sample.pp
Notice: Compiled catalog for my-first-app.c.graphite-playground.google.com.internal in environment production in 1.31 seconds
Notice: /Stage[main]/Main/File[/var/www/html/index.html]/ensure: removed
Notice: /Stage[main]/Main/File[/var/www/html/index.php]/ensure: defined content as '{md5}7b787125b1026cc7582f46a127f58115'
Notice: Applied catalog in 0.67 seconds
[root@my-first-app ~]#
```

Now refresh your browser (where the 'coming soon' page was showing) and you
should see something like this:

```
Now it is Sun, 25 Jun 2017 07:38:19 +0000
```

## Updating time zone

You will notice that the time is apparently not correct. Actually it is (the
machine has its clock synced to an atomic clock). The "problem" is that it is
showing the time in "UTC" (Greenwich) time zone "+0000".

PHP `date('r')` command draws the time zone from the machine, so we need to
adjust the machine time zone accordingly. (There are more sophisticated ways of
dealing with global time, but they are beyond this quick start tutorial).

You'll never guess: we'll be using Puppet to do that :) We'll write this Puppet
manifest once and use it anytime we need. As before put it in your file and
apply it using Puppet:

```puppet
include apache
include apache::mod::php

file { '/etc/php.d/timezone.ini':
  ensure  => file,
  content => join([
      '[Date]',
      'date.timezone = America/Los_Angeles',
    ]),
  notify  => Class['apache::service'],
}
```

The expected output:

```
[root@my-first-app ~]# puppet apply time.php
Notice: Compiled catalog for my-first-app.c.graphite-playground.google.com.internal in environment production in 1.50 seconds
Notice: /Stage[main]/Main/File[/etc/php.d/timezone.ini]/ensure: defined content as '{md5}34cb0851c4094296a53415c09486c53d'
Notice: /Stage[main]/Apache::Service/Service[httpd]: Triggered 'refresh' from 1 events
Notice: Applied catalog in 1.80 seconds
[root@my-first-app ~]#
```

The `include` and `file` portions are not new. We're just creating another file
that PHP will read to configure time zone (following PHP [date.timezone
docs][php-date-timezone]). You can use any of the [Supported
Timezones][php-timezones] described.

The new part here is the `notify =>` line. This one is critical for you to
understand as it is a core Puppet strength. This tells Puppet to notify the
service (and reconfigure itself) if there are changes to the file being applied.
Notice the `Service[httpd]: Triggered 'refresh'` in your apply output.

**Why?** PHP reads this file when the Apache web server starts up. That means if
you create the file and put it there nothing happens until next time the Apache
server restarts (or reloads gracefully).

### What did Puppet do under the covers?

Puppet is now watching the file you defined, and if **and only if** there are
changes to that file it will tell Apache to reload gracefully, picking up the
changes.

### Try without Puppet yourself

1. Change `/etc/php.d/php.ini` to some other time zone
2. Reload your web page
3. You will notice that nothing changed
4. Now execute `systemctl restart httpd`
5. Reload the page on the browser
6. Now it shows the correct time zones.

Puppet just did that for you without needing to resort to understanding which
command on the operating system does that (systemctl is for Cent OS). **Other
operating systems uses different commands that you have to know to manage
them.** Note that you also did not need to know all the inner details of Apache
or PHP get this this going.

> *[Pro Tip]* Before you deploy your application for real (what we call
> production) I suggest you do understand better Apache and PHP. How to
> configure it properly: to run fast, secure, reliable. It sucks when you put
> your site and some hacker vandalizes it, or worse, steals all your data.

That's it for now. I hope you got the basis of what Puppet is great at.

Ah... added bonus: these manifests you wrote will run on any Linux operating
system without changes. It is what we call _portable code_.


[PHP]: https://www.php.net
[php-date-timezone]: http://php.net/manual/en/datetime.configuration.php#ini.date.timezone
[php-timezones]: http://php.net/manual/en/timezones.america.php
