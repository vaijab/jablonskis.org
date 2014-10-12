---
date: 2011-11-06 16:03:24+00:00
layout: post
slug: free-dns-hosting-what-happened
title: Free DNS Hosting - What Happened?
categories:
- linux
- services
- articles
tags:
- dns
- dyndns
- entrydns
- free
- hosting
- linux
- powerdns
- rails
- ruby
- service
---

You may wonder why I am writing about yet another free DNS hosting service,
after all there are quite a few of them out there already.

If you think so, then you're probably right, there are at least 4 DNS services
where you can host your own domain/zone for free, but there is always a catch.
They either start charging you if you want to add more domains/zones or if your
domain is a popular one and gets a lot of queries which need to be answered by
their name servers. There are other limitations too, like you can only have a
subdomain of their already preconfigured domain names. And if there are no
usage limitations then is usability limitation - UI sucks too much (again,
other people might find their UI usable).

I had a hard time finding one which is worth using it, but that's probably me -
I like minimalism.


### Old Good Service

I remember there was one great service, which is long gone now. Those guys used
to provide an awesome service for free, without any usage limitations as far as
I remember, their UI was very simple, written in PHP and was fast too.
Obviously there was no support at all - they clearly said that, but that's fair
enough.


### New Good Service - the beginning

Please keep in mind that this post is not a marketing promotion of the service,
it's just a genuine and honest stuff I would like to share with you.

So I came up with an idea that I need to do something about it, I contacted my
mate who is an incredible software developer. We had a chat on IRC and decided
that we could totally build a system and let people use it for free with no
limits.

We love what we do, I love linux systems, love open source and love FREE stuff,
he is pretty much the same + he loves coding.

Obviously, we needed a bit of cash just to start with, so we just put a couple
of hundreds of British pounds and bought some VPSes with a fair amount of
allowed bandwidth per month. The next important bit of this project was to
finish it - there are lots of people who are enthusiastic about stuff, they
start it, but never finish it, but apparently we have finished our project in 3
months or so.


### New Good Service - the present

And here we have it - [EntryDNS](https://entrydns.net). The service is nowhere
perfect and has a few problems which are yet to be solved.

Let me talk a bit about what is currently supported and what's coming. EntryDNS
currently has 3 name servers - 2 in the UK and 1 in the US. We have limited
funds at the moment, but we do hope people will help us out by
[donations](https://entrydns.net/pages/donate). Since DNS is pretty light we do
not need a huge funding to keep it going. Our aim is to have as many nodes as
possible spread out across the globe, we are working on a design how we will
achieve that technically and we are nearly there! :-)

EntryDNS supports major DNS resource records: A, AAAA, NS, CNAME, TXT, MX, SRV,
if you think there is an important RR to be added - give us a shout (see below
for contact details). Low TTL values are allowed too. Instant updates and so
on.

We tried to make our UI as simple as possible. It may not be very user
friendly, especially for people who know nothing or very little about DNS, but
please let us know how we could improve that.

We are working on some unique features which will allow users to share their
domains and subdomains with other users.


### The Future

It all depends on you guys. If you believe in what we do and if you like free
services supported by generous users then EntryDNS is not going anywhere.
We may even open source our code and system's design, so everyone will be able
to add new features and support the platform.
I, personally, enjoy being able to give something back to people.


### Links and stuff
* Follow EntryDNS on twitter - [@entrydn](https://twitter.com/#!/entrydns)[s](https://twitter.com/#!/entrydns)
* EntryDNS homepage - [https://entrydns.net/](https://entrydns.net/)
* People behind it - [https://entrydns.net/pages/team](https://entrydns.net/pages/team)
* Want to help? - [https://entrydns.net/pages/contact](https://entrydns.net/pages/contact)






