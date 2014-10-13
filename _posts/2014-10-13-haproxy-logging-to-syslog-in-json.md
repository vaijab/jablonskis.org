---
layout: post
title: "HAProxy Logging to Syslog in JSON"
date: 2014-10-13T23:13:44+01:00
modified:
comments: true
tags: [haproxy, linux, docker, syslog, json]
---
HAProxy is amazing piece of software. It is rock solid, extremely flexible and
has been proven by many comanies which depend on it.

I use haproxy as a load balancer and service discovery. It runs on each CoreOS
node and proxies traffic through to healthy services running on CoreOS cluster
inside docker containers. This post is not about how that works, but rather how
to get haproxy log to syslog and in json.

HAProxy log contains a lot of useful information and can be very helpful when
you're trying to debug some issue with your applications. Say you're pushing
all your logs to ElasticSearch, so that they can be easily searchable from one
place. I like when apps log in already structured format if possible, instead
of relying on some central log pre-processor.

HAProxy has no native json log format support, but it allows you to define your
own log format. In `haproxy.cfg` global defaults section define custom,
json-like `log-format`, then in global `haproxy.cfg` section, tell it to log to
`/dev/log`, see full example below:

{% highlight bash %}
global
  daemon
  maxconn 4096
  spread-checks 5
  log /dev/log local0

  defaults
    mode tcp
    log global
    option log-health-checks
    # make sure log-format is on a single line
    log-format {"type":"haproxy","timestamp":%Ts,"http_status":%ST,"http_request":"%r","remote_addr":"%ci","bytes_read":%B,"upstream_addr":"%si","backend_name":"%b","retries":%rc,"bytes_uploaded":%U,"upstream_response_time":"%Tr","upstream_connect_time":"%Tc","session_duration":"%Tt","termination_state":"%ts"}
{% endhighlight %}

A few things to note here is that haproxy does not allow you to define error
log format, so error logs will be just in plain unstructured format.

Let me know if you're interested how to get haproxy logs over the fence
(outside docker container).

