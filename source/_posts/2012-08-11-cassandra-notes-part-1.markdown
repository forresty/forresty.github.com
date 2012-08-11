---
layout: post
title: "Notes and Quotes of Cassandra The Definitive Guide, Part-1: Introduction"
date: 2012-08-11 15:11
comments: true
categories:
---

Following are the notes I took while reading [_Cassandra The Definitive Guide_](http://www.amazon.com/Cassandra-Definitive-Guide-Eben-Hewitt/dp/1449390412). I am taking a client work that requires me to use Cassandra, so I picked up this book to get familiar with it. It was a good read.

This article is for Chapter 1 the introduction of Cassandra and in general NoSQL. Since I am not an expert in this field, it contains a lot of quotes from the book. In fact, there are so many quotes in this article that many sentences not appear in blockquotes are actually quotes as well. Apologies for that.

I expect myself to write at least one more article on this book, which will be more on the technical details as described in the rest of the book.

<!-- more -->

## History and Background

Cassandra was **open-sourced by Facebook** in July 2008. It was **strongly influenced by Amazon's Dynamo** with its Dynamo-style replication model with **no single point of failure**, but adds a more powerful **"column family"** data model.

The code was released as an open source Google Code project in July 2008. During its tenure as a Google Code project in 2008, the code was updateable only by Facebook engineers, and little community was built around it as a result. So in March 2009 it was moved to an Apache Incubator project, and on February 17, 2010 it was voted into a top-level project. Cassandra graduated from the Apache Incubator in March 2010.

**How Did Cassandra Get Its Name?**

In Greek mythology, Cassandra was the daughter of King Priam and Queen Hecuba of Troy. Cassandra was so beautiful that the god Apollo gave her the ability to see the future. But when she refused his amorous advances, he cursed her such that she would still be able to accurately predict everything that would happen—but no one would believe her. Cassandra foresaw the destruction of her city of Troy, but was powerless to stop it.

## Hierarchical Databases, Relational Databases and NoSQL

### Hierarchical Databases

[Information Management System](http://en.wikipedia.org/wiki/IBM_Information_Management_System) (IMS) by IBM was a very successful hierarchical database which is accessible over a **TCP/IP interface** and is available from a variety of languages. It was difficult to use but nevertheless achieved wide adoption. **IMS was built for use in the Saturn V moon rocket.**

Then a new model came along: relational databases. And the rest was history.

### Relational Databases

Dr. Edgar F.Codd at IBM wrote a paper **"A Relational Model of Data for Large Shared Data Banks"** which is still available at [http://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf](http://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf) became the foundational work for relational database management systems.

Then a lot of things happened. Then relational databases became a key facet of the modern technology and business landscape.

#### SQL as We All Known and Loved

SQL is a huge part of relational databases. It was first officially adopted as an ANSI standard in 1986, and is powerful for a variety of reasons:

- great expressive power
- easy to use
- easy integration with RDBMS

#### Transactions, ACID-ity, and two-phase commit

A database transaction is, as Jim Gray puts it, **“a transformation of state” that has the ACID properties** (see [http://research.microsoft.com/en-us/um/people/gray/papers/theTransactionConcept.pdf](http://research.microsoft.com/en-us/um/people/gray/papers/theTransactionConcept.pdf))

##### What is ACID?

> ACID is an acronym for **Atomic**, **Consistent**, **Isolated**, **Durable**, which are the gauges we can use to assess that a transaction has executed properly and that it was successful:

> **Atomic**
>
> Atomic means “all or nothing”; that is, when a statement is executed, every update within the transaction must succeed in order to be called successful. There is no partial failure where one update was successful and another related update failed. The common example here is with monetary transfers at an ATM: the transfer requires subtracting money from one account and adding it to another account. This operation cannot be subdivided; they must both succeed.
>
> **Consistent**
>
> Consistent means that data moves from one correct state to another correct state, with no possibility that readers could view different values that don’t make sense together. For example, if a transaction attempts to delete a Customer and her Order history, it cannot leave Order rows that reference the deleted customer’s primary key; this is an inconsistent state that would cause errors if someone tried to read those Order records.
>
> **Isolated**
>
> Isolated means that transactions executing concurrently will not become entangled with each other; they each execute in their own space. That is, if two different transactions attempt to modify the same data at the same time, then one of them will have to wait for the other to complete.
>
> **Durable**
>
> Once a transaction has succeeded, the changes will not be lost. This doesn’t imply another transaction won’t later modify the same data; it just means that writers can be confident that the changes are available for the next transaction to work with as necessary.

##### What's wrong with Transactions?

**Transactions are expensive and we might not need it all the time, and it become difficult under heavy load.** When you first attempt to horizontally scale a relational database, you must now account for _distributed transations_, where the transaction isn’t simply operating inside a single table or a single database, but is spread across multiple systems. **In order to continue to honor the ACID properties of transactions, you now need a transaction manager to orchestrate across the multiple nodes.**

In order to account for successful completion across multiple hosts, the idea of a **two-phase commit** (sometimes referred to as “2PC”) is introduced. But then, because two-phase commit locks all associate resources, it is useful only for operations that can complete very quickly.

Two-phase commit blocks, so clients must **wait for a prior transaction to finish before they can access the blocked resource.** After submitting a commit operation, the node will wait for the coordinator to send a commit response (or a rollback response if, say, a different node can’t commit).

> **So in order to account for these shortcomings in two-phase commit of distributed transactions, the database world turned to the idea of compensation.** Compensation, often used in web services, means in simple terms that the operation is immediately committed, and then in the event that some error is reported, a new operation is invoked to restore proper state.

> There are a few basic, well-known patterns for compensatory action that architects frequently have to consider as an alternative to two-phase commit. These include **writing off the transaction if it fails, deciding to discard erroneous transactions and reconciling later**. Another alternative is to **retry failed operations later on notification**.

> **In a reservation system or a stock sales ticker, these are not likely to meet your requirements. For other kinds of applications, such as billing or ticketing applications, this can be acceptable.**

Gregor Hohpe, a Google architect, wrote a wonderful and often-cited blog entry called **“Starbucks Does Not Use Two-Phase Commit.”** It shows in real-world terms how difficult it is to scale two-phase commit and highlights some of the alternatives that are mentioned here. [http://www.eaipatterns.com/ramblings/18_starbucks.html](http://www.eaipatterns.com/ramblings/18_starbucks.html)

#### Schema

Schema is how you map your domain objects into a relational model. Since we are application developers, we are all familiar with database schema and Object-Relational Mapping (ORM).

However:

> … **you are forced to create tables that don’t exist as business objects in your domain**. For example, a schema for a university database might require a Student table and a Course table. **But because of the “many-to-many” relationship here (one student can take many courses at the same time, and one course has many students at the same time), you have to create a join table.** This pollutes a pristine data model, where we’d prefer to just have students and courses. It also forces us to create more complex SQL statements to join these tables together. The join statements, in turn, can be slow.
>
> … **not all schemas map well to the relational model**. One type of system that has risen in popularity in the last decade is the **complex event processing system, which represents state changes in a very fast stream**. It’s often useful to contextualize events at runtime against other events that might be related in order to infer some conclusion to support business decision making. Although event streams could be represented in terms of a relational database, it is an uncomfortable stretch.

#### Sharding and shared-nothing architecture

A way to attempt to scale a relational database is to introduce sharding to your architecture.

> … The idea here is that you **split the data** so that instead of hosting all of it on a single server or replicating all of the data on all of the servers in a cluster, you divide up portions of the data horizontally and host them each separately.

There are three basic strategies for determining shard structure:

> **Feature-based shard or functional segmentation**
>
> Using this strategy, the data is split not by dividing records in a single table (as in the customer example discussed earlier), but rather by splitting into separate databases the features that don’t overlap with each other very much. For example, at eBay, the users are in one shard, and the items for sale are in another.
>
> **Key-based sharding**
>
> In this approach, you find a key in your data that will evenly distribute it across shards. A not-so-good example is you could divide your customer records across 26 machines, one for each letter of the alphabet.
>
> **Lookup table**
>
> In this approach, one of the nodes in the cluster acts as a “yellow pages” directory and looks up which node has the data you’re trying to access. **This has two obvious disadvantages.** The first is that you’ll take a performance hit every time you have to go through the lookup table as an additional hop. The second is that the lookup table not only becomes a bottleneck, but a single point of failure.

Sharding could be termed a kind of “shared-nothing” architecture that’s specific to databases. **A shared-nothing architecture is one in which there is no centralized (shared) state, but each node in a distributed system is independent, so there is no client contention for shared resources.** The term was first coined by Michael Stonebraker at University of California at Berkeley in his 1986 paper “The Case for Shared Nothing.”

Shared Nothing was more **recently popularized by Google**, which has written systems such as its Bigtable database and its MapReduce implementation that do not share state, and are therefore capable of **near-infinite scaling**. **The Cassandra database is a shared-nothing architecture**, as it has no central controller and no notion of master/slave; all of its nodes are the same.


### NoSQL

Why NoSQL?

Let's have a look at how we typically address relational database scaling problems:

> We typically address these problems in one or more of the following ways, sometimes in this order:
>
> - Throw hardware at the problem by **adding more memory, adding faster processors, and upgrading disks**.
> - … add hardware in the form of **additional boxes in a database cluster**. Now you have the problem of data replication and consistency during regular usage and in failover scenarios. You didn’t have that problem before.
> - Now we need to update the configuration of the database management system. This might mean optimizing the channels the database uses to write to the underlying filesystem. We turn off logging or journaling, which frequently is not a desirable (or, depending on your situation, legal) option.
> - Having put what attention we could into the database system, we turn to our application. We try to **improve our indexes. We optimize the queries.**
> - We employ a **caching layer**.
> - We turn our attention to the database again and decide that, now that the application is built and we understand the primary query paths, we can duplicate some of the data to make it look more like the queries that access it. **This process, called denormalization, is antithetical to the five normal forms that characterize the relational model, and violate Codd’s 12 Commandments for relational data.** We remind ourselves that we live in this world, and not in some theoretical cloud, and then undertake to do what we must to make the application start responding at acceptable levels again, **even if it’s no longer “pure.”**

It's now obvious that we have **mistakenly** learned over the years that **a relational database is a one-size-fits-all solution**.

> Because of some of the **inherent design decisions in RDBMS**, it is not always as easy to scale as some other, more recent possibilities that take the structure of the Web into consideration. But it’s not only the structure of the Web we need to consider, but also its phenomenal growth, because as more and more data becomes available, **we need architectures that allow our organizations to take advantage of this data in near-time to support decision making and to offer new and more powerful features and capabilities to our customers**.

Let's look at some statistics:

> - YouTube serves 100 million videos every day.
> - Chevron accumulates 2TB of data every day.
> - In 2006, the amount of data on the Internet was approximately 166 exabytes (166EB). In 2010, that number reached nearly 1,000 exabytes. An exabyte is one quintillion bytes, or 1.1 million terabytes. To put this statistic in perspective, 1EB is roughly the equivalent of 50,000 years of DVD-quality video. 166EB is approximately three million times the amount of information contained in all the books ever written.
> - Wal-Mart’s database of customer transactions is reputed to have stored 110 terabytes in 2000, recording tens of millions of transactions per day. By 2004, it had grown to half a petabyte.
> - **The movie Avatar required 1PB storage space**, or the equivalent of a single MP3 song—if that MP3 were 32 years long (source: [http://bit.ly/736XCz](http://bit.ly/736XCz)).
> - As of May 2010, Google was provisioning 100,000 Android phones every day, all of which have Internet access as a foundational service.
> - In 1998, the number of email accounts was approximately 253 million. By 2010, that number is closer to 2 billion.

Considering the now we faced "Web Scale" problem, here is a piece of interesting historical story:

> It has been said, though it is hard to verify, that the 17th-century English poet **John Milton** had actually **read every published book on the face of the earth**. Milton knew many languages (he was even learning Navajo at the time of his death), and given that the total number of published books at that time was in the thousands, this would have been possible. The size of the world’s data stores have grown somewhat since then.

**In a world now working at web scale and looking to the future, NoSQL is a great tool at hand for us developers to tackle the scaling problem.**

Apache Cassandra might be one part of the answer.

But also we must be aware of the **NoSQL-ALL-THE-THINGS** attempts:

> If massive, elastic scalability is not an issue, the trade-offs in relative complexity of a system such as Cassandra may simply not be worth it.

## The Cassandra Elevator Pitch

This is a summary of what Cassandra is all about. Since I am not familiar with it (yet), it's basically just quotes from the book.

### Cassandra in 50 Words or Less

> **Apache Cassandra is an open source, distributed, decentralized, elastically scalable, highly available, fault-tolerant, tuneably consistent, column-oriented database that bases its distribution design on Amazon’s Dynamo and its data model on Google’s Bigtable. Created at Facebook, it is now used at some of the most popular sites on the Web.**

### Distrubuted and Decentrailized

> … Cassandra is distributed, which means that it is capable of running on multiple machines while appearing to users as a unified whole. **In fact, there is little point in running a single Cassandra node.**
>
> … You can **confidently write data to anywhere in the cluster** and Cassandra will get it.
>
> Once you start to scale many other data stores (MySQL, Bigtable), some nodes need to be set up as masters in order to organize other nodes, which are set up as slaves. Cassandra, however, is decentralized, meaning that every node is identical; no Cassandra node performs certain organizing operations distinct from any other node. **Instead, Cassandra features a peer-to-peer protocol and uses gossip to maintain and keep in sync a list of nodes that are alive or dead.**
>
> In many distributed data solutions (such as RDBMS clusters), you set up multiple copies of data on different servers in a process called replication, which copies the data to multiple machines so that they can all serve simultaneous requests and improve performance. **Typically this process is not decentralized as in Cassandra, but is rather performed by defining a master/slave relationship.**
>
> … **The decentralized design is therefore one of the keys to Cassandra’s high availability**. Note that while we frequently understand master/slave replication in the RDBMS world, **there are NoSQL databases such as MongoDB that follow the master/slave scheme as well.**


### Elastic Scalability

> **Scalability is an architectural feature of a system that can continue serving a greater number of requests with little degradation in performance.**

> Elastic scalability refers to a special property of horizontal scalability. It means that your cluster can seamlessly scale up and scale back down. … You don’t have to restart your process. You don’t have to change your application queries. You don’t have to manually rebalance the data yourself. **Just add another machine—Cassandra will find it and start sending it work.**

### Tuneable Consistency

**Consistency essentially means that a read always returns the most recently written value.**

> … scaling data stores means making certain trade-offs between **data consistency**, **node availability**, and **partition tolerance**.

**Cassandra is frequently called “eventually consistent”.**

> Eventual consistency means on the surface that all updates will propagate throughout all of the replicas in a distributed system, but that this may take some time. Eventually, all replicas will be consistent.

When considering consistency, availability, and partition tolerance, we can achieve **only two of these goals** in a given distributed system (See **CAP Theorem** later in this article). At the center of the problem is data update replication. To achieve a strict consistency, all update operations will be performed synchronously, meaning that they must block, locking all replicas until the operation is complete, and forcing competing clients to wait. A side effect of such a design is that during a failure, some of the data will be entirely unavailable. **As Amazon CTO Werner Vogels puts it, “rather than dealing with the uncertainty of the correctness of an answer, the data is made unavailable until it is absolutely certain that it is correct”** ("Dynamo: Amazon’s Highly Distributed Key-Value Store” :[http://www.allthingsdistributed.com/2007/10/amazons_dynamo.html](http://www.allthingsdistributed.com/2007/10/amazons_dynamo.html))

We could alternatively take an optimistic approach to replication, propagating updates to all replicas in the background in order to avoid blowing up on the client. **The difficulty this approach presents is that now we are forced into the situation of detecting and resolving conflicts.** A design approach must decide whether to resolve these conflicts at one of two possible times: during reads or during writes. **That is, a distributed database designer must choose to make the system either always readable or always writable.**

**Dynamo and Cassandra choose to be always writable**, opting to defer the complexity of reconciliation to read operations, and realize tremendous performance gains. The alternative is to reject updates amidst network and server failures.

> **In Cassandra, consistency is not an all-or-nothing proposition, so we might more accurately term it “tuneable consistency” because the client can control the number of replicas to block on for all updates.** This is done by setting the _consistency level_ against the _replication factor_.

> The __replication factor__ lets you decide how much you want to pay in performance to gain more consistency. **You set the replication factor to the number of nodes in the cluster you want the updates to propagate to** (remember that an update means any add, update, or delete operation).

> The __consistency level__ is **a setting that clients must specify on every operation** and that allows you to decide how many replicas in the cluster must **acknowledge a write operation** or **respond to a read operation** in order to be considered successful. **That’s the part where Cassandra has pushed the decision for determining consistency out to the client.**

> So if you like, you could **set the consistency level to a number equal to the replication factor**, and **gain stronger consistency at the cost of synchronous blocking operations that wait for all nodes to be updated and declare success before returning**. This is not often done in practice with Cassandra, however, for reasons that should be clear (it defeats the availability goal, would impact performance, and generally goes against the grain of why you’d want to use Cassandra in the first place). **So if the client sets the consistency level to a value less than the replication factor, the update is considered successful even if some nodes are down.**

### Brewer’s CAP Theorem

> While working at University of California at Berkeley, Eric Brewer posited his CAP theorem in 2000 at the ACM Symposium on the Principles of Distributed Computing. The theorem states that within a large-scale distributed data system, there are three requirements that have a relationship of sliding dependency: **Consistency**, **Availability**, and **Partition Tolerance**.
>
> **Consistency**
>
> All database clients will read the same value for the same query, even given concurrent updates.
>
> **Availability**
>
> All database clients will always be able to read and write data.
>
> **Partition Tolerance**
> 
> The database can be split into multiple machines; it can continue functioning in the face of network segmentation breaks.

Brewer’s theorem is that in any given system, you can strongly support only two of the three. **This is analogous to the saying you may have heard in software development: “You can have it good, you can have it fast, you can have it cheap: pick two.”**

The CAP theorem was formally proved to be true by Seth Gilbert and Nancy Lynch of MIT in 2002. In distributed systems, however, it is very likely that you will have network partitioning, and that at some point, machines will fail and cause others to become unreachable. Packet loss, too, is nearly inevitable. **This leads us to the conclusion that a distributed system must do its best to continue operating in the face of network partitions (to be Partition-Tolerant), leaving us with only two real options to choose from: Availability and Consistency.**

**Where different databases appear on the CAP continuum**

![Where different databases appear on the CAP continuum](http://forresty.com/images/CAP-databases.png)

> **In this depiction, relational databases are on the line between Consistency and Availability, which means that they can fail in the event of a network failure (including a cable breaking).** This is typically achieved by defining a single master server, which could itself go down, or an array of servers that simply don’t have sufficient mechanisms built in to continue functioning in the case of network partitions.

What does it mean in practical terms to support only two of the three facets of CAP?

> **CA**
>
> **To primarily support Consistency and Availability means that you’re likely using two-phase commit for distributed transactions.** It means that the system will block when a network partition occurs, so it may be that your system is limited to a single data center cluster in an attempt to mitigate this. If your application needs only this level of scale, this is easy to manage and allows you to rely on familiar, simple structures.
>
> **CP**
>
> To primarily support Consistency and Partition Tolerance, you may try to advance your architecture by setting up data shards in order to scale. Your data will be consistent, **but you still run the risk of some data becoming unavailable if nodes fail.**
>
> **AP**
>
> To primarily support Availability and Partition Tolerance, your system **may return inaccurate data**, but the system will **always be available**, even in the face of network partitioning. **DNS is perhaps the most popular example of a system that is massively scalable, highly available, and partition-tolerant.**

### Row-Oriented

> … although it’s not wrong to say that Cassandra is columnar or column-oriented, it might be more helpful to think of it as an indexed, row-oriented store.

**Cassandra stores data in what can be thought of for now as a multidimensional hash table.** That means you don’t have to decide ahead of time precisely what your data structure must look like, or what fields your records will need. This can be useful if you’re in startup mode and are adding or changing features with some frequency. It is also attractive if you need to support an Agile development methodology and aren’t free to take months for up-front analysis.

> … Cassandra requires a shift in how you think about data. Instead of designing a pristine data model and then designing queries around the model as in RDBMS, **you are free to think of your queries first, and then provide the data that answers them**.

### Schema-Free

**Cassandra requires you to define an outer container, called a keyspace, that contains column families.** The keyspace is essentially just a logical namespace to hold column families and certain configuration properties. The column families are names for associated data and a sort order. **Beyond that, the data tables are sparse, so you can just start adding data to it, using the columns that you want; there’s no need to define your columns ahead of time.** Instead of modeling data up front using expensive data modeling tools and then writing queries with complex join statements, Cassandra asks you to model the queries you want, and then provide the data around them.

## Use Cases for Cassandra

- Large Deployments
- Lots of Writes, Statistics, and Analysis
- Geographical Distribution
- Evolving Applications

## Who is Using Cassandra?

- **Twitter** is using Cassandra for analytics. In a much-publicized blog post (at [http://engineering.twitter.com/2010/07/cassandra-at-twitter-today.html](http://engineering.twitter.com/2010/07/cassandra-at-twitter-today.html), Twitter’s primary Cassandra engineer, Ryan King, explained that **Twitter had decided against using Cassandra as its primary store for tweets**, as originally planned, but would instead use it in production for several different things: for real-time analytics, for geolocation and places of interest data, and for data mining over the entire user store.
- Mahalo uses it for its primary near-time data store.
- **Facebook** still uses it for inbox search, though they are using a proprietary fork.
- Digg uses it for its primary near-time data store.
- Rackspace uses it for its cloud service, monitoring, and logging.
- **Reddit** uses it as a persistent cache.
- Cloudkick uses it for monitoring statistics and analytics.
- Ooyala uses it to store and serve near real-time video analytics data.
- SimpleGeo uses it as the main data store for its real-time location infrastructure.
- Onespot uses it for a subset of its main data store.


## Interesting Quotes from Book

If at first the idea is not absurd, then there is no hope for it. -- Albert Einstein

If I had asked people what they wanted, they would have said faster horses. -- Henry Ford

If you can’t split it, you can’t scale it. -- Randy Shoup, Distinguished Architect, eBay

An invention has to make sense in the world in which it is finished, not the world in which it is started. -- Ray Kurzweil
