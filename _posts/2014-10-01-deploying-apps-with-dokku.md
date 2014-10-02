---
title: "Deploying Apps With Dokku"
layout: post
published: true
comments: true
categories: []
tags: [dokku clojure cloudsigma]
---

Last weekend I was fortunate enough to participate in the [Clojure
Cup](https://clojurecup.com) 2014 competition.  Clojure Cup is a competition
where teams of 1-4 people build and deploy applications from scratch in 48
hours. The apps must be written in Clojure, ClojureScript, or both. I was in a
team of three representing the [Austin Clojure Meetup
Group](http://www.meetup.com/Austin-Clojure-Meetup). Though the Clojure Cup
rules did not specify anything about your application other than the languages
it had to be written in, it did seem like web applications were the preferred
application type. We decided not to go against the grain and build ourselves a
web app. This post details the process we had to go through to deploy our
Clojure-based web app with Dokku and CloudSigma.

## Dokku Overview

[Dokku](https://github.com/progrium/dokku) is a "Docker powered mini-Heroku".
[Docker](https://www.docker.com/) is a Linux container system for easily
managing processes. [Heroku](https://heroku.com) is a PaaS which allows you to
deploy several different applications easily. Dokku utilizes Docker while
providing the ease of deployment normally found only with Heroku. Please
familiarize yourself with the technology of Docker and the workflow of Heroku
as they will help you better understand Dokku.


## Step 1 - Server Configuration

As participants of Clojure Cup we were provided a free trial to the
[CloudSigma](https://www.cloudsigma.com/) cloud service, which we were required
to use to ensure an equal playing field amongst all applications. Though this
post will focus on some CloudSigma specific configuration steps, Dokku is not
limited to only being used on CloudSigma systems. Dokku can be installed on any
Ubuntu 14.04 or 12.04 64-bit servers. I would advice you to generalize any
CloudSigma specific steps to fit your setup or to skip the server configuration
if you already have a server ready to go.

### CloudSigma Specific Configuration

#### Create New Server

1. [Create a new server](https://zrh.cloudsigma.com/ui/#/servers/create)
2. In Properties, name it **Dokku Test**
3. In Properties, add your public SSH key
4. Save
5. In Drives, click **Attach Drive**, then click **Drive from Marketplace**
6. Search for **Ubuntu 14.04 Server**, then click **Attach Drive**
7. Save
8. [Go to your drives](https://zrh.cloudsigma.com/ui/#/drives)
9. Click on your new Ubuntu Server drive
10. In Properties, name it **Ubuntu 14.04 Sever Dokku Test**
11. In Install Notes, take note of the default username and password for the drive
12. Save
13. [Go back to your servers](https://zrh.cloudsigma.com/ui/#/servers)
14. Click on **Dokku Test**
15. Hit the **Start** button

#### SSH into Server

1. In Properties, take note of the Public IP for your server
2. [Go to your Dokku test drive](https://zrh.cloudsigma.com/ui/#/drives)
3. In Install Notes, take note of the default username and password for the drive
4. SSH into the server and change the user password:

```
$ ssh <drive-username>@<server-public-ip>
```

The server might automatically ask you to change the default password. If it
doesn't, change it manually:

```
$ sudo passwd <drive-username>
```

## Step 2 - Dokku Configuration

### Dokku Installation

Now that the server is ready to go we need to install and configure Dokku on it.

If on Ubuntu 12.04, first install some dependencies:

```
$ sudo apt-get install -y python-software-properties
```

Run the Dokku installer:

```
$ wget -qO- https://raw.github.com/progrium/dokku/v0.2.3/bootstrap.sh | sudo DOKKU_TAG=v0.2.3 bash
```

Check if dokku is installed, if not retry the last step:

```
$ dokku version
```

The Dokku installer has done a few things to bootstrap itself. Some of those things are:

- created a ```dokku``` user
- created a file structure in ```/home/dokku``` to host your apps configurations
- installed several necessary packages like Docker

### Accessing Dokku

Login to server with provided username, not the ```dokku``` user, and provide your SSH key to Dokku:

```
$ cat ~/.ssh/authorized_keys | sudo sshcommand acl-add dokku dokkutest
```

This used an SSH wrapper installed and endorsed by Dokku called
[sshcommand](https://github.com/progrium/sshcommand).

### Domain Configuration

Dokku has created several files under ```/home/dokku``` which are used to configure
the domain from which your apps will be served. If you navigate there you will
find the following files:

```
# tree
.
├── HOSTNAME
├── VERSION
└── VHOST
```

#### HOSTNAME

If you have a domain name you should edit this file so that it contains that
domain name. If you don't have a domain name you should edit the file to have
your server's public IP address.

#### VHOST

##### Subdomain

```
<your-app>.<your-domain>.com
```

If you want the URI of your app to look like that, specifying a ```VHOST``` file
will give provide a subdomain with the name of your app. The ```VHOST``` file
should contain the same thing your ```HOSTNAME``` file contains. **This will only
work with a domain name, it will not work with only an IP address**.

#### Application Port

```
<your-domain>.com:<app-port>
```

or

```
<server-public-ip>:<app-port>
```

If you don't want a subdomain or don't have a domain name your only option is
to use a port number. The ```VHOST``` file should be removed or be empty for
this effect.

## Step 3 - Deploy your Application

Now that both our server and Dokku are fully configured we can now focus on
deploying our application. Because Dokku uses
[Buildstep](https://github.com/progrium/buildstep) it supports deploying many
types of applications for Node.js, Ruby, Python and
[more](https://github.com/progrium/buildstep#supported-buildpacks). For now we
will just deploy the [Heroku Node.js sample
app](https://github.com/heroku/node-js-sample).

Follow the steps on your development machine:

Clone the node-js-sample app:

```
$ git clone https://github.com/heroku/node-js-sample
$ cd node-js-sample
```

Add the remote location to dokku:

```
$ git remote add dokku dokku@<server-public-ip>:node-js-sample
```

Push the application:

```
$ git push dokku master
```

Once it is done pushing and building you should see where your application was
deployed.

## Closing Useful Tips

- Dokku has facilities to support [plugins](https://github.com/progrium/dokku/wiki/Plugins) which provide useful functionality, like easy management of databases.
- Dokku is "a mini-Heroku" so reading some Heroku specific documentation helps with Dokku.
- Web applications, like the sample nodejs app, are expected to be exposed through port 5000 unless an environment variable is specified.
- Explore dokku with ```dokku help```
