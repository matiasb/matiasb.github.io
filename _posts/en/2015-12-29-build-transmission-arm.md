---
layout: post
title: Build Transmission for ARM
modified:
author: matiasb
categories: [en]
comments: true
excerpt: Fourth post on a series about building Transmission snap package.
tags: [snappy, ubuntu]
image:
  feature:
date: 2015-12-29T20:00:00-03:00
---

*This is part of a [series](/tags/#snappy) describing how I got [Transmission](https://uappexplorer.com/app/transmission.matiasb) compiled and running for Snappy Ubuntu Core.*
{: .notice}


#### Snappy in a Raspberry Pi 2

You can get Snappy Ubuntu Core running in a Raspberry Pi 2 by following the instructions here:
[https://developer.ubuntu.com/en/snappy/start/raspberry-pi-2/](https://developer.ubuntu.com/en/snappy/start/raspberry-pi-2/)

Our plan will be to compile Transmission from source there, and get our ARM binaries.

As mentioned in the [first post](/en/getting-started-with-snappy) of this series, we don't have the usual tools and packages available in Snappy. We could compile the ARM binaries using something different, but let's see how we can get a "classic Ubuntu" inside Snappy itself through LXD.

### What is LXD?

[LXD](https://linuxcontainers.org/lxd/) is a container "hypervisor", built on top of LXC, and it is available in the Snappy store. Then, the idea is to create a container for an Ubuntu 14.04 LTS running in Snappy where we can build Transmission as we did it before.

Assuming you are already logged in in your RPi2 Snappy instance, to get LXD installed:

{% highlight bash %}
(RaspberryPi2)ubuntu@localhost:~$ sudo snappy install lxd
Installing lxd
Starting download of lxd
22.32 MB / 22.32 MB [=============================================================================================================] 100.00 % 1.11 MB/s 
Done
Starting download of icon for package
24.52 KB / 24.52 KB [============================================================================================================] 100.00 % 93.94 KB/s 
Done
Name        Date       Version Developer 
ubuntu-core 2015-12-09 4       ubuntu    
lxd         2016-01-03 0.21-1  stgraber  
webdm       2016-01-03 0.11    sideload  
pi2         2015-11-13 0.16    canonical
{% endhighlight %}

You can get familiar with LXD checking its [documentation](https://linuxcontainers.org/lxd/getting-started-cli/).

### Compiling Transmission in Snappy on ARM using Ubuntu 14.04 container

First, we will create our container:

{% highlight bash %}
(RaspberryPi2)ubuntu@localhost:~$ lxc remote add images images.linuxcontainers.org
Generating a client certificate. This may take a minute...
If this is your first run, you will need to import images using the 'lxd-images' script.
For example: 'lxd-images import ubuntu --alias ubuntu'.

(RaspberryPi2)ubuntu@localhost:~$ lxc launch images:ubuntu/trusty/armhf trusty
Creating trusty done.
Starting trusty done.

(RaspberryPi2)ubuntu@localhost:~$ lxc exec trusty bash
root@trusty:~#
{% endhighlight %}

There we are, inside a full Ubuntu 14.04 LTS inside Snappy!

At this point we can replicate the steps described in [Build Transmission from source](/en/building-transmission-from-source) (we may need to install wget: `apt-get install wget`). All dependencies, and Transmission itself, are ready to be compiled in ARM architecture.

It will take a little longer to compile, but we will get our ARM binaries that will allow us to extend our snap package to support a new architecture.


Coming next
-----------

And that's it, we have compiled Transmission for ARM. Next steps will be:

* [Build multi-architecture snap package](/en/build-multiarchitecture-package/)
* Build Transmission using snapcraft
