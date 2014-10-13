---
date: 2011-10-26 15:50:47+00:00
layout: post
slug: starting-kickstart-installation-via-grub
comments: true
title: Starting Kickstart installation via GRUB
categories:
- kickstart
- linux
- articles
tags:
- centos
- centos6
- grub
- installation
- kickstart
- linux
- pxe
- rebuild
---

## Intro

How many of our managed servers have PXE boot capable NICs? Who really wants to
move his ass and crawl to a data centre to manually build a server? Trust me -
there are people who really enjoy doing that. But I do not, and whoever is
reading this post does not either I guess.


## Doing stuff

I will share with you a nice little trick which can be useful if you cannot
boot off your servers of a PXE and start a kickstart installation. Please note
I am going to be installing CentOS 6.0. You can start kickstart installation


### Prerequisites
* An actual server/workstation you want to rebuild
* A pretty much any working Linux distribution on the server/workstation you're
  rebuilding which you have root access to
* A working network and DHCP server (you can use static IPs - requires
  additional kernel parameters)
* A common sense (the most important part)


### Download kernel and initrd images

{% highlight bash %}
# cd /boot
# wget http://mirror.centos.org/centos-6/6/os/x86_64/images/pxeboot/vmlinuz
# wget http://mirror.centos.org/centos-6/6/os/x86_64/images/pxeboot/initrd.img
{% endhighlight %}


### Edit grub.conf

{% highlight bash %}
default=0
title CentOS Linux PXE install
        root (hd0,0)
        kernel /vmlinuz ks="http://repo.server.com/ks/server_kickstart_config.cfg" ksdevice=eth0 vnc
        initrd /initrd.img
{% endhighlight %}

Notice I pulled the kickstart config from a local web server, you can use
either http, nfs etc. Make sure the above new entry in your `grub.conf`
represents `default=0` position number.

Save the config and restart the box. The installation will start a VNC server
on your box which you can connect to. If you have access to DHCP server, then
it's easy to get the IP which was assigned to the box you're rebuilding,
otherwise you can tell the installation to connect to your client box.

Have fun :-)
