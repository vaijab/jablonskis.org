---
layout: post
title: "Systemd Journal Logging Inside Docker Container"
date: 2014-10-12T22:27:01+01:00
modified:
categories: [docker, systemd]
tags: [docker, journald, systemd, logging, containers]
---
There are a lot of companies which are still discovering puppet, chef or
saltstack. Sadly, there are ones which are yet to discover configuration
management. While companies like Google have been running everything inside
containers for over a decade.

I always try to keep up with ever changing technologies and best practices. I
have been using docker for quite some time and recently have started to run
production inside containers. This new approach opens up a lot of questions and
challenges. I am going to share my experience and tricks about running services
inside containers. This time I will touch on logging.

A known practice is to run single process containers, but sometimes you want to
run multiple processes. For example, haproxy and confd. You want to ensure that
if confd or haproxy dies, you want to make sure that something brings either of
them up, in my case that something is systemd. I base my images off of fedora
which comes with systemd and journal. I use systemd extensively in CoreOS as
well, so it makes sense to use it inside containers where it is needed. As you
know, systemd comes with journald. I love both systemd and journal, they solve
so many problems, especially when it comes to docker logging, which I
will talk about in my next blog post.

I have both haproxy and confd running inside a docker container as systemd
services, which means that anthing they spit out to stdout/stderr is eaten by
journal. In more traditional infrastructure, that is a priceless feature, but
in container case this behavior can be undesirable, however it is very easy to
to tell journal to forward its messages to console (`/dev/console`).

When building docker image with systemd, make sure to add a modified
`journald.conf` file with `ForwardToConsole=yes` to
`/etc/systemd/journald.conf`. Beware that journal will only forward messages to
console if it sees that there is a TTY available, so you need to run docker
container with `docker run -t <...>`.

