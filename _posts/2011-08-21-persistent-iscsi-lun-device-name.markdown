---
date: 2011-08-21 22:11:45+00:00
layout: post
slug: persistent-iscsi-lun-device-name
title: Persistent iSCSI LUN Device Name
categories:
- linux
- articles
tags:
- howto
- iscsi
- linux
- san
- udev
---

I spent a bit of time figuring out how to get this achieved, so thought it is
worth noting for the future reference. I will try to make this quick assuming
you have knowledge about iSCSI software initiators in Linux.

Tested on CentOS 6.0 it may work on CentOS 5.0 and alternatives. Software used:
	
* udev-147-2.29.el6.x86_64
* iscsi-initiator-utils-6.2.0.872-10.el6.x86_64

Steps to make this work:
	
* First add/create a file `/etc/scsi_id.config` (you may need to create a new file): `options=--whitelisted --replace-whitespace`
* Connect your iSCSI target to the system (I assume you know how to do that)
* Then you need to get an ID of the LUN (let's say it is `/dev/sdc` for now):
{% highlight bash %}
/sbin/scsi_id --whitelisted --replace-whitespace /dev/sdc
UNIQUE_UUID_OF_A_BLOCK_DEVICE
{% endhighlight %}

* Next, create udev rules file `/etc/udev/rules.d/20-persistent-iscsi.rules`:
{% highlight bash %}
KERNEL=="sd[a-z]", SUBSYSTEM=="block", PROGRAM="/sbin/scsi_id --whitelisted --replace-whitespace /dev/$name", RESULT=="UNIQUE_UUID_OF_A_BLOCK_DEVICE", NAME="iscsi/persistent-lun"
KERNEL=="sd[a-z][0-9]*", SUBSYSTEM=="block", PROGRAM="/sbin/scsi_id --whitelisted --replace-whitespace /dev/$name", RESULT=="UNIQUE_UUID_OF_A_BLOCK_DEVICE", NAME="iscsi/persistent-lun%n"
{% endhighlight %}

You can replace `NAME="to_whatever_you_want"`, I just like to use `/dev/iscsi/`
location for iSCSI LUNs attached to the system.
	
* Reload udev rules:
`udevadm control --reload-rules`
* Log the iSCSI LUN out and back in again, udev will assign the new device name for the LUN you specified.

I will not go through the basics of writing udev rules, but
basically `NAME=desired_device_name` sets the name of the device and `%n` is a
kernel number i.e. `/dev/sda1` would be `%n==1`.
