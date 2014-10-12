---
date: 2011-06-17 00:02:20+00:00
layout: post
slug: howto-setup-asterisk-pbx-server
title: Howto Setup an Asterisk PBX Server
categories:
- articles
tags:
- asterisk
- howto
- linux
- pbx
- voip
---

### Introduction

Asterisk is software that turns an ordinary computer into a communications
server. Asterisk powers IP PBX systems, VoIP gateways, conference servers and
more.

This howto covers topics and issues installing, configuring and managing
Asterisk server on Fedora Linux version 12 to 14. It is based on Asterisk
1.6.2.


### Installation

I assume you are going to be using Asterisk and its components packages which
Fedora supplies via yum 'updates' repository, which is enabled by default.

Required packages for a simple Asterisk installation. Is is always a good
practice to install sound files for the most common codecs,
so asterisk's core does not have to convert the sound files on the fly -
`asterisk-sounds-core-en-<codec>`

{% highlight bash %}
asterisk
asterisk-sounds-core-en
asterisk-sounds-core-en-alaw
asterisk-sounds-core-en-g722
asterisk-sounds-core-en-g729
asterisk-sounds-core-en-gsm
asterisk-sounds-core-en-ulaw
asterisk-sounds-core-en-wav
asterisk-voicemail # (if you need voicemail support)
iax # (if you need IAX protocol support)
{% endhighlight %}

Packages installation using yum install:

{% highlight bash %}
yum install asterisk asterisk-sounds-core-en asterisk-sounds-core-en-alaw \
  asterisk-sounds-core-en-g722 asterisk-sounds-core-en-g729 \
  asterisk-sounds-core-en-gsm asterisk-sounds-core-en-ulaw \
  asterisk-sounds-core-en-wav asterisk-voicemail iax
{% endhighlight %}

Turn on asterisk service to start on boot

`chkconfig --level 345 asterisk on`


### Pre-configuration

First of all let's start the Asterisk service just to make sure it starts without any errors:
`service asterisk start`

Once the service is started and running you can connect to Asterisk by simply
running `asterisk -r` - the -v option tells the verbosity level of the
Asterisk's core (it can be set via Asterisk's command line), which is optional.
You will get into the Asterisk'a command prompt:

{% highlight bash %}
# asterisk -r
Asterisk 1.6.2.16.1, Copyright (C) 1999 - 2010 Digium, Inc. and others.
Created by Mark Spencer
Asterisk comes with ABSOLUTELY NO WARRANTY; type 'core show warranty' for details.
This is free software, with components licensed under the GNU General Public
License version 2 and other licenses; you are welcome to redistribute it under
certain conditions. Type 'core show license' for details.
=========================================================================
Connected to Asterisk 1.6.2.16.1 currently running on zs (pid = 20283)
Verbosity is at least 10
hostname*CLI>
{% endhighlight %}

The main components are the following:

* `core` - Asterisk's core engine
* `dialplan` - Asterisk's logic which you will have to define
* `sip` - The SIP protocol engine
* `iax2` - The IAX2 protocol engine


### Configuration
#### Config files

The files are stored in /etc/asterisk directory. The main configuration files are:

* `sip.conf` - This is where we'll configure the SIP protocol
* `extensions.conf` - A dialplan configuration file, this is where all the logic we define goes to
* `iax.conf` - This is the IAX2 protocol configuration file


#### Backup existing files

Asterisk packaged installation comes with already populated config files, it
has some good examples, but they are unrelated to what we are going to achieve,
so it is a good idea to backup them using:

{% highlight bash %}
cd /etc/asterisk
mv sip.conf sip.conf.sample; touch sip.conf
mv iax.conf iax.conf.sample; touch iax.conf
mv extensions.conf extensions.conf.sample; touch extensions.conf
{% endhighlight %}


#### SIP Protocol

Since the SIP protocol is the most common one, I will cover its configuration
in this howto. The sip.conf file contains channel and users (phones)
configuration, login details etc. The main channel is [general] which is
reserved for general/default SIP engine configuration. Many options can be set
per channel or trunk channel. A new channel starts with [channel-name] and ends
where the new channel starts.

The basic structure of the sip.conf file is:

{% highlight bash %}
;this is a comment
[general]
context=default ;if no destination is specified the call will go to the default context
allowoverlap=no
bindport=5060 ;port number to bind (default)
bindaddr=0.0.0.0 ;address to bind (all by default)
srvlookup=yes ;do DNS lookups
directmedia=no ;use indirect media (partially solves SIP over NAT issues)
bandwidth=low ;enable most common codecs
disallow=lpc10 ;makes your voice sound like a robot, so we disable this codec :-)

