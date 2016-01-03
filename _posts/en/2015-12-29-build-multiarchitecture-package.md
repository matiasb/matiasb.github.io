---
layout: post
title: Build multi-architecture snap package
modified:
author: matiasb
categories: [en]
comments: true
excerpt: Fifth post on a series about building Transmission snap package.
tags: [snappy, ubuntu]
image:
  feature:
date: 2015-12-29T20:01:00-03:00
---

*This is part of a [series](/tags/#snappy) describing how I got [Transmission](https://uappexplorer.com/app/transmission.matiasb) compiled and running for Snappy Ubuntu Core.*
{: .notice}

Snap packaging, updated
-----------------------

Since we have a new set of binaries for ARM (for Transmission and its dependencies), we will need to update the snap package. The process will be based on what I described in the [third part of this series](/en/snap-packaging-transmission).

We need to include the ARM binaries in the package:

    armv7l/
        lib/*                   # compiled ARM dependencies/libraries (copied from the deps build)
        transmission-daemon     # ARM binary for the daemon (copied from the transmission build)
    meta/*
    web/*
    x86_64/*
    settings.json
    transmission-daemon         # custom script to run the daemon in snappy

Note the above is the same files we had, plus the `armv7l` directory (armv7l is the value returned by the `uname` command below) containing the ARM compiled binaries and deps.

The wrapper script to run the daemon from our initial package already detected the architecture and chose the right binary to run, so there is nothing to change there:

{% highlight bash %}
#!/bin/bash
ARCH=$(uname -m)
PATH=${SNAP_APP_PATH}/${ARCH}/lib/:$PATH
LD_LIBRARY_PATH=${SNAP_APP_PATH}/${ARCH}/lib/:$LD_LIBRARY_PATH
export PATH LD_LIBRARY_PATH

TRANSMISSION_HOME=${SNAP_APP_DATA_PATH}
TRANSMISSION_WEB_HOME=${SNAP_APP_PATH}/web
export TRANSMISSION_HOME TRANSMISSION_WEB_HOME

DEFAULT_SETTINGS=${SNAP_APP_PATH}/settings.json
SETTINGS_FILE=$TRANSMISSION_HOME/settings.json
if test -f $SETTINGS_FILE;
then echo "Using existent settings";
else cp $DEFAULT_SETTINGS $SETTINGS_FILE; fi

# fire the binary
exec ${SNAP_APP_PATH}/${ARCH}/transmission-daemon -f --logfile "${SNAP_APP_DATA_PATH}/transmission-daemon.log"

# never reach this
exit 1
{% endhighlight %}

Back to snap packaging, we need to list the new supported architecture:

{% highlight yaml %}
name: transmission
icon: meta/transmission.png
vendor: matiasb <xxxxx@xxxx.com>
version: 2.84.2
architectures: [amd64, armhf]
services:
 - name: transmission-daemon
   description: Transmission bittorrent client daemon.
   start: ./transmission-daemon
   caps:
        - networking
        - network-service
        - network-client
   ports:
      external:
         ui:
            port: 9091/tcp
            negotiable: no
         peers_listening:
            port: 51413/tcp
            negotiable: no
{% endhighlight %}

Finally, we just need to rebuild our package:

{% highlight bash %}
$ snappy build
Generated 'transmission_2.84.2_multi.snap' snap
{% endhighlight %}


Coming next
-----------

And that's it, we have a multi-architecture Transmission snap package! You can install this same package in amd64 and ARM architectures, indistinctly.

Next steps will be:

* Build Transmission using snapcraft
