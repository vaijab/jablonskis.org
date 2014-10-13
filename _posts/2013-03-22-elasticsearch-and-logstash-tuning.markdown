---
date: 2013-03-22 00:50:29+00:00
layout: post
slug: elasticsearch-and-logstash-tuning
comments: true
title: ElasticSearch and Logstash Tuning
categories:
- howto
- articles
tags:
- elasticsearch
- linux
- logstash
- scaling
- tuning
---

I was slightly familiar with elasticsearch and logstash before at a very
minimum level. But just a couple of days ago I had a chance to play with both
toys at a larger scale. I was given a box with elasticsearch, redis and
logstash already running, it was actually barely alive, so overwhelmed,
elasticsearch was constantly timing out and redis in-memory database was
running out of allocated memory.

It was a pretty simple setup: around 10 logstash instances running on nginx
load balancers pushing logs to a redis instance, logstash running on the same
box  reading from redis and pushing into elasticsearch, also on the same
machine. The only issue is that the amount of data is pretty big, around
350GB/day of compressed indexes in elasticsearch.

As usual, before trying to do random changes, I normally do a lot of reading
just to understand how things work at a deeper level. I found out quite a lot
about elasticsearch and especially its magic, which I think is really cool.

I will share some of my thoughts and details of how I optimized the
elasticsearch as well as logstash setup I had.

As I mentioned, ES does a lot of magic under the bonnet and makes some
assumptions and decisions for you, which makes ES horizontal scaling like a
walk in a park compare to MySQL for example. But at the same time it cannot
predict your ES usage intentions.

ES is written in Java and obviously runs inside a JVM. The most apparent JVM
option is -Xmx. I set it to about 50% of the total physical memory, it happened
to be a 64GB of RAM machine, so I set the `ES_HEAP_SIZE` size to 32GB. It's
important to understand, that it's a Java application which stores indexes on a
file system, which has a cache too and JVM actually likes that, so leaving at
least 40% of total RAM available to the file system cache is a great idea.

Memory intensive Java application does not feel comfortable when its memory
gets paged out, so to prevent this from happening, set the following option,
which will also tell the JVM to lock the whole 32GB of memory at start time:
{% highlight bash %}
bootstrap.mlockall: true
{% endhighlight %}

ES by default assumes that you're going to use it mostly for searching and
querying, so it allocates 90% of its allocated total HEAP memory for searching,
but my case was opposite - the goal is to index vast amounts of logs as quickly
as possible, so I changed that to 50/50:
{% highlight bash %}
indices.memory.index_buffer_size: 50%
{% endhighlight %}

Another option to consider is the frequency of a translog flushes, by default
translog is flushed for each shard of each index every 5k operations, which in
my case was almost every second, so I changed that to every 50k operations:
{% highlight bash %}
index.translog.flush_threshold_ops: 50000
{% endhighlight %}

By default ES will split your indexes into 5 shards, it's probably a good
default, but it highly depends on how you plan to scale your ES cluster. In my
case I have no plans to scale it to more than maybe two ES boxes in total, so I
decided to use 3 shards per each index.
{% highlight bash %}
index.number_of_shards: 3
{% endhighlight %}

You can change a number of shards per existing index later on, if you choose
to, but I'd highly discourage you from doing so, because ES would then need to
re-index and to shuffle data around, which is a very expensive operation, but
again, it depends on the amount of data you have and other criteria.

Next, another very important detail to understand about ES is thread pools. ES
has multiple thread pools for each function: search, index, bulk, merge, etc.
Most important in my case, are search and index thread pools. By default ES
does not have a hard limit on the number of threads it would spawn to serve new
requests, but problem is that hardware capacity is limited in reality.

I wish ES did a better job here, maybe implementing some sort of dynamically
adaptive mechanism would be a good place to start? This is really something you
would have to experiment with. In my situation, I
am mostly interested in writes, so my configuration looks like this:

{% highlight bash %}
# Search thread pool
threadpool.search.type: fixed
threadpool.search.size: 20
threadpool.search.queue_size: 100

# Index thread pool
threadpool.index.type: fixed
threadpool.index.size: 60
threadpool.index.queue_size: 200
{% endhighlight %}

Logstash. Another brilliant application. As mentioned earlier, I use it to ship
Nginx logs to Redis database, which acts like a buffer between logstash and ES.
Logstash processes run on each Nginx box ship logs to Redis and on the other
side of the Redis DB there is another logstash which pulls logs from Redis and
pushes into ES. I discovered that this single logstash was a bottleneck too
once I got ES tuned up.

Logstash by design is fairly simple, so it does not have a lot of options for
tuning, though I did a couple changes to its default configuration.

First, gave it 2GB of RAM. Second increased a number of Redis input threads to
8, which made a huge difference, because previously there was a single thread
trying to pull data at the same rate as other 10 logstash processes pushing in.

So there was my almost two days of playing with ES and logstash and trying to
make it work better. I feel pretty confident that there are still lots that can
be done to improve the performance of both logstash and elasticsearch.
