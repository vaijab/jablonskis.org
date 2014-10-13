---
date: 2011-07-07 23:10:27+00:00
layout: post
slug: desperate-for-centos-6
comments: true
title: Desperate for CentOS 6!
categories:
- articles
- linux
tags:
- bash
- centos
- centos6
- linux
---

If you are also very desperate for CentOS 6.0 as I am, you might find the below
one-line script useful to check if any of the official CentOS mirrors has got
6.0 already synced.

This tiny script automates the process of going through the mirror sites and
checking if 6.0 folder exists :-)

You will need curl, ncftp and obviously bash for the script to work. You
can add it to cron, so it gets run every 4 hours or so and you can setup some
sort of notification when CentOS 6.0 appears on one of the mirrors.

{% highlight bash %}
# so here it is, just copy and paste it to your terminal
for n in $(curl -s http://www.centos.org/modules/tinycontent/index.php?id=31 http://www.centos.org/modules/tinycontent/index.php?id=34 http://www.centos.org/modules/tinycontent/index.php?id=30 | egrep -o --color=never "ftp://([/.A-Za-z0-9-]+)"); do ncftpls -1 -t 1 $n | grep ^6 > /dev/null; if [[ $? -eq 0 ]]; then echo "Mirror $n has CentOS 6"; fi; done 2>/dev/null
{% endhighlight %}

I know some people will go mad at me probably for this idea, so I do not
encourage you to run this script especially very often! :-)

I just thought I will write a quick blog post about this crazy idea :-)
