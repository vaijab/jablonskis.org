---
date: 2011-09-25 20:34:50+00:00
layout: post
slug: howto-rebuild-centos-6-0-linux-kernel
comments: true
title: Howto Rebuild CentOS 6.0 Linux Kernel
categories:
- articles
---

Here is a quick step by step tutorial howto rebuild a CentOS 6 kernel (it may
work with older CentOS). **WARNING: never build RPMS as root!!!**

I will be using a kernel version `2.6.32-71.29.1.el6` as an example.
* Grab an SRPM from (http://mirror.centos.org/centos/6/updates/SRPMS/kernel-2.6.32-71.29.1.el6.src.rpm)
* Install the rpm (as a regular user, this will install the source RPM into /home/$USER/rpmbuild/ directory):
`rpm -ivh kernel-2.6.32-71.29.1.el6.src.rpm`

#### Now there are two ways of doing it:
##### 1. If you want to apply your own patches or modify the kernel in any other way:
	
* Run the following command to unpack the kernel source and apply required
  patches etc: `rpmbuild -bp /home/$USER/rpmbuild/SPECS/kernel.spec`
* Configure the kernel: `cd /home/$USER/rpmbuild/BUILD/kernel-2.6.32-71.29.1.el6/linux-2.6.32-71.29.1.el6.x86_64/ && make menuconfig`
* After you have done all necessary changes to the kernel, it's time to compile
  it and build SRPM and RPM: `make -j<number of cores> rpm`
* Wait - it will take some time depending on what box the kernel is being built on
* Your custom kernel is built and RPM is created in: `/home/$USER/rpmbuild/RPMS/x86_64/kernel-2.6.32-1.x86_64.rpm`


##### 2. Use this method if you just want to edit kernel's config and rebuild
it, for example you want ext4 filesystem support in the kernel:
* Edit the config, for x86_64 the config is in `/home/$USER/rpmbuild/SOURCES/config-generic-rhel`
* Rebuild the kernel: `rpmbuild -ba /home/$USER/rpmbuild/SPECS/kernel.spec`
* Wait and enjoy your custom kernel


The new RPM will be placed into `/home/$USER/rpmbuild/RPMS/x86_64/kernel-version.x86_64.rpm`.
