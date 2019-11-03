---

title: Building Transmission from source
categories: [en]
excerpt: Second post on a series about building Transmission snap package.
tags: [snappy, ubuntu]
date: 2015-10-30T21:36:19-03:00
---

*This is part of a [series](/tags/#snappy) describing how I got [Transmission](https://uappexplorer.com/app/transmission.matiasb) compiled and running for Snappy Ubuntu Core.*
{: .notice}


Transmission
------------

Transmission[^1] is a open-source, cross-platform bittorrent client. There are multiple remote clients you can use to connect to a running transmission daemon (through its RPC API) to manage your downloads.

Our goal, compile and create a snap package for the latest stable release of Transmission (2.84), particularly, its daemon.

[^1]: http://www.transmissionbt.com/


Building Transmission
---------------------

To build Transmission (written in C) we will need a few tools installed first; there are also some dependencies/libraries that we have to compile in order to get the final binaries. We will need to provide those compiled libraries in our snap package too (because our application will run isolated in Ubuntu Core).

I will be running all this building steps in my host computer, running Ubuntu 64 bits. This means I'll be getting amd64 binaries as a result (which means, after packaging we will have an amd64 snap package, runnable in our Snappy VM). In a future post I will hopefully describe how to build Transmission for ARM, and how to package it in a multi-architecture snap.

### Installing required tools

{% highlight bash %}
$ sudo apt-get install install build-essential pkg-config autoconf automake
{% endhighlight %}

### Getting dependencies and Transmission sources

To build Transmission we will need the following libraries:

* [libevent](http://libevent.org/) (I used 2.0.22-stable)
* [zlib](http://zlib.net/) (1.2.8)
* [openssl](http://openssl.org/) (1.0.2d)
* [libcurl](http://curl.haxx.se/) (7.45)

The latest versions of each should work too.

### Compiling dependencies

I created a directory where I'm going to download sources (deps), and another where compiled libraries will be installed:

{% highlight bash %}
$ mkdir deps
$ mkdir build
{% endhighlight %}

Let's download and compile dependencies:

{% highlight bash %}
$ cd deps
{% endhighlight %}


#### libevent

{% highlight bash %}
$ wget https://github.com/libevent/libevent/releases/download/release-2.0.22-stable/libevent-2.0.22-stable.tar.gz
$ tar xvfz libevent-2.0.22-stable.tar.gz
$ cd libevent-2.0.22-stable
$ ./configure --prefix=/home/matiasb/projects/snappy/build
$ make
$ make install
$ cd ..
{% endhighlight %}


#### zlib

{% highlight bash %}
$ wget http://zlib.net/zlib-1.2.8.tar.gz
$ tar xvfz zlib-1.2.8.tar.gz
$ cd zlib-1.2.8
$ ./configure --prefix=/home/matiasb/projects/snappy/build
$ make
$ make install
$ cd ..
{% endhighlight %}


#### openssl

{% highlight bash %}
$ wget http://openssl.org/source/openssl-1.0.2d.tar.gz
$ tar xvfz openssl-1.0.2d.tar.gz
$ cd openssl-1.0.2d
$ ./config --prefix="/home/matiasb/projects/snappy/build" shared zlib-dynamic -I"/home/matiasb/projects/snappy/build/include" 
$ make
$ make install
$ cd ..
{% endhighlight %}

(note we compile against our recently compiled zlib library)

#### libcurl

{% highlight bash %}
$ wget http://curl.haxx.se/download/curl-7.45.0.tar.gz
$ tar xvfz curl-7.45.0.tar.gz
$ cd curl-7.45.0
$ ./configure --prefix="/home/matiasb/projects/snappy/build" --with-zlib="/home/matiasb/projects/snappy/build/" --with-ssl="/home/matiasb/projects/snappy/build"
$ make
$ make install
$ cd ..
{% endhighlight %}

(note we compile against our recently compiled zlib and openssl libraries)

### Compiling Transmission

We will compile Transmission, daemon only, against our just compiled libraries.

{% highlight bash %}
$ wget http://download.transmissionbt.com/files/transmission-2.84.tar.xz
$ tar xvfJ transmission-2.84.tar.xz
$ cd transmission-2.84
$ ./configure --prefix=/home/matiasb/projects/snappy/build/transmission --without-gtk --disable-libnotify --disable-mac --disable-wx --disable-beos --enable-utp --disable-nls --enable-inotify --enable-lightweight --disable-cli --enable-daemon --with-zlib=/home/matiasb/projects/snappy/build PKG_CONFIG=/usr/bin/pkg-config PKG_CONFIG_PATH=/home/matiasb/projects/snappy/build/lib/pkgconfig CPPFLAGS=-DTR_EMBEDDED
$ make
$ make install
$ cd ..
{% endhighlight %}

If everything went ok, you should have your Transmission binaries in the build/transmission/bin directory.

To give it a try you could do:

{% highlight bash %}
$ cd ../build/transmission/bin
$ export LD_LIBRARY_PATH=/home/matiasb/projects/snappy/build/lib
$ ./transmission-daemon -f
{% endhighlight %}


Coming next
-----------

And that's it, we have transmission daemon compiled and running, completely isolated from system dependencies. Next steps will be:

* [Manually snap packaging Transmission](/en/snap-packaging-transmission)
* [Build Transmission for ARM](/en/build-transmission-arm/)
* [Build multi-architecture snap package](/en/build-multiarchitecture-package/)
* Build Transmission using snapcraft
