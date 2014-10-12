---
date: 2011-11-11 01:24:14+00:00
layout: post
slug: howto-log-bash-history-to-syslog
title: HOWTO Log Bash History to Syslog
categories:
- bash
- linux
- tricks
- articles
tags:
- bash
- central
- history
- linux
- log
- logging
- shell
- syslog
---

I will show you a very nice and non-intrusive way how to log bash history to a
syslog. You may wonder what problem I was trying to solve? Right, the issue is
that I want to log root users bash history from multiple servers to a central
syslog box. I also want to preserve root users history on each server for
convenience purposes.

But first let's talk about my failed attempts to do so.


### Failed attempts
* Hacking bash source code.
Really? We are talking about a server farm, who would like to maintain custom
built bash packages, make sure that security fixes and bugs are also pushed to
your custom bash sources etc etc? - No one! Unless you have nothing else to do.
* Using trap. If you google for a solution you will come across [this blog
post.](http://blog.rootshell.be/2009/02/28/bash-history-to-syslog/) It sounds
like a feasible idea until you actually try it. Using bash built-in trap
command which allows you to basically catch users input and then pipe it to a
logger command.

There are few problems taking this approach, first user input does not get
logged on logout, second, if you press enter multiple times the previous
command gets logged multiple times, third, how do you log shell's PID?

* Using script (typescript). Sounds like a good idea too, but first
  `script` logs input and output what happens in the terminal. What is wrong
  with that? Well try to tail the same file you're logging to - you'll see what I
  mean. :-)


### Successful attempt

There is an updated trick below which logs to syslog as well as writes commands
to .bash_history file so you do not lose your bash history.

`PROMPT_COMMAND='history -a >(tee -a ~/.bash_history | logger -t "$USER[$$] $SSH_CONNECTION")'`

Is that simple? - Yes, it is! You can put the command (not the whole line, but
just the command) into `/etc/sysconfig/bash-prompt-default` and make it
executable. This will log all users bash history system wide to a syslog. Or
you can add it to `~/.bashrc` file per user - it's up to you. Surely, it's
possible to tell the logger to send messages to specific log facility and so
on, but that's out of the scope of this blog post. The way I do, I put that
into `/root/.bashrc`, configure rsyslog to log to central log server, which
then writes messages to a separate file. Please don't say - "oh that's easy to
bypass, this is another failed attempt". Yes it is easy to bypass, but that's
not the thing I am trying to achieve. At the end of the day root can bypass
pretty much everything. This is a solution if you want to have some kind of
audit trail what happens on your systems when you have multiple sys admins.

Your input is always welcome in the comments section below.

PS: Thanks to [Steve Harris](https://twitter.com/#!/theno23) for an awesome
brainstorming session which helped me to come up with this idea.
