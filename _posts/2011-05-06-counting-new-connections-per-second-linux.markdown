---
date: 2011-05-06 00:55:11+00:00
layout: post
slug: counting-new-connections-per-second-linux
comments: true
title: Counting NEW Connections per Second on a Linux Firewall
categories:
- linux
- networking
- articles
tags:
- conntrack
- iptables
- linux
- netfilter
- networking
---

Thought I will write a tiny post about how to easily monitor NEW (or any other state)
connections per second on a Linux firewall. The approach I have chosen seems to be really
easy and simple one-liner.

Kernel modules and packages you are going to need:

  * ip_conntrack iptables kernel module loaded or compiled in to the kernel
  * conntrack-tools package installed
  * libnetfilter_conntrack package installed
  * pv (if not installed already) package installed


Depending on your distribution (I have tested it on Fedora 14 and Centos5.5 and 5.6),
obviously Fedora has the above two packages in its repository, but for example Centos does
not, so if you use Centos, you can get them from:
[http://centos.alt.ru/pub/conntrack-tools/0.9.15/RHEL/RPMS/](http://centos.alt.ru/pub/conntrack-tools/0.9.15/RHEL/RPMS/).
The `pv` package is available from Fedora repository, but Centos does not have it, so you might need
to add epel repo or just get the RPM from epel repo online (most people have epel repo configured already).

Once you have got required module and libraries in place, then just simply run:

{% highlight bash %}
conntrack -E -e NEW | pv -l -i 1 -r > /dev/null
{% endhighlight %}

The self updating output should look similar like the one below:

`[   50/s ]`

A little explanation of the command line above:
	
  * `conntrack -E -e NEW` - display a real-time event log with event-mask 'NEW'
  * `pv -l -i 1 -r` - pv is a pipe viewer `-l` turns the line mode for counting lines instead of bytes,
waits 1 second between updates (`-i 1`) and `-r` turns the rate counter on
  * `> /dev/null` - redirects the output from `conntrack -E -e NEW` to `/dev/null` at the end

I find it a very simple 'one-liner' which comes in handy sometimes when I want to quickly count the NEW connections per second my firewalls are dealing with.

If you know better or other ways of doing it, please post that in the comments section below.
