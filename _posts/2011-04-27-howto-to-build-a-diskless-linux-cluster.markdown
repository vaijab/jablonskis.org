---
date: 2011-04-27 14:23:37+00:00
layout: post
slug: howto-to-build-a-diskless-linux-cluster
title: Howto to Build a Diskless Linux Cluster
categories:
- linux
- articles
tags:
- centos
- cluster
- hpc
- linux
---

### Intro

Recently I had some joy to build a diskless linux cluster for parallel regexing.
So I decided to document it how I accomplished that using free and open source software.
There is very little or very old documentation on how to build a diskless cluster using
linux distributions.

This howto explains the setup of using a single compressed root file system image as a
ramdisk for the slave nodes.


### My setup/configuration


Since the cluster is going to be used for regexing of some useful data out of millions of
small files, it is going to be very CPU and RAM intensive process. I am going to use the
following hardware:

  * 10 x slave nodes: AMD CPUs - 24 cores, 32GiB of RAM
  * 1 x master node: AMD CPUs - 8 cores, 8GiB of RAM, RAID5 array of 8 SATAII disks
  * 1 x network switch

Software/packages required:
	
  * Centos Linux 5.6
  * nfs client/server
  * dhcpd
  * tftp-server
  * syslinux
  * xinetd
  * chroot

Additional software may be installed:

  * ntp
  * pssh (parallel SSH)

Networking and hostnames:
	
  * Subnet - 10.0.0.0/24
  * Gateway - 10.0.0.254
  * Master node hostname - m0.example.com (10.0.0.1)
  * Slave nodes hostnames - s{0..9}.example.com (10.0.0.10-10.0.0.19)


### Prerequisites


I suppose you have all the servers connected to the network switch and a
router which does routing or NAT at least for your master node to be able to
download software needed (unless you have a local yum repo as I do). Also I
believe you have some CentOS or RHEL based distributions administration skills
and general common sense. :-)


### Master node installation and config


The first step in building your diskless cluster is to install an OS on the master
node in this case I am using CentOS5.6. The master node's base installation is going
to be used for slave nodes root filesystem image, so initially try to keep it as minimal
as possible (obviously install the required packages for the purpose of your cluster).

The things master node is going to provide to slave nodes are:

  * dhcp server
  * pxe/tftp boot server
  * read-only /usr NFS export
  * NTP time server


Install ntp package, so both the master node and slave nodes will have it: `yum install ntp`


#### Build a root file system image


Once the OS is installed and most recent updates are applied we can start building a root file system image for the slave nodes. The process is pretty straight forward: create a file of 512MiB in size, create a file system on the loop file, mount it, copy and create required files/directories, edit configuration files, chroot into the new environment, create users, enable/disable services etc. Here is a basic script which can be used to do some of the mentions steps:

{% highlight bash %}
#!/bin/bash

# zooz <jablonskis@gmail.com>
# a script to create a basic compressed rootfs for diskless nodes

# set variables
# size in megabytes
rootfs_size="512"

# set mount point for the rootfs
mount_point="rootfs-loop"

# create a rootfs file
dd if=/dev/zero of=rootfs bs=1k count=$(($rootfs_size * 1024))

# create an ext3 file system
mkfs.ext3 -m0 -F -L root rootfs

# create a mount point
mkdir -p $mount_point

# mount the newly created file system
mount -t ext2 -o loop rootfs $mount_point

# cd into it and create required directory structure
cd $mount_point && mkdir -p bin boot dev etc home lib64 \
mnt proc root sbin sys usr/{bin,lib,lib64} var/{lib,log,run,tmp} \
var/lib/nfs tmp var/run/netreport var/lock/subsys

# copy required files into created directories
cp -ap /etc .
cp -ap /dev .
cp -ap /bin .
cp -ap /sbin .
cp -ap /lib .
cp -ap /lib64 .
cp -ap /var/lib/nfs var/lib
cp -ap /usr/bin/id usr/bin
cp -ap /root/.bashrc root/
cp -ap /root/.bash_profile root/
cp -ap /root/.bash_logout root/

# set required permissions
chown root:lock var/lock

# cd out of the mount point
cd ..
{% endhighlight %}

The above script creates a rootfs ext3 file with the directory structure and populates
it with required binaries and libraries for the system to be able to boot off. You should
have the rootfs mounted on `/root/rootfs-loop`. Now you can bind mount /usr and chroot to the environment:

