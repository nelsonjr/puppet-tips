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

```
rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
yum install puppet-agent
```


[Puppet Enterprise]: https://puppet.com/product/puppet-enterprise
[modules]: https://forge.puppet.com
[Developer Console]: https://cloud.google.com/console
[Google Cloud SDK]: https://cloud.google.com/sdk
