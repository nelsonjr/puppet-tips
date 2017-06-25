# Installing Puppet on a Google Cloud Platform machine

## Why Puppet?

Puppet helps automate deployment, and has a vast amount of [modules][] with
deployment scripts ready for use. We'll leverage those modules to create a
simple, yet easy to maintain and reliable deployment setup.

## Get a machine

We need a machine to work on :). You can get a machine by using the
[Developer Console][] or the 'gcloud' tool (a.k.a. [Google Cloud SDK][]).

This tutorial is based on Cent OS 7, so make sure your machine is created using
that operating system, or you'll need to change the commands to match your other
OS.

Using the gcloud tool:

```
gcloud compute instances create my-first-app \
  --zone=us-central1-a --image-family=centos-7 --image-project=centos-cloud
```

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

```
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


[Puppet Enterprise]: https://puppet.com/product/puppet-enterprise
[modules]: https://forge.puppet.com
[Developer Console]: https://cloud.google.com/console
[Google Cloud SDK]: https://cloud.google.com/sdk
