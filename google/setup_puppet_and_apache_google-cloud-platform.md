# Puppet & Apache on a Google Cloud Platform machine

* [Get a machine](#get-a-machine)
* [SSH to the machine](#ssh-to-the-machine)
* [Becoming 'root'](#becoming-root)
* [Install Puppet](#install-puppet)
   * [Get Puppet client on your path](#get-puppet-client-on-your-path)
* [Simplest Puppet manifest](#simplest-puppet-manifest)
* [Installing software](#installing-software)
   * [The "hard" way (automated, pero-no-mucho)](#the-hard-way-automated-pero-no-mucho)
   * [Like a boss!](#like-a-boss)
      * [Installing the puppetlabs-apache module](#installing-the-puppetlabs-apache-module)
      * [Turning off warnings for puppetlabs-apache module](#turning-off-warning-for-puppetlabs-apache-module)
      * [Using the puppetlabs-apache module](#using-the-puppetlabs-apache-module)
* [Uninstall and try again (just for fun)](#uninstall-and-try-again-just-for-fun)


## Get a machine

We need a machine to work on :). You can get a machine by using the
[Developer Console][] or the 'gcloud' tool (a.k.a. [Google Cloud SDK][]).

This tutorial is based on Cent OS 7, so make sure your machine is created using
that operating system, or you'll need to change the commands to match your other
OS.

Using the gcloud tool:

```
gcloud compute instances create my-first-app \
  --machine-type=f1-micro --zone=us-central1-a \
  --image-family=centos-7 --image-project=centos-cloud
```

_(If you are typing the command above instead of copying/pasting do not type the
'\\' (backslash) characters at the end of the line. Type everything but the '\\'
in a single line instead)_

This example uses `f1-micro` as the machine type. That's the cheapest machine
available (it is limited in CPU and memory). For these examples it is enough,
but for serving your real application you should use something more robust, at
minimum `n1-standard-1`.

## SSH to the machine

We'll be working inside the machine, so let's SSH to it so we can run commands
on it. If you created with the Developer Console click the SSH button for the
machine we'll be working on.

If you created using gcloud, you can SSH using gcloud tool:

```
gcloud compute ssh my-first-app --zone=us-central1-a
```

## Becoming 'root'

All the commands we'll be running to set the machine are privileged, so we need
to become super user. On Linux that's done with sudo command. Once you become
super user the prompt will change from '$' to a '#'.

Type on your SSH connection:

```
sudo -i
```

The screen should look like this:

```
[nelsona@my-first-app ~]$ sudo -i
[root@my-first-app ~]# 
```

So make sure all commands from now on are typed as super user (note the '#').

## Install Puppet

We'll be using the Puppet Open Source. For more sophisticated and corporate
grade deployments you may want to look into [Puppet Enterprise][] (PE). Don't
look into PE now. That was just FYI and future use.

Run:

```
rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
yum install puppet-agent
```

... and follow the screen prompts.

To confirm Puppet is installed:

```
rpm -qa | grep puppet
```

It should show `puppet-agent` in the system:

```
# rpm -qa | grep puppet
puppetlabs-release-pc1-1.1.0-5.el7.noarch
puppet-agent-1.10.4-1.el7.x86_64
```

### Get Puppet client on your path

As you just installed Puppet it is not yet available on your application path.
The easiest way to get it available is to exit the super user shell and come
back in:

```
[root@my-first-app ~]# puppet
-bash: puppet: command not found
[root@my-first-app ~]# exit
logout
[nelsona@my-first-app ~]$ sudo -i
[root@my-first-app ~]# puppet
See 'puppet help' for help on available puppet subcommands
[root@my-first-app ~]# 
```

## Simplest Puppet manifest

Create a file with the following contents. Let's call it `hello.pp` (`pp` is
Puppet's manifest extension):

```puppet
notify { 'hello':
  message => 'This is my first Puppet app'
}
```

To run a Puppet manifest (without a server) we use `apply`:

```
puppet apply <your-file>
```

In the `hello.pp` case it should have an output like this:

```
[root@my-first-app ~]# puppet apply hello.pp 
Notice: Compiled catalog for
my-first-app.c.graphite-playground.google.com.internal in environment production
in 0.09 seconds
Notice: This is my first Puppet app
Notice: /Stage[main]/Main/Notify[hello]/message: defined 'message' as 'This is
my first Puppet app'
Notice: Applied catalog in 0.01 seconds
[root@my-first-app ~]# 
```

## Installing software

### The "hard" way (automated, pero-no-mucho)

You can tell Puppet to install software on your machine easily. You just declare
a `package` block. For example, to install Apache (web server) on our Cent OS
machine we create a file, say `apache.pp`:

```puppet
package { 'httpd':
  ensure => installed,
}
```

Now we apply the file:

```
[root@my-first-app ~]# puppet apply apache.pp
Notice: Compiled catalog for my-first-app.c.graphite-playground.google.com.internal in environment production in 0.79 seconds
Notice: /Stage[main]/Main/Package[httpd]/ensure: created
Notice: Applied catalog in 3.24 seconds
[root@my-first-app ~]# 
```

Notice the `ensure: created` in the log above. That means that Puppet just
installed Apache on your machine.

Let's confirm that:

```
[root@my-first-app ~]# rpm -qa | grep httpd
httpd-tools-2.4.6-45.el7.centos.4.x86_64
httpd-2.4.6-45.el7.centos.4.x86_64
[root@my-first-app ~]# 
```

*COOL PART:* The coolest part is that Puppet is *not* a scripting language. That
means that you didn't ask Puppet to install Apache. Instead, you asked Puppet to
_ensure Puppet is installed_. That means if Apache is already installed, Puppet
will not do anything to your machine. Let's run it again:

```
[root@my-first-app ~]# puppet apply apache.pp
Notice: Compiled catalog for my-first-app.c.graphite-playground.google.com.internal in environment production in 0.98 seconds
Notice: Applied catalog in 0.12 seconds
[root@my-first-app ~]# 
```

As you can see now Puppet did not do anything.

### Like a boss!

Puppet has an extensive library of modules that you can leverage. Instead of
dealing with things specific to Apache (for example which is the correct package
name for my Operating System [in CentOS is 'httpd' but in Debian is 'apache2']
we can use Puppet's module to hide all that from us.

#### Installing the [`puppetlabs-apache`][] module

We'll download and install a module that will teach Puppet how to deal with
Apache like a boss:

```
puppet module install puppetlabs-apache
```

That should show something like this:

```
[root@my-first-app ~]# puppet module install puppetlabs-apache
Notice: Preparing to install into
/etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-apache (v1.11.0)
  ├── puppetlabs-concat (v2.2.1)
  └── puppetlabs-stdlib (v4.17.1)
[root@my-first-app ~]# 
```

Let's confirm they are all there:

```
puppet module list
```

And it should show:

```
[root@my-first-app ~]# puppet module list
/etc/puppetlabs/code/environments/production/modules
├── puppetlabs-apache (v1.11.0)
├── puppetlabs-concat (v2.2.1)
└── puppetlabs-stdlib (v4.17.1)
/etc/puppetlabs/code/modules (no modules installed)
/opt/puppetlabs/puppet/modules (no modules installed)
[root@my-first-app ~]# 
```

#### Turning off warning for [`puppetlabs-apache`][] module

Background: Puppet is phasing out their old 3.x version and
[puppetlabs-apache][] module did not get converted yet, so lots of warnings
would be printed until they (or the community) update this module.

To avoid having to see those warnings go by we'll turn warnings off. This should
be a temporary action and you should turn it back on as soon as possible.

To be cute we'll use Puppet to configure Puppet. I find it cool a tool can
configure itself :)

```puppet
file_line { 'turn-warnings-off':
  path  => '/etc/puppetlabs/puppet/puppet.conf',
  line  => 'strict = off',
  match => 'strict =',
}
```

As before put the code above in a file, e.g. warn-off.pp, and issue `puppet
apply warn-off.pp` to execute it.

Note: This manifest uses `file_line` which is provided by the
`puppetlabs-stdlib` module, which was installed automatically by the
`pupplabs-apache` module. In case you did not install the Apache module and
still wish to run the manifest above, install the `stdlib` module manually:

```
puppet module install puppetlabs-stdlib
```

Alternatively you could have done the command `puppet config set strict off`,
but you'd need to do on every computer you setup. By putting in the Puppet
manifest you do *not* have to remember any setup steps. That's my preferred
method.

##### Putting warnings back

When you need to put the warnings back you revert the steps you did in this
section. You can either run a command `puppet config set strict warning` or
update your manifest to have `warning` instead of `off`.


#### Using the [`puppetlabs-apache`][] module

Create a new file with the manifest below. Let's call it apache-via-mod.pp:

```puppet
include apache
```

and apply it:

```
puppet apply apache-via-mod.pp
```

Now a million lines will go by. Don't panic, it's okay :) Puppet is catching up
with the module and installing what it needs.

*NOTE* Ignore all the yellow lines. There is a version mismatch on Puppet
modules but that does not affect this tutorial. I filed a bug to track that and
eventually update this ;-) [link to
bug](https://github.com/nelsonjr/puppet-tips/issues/2)

*Nevertheless* you will notice that the `ensure: created` is not in that list,
which means Puppet already detected Apache and did nothing.

*FYI (for later)*: The [puppetlabs-apache][] module is very, very, very well
done and comprehensive. Its full documentation and glory can see here:
[http://forge.puppet.com/puppetlabs-apache](
  http://forge.puppet.com/puppetlabs-apache)


## Uninstall and try again (just for fun)

Just so we can see Puppet in action, let's:

1) Uninstall Apache

	```
	yum remove -y httpd
	```

2) Confirm Apache is not present

	Confirm httpd is not there anymore (same command as you used to verify it was
	there).

3) Run Puppet again

	Now run through the previous section again. Prefer the `puppetlabs-apache`
	instead of the manual this time, as we'll use the module going forward.

4) Confirm Apache is present

	Same as step #2.


# What's Next?  What's the use of all of this?

Now that we have a computer setup with a Web server it is time we serve traffic
to the Internet. Follow the [Simplest Web page on Google Cloud Platform with
Puppet][next] to get it done.



[next]: first_web-app_google-cloud-platform.md
[Puppet Enterprise]: https://puppet.com/product/puppet-enterprise
[modules]: https://forge.puppet.com
[Developer Console]: https://cloud.google.com/console
[Google Cloud SDK]: https://cloud.google.com/sdk
[`puppetlabs-apache`]: https://forge.puppet.com/puppetlabs-apache
[puppetlabs-apache]: https://forge.puppet.com/puppetlabs-apache
