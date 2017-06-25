# Setting up PHP

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
Notice: /Stage[main]/Main/File[/var/www/html/index.php]/ensure: defined content as '{md5}28d8c2a602a204496942c2d77989035e'
Notice: Applied catalog in 0.67 seconds
[root@my-first-app ~]#
```

Now refresh your browser and you should see something like this:

```
Now it is Sun, 25 Jun 2017 07:38:19 +0000
```


[PHP]: https://www.php.net
