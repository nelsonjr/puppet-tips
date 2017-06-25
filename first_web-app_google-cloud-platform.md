# Simplest Web page on Google Cloud Platform with Puppet

You need to have Puppet and Apache installed. Follow the steps on
setup [Setup Puppet and Apache][] before continuing.

## Creating your first page

Create a Puppet manifest, say coming-soon.pp with:

```puppet
include apache

file { '/var/www/html/index.html':
  ensure  => present,
  content => 'My first page served. Full app coming soon!',
}
```

and apply it:

```
[root@my-first-app ~]# puppet apply coming-soon.pp 
Notice: Compiled catalog for my-first-app.c.graphite-playground.google.com.internal in environment production in 1.43 seconds
Notice: /Stage[main]/Main/File[/var/www/html/index.html]/ensure: defined content as '{md5}4fc3570416d1d43fbccb01c7bfd67a27'
Notice: Applied catalog in 0.65 seconds
```

## Opening Firewall

For security Google Cloud Platform built-in firewall blocks all access to your
machines. But that means nobody can see your site, so we need to "poke a hole"
on the firewall:

```
gcloud compute instances add-tags my-first-app --tags http-server \
  --zone=us-central1-a
```

The special `http-server` tag will tell Google that your machine is a web server
and the default port should be allowed in.

### Using Developer Console

If you don't have the [gcloud][] tool you can use the Developer Console for this
step:

1) Click on the machine
2) Click Edit
3) Check the [x] Allow HTTP traffic
4) Save

## Get your machine IP address

The `External IP` shows in the Developer Console. After the command above
completes you can refresh the page and click on it. You can put that on your
browser and see a blank page.

If you're using the [gcloud][] tool you can execute:

```
gcloud compute instances list my-first-app --zone=us-central1-a
```
Take the `EXTERNAL_IP` and put on your browser.

## See your page

You should see your page with whatever you put in the `content` section of the
Puppet file.

## Edit your content (just for fun)

1) Go back and change your Puppet file `content` to something else
2) Apply the file again
3) Refresh your browser

You should see your new content.


[Puppet and Apache on GCP]: 
[Setup Puppet and Apache]: setup_puppet_and_apache_google-cloud-platform.md
