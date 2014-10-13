---
date: 2013-01-31 01:55:39+00:00
layout: post
slug: making-ubuntu-a-better-place
comments: true
title: Making Ubuntu a Better Place
categories:
  - linux
  - rants
  - ubuntu
  - articles
tags:
  - linux
  - rants
  - tips
  - ubuntu
---

I will try to be open minded here as much as I can. I am a huge fan of
rpm-based Linux distributions, such as Fedora and CentOS/RHEL. They are super
clean, stable, *predictable*. Usability is great too.

I don't care about care that Fedora comes with Gnome3 by default and some say
that Gnome3 sucks is unstable and usability is terrible, but I like it. All I
need from desktop environment is to be able to quickly launch my favourite
applications, gnome-terminal, Google chrome, xchat and audacious without
touching my mouse. Gnome3 does that extremely well.

But I am here to talk not about desktop environments. Let's talk about running
Ubuntu Linux on servers, either in the cloud, physical boxes or on your
workstation as virtual machines.

I have to admit, I am not familiar with Ubuntu/Debian Linux distributions as
much as I am with CentOS or Fedora. I have been running the latter two on
servers for quite a few years and they are perfect.

So why Ubuntu Linux, if I love rpm-based distros and they're so perfect? Well,
things surrounding me and requirements change and that is a good thing. They
say that sysadmins or operations people in general hate when stuff, which has
been working for ages, change, but I love it.


> If you don't like something, change it and if you cannot change it, then
> change the way to think about it.
>
> -- Mary Engelbreit


In the very near future I will manage Ubuntu servers across the globe for a
very cool startup in London, so I have started looking into Ubuntu more closely
and with much more open mind than ever before. My first impression was "WTF?
Who built this damn thing?"

Let me rant about Ubuntu disastrous parts, what's wrong about it and how to fix
it.

My journey started a week or so ago, first I span up an Ubuntu 12.04 Vagrant
box and started looking into the operating system. What interest me is the
kernel, packaging system, service management and other bits.

The out of the box kernel build didn't seem that much different, compare to
CentOS 6.3 for instance, so I am not worried about that for now.

The next thing which is important to me is packaging system. First there is a
dozen different commands to do basic package management, like apt-get to
install packages, apt-cache to search, dpkg to do other stuff and so on. Why
cannot be there a simple tool installed by default? Why do I have to install
some tools like aptitude just to make it more usable. The next disastrous
element of packaging system is services being started automatically after a
package is installed. Why in the hell would anyone want that? Please if anyone
knows a good reason, tell me, but don't waste your efforts by saying bullshit
like "oh it's really convenient, you don't have to do it after blah blah". That
might be okay for noobs or people with [Hemispatial
Neglect](https://en.wikipedia.org/wiki/Hemispatial_neglect) syndrome.

More ranting about packaging. This thing is so annoying that I don't even want
to think about it. It is this all interactiveness during package installation.
Come on, please stop making packages that suck and make my terminal pink or
whatever is the colour of that stupid terminal-based prompt. What's more funny
is that if I  turn this interactiveness off, some packages fail to install. WTF
Ubuntu?

Services. Cannonical did a great job for developing Upstart, a SysV init system
replacement, but they forgot one simple thing - a tool which allows you simply
do "<insert_tool_name_here> sshd off". Or in other words - easily
enable/disabled services startup on boot. Instead what they did was "invented"
a gazillion different ways to achieve that, but none of them is as simple as
just a single command. But seems that Cannonical is planing to move to systemd
in the future, so that's going to be solved hopefully.

Of course, there are many other minor things which makes Ubuntu a strange place
to me, but that's probably just a personal preference, so I will not go into
details.

Ubuntu is the most popular Linux distribution after all and I think it deserves
it. Cannonical marketing is pushing Ubuntu really hard out into the wild. There
is a huge and growing user base and it's growing. But most importantly, there
is an LTS edition, for people who need to run the internet :-)

So think about it, are there any alternative to Ubuntu really (I am not talking
about fat-rich corporations)? Fedora - it's a great distribution, especially
for developers, backed by a very successful company, but it lacks long term
support. It comes out every half year and its lifespan is very short. CentOS -
it's great too, but there have been some issues with "cloning" RHEL 6.x major
release, it took CentOS people quite a long time to do it also it's based off
of a commercial OS, which means one can never know when Redhat is going to
close the tap. They already made some changes to the way rhel kernel is
packaged and released as a source RPM, just to screw Oracle, but at the same
time they screwed others too. There is Suse Linux which I do not know much
about, seems that Germans are crazy about it so ask them.

What is left at the end is Ubuntu. Despite the fact that there a number of
freakingly wrong things about it, I am going to use and give it more love. I am
sure I can adapt pretty quickly and work around some annoyances.

Here are some tips for people who feel the same way how to "fix" some of the
issues I talked above.

* Prevent services from starting post package install. Create /usr/sbin/policy-rc.d file with content and make it executible.

{% highlight bash %}
# cat /usr/sbin/policy-rc.d
#!/bin/bash
exit 101
{% endhighlight %}
	
* Disable interactive packages installation (if you know a way how to configure
  this without being prompted for a selection, let me know). Run the below
command and select `noninteractive` and then `critical`, which means that it
will only prompt you for the most life-critical input.

{% highlight bash %}
# dpkg-reconfigure debconf
{% endhighlight %}

If you know a tool which would allow me to enable/disable service startup on
boot, also if you have some cool tips how to make Ubuntu more
sysadmin-friendly, let us know in the comments below.

And finally, trolls and flame warriors, don't waste your time on flame wars
here, no one gives a shit about it.
