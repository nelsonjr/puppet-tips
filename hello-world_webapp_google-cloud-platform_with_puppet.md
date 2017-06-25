# Installing Puppet on a Google Cloud Platform machine

## Why Puppet?

Puppet helps automate deployment, and has a vast amount of [modules][] with
deployment scripts ready for use. We'll leverage those modules to create a
simple, yet easy to maintain and reliable deployment setup.

## Step 1: Get a machine

We need a machine to work on :). You can get a machine by using the Developer
Console or the 'gcloud' tool (a.k.a. Google Cloud SDK).

This tutorial is based on Cent OS 7, so make sure your machine is created using
that operating system, or you'll need to change the commands to match your other
OS.

Using the gcloud tool:

```
gcloud compute instances create my-first-app \
  --zone=us-central1-a --image-family=centos-7 --image-project=centos-cloud
```

## Step 2: SSH to the machine

We'll be working inside the machine, so let's SSH to it so we can run commands
on it. If you created with the Developer Console click the SSH button for the
machine we'll be working on.

If you created using gcloud, you can

```
gcloud compute ssh my-first-app --zone=us-central1-a
```

## Step 3: Install Puppet

We'll be using the Puppet Open Source. For more sophisticated and corporate
grade deployments you may want to look into [Puppet Enterprise][] 


[Puppet Enterprise]: https://puppet.com/product/puppet-enterprise
[modules]: https://forge.puppet.com
