---
layout: post
title: Getting Started With Snappy
modified:
author: matiasb
categories: [en]
comments: true
excerpt: First post on a series about building Transmission snap package.
tags: [snappy, ubuntu]
image:
  feature:
date: 2015-10-20T21:36:19-03:00
---

*This is part of a series describing how I got [Transmission](https://uappexplorer.com/app/transmission.matiasb) compiled and running for Snappy Ubuntu Core.*
{: .notice}


What is Snappy Ubuntu Core?
---------------------------

Snappy Ubuntu Core[^1] is the Ubuntu version for clouds and devices, featuring transactional updates on a minimal server image.

One of the first noticeable differences from the regular Ubuntu version is that there is no apt-* commands and you cannot install traditional `.deb` packages; you manage your applications through the `snappy` utility that lets you install and update snap packages.

Snap packages are the base for Ubuntu Core to provide stronger security guarantees (applications run confined using AppArmor), and it should be relatively easy to build (or port) your app and make it available in the Snappy store.

[^1]: https://developer.ubuntu.com/en/snappy/


Running Ubuntu Core in Virtual Box
----------------------------------

Let's get our first contact with a running Ubuntu Core!

One quick way to get Ubuntu Core running for a first time and make the first experiments is to download an OVA container and plug it in using VirtualBox (or [other capable hypervisor](https://developer.ubuntu.com/en/snappy/start/#ova)).

You can [download 15.04 from here](http://cloud-images.ubuntu.com/ubuntu-core/15.04/core/stable/current/core-stable-amd64-cloud.ova). Then, you need to [import the image into VirtualBox](http://docs.oracle.com/cd/E26217_01/E26796/html/qs-import-vm.html). Finally, in order to use the OVA images you need to [prepare a Cloud-Config ISO](https://developer.ubuntu.com/en/snappy/start/#ova).

At this point you should be able to start your Ubuntu Core setup and [try the Snappy tour](https://developer.ubuntu.com/en/snappy/tutorials/using-snappy/)!

Since we would like to access installed snaps and push our own, it is a good idea to enable network access to our Ubuntu Core machine. I just enabled the `Bridged Adapter` as `lxcbr0` in *Network settings* (you could also set up port forwarding). That will allow you to get *WebDM* in your browser at *\<machine-ip\>:4200*, as well as your other installed packages' exposed UIs or ssh from your terminal.


Installing tools and examples
-----------------------------

To build our snaps we'll need to install some tools in our host:

{% highlight bash %}
$ sudo add-apt-repository ppa:snappy-dev/tools
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install snappy-tools bzr
{% endhighlight %}

After this you should have the snappy utilities in your system. Try for example:

{% highlight bash %}
$ snappy --help
{% endhighlight %}

You can also branch the Snappy examples from Launchpad to get a first sight of how a snap package looks like:

{% highlight bash %}
$ bzr branch lp:~snappy-dev/snappy-hub/snappy-examples
{% endhighlight %}

More on package format later, when we build our snap from scratch.


Completing the cycle
--------------------

To get a taste of the full experience, let's build one of the examples and push it to our running instance.

{% highlight bash %}
$ cd snappy-examples/hello-world
$ snappy build
Generated 'hello-world_1.0.18_all.snap' snap
$ snappy-remote --url=ssh://10.0.3.202 install hello-world_1.0.18_all.snap
=======================================================

Installing hello-world_1.0.18_all.snap from local environment
Installing /tmp/hello-world_1.0.18_all.snap
2015/10/31 16:55:25.742623 verify.go:85: Signature check failed, but installing anyway as requested
Name          Date       Version      Developer 
ubuntu-core   2015-09-17 5            ubuntu    
hello-world   2015-10-31 IESTYKQfgHgH sideload  
webdm         2015-06-14 0.9          ubuntu  
generic-amd64 2015-06-05 1.1.1        ubuntu  

=======================================================

{% endhighlight %}

(use your VirtualBox machine IP address instead of 10.0.3.202, of course).

The signature check is expected to fail because we are manually installing the package and not getting it from the store. Also note that the package is marked as `sideload`.

If you now check, in your Ubuntu Core instance, you should have `hello-world` binaries available:

{% highlight bash %}
(amd64)ubuntu@ubuntu-snappy:~$ hello-world.
hello-world.echo     hello-world.env      hello-world.evil     hello-world.sh       hello-world.showdev  hello-world.usehw    
{% endhighlight %}

Coming next
-----------

And that's it, we have our initial development environment setup. Next steps will be:

* [Build Transmission from source]({% post_url en/2015-10-30-building-transmission-from-source %})
* Build initial snap package
* Build Transmission for ARM (using Raspberry 2 setup)
* Build multi-architecture snap package
* Build Transmission using snapcraft