{% highlight bash %}
# bind mount /usr
mount -o bind /usr rootfs-loop/usr

# chroot to the new environment
chroot rootfs-loop /bin/bash
{% endhighlight %}

Now you are in your new (node) environment. The following steps are necessary for the
nodes to function properly:
	
  * _/etc/fstab _the contents of this file should look like below:


{% highlight bash %}
/dev/ram0               /              ext3    defaults        0 0
tmpfs                   /dev/shm       tmpfs   defaults        0 0
devpts                  /dev/pts       devpts  gid=5,mode=620  0 0
sysfs                   /sys           sysfs   defaults        0 0
proc                    /proc          proc    defaults        0 0
m0.example.com:/usr     /usr           nfs     ro        0 0
{% endhighlight %}

	
  * `/etc/hosts`:

{% highlight bash %}
127.0.0.1    localhost.localdomain localhost
::1        localhost6.localdomain6 localhost6

10.0.0.1 m0.example.com
10.0.0.10 s0.example.com
10.0.0.11 s1.example.com
10.0.0.12 s2.example.com
10.0.0.13 s3.example.com
10.0.0.14 s4.example.com
10.0.0.15 s5.example.com
10.0.0.16 s6.example.com
10.0.0.17 s7.example.com
10.0.0.18 s8.example.com
10.0.0.19 s9.example.com
{% endhighlight %}

	
  * `/etc/sysconfig/network`


Leave HOSTNAME unset - the slave nodes will get their hostnames from DHCP.

{% highlight bash %}
NETWORKING=yes
NETWORKING_IPV6=no
HOSTNAME=
{% endhighlight %}

	
  * `/etc/sysconfig/network-scripts/ifcfg-eth0`


Make sure HWADDR is unset, I will not explain why :-)

{% highlight bash %}
DEVICE=eth0
BOOTPROTO=dhcp
HWADDR=
ONBOOT=yes
{% endhighlight %}

It is always a good idea to have time synced up with the master node or some external
NTP source, so we will enable NTP service and point it to sync up with the master node.

`chkconfig ntpd on`

Edit `/etc/ntpd.conf` file and set `server` option to `m0.example.com` (configuring NTPD is out of the scope of this post)

Some additional things you may consider doing: disable all unnecessary  services, create users, add public ssh keys (to make nodes management a  lot easier), configure remote syslog etc.

The next step is to exit chrooted environment, umount the image and compress it:

{% highlight bash %}
exit
umount rootfs-loop/usr
umount rootfs-loop
gzip -c rootfs | dd of=rootfs.gz
{% endhighlight %}

Now you have `rootfs.gz` root file system image.


#### Master node configuration

Right. Let's configure the main services on the master node which are vital
for the slave nodes. I assume you have the network interface configured with
static IPs and host name set correctly to `m0.example.com` (or whatever naming you decided to use).

Now install the necessary software/packages:

`yum install xinetd dhcp syslinux tftp-server`


###### DHCP server configuration - /etc/dhcpd.conf_

Here is the very basic DHCP daemon config file:

{% highlight bash %}
ddns-update-style interim;
ignore client-updates;

subnet 10.0.0.0 netmask 255.255.255.0 {
	# supposedly your router has 10.0.0.254 address
	option routers		10.0.0.254;
	option subnet-mask	255.255.255.0;

	# address of the tftpboot server
	next-server 10.0.0.1;
	filename "pxelinux.0";
	default-lease-time 432000;
	max-lease-time 432000;
}

# fixed IP configuration for s0.example node
host s0.example.com {
	fixed-address 10.0.0.10;
	hardware ethernet AA:BB:CC:DD:EE:FF;
	option host-name "s0.example.com";
}
{% endhighlight %}

If you have more slave nodes creating host configuration for every single one can be
painful, so I wrote a simple bash script to easy it up a bit. What you need is a file,
let's say `host_ip_mac.txt`, which contains:

{% highlight bash %}
s0.example.com    10.0.0.10    AA:BB:CC:DD:EE:FF
s1.example.com    10.0.0.11    AA:BB:CC:DD:EE:00
s2.example.com    10.0.0.12    AA:BB:CC:DD:EE:11
s4.example.com    10.0.0.13    AA:BB:CC:DD:EE:22
{% endhighlight %}

