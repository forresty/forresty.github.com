---
layout: post
title: "Cassandra and Ruby"
date: 2012-10-18 11:36
comments: true
categories:
---

I've been working on a project that requires the usage of Cassandra and Ruby for some time already, and the process is at the same time quite painfully slow and rewarding for me.

<!-- more -->

To be honest, working with Cassandra using Ruby is not a pleasant experience, especially for a not-so-experienced programmer like me. IMHO the main problems are:

1. Cassandra itself, is not so wonderful as promised. Sitting on the AP side of the CAP theorem, that is, unlike traditional RDBMSs or newer NoSQL breeds such as MongoDB and Redis, Cassandra is always available and partitioning-tolerant, while sacrificing consistency. Being open-sourced by Facebook and later adopted by Twitter and Reddit etc, it solves very specific problems for them: inbox search for Facebook, real-time analytics for Twitter, and persistent cache for Reddit. THEY DON'T USE IT AS MAIN STORAGE ENGINES AND THERE ARE NOT MANY PEOPLE IN THE WILD USING IT. That alone can tell something.

2. Cassandra is not very Ruby-friendly, or better put, there is not a huge community of Ruby users playing with it, perhaps due to somewhat conflicts in philosophy? Unlike widely available options of open source client libraries for other storage engines, most notably ActiveRecord from Rails for traditional RDBMS, or Mongoid for MongoDB, there are only low-level libraries such as thrift_client gem and cassandra gem by Twitter, the only good high-level library I can find is data-axle's fork of cassandra_object, which only has 28 watchers on GitHub. So in that sense, I do feel lonely a little bit, good thing is I've already gotten used to that.

Well, this is not a post about the downside of using Cassandra with Ruby, so I'd just stop the complaint. If I am really as good at coding and hacking as I imagined myself to be, I would be much happier. So I only have myself to blame.

So back to the main topic.

Thrift interface is Cassandra's lowest level API interface, it's quite like the ProtocolBuffer from Google or the (seemingly more sassy) MessagePack. Though CQL is later added as an interface option, the CQL channel is actually built upon thrift interface - that is, every time you execute a CQL query against a Cassandra node, your query is packaged as a Thrift RPC call to that node, and you get your results back as a packet of binary thrift formatted data. Performance-wise, various people have done experiments and the difference between raw thrift calls and cql queries is not apparent, which is quite understandable. Besides Thrift, there used to be an alternative RPC interface based on Avro which is a bit similar to Thrift, the reasons why it was introduced in the first place and why later got dropped I never bother myself to look into.

Since Cassandra is written in Java, so naturally its Thrift server is implemented in Java. So typically when you use Cassandra with Ruby, you are going down on this route:

```
cassandra gem (from Twitter, in Ruby) --->
thrift_client gem (Twitter, simply a wrapper around thrift gem, Ruby) --->
thrift gem (from the Thrift team, part of Thrift source code, Ruby & C) --->
[the inter-webz, TCP/IP socket] --->
Thrift server (from the Cassandra team, in Java) --->
Cassandra internal API (Cassandra, Java) --->
Cassandra (hooray!)
```

Well, what I am doing (and a bunch of other people) is throwing on top of these another layer, the data-axle's fork of cassandra_object, which is modeled based on the awesome ActiveModel project introduced in Rails 3 and uses the cassandra gem from Twitter internally, and then trying to tame it to my specific needs, and hack yet another layer of business logic on top of it. So you can imagine when things went wrong, I'd be forced to go down each layer trying to figure out what the fuck is going on. Since there is no a huge community, I am thankful that my hair is short so I don't have the risks of getting them pulled off by the irritated me.

But it's quite fun and so far I am still enjoying it. I see myself forking the top most layers, learning the hell out of every bit of technical details, and hopefully I don't need to go down to the very bottom. During the whole process I am often overwhelmed by a mixed feeling: the hatred towards myself and the regret that I didn't become a hard-core tech nerd as early as when I am still in college, and the joy of challenging myself to understand every piece of details down the road and finally getting close to it.

Let see how far it goes.