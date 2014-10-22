---
layout: post
comments: true
title: "Running CoreOS in EC2 Autoscaling Group"
date: 2014-10-22T23:11:50+01:00
categories: [coreos, aws]
tags: [coreos, etcd, aws, ec2, autoscaling]
---
My life managing systems, especially distributed, has gone much easier since I
started using more and more of CoreOS awesomeness.

> CoreOS is a new Linux distribution that has been rearchitected to provide
> features needed to run modern infrastructure stacks.

The main features of CoreOS which I like are that it is very minimal, read-only
and most importantly comes with [etcd](https://github.com/coreos/etcd) out of
the box.

As it stands, CoreOS is not designed to run in EC2 auto scaling group. Mostly
because to create an ASG launch configuration, you have to specify which AMI id
to use for launching instances. CoreOS has its own update mechanism, similar to
ChromeOS and is based on [Omaha spec](https://coreos.com/docs/coreupdate/custom-apps/coreupdate-protocol/).
CoreOS cluster can be auto-updated whenever there is a new release of the
channel that you are running, while ASG will always start new instances using
the AMI you originally specified. This could mean that new machines which are
added to the cluster can take long time to catch up with the rest of the
cluster in terms of CoreOS version. Also, worth noting that they may never be
able to join the cluster, because of incompatible versions of etcd (not that
this is the case right now, but it might be if AMI in ASG launch configuration
is very old).

If you're using EC2 ASG, you probably doing it via CloudFormation templates. CF
is very convenient, but it has its limits. One of which you have to be very
careful about is CF stack updates. Each resource has a different update policy.
Most dangerous are the ones which require a resource replacement if some of its
properties are to be updated. In our case, it is `ImageId` property of
`AWS::AutoScaling::LaunchConfiguration` type. Currently there is no way to say:
hey, don't touch currently running instances, but when you launch new ones,
make sure to use an updated `ImageId`.

What I do is I have automatic CoreOS updates disabled by default. You can do that via `user-data` using [cloud-config](https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/#coreos):

{% highlight yaml %}
#cloud-config

coreos:
  update:
    reboot-strategy: off
{% endhighlight %}

This means, that you will have to roll out CoreOS updates as part of the
CloudFormation stack update. This works for small or mid-size clusters. However, there
is a catch here. As I mentioned before, CoreOS comes with etcd and I am sure
you use etcd if you have more than one CoreOS node. The problem with CF stack
updates and EC2 ASG in general is that it will happily carry on killing and bringing
nodes one by one by default. ASG has no idea about your cluster state. Your
cluster can get into a broken state very quickly, due to lack of etcd quorum, ASG
replaces instances too quickly.

At etcd cluster formation time, you can set etcd member remove delay timeout, by
default it is quite high for our needs. It configures etcd with the minimum
time in seconds that a machine has been observed to be unresponsive before it
is removed from the cluster. What we need to do is to configure ASG to wait as
long as twice the amount of time it takes for etcd to notice and remove dead nodes
from the cluster.

ASG has an update policy for rolling updates. Let's set `PauseTime` to `PT10M`
and `MinInstancesInService` to `N-1`, `N` being a desired number of nodes in
your CoreOS cluster.

Again, etcd can be configured via the same `cloud-config` configuration using EC2 `user-data`:

{% highlight yaml %}
#cloud-config

coreos:
  update:
    reboot-strategy: off
  etcd:
    discovery: https://discovery.etcd.io/new  # <-- get a new token
    # some other etcd params
    # peer-addr etc
    cluster-remove-delay: 300
{% endhighlight %}

Please note, that you cannot change `cluster-remove-delay` parameter if your
cluster is already formed.

However, I have noticed that sometimes etcd would get stuck and won't re-elect
a new leader. I have not had a chance to debug that in more detail, but I
suspect that's a bug in etcd. I am aware that etcd team is working hard at
addressing these issues in version 0.5.0. One way of forcing re-election is by
stoping etcd service on all CoreOS nodes and starting them again:

{% highlight bash %}
$ sudo systemcl stop etcd.service; sleep 5; sudo systemctl start etcd.service
{% endhighlight %}

I am sure that people have different ways of running CoreOS on EC2 and maybe in
ASG. If you do, please share.

