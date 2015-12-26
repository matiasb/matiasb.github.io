---
layout: post
title: Manually snap packaging Transmission
modified:
author: matiasb
categories: [en]
comments: true
excerpt: Third post on a series about building Transmission snap package.
tags: [snappy, ubuntu]
image:
  feature:
date: 2015-12-26T21:36:19-03:00
---

*This is part of a [series](/tags/#snappy) describing how I got [Transmission](https://uappexplorer.com/app/transmission.matiasb) compiled and running for Snappy Ubuntu Core.*
{: .notice}


Snap packaging
--------------

Having Transmission compiled binaries, we can now build our first snap package[^1]. For that we will need the following extra files: `package.yaml` (for the package metadata) and `readme.md`.

[^1]: https://developer.ubuntu.com/en/snappy/guides/packaging-format-apps/

The package directory/files structure will be:

    meta/
        package.yaml
        readme.md
        transmission.png
    web/*                       # HTML/static assets for the web UI (copied from the transmission build)
    x86_64/
        lib/*                   # compiled dependencies/libraries (copied from the deps build)
        transmission-daemon     # binary for the daemon (copied from the transmission build)
    settings.json
    transmission-daemon         # custom script to run the daemon in snappy

Note the above is just how I decided to organize it, but you can change/adapt as desired. The only required bits are the meta directory and the `package.yaml` and `readme.md` files in there.

A wrapper script to run the daemon is added to define custom environment variables[^2][^3] to run Transmission daemon taking into account Snappy filesystem structure[^4]. Also, a custom default settings file is provided that enables access to web UI from the local network.

If you take a look, this script will also handle multiple architecture binaries (that will be useful once we compile Transmission for ARM, for example).

[^2]: https://trac.transmissionbt.com/wiki/EnvironmentVariables
[^3]: https://developer.ubuntu.com/en/snappy/guides/security-policy/
[^4]: https://developer.ubuntu.com/en/snappy/guides/filesystem-layout/

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

Back to snap packaging, the minimal `package.yaml`[^5] for our Transmission package could look something like:

{% highlight yaml %}
name: transmission
icon: meta/transmission.png
vendor: matiasb <xxxxx@xxxx.com>
version: 2.84.2
architectures: [amd64]
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

[^5]: https://developer.ubuntu.com/en/snappy/guides/package-metadata/

As you can see we are providing our package metadata (icon, version, architectures, etc) and describing how the service is run.

Since the daemon will act as a server and a client, and require access to networking capabilities, we set the respective `caps` property. We also declare the `ports` we will need to be open, for the web UI (under the `ui` label, convention used by webdm to link to the web app) and for the (optional) peers listening.

To build our package, in the package root directory, run:

{% highlight bash %}
$ snappy build
Generated 'transmission_2.84.2_amd64.snap' snap
{% endhighlight %}

If everything went ok, you can push it to your Snappy instance as we see in the first post:

{% highlight bash %}
$ snappy-remote --url=ssh://10.0.3.202 install transmission_2.84.2_amd64.snap
=======================================================

Installing transmission_2.84.2_multi.snap from local environment
Installing /tmp/transmission_2.84.2_multi.snap
2015/12/26 20:02:25.892810 verify.go:85: Signature check failed, but installing anyway as requested
Name          Date       Version      Developer 
ubuntu-core   2015-09-17 5            ubuntu    
transmission  2015-12-26 IIdOOefXdbVe sideload  
webdm         2015-06-14 0.9          ubuntu    
generic-amd64 2015-06-05 1.1.1        ubuntu    

=======================================================

{% endhighlight %}


### Security issues

We have Transmission daemon running, but you will notice a few issues: the web UI freezes, and checking the logs [^6] you will see apparmor denials when trying to access mounts and when attempting to do `quotactl` syscalls.

[^6]: see Debugging section, https://developer.ubuntu.com/en/snappy/guides/security-policy/

This is because our package runs confined, and access to some resources is not allowed by default, for security reasons. You could customize apparmor and security profiles to get access to those resources[^7], but this will trigger a manual review when uploading the package to the store and it will likely be rejected, since you will be breaking the default confinement. So, instead, let's try to find a work around.

[^7]: https://developer.ubuntu.com/en/snappy/guides/security-policy/

Checking Transmission source code (this file: `libtransmission/platform-quota.c`), you will notice that `mounts` and `quotactl` calls are related to the free space checks the daemon does. You can also see that there are different implementations for different platforms, guarded by compilation flags. Then, we can tweak our compilation flags to use a different implementation, particularly the one based on `statvfs`, which works without any extra requirement in Snappy, and so it won't require a custom profile and/or trigger a manual review.

After re-compiling with those changes, re-packaging and pushing to our Snappy instance, everything works as expected and there is no error in the logs!


### Exposing other binaries

If you want, you could also provide extra binaries (and/or services) in your package. For example, we could also include and provide binaries for the extra Transmission tools, like `transmission-remote`.

For that, we will be updating our `package.yaml` and add the required extra files: the compiled binary and a wrapper script:

    meta/
        package.yaml
        readme.md
        transmission.png
    web/*
    x86_64/
        lib/*
        transmission-daemon
        transmission-remote     # binary for the remote tool (copied from the transmission build)
    settings.json
    transmission-daemon
    transmission-remote         # script to run remote, similar (simplified) to the one for daemon

In the `package.yaml` description we add a new `binaries` keyword where we list our new executable entry:

{% highlight yaml %}
name: transmission
icon: meta/transmission.png
vendor: matiasb <xxxxx@xxxx.com>
version: 2.84.2
architectures: [amd64]
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
binaries:
 - name: remote
   exec: transmission-remote
   caps:
     - networking
     - network-client
{% endhighlight %}


Repackaging and pushing to our instance will make the `transmission.remote` executable available.


Coming next
-----------

And that's it, we have a working Transmission snap package for amd64. Next steps will be:


* Build transmission for ARM (using Raspberry 2 setup)
* Build multi-architecture snap package
* Build transmission using snapcraft