;SIP provider registration
;register = username:secret@sip.provider.tld

;simple configuration for user (extension) 1001
[1001]
type=friend ;type friend allows a user/phone to make and receive calls
host=dynamic ;host is needed when incoming calls comes in, Asterisk will take a note of phone's IP upon registration
secret=super_strong_secret
context=users ;the context for inbound and outbound calls in this case (because type is friend)
callerid="Name Lastname <1001>"
{% endhighlight %}


#### Dial Plan

The main Asterisk PBX logic is configured in extensions.conf file. This file
again is separated by [sections]. The already reserved sections/contexts are
`[globals]` and `[general]`.
`[globals]` is usually empty, but can contain some options or it can be used
for variable assignments. Let's create a basic extensions.conf configuration
file:

{% highlight bash %}
[globals]

[general]
autofallthrough=yes
static=yes

[default] ;create a default context, so if incoming caller did not specify an extension the call will end up in this context
exten => s,1,Verbose(1,Unrouted call handler) ;send a custom message in the log
exten => s,n,Dial(SIP/1001) ;call extension 1001 (remember, we configured 1001 channel in sip.conf)
exten => s,n,Hangup() ;and finally hang up once the call is finished, it is always a safe bet to do that

[users] ;create a users context, this is where configured users/phones will end up (remember? we placed user 1001 in 'users' context)
exten => 1001,1,Dial(SIP/1001)
exten => 1001,n,Hangup()

exten => 5000,1,Echo() ;create a simple echo test on ext 5000
exten => 5000,n,Hangup()
{% endhighlight %}

So now you should have the working configuration files for SIP channels and a
simple dialplan. Let's move on to the Asterisk basic management and control.


#### Asterisk and NAT

Probably the worst case is when an Asterisk PBX is behind one NAT and clients
connecting to the PBX are behind another NAT and network. Asterisk can handle
this situation pretty well if you tell it to. There few steps which need to be
done in order to make Asterisk property translate SIP and RTP data over NAT:

Do port forwarding on the firewall which does NAT for the Asterisk server of the following ports:
`5060/udp` (SIP traffic), `10000-20000/udp` (RTP traffic) - ports range can be
changed on /etc/asterisk/rtp.conf file.

Add the following options to `/etc/asterisk/sip.conf` file:

{% highlight bash %}
[general]
externip=1.1.1.1
localnet=192.168.1.0/255.255.255.0
qualify=yes
canreinvite=no

externip=1.1.1.1 - set it to your external IP
localnet=192.168.1.0/255.255.255.0 - set it to your local subnet (more than one 'localnet' option can be used)
qualify=yes - set it to 'yes', so Asterisk will keep connections open over the NAT
canreinvite=no - set it to no, so Asterisk does not send re-invites (always stays in between the current call)
{% endhighlight %}


### Management

Now we have configured basic components of the Asterisk PBX, but our PBX system
is running off the sample configuration files, remember (we renamed the files
after we have started the Asterisk service)?
I believe you still have the terminal open with a connection to the service
(asterisk -r). Asterisk control is pretty simple, I will list a few of the main
commands just to get you started:

* `core set verbose 10` - increases/decreases verbosity level of the core (10 is the most verbose output)
* `core restart` - restarts the Asterisk core, very rarely needed, unless you are changing core configuration
* `core show channels` - displays active channels/calls and processed calls
* `sip set verbose 10` - increases/decreases verbosity level of the sip engine (10 is the most verbose output)
* `sip set debug on` - sets debugging on of the sip engine
* `sip show users` - shows configured users in sip.conf
* `sip reload` - reloads the sip engine and rereads the sip.conf file
* `dialplan reload` - reloads the dialplan enginer and rereads extensions.conf file
* `core show applications` - shows all the applications which can be used in your dialplan
* `core show functions` - shows all the functions which can be used while building a dialplan

There are many more commands, but I will not cover them all here obviously.


### Final Words

By now you should have a basic fully working PBX system. What you need to do is
to add more users/extensions, make sure you create a dial plan for them too and
configure your phones/softphones and you can start making calls for free.


### Final words

I do apologise if I made some mistakes or typos as I was writing everything out
of my head, I did not have to test this actual setup. If you find any mistakes
please let me know and I will correct that. Otherwise I hope it is going to be useful for somebody.