And then the below script, say named `dhpd-conf-gen` (make it executable of course):

{% highlight bash %}
#!/bin/bash

# takes three arguments from stdin and creates dhcpd
# config for each node
# hostname ip mac
# multiple lines can be passed on

while read -r hostname ip mac
 do
 echo "host $hostname {"
 echo -e "\tfixed-address $ip;"
 echo -e "\thardware ethernet $mac;"
 echo -e "\toption host-name \"$hostname\";"
 echo -e "}"
 echo
done
{% endhighlight %}

Run it and it will spit the config snippets for every node you listed in
`host_ip_mac.txt` file and then just paste it into the `dhcpd.conf` file:

{% highlight bash %}
cat host_ip_mac.txt | ./dhcpd-conf-gen
host s0.example.com {
	fixed-address 10.0.0.10;
	hardware ethernet AA:BB:CC:DD:EE:FF;
	option host-name "s0.example.com";
}

...
{% endhighlight %}

Start the service and make sure it is set to start on boot:

`service dhcpd start && chkconfig dhcpd on`


###### tftp boot server configuration - /etc/xinetd.d/tftp

{% highlight bash %}
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /tftpboot -v
        disable                 = no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
{% endhighlight %}

Start the service and make sure it is set to start on boot:

`service xinetd start && chkconfig xinetd on`


###### PXE boot configuration

PXE boot loader and its configuration file as well as the linux kernel
and rootfs.gz image will have to be copied under `/tftpboot` directory:

{% highlight bash %}
# create directories required for pxe bootloader
mkdir -p /tftpboot/{linux,pxelinux.cfg}

# copy pxe boot loader (comes with syslinux package)
cp /usr/lib/syslinux/pxelinux.0 /tftpboot/

# copy linux kernel so it can be passed onto nodes by a pxe bootloader
cp /boot/vmlinuz-$(uname -r) /tftpboot/linux

# copy linux root filesystem image
cp /root/rootfs.gz /tftpboot/linux
{% endhighlight %}

Create a PXE bootloader config file - `/tftpboot/pxelinux.cfg/0A0000`

{% highlight bash %}
# default is label 'linux'
# boots a linux kernel and mounts rootfs.gz as a root file system on a 512MiB ramdisk
default linux

label	linux
	kernel linux/vmlinuz
	append initrd=linux/rootfs.gz root=/dev/ram ramdisk_size=524288 rw ip=dhcp
{% endhighlight %}

The above config looks similar to the one we used to have in happy LILO days, remember?
The `append` kernel line parameters pass the `rootfs.gz` image as a root file system,
which is then mounted on `/dev/ram0`, 512MiB in size as read-write (there is no point
to mount ro and then remount it rw).

`0A0000` is 10.0.0.x converted into HEX, which means that the above config is valid for
the nodes with 10.0.0.x IPs. For information about the way `pxelinux` finds its configuration
files can be found [here](http://syslinux.zytor.com/wiki/index.php/PXELINUX#How_do_I_Configure_PXELINUX.3F).


###### NFS server configuration

NFS server configuration on Linux and most Unix-like systems is very
simple - in our case you will need `/etc/exports` below:
`/usr		10.0.0.0/24(ro,no_root_squash)`

Start NFS services and make sure it is set to start on boot:
`service nfs start && chkconfig nfs on`


### Powering on slave nodes

Now once we've got everything (I believe) in place we can power on slave nodes. So fingers
crossed and if you added a bit of your brain too while following this howto you should have
a fully working cluster for high performance tasks (what tasks? - I will leave it for
your imagination :-) ).


### Cluster Management

If you've forgotten something or want to add more features or change the config files
for your slave nodes, scroll back up and follow instructions how to mount the rootfs
image and chroot to the new environment, then unmount it and compress it over to `/tftpboot/linux/rootfs.gz`.

Also if you screwed something on the slave nodes you can always power cycle them and they will be as fresh as new :-)

Slave nodes management and control can be done using an awesome tool written in Python called [pssh](http://code.google.com/p/parallel-ssh/).


### Final words

This was my second public blog post, so please post your comments with your suggestions, questions and fixes and I will try to answer them all.

Enjoy! Hope it will be useful for some people and if it isn't, then I will definitely benefit from this post some time in the future.
