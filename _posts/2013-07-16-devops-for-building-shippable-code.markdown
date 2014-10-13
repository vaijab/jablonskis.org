---
date: 2013-07-16 22:33:03+00:00
layout: post
slug: devops-for-building-shippable-code
comments: true
title: 'DevOps for Building Shippable Code '
categories:
- DevOps
- Educational
- howto
- articles
tags:
- ansible
- chef
- culture
- devops
- linux
- practices
- puppet
- salt
---

The word DevOps has been out there for a while. Some believe it is just a buzz
word, others - it's a work culture and recruiters insist that it's a job
position, but let's leave that to them. We shall try to eliminate silos between
Operation and Developer teams and not to create another silo called 'DevOps
Engineer'. There is a lot of fuzz about this already, so let's move on to some
real practices on how we can work together to get our code out the door.

I work in Operations team, so this post is going to be more from an Ops
perspective. First we should understand how important code shipping in general
is. We all like to work on some cool things, try out new languages,
configuration management frameworks and so on, but the most important mission
of all is to be able to ship new code fast, reliably and later to maintain it.

In a more traditional organization developers write code, throw it over a fence
to a QA team (if one exists) and mark the task as done. The QA team takes it
over, writes some functional tests or most likely does some manual testing, if
all okay - passes over to Operations team and marks the QA task as done. The
Ops team picks it up and tries to work around some design decisions that've
been made by the Dev team, just to ship the code to production. This process
from writing code to having it running in production can take a very long time
and sometimes fail.

There are lots of companies and sadly a lot of people who insist on the above
culture and that's why we do not work for or with them. There is no DevOps
Bible or a defined list of rules what it actually is.

DevOps culture is very simple, you just need similar-minded people, otherwise -
good luck. To get started, first start using the word 'we' when talking about
Dev, Ops or both teams. There is no such thing as 'it's an Ops or Devs
problem', no - it is our problem. We're all in the same boat, so let's keep it
on the water, not under. Another important fact is that DevOps can be anything
as long as we all work together and get the our shit done in an efficient, fast
and simple way.

Let's talk about some practices that should help us work together. Most of us
are used to working in our own silos, so it's not unusual for Operations people
not to be up to speed with development best practices and Developers team with
Ops practices on running their software in an environment other than a single
laptop. So let's educate each other.


**Communication**. I cannot stress enough how important this is. A few tips how this could be improved:

* Get Ops or Devs into your morning standups.
* Listen to what each member has to say and try to help each other with the issues they are having.
* Use same communication tools, be it IRC, Skype or something else and make
  sure everyone is easily accessible. Please, no email.

When designing your software architecture or infrastructure, make sure whoever
is interested get involved. Consistency and knowledge sharing is the key here.
So let's talk about some tips that could help both teams to build and ship
software faster.

Build service oriented infrastructure. Design and build simple, small and
API-based services. Stop writing a bunch of random scripts and relying on
cronjobs. There is nothing wrong with scripts and cronjobs, but we definitely
can do better than that.

Think about nothing-shared infrastructures. It is so cheap to spin up hundreds
of virtual instances nowadays from both operational and time perspective.

Snowflake hand-crafting is beautiful, but don't apply it to servers. Use
configuration management tools, like Salt, Puppet, Chef or Ansible and let them
do the crafting. If you're small, start with Salt or Ansible, especially if
you're a developer. Build your software with configuration management practices
in mind - make it configurable. If you can use a plain-text config file, please
do so instead of choosing some random database or some other binary format to
store a port number on which your service listens on.  Build your software with
one environment in mind - let CM tools deal with configuration differences
between different environments.

Build your configuration management infrastructure, so that you do not repeat
yourself. Write generic nginx, httpd, firewall, etc modules, that can be reused
many times and have no dependencies on other modules. Separate your CM code
from configuration data. Try to abstract your infrastructure by simply writing
structured configuration data. Some tools like Salt or Ansible are designed to
do that by default, others like Puppet or Chef allow you to do that as well.

Talk to Ops before jumping on 'the-most-popular-programming-language-today'
wagon. See if whatever you're about to invest a lot time in is actually simple
to maintain and run.

Do yourselves a favour and use distribution package managers. If you're Google
or <sarcasm>Canonical</sarcasm>, then you might build your own packaging
format, otherwise invest some time in learning DEB or RPM packaging, which is
not that difficult. Try and use distro-provided packages first or bundle all
the dependencies in a self-contained package. Build packages in a clean and
reproducible environment, there are great tools like pbuilder for deb packages
and Mock for RPM-type package building. If you're lazy to learn how to package
your software, use FPM. FPM is a great tool, but it has it's downsides and
uses.

Use distro-provided service managers, if you're an Ubuntu shop, then use
upstart, if Debian / Fedora and soon RHEL/CentOS - systemd. Systemd is
absolutely brilliant.

Make your software log and make it log a lot. Do not log everything to error
logs by default. Use a consistent place where your software writes logs to,
either be it syslog or /var/log/<service_name>/logfile.log. Ship all of your
logs to a central log server for processing and indexing, Logstash +
ElasticSearch + Kibana are great tools for doing just that.

Build built-in poke-endpoints. If it's an HTTP-based service, endpoints like
below are always invaluable.

* /version - returns a current running software version
* /stats - some internal stats which could be queried by monitoring system
* /debug - more verbose information, if enabled

Pick a metrics software. Graphite is a pretty popular one. Make your
software/service send stats and other useful-to-graph information to Graphite
for later storage and correlation.

Write README.md files and make sure they are checked in to your version control
system together with code. README.md could just have some basic information
like:
* What's the purpose of this service?
* What does it actually do? Makes coffee or something more interesting?
* What are the dependencies?
* How to build it?
* How to start / stop it?
* What does it talk to?
* Configuration files

Everyone in your company should be able to deploy your software as many times
as they want and with minimal or zero risk. Etsy and Netflix has done it right.
I recommend you to read their posts, watch some cool talks and make use of some
of their great open source tools.

When it comes to deployment, again make sure Ops and Dev build that together.
Whatever you build or chose an existing solutions - it needs to be simple,
flexible and easy to extend. If you make use of OS packaging, configuration
management and other tools what they are great for, then your deployments
should be fairly straight forward.

There is so much more I could share with you, but I might leave it for a part
2, since this post turned out to be quite long.

By the way, if you got this far. We at
[TagMan](http://eu.tagman.com/about/careers/) are hiring. If you're
London-based and would like to work within our great DevOps team, get in touch.
