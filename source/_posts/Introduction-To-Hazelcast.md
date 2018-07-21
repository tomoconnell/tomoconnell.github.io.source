---
title: Introduction To Hazelcast
date: 2018-07-13 00:23:17
tags:
---
## Table of Contents
### Introduction to Hazelcast    
### Overview    
1.    Hazelcast, Briefly    
    1.1.    Clustered    
    1.2.    In Memory   
    1.3.    Data Grid    
    1.4.    With Sharding    
    1.5.    And Monitoring    
2.    What’s it for?    
3.    Before you start    
    3.1.    Java 8    
    3.2.    IDE    
    3.3.    Maven    
4.    Maven Dependencies    
    4.1.    Server    
    4.2.    Client    
5.    Programming Models    
    5.1.    Embedded Member    
    5.2.    Dedicated Member    
    5.3.    Lite Member    
    5.4.    Clients    
6.    A Simple Spring Boot Cluster Member    
7.    A Simple Client    
8.    Configuring Hazelcast    
9.    A Slightly More Robust Server    
10.    A Slightly Better Client    
    10.1.    Simple Map Access    
    10.2.    Near Cache Access    
    10.3.    Simple Query Operations    
11.    Aggregations    
    11.1.    Entry-Processors    
    11.2.    Runnable Tasks    
    11.3.    Callable Tasks    
12.    Events    
13.    Interceptors    
14.    What Just Happened?    



## Introduction to Hazelcast
### Overview
### Hazelcast, Briefly
Really briefly, Hazelcast is a clustered, in-memory data-grid that uses sharding for data distribution and supports monitoring.
### Clustered
Clustering refers to how some network-centric software remains resilient and highly available. Because of the clustering – i.e. the discovery process – you can just start processes, either members or clients, as you need and they find each other and form a consistent whole. Members join, the load spreads out; members terminate and the load is absorbed by the others.
### In Memory
The sweet spot of in-memory storage – obviously – is purely in-memory. This is an ideal use-case for Hazelcast. You can scale the storage in a number of ways and they all work well. For the open-source edition, you pick your JVM size, based upon your own testing and tuning – there are no good portable recommendations for that, you just have to test. You pick your backups – I don’t think I’ve ever gone off the default (1 backup) for in-memory storage. If it’s worth storing, it’s probably worth having a backup, but more than one backup is probably not a useful idea. Consider adding persistence, if you think you need multiple backups. Persistence takes the IMDG out of the purely in-memory range, but gives you access to huge persistent storage – anything that you can hit from a Java process can be incorporated into your persistence architecture. I don’t think I’ll have time to talk about this in any meaningful depth here.
### Data Grid
It’s a data grid. That means a lot of different things and marketing terminology can obscure this, but basically, it’s data held in caches that is available for retrieval by members or clients, processing – either in-place or in different processes, that can support events, triggers, transformations and basically, anything you can think up and code.
The tools available in this grid, particularly are the most robust among IMDG frameworks. Everyone supports some flavor of map or cache, but this tools provides a really wide array of distributed processing possibilities. There are maps, queues, lists, sets – all extending the collections classes. There are also RingBuffers and Multimaps. Alongside queues, there are topics and even reliable topics, for more messaging options. The available concurrency utilities include locks, semaphores, atomic-longs, atomic-references, the id-generator and a countdown-latch. CRDTs (conflict free replicated data-types) are being added, starting with the PN-counter. I don’t think that there’s really any competition in this area.
### With Sharding
Sharding – Hazelcast calls it partitioning – is a means of horizontally partitioning the data across multiple member processes. You can think of the Hazelcast shards (partitions) as the hash buckets of a distributed hash map. Each cluster uses a configured number of partitions – the default is 271 and that often need not be changed. A single member “cluster” (non-clustered cluster?) would get all the partitions, without backups. As the first member joins, roughly half the partitions are transferred to that new member and backups are created at that time. As each subsequent member joins, some basically equal fraction of the partitions are transferred to that member – both primary data and backups.
When members leave, the backups become the primary data partitions and new backups are created on the remaining members.
### And Monitoring
I’m pretty sure that someone will argue that monitoring doesn’t belong here, but I’ve been working with distributed systems for a while and I really feel that the lack of monitoring is the first step toward failure. Headless systems are often not well understood and sometimes ignored by ops – and who can blame them. If we give them a black box with no outputs, there’s really nothing to do. Hazelcast supports JMX and a management console, so easy monitoring is available; you see issues coming in advance of major problems and – better yet – you can set alerting thresholds that will allow the system to call for help.
### What’s it for?
It’s for almost any programming task – broadly speaking, the three major areas are caching, distributed processing and distributed messaging. The central theme of all the applications is big, fast data. Big data is good, big fast data is awesome. Hazelcast has been around long enough to have serious adoption in various industries – airlines, telcos, finance, crypto currency applications and on and on. Does it need to be big? Not really – it’s nice to have an infrastructure that can start small and grow enormously.
The processing capabilities have been enormously augmented by Jet – fast stream and batch processing, built on top of the IMDG. That’s a deep topic, all on its own – and will be covered in many other places.

3. Before you start
There are only a few things you need, to get going -
3.1 Java 8
Java 8 is probably the most widely used jdk/jre right now and is the preferred one to start with.
3.2 IDE
Eclipse, IntelliJ and NetBeans seem to make up about 97% of the IDE users and any of them work well with Hazelcast/maven. If you’re not using an IDE, you may want to re-think what you’re doing. The sample code in the github repo was tested in eclipse, as well as from the command line, so it’s an easy import.
3.3 Maven
There are lots of build tools. Maven works well enough and it doesn’t bother me. Gradle is probably better, but I’m not really a fan – but then, I really liked make. Maven is really pretty adequate for this.
Maven Dependencies
All the dependencies for Hazelcast – any edition – are available on public maven repositories, in addition to download. You can download the open-source code, too, for that matter.
Server
The server will be in one of two forms ‘hazelcast’ or ‘hazelcast-all’, which also includes client dependencies.
Client
The Hazelcast client is generally included from ‘hazelcast-client’, and that’s the only addition to your client app build.
Programming Models
Hazelcast is a toolkit – I’ll probably say that more than once. There are common patterns you can employ, but – really – it’s just java. You can design your own infrastructure to meet your needs in any way you see fit. I’d like to just talk a little about some common deployment models.
Embedded Member
Embedded members are really the easiest way to get started and, for some things – they may be all that you need. An embedded client is a java program, where you create a HazelcastInstance. That will load the launch the framework and form a cluster with any compatibly configured members they find on the network – that all depends upon your network and discovery configuration.
Dedicated Member
A dedicated member is a Hazelcast process dedicated to storage and a few other things. It won’t run your code, except for a few server-side-specific instances – entry processors, Executor Tasks (Callables and Runnables), event code (Listeners and Interceptors) and persistence code (MapLoader and MapStore).
The advantage of this approach, over the embedded model is that scalability will always become more important than simplicity. With this, you can scale your storage fleet separately from your client fleet. If your storage demands soar, but the processing doesn’t, you just scale these members. If you introduce new processing demands for the same, or similar, data loads, you just add clients. 
In general, this is going to be the best basic approach to almost any IMDG system.
Lite Member
Lite members are interesting – they join the cluster – unlike clients that just make a client-specific TCP connection. The do not, however, host data. They are for a small number of kinda advanced things – they may be used for class-loading or they may be used as high-performance compute clients. You could, for example direct runnable and callable tasks to Lite members – if they require data that’s spread across the cluster for some kind of computation or processing.
Clients
Clients are java programs that include the client (i.e. Hazelcast-client) jar in their build, read or create config that helps them find a cluster and perform the widest scope, typically, of client requests. These will be in your web-clients, your command-line tools or anywhere you need to interface your systems with Hazelcast. Don’t think, though, that because they’re clients, your going to be doing all your processing there. Well written clients will use server-side constructs – particularly entry-processors, aggregation and executor tasks, to delegate processing requests from (could be) single-threaded clients onto a (should be) massively scalable clustered storage and processing environment.
Not everything will be delegated to the back-end, of course. Many, many clients simply require extremely low-latency access to fast big data that isn’t changed too often and isn’t changed (ideally) by separate clients (i.e. sticky sessions are good). For these near-caches are extremely effective. Each member can host – within its process space – potentially large sub-sets of data that is being actively managed by the cluster. We’re talking mostly about the open-source version here, but it’s worth noting that the enterprise HD edition will allow off-heap near-caches, giving you low latency access to – potentially – many gigabytes of near-cache data in each client. This has a broad range of applications, across industries – real-time inventory for e-commerce and fraud-detection for credit-card processors are two interesting ones. Note that in neither of these, is the data static – that’s not a requirement. But the data is read much more often than it’s changed, making both of these ideal cases for near-caching.



A Simple Spring Boot Cluster Member
So, finally a little more code. All of this is on github, so you can look at the POM (pom.xml) file there. It’s really basic, you need the spring-boot parent entry and just the Hazelcast dependency (from above).
Make you main class a spring boot application with the “@SpringBootApplication” annotation. I’m adding the “@Configuration” annotation, so I can have one class that serves up the beans and executes them. 

A Simple Client
As I’ve said, 
Configuring Hazelcast
So, we already ran code – why talk about configuration now? Because the really simple examples take all the defaults. While they’re interesting to run, you wouldn’t really go much past the hello-world kind of thing with that.
The main things you’ll want to change are the network configuration – particularly the join configuration – and the definition of maps and caches.
The join configuration dictates how members find each other. It starts with multicast – which probably won’t fly in most prod environments – as multicast traffic is generally frowned upon by network administrators. TCP discovery is great, if you know the addresses and they don’t change (much). If you have a data-center with dedicated hardware, that could work really well. In a cloud, however, this falls apart – you don’t know the IPs up front and they’re pretty much guaranteed to change. There are cloud-based cluster discovery plugins that will make that work well and easily, but that’s too much for this intro.
 Maps  (IMAP) are the workhorse distributed storage data-structure and caches (ICache) are basically the same thing, but for the JCache (JSR107) API. By default, you can create a map using default configuration and you get a container with no limitations on it. The data is not bounded and it doesn’t expire and it isn’t evicted – the problems with that should be obvious. The system will store and retain data until it runs out of memory and that’ ugly. For any IMDG, you want to be careful about managing memory and Hazelcast has lots of options, using either a number of objects in the cache, or an absolute size of memory that can be used, or thresholds on the heap-utilization that will trigger eviction. In addition to evicting on space, you can set an expiration interval on your data – you decide up front, how long the data is likely to be worth caching – i.e. your online session data is rarely, if ever, good for a full day and probably isn’t worth caching for even an hour (unless it’s being accessed).
A Slightly More Robust Server
We can do better on the server code.

That’s a fairly concise server – even allowing for my idiosyncratic indentation (I like white-space). Everything that it’s going to do is going to be injected by Spring Boot.
It relies on some configuration code -



Spring Boot will create a default instance of Hazelcast – which may not give you what you want. I like having a bean for the config and one for an instance. 
Here’s the CommandLineRunner that makes the spring app work -

 


A Slightly Better Client
We can do more with the client code. The application will look like 
Simple Map Access
This part is easy – a Hazelcast IMap “is a” java.util.Map – so you can take existing code for the java collections API and just repurpose it. I don’t want to get too far astray, but here’s a little code showing how easy that can be.



In that bit, there’s a method that creates a map and uses it – sort of trivially, but it’s working code. 
In the second method – “hzMethod” –  the only change was to use the injected Hazelcast instance (injected, via annotation), to get a reference to a distributed map in the IMDG. There’s no magic – Hazelcast is designed so that you can swap it in that easily, using the – really better be – familiar collections API. But, back to that comment for a second – should the declaration have been IMap, not Map? It depends – it could be, but it doesn’t need to be. Hazelcast maps implement the java.util implements, so that’s valid – but maybe not useful. In a minute, we’re going to use some hz specific methods on the map and to make those visible, you’d want to change the declaration. If you’re just doing put, get, size, remove and all of those, then no. One interesting note on that – it’s easy to forget that ‘put’ returns the old mapping, as it inserts the new. I don’t think I’ve ever actually seen code that used that, but, there it is. Think about that in a network environment, though – when you do a ‘put’, hz – conforming to the contract – returns the old mapping. Over the network. Incurring serialization. For no reason. Because nobody. Ever. Looks. At. It. Hazelcast has added a ‘set’ method, that works like ‘put’, save that it doesn’t return the value. This may seem like small stuff, but think about a heavily utilized production environment, getting a surge of requests; you’re busy and half of that flavor of network traffic is stuff you’re never going to look at. Change two letters in your code and the network traffic drops – possibly by lots. I’d do it.
Keep in mind, however, that there are differences. It’s a distributed map – absent security configuration, other clients/other threads can use the same map. If you test the size of a new in process map, that you create in your thread, the size will be ‘0’. When you get a reference to a distributed collection from the IMDG, it will create it – if required – or return a reference to an existing collection, if it’s already been created. This can be a very powerful feature – you can pre-populate a collection, from a persistent store or any other data-source. Your client code will be smaller and simpler, because you can make assumptions about it. If you’re using a map for a scratchpad cache, however, keep in mind that you may want to create unique map instances or manage data, so that your thread doesn’t collide with other clients.


Near Cache Access
Hazelcast supports second-level, or edge caching in client processes and refers to it as ‘near caching’. Near caches are almost transparent to your code – there are things you need to be aware of, though.
Each mapping in the near cache is, fundamentally, managed by the IMDG member that owns the ‘master’ copy of the data. It may be cached on multiple clients – in multiple apps – and each caching client app may define their own policy for managing updates. The data in your near cache may be stale – you probably shouldn’t cache things for overly long times (you set the expiry interval in config). You should be careful about using near caches for things that are updated frequently and very careful about using them for things that are updated from multiple points. For a web application with sticky sessions, you should be able to count on certain objects being in only one client process – that’s a good scenario. 
Simple Query Operations
SQL Queries
Hazelcast is not an SQL database or – really – a SQL query tool, but it provides a very workable, robust subset of SQL query functionality. It’s very accessible for developers. If you have an SQL background, this is nothing; if you don’t, it’s still pretty intuitive. The SqlPrecicate encapsulates the ‘where’ clause of a query. For IMDG data extraction, that’s probably the part you really care about. Having said that it’s not a relational database, I should point out that one of the nice distinctions there is that, since you’re dealing with – generally – purely in memory data, this is going to be very fast.



I think that shows how easy it is to bridge an SQL background, with the IMDG SQL-like query. The big caveat here is joins – out of the box, the IMDG is not a really good tool for joins, because of the nature of the data. We split it up, because it’s big data; because it’s big data, joining it back together is a more specialized thing. I mentioned  Jet earlier; that’s the tool to use for that. Again, that’s too much to fit in here. Hazelcast can certainly do joins, but not in an intro that’s already seeming kind of wordy to me.


Predicate Queries – Criteria API
For Java developers who really never liked SQL – lots of people – there’s also a pure java approach to querying the IMDG –  the criteria API.



It’s probably obvious – but let me talk about what that’s doing. From a client, you’re setting up arbitrarily complex criteria – in both the SQL and the criteria APIs – and dispatching the evaluation to the grid. I think that’s pretty cool. I’ve seen people do things like bring the entire map to a client, to iterate over it, for reasons. Bad reasons. With hz, you describe what you want back and dispatch the evaluation and extraction to your entire server fleet, seeing only correct results. This is a reporting or analysis kind of thing – I wouldn’t look up everyone whose last name starts with ‘D’, to do any kind of processing where the employee object would be updated, of course. Does this make the IMDG a database? Nope. Does it give you highly leveraged tools for fast access to big data? Yep. 

One thing that, looking at this, really needs to be mentioned, if only briefly, is memory in the query client. I’ve described hz as a fast big-data framework, so clearly, the query operations can quickly retrieve very large volumes of data. If you had, say 20 storage members and queried the cluster and brought back 1GB, or so, of data from each, you’d be looking at 20GB of data from one query. You may, or may not, have that much memory available in your client. The fix for that is paging-predicates. These predicates wrap (logically subsume the functions of) other predicates, so that you get the same logical comparisons – the same filtering – wrapped in a container that lets you bring results back in batch sizes that you specify – it’s as though you’re reading a printed book and seeing one page at a time.


This was only taking the previously used predicate and returning the results in paged sets, with the page-size set to 10 in this example.



Aggregations
Aggregations, or as they’re now called “fast aggregations” allow data query and transformation to be dispatched to the cluster. It can be extremely effective.
I’m going to keep using the Employee class and the “employees” map that I had in the other examples and do a quick and dirty aggregation. I could just do the aggregation across the entire entry-set of the map, but I like using a predicate to filter, or map, the objects, before the aggregator does the reduction on them. Department 13 is used to represent group-W – you know those people; they’re everywhere. 

This is going to create an anonymous class, which is quick and easy and highly effective for this.



That code is probably pretty self-explanatory, let me make sure.
The predicate will ensure that only matching elements from the distributed map are included in the calculation. The aggregator will ‘accumulate’ data – examining the matching subset and adding the age into the sum – but, where does that happen. The accumulate call is called on each storage member (i.e. not the clients and not Lite members); it’s passed each filtered (by deptPredicate) matching entry and it accumulates the raw values. Note that these run in parallel on each member involved. Because the data is partitioned across members and only a filtered sub-set is processed, it’s going to be very fast.  In the second phase, each of these aggregator instances are returned to the caller for processing – the instance of the anonymous aggregator class examines each returned aggregator (instances of the same anonymous class) and combines all the raw results. In that part of the code, because this wasn’t a concrete class, it’s necessary to call the “class.cast()” method, to allow access to the count and the sum. The final step, also in the calling member, the ‘aggregate’ method is invoked and performs the simple calculation of average age. I think that this is really not much code, for that kind of power.

There are also a very complete set of built in aggregators for things like min/max/count/avg of different numeric types, distinct for any comparable type and so on. 
They can be used with very little setup, like this -



Implicit there was a static import of ‘distinct’, but if you’re reading this, I guess you got that.


Entry-Processors
Entry processors are really pretty cool – you can do highly efficient in-place processing with a minimum of locking. Consider what people often end up doing to work with remote objects – lock a key, fetch the value, mutate the value, put it back then (in a finally block, I hope) unlock the key. That’s four network calls, to start with – three if you’re only looking at the data and not updating the central source. Your objects may be large and incur significant cost in terms of CPU and network for serialization and transport. They may not, but I’m trying to make a point.
Entry processors allow you to dispatch a key-based ‘task’ object across the LAN – directly to the member owning a key, where it is executed in a lock-free, thread-safe fashion. Too good to be true? Not really – Hazelcast has an interesting threading model that allows this to happen. That’s a topic worth its own discussion, too, so take my word for it, for now.

Here’s a brain dead simple entry processor example – but it’s still a really useful approach:



Here’s the entry processor that is called in this –


I’m describing the simplest case, but as simple as it is, this is a very powerful tool to have. From your client, you can dispatch computation to your servers, no locking, no contention, efficient user of the LAN. But – you’re waiting for the result to be returned, so it may not seem like a really big deal. Hazelcast also has a number of options to run synchronously, asynchronously and on multiple entries.


This is a really useful set of calls – the first one ‘executeOnKey’ – does exactly that; it makes one direct call to the key owner and executes synchronously on that entry. The next two execute asynchronously – your client code doesn’t need to wait for long running operations. A word of warning, though – long running entry-processors can be truly evil – use off-loadable in your code, by annotation, to tell hz to move the operation off of the default threading structure.

Here’s an async call – similar in function to the first.



I like this one – submit the processing, tag it with a callback and be on your way. Where would you use that? Broad question – it really depends – but if you have a stream of data requests that need to be initiated without the caller needing to see a result, this may be perfect. If you’re going to do this, be sure to read ahead to the section on back-pressure.

The execution callback executes in the caller’s process space, so it’s your notification that the invocation is complete. This one just logs, like this:


Because nothing is free – there’s a little bit of server-side configuration to go with this. Back-pressure, both in the client and in server members is important, when using asynchrony. Configuring it can be done with system properties, like this:



Back-pressure is a topic worth some consideration. Hz is using its threading model to execute these, so there’s a limit to how many invocations can be in flight at any one point in time. The absolute number doesn’t matter – that would depend upon the size of your cluster and the number of CPUs/cores/physical threads. What’s likely to be interesting is how many can be queued up for one partition – by default, mutating entry-processors operate on a ‘partition thread’. I don’t want to open a really big topic, but in configuring your cluster, you know how many physical threads you have, so you can configure the partition thread count to be some sensible number. Too few and you’ll have contention, too many – one-to-one sounds ideal; it rarely is – and you won’t get good resource utilization. I won’t even try to estimate that kind of configuration here – knowledgeable performance experts advocate leaving it the hell alone, unless and until you really know what the hell you’re doing.


Runnable Tasks
These are simply java runnable objects that are dispatched to one or more members. Keep in mind – the salient part of the signature is “public void run()” – i.e. nothing is returned. The way that they’re dispatched to the members is very flexible – it can be one member, all members, a member that owns a key or you can select members by an attribute you have set on them.
Here’s an example of running something on the member that owns a key –


The Runnable object is pretty ordinary, with the caveat that it needs to be serializable. This is an interesting topic and I’m going to touch on it in a minute. This runnable is also ‘HazelcastInstanceAware’, so that, when it’s set up on the target node, the hz framework will inject the correct instance, so that it can communicate with the cluster.


This code wasn’t particularly profound, but there’s one cool aspect to it – you can direct processing to a member that owns a key (or other members) and process that key and or other keys in multiple maps. So, complex manipulation may be performed outside your client, eliminating multiple network round-trips. They data need not come all from one member, either – there are no restrictions on that. It can be significant performance boost to design your data, so that related items are all within one node – then this kind of task will tend not to make network calls but will not be restricted from doing so.


Callable Tasks
As with runnable, callable tasks are dispatched to one or more members, but offer more options for things like bringing back data.
Here’s a really simple callable that will be dispatched to a member, log some noise to show it ran, and return the partition count. There are better ways to monitor or manage partitions, but this should just show how you get a value – easily – from a member.



That’s just looking at the member, which you might want to do. Importantly, If you wanted to get/set/remove data, run a query, or any other hz operation, you can do that from that code.
The call is easy – as with the collections, it draws heavily from the Java Executors API 


Events
There are lots of events, but for just right now, I only want to talk about some of the data events – listeners and interceptors. This is still a fairly big topic, so let’s talk about a workable subset of it. Within data-data events, there are what are called ‘map events’ and ‘entry events’. Map events are called for map-level changes, specifically “clear()” or “evictAll()”. Entry events are called after changes to map-entries and there’s an interesting set of those changes – there are events for entries being added, removed, updated and evicted. No listener for get, however. Does that seem odd? It may. The entry events are only for changes – in that context, it makes sense (there’s an interceptor for ‘get’ – just wait).
Events may be added in a number of ways – they can be added in configuration, or programmatically. In addition to that, they can be done from a member or a client. Within each member, you get local events – events triggered by (after) data-mutating events within that JVM. 
Here's a simple example of listening for entry



Adding the entry-added listener could be done in config, but I like the Java API for doing it, personally -


This code will add the entry listener – listening only for entries being added. The boolean parameter tells hz that the value should be available (i.e. “getValue()”) in the entry-event that’s going to be delivered to the listener.

Clients may add these listeners, also – in addition to client lifecycle events, cluster membership events and distributed object creation/deletion. So, they may be notified of their own client-lifecycle: starting, started, shutting-down and shutdown; they may be notified of membership changes, storage members joining and leaving and they may be notified of distributed object creation or destruction – maps, caches, queues and all. A word of caution, though – high volume activity will create high volumes of events – look carefully at your resources: client CPU/RAM and especially network. Think in a distributed perspective and put the listener where it needs to be, not simply where it seems convenient.
Interceptors
Interceptors are closely related to events, but have some differing semantics
What Just Happened?
This may be a little more than a simple intro, but it’s also just a little of what you can do with Hazelcast. I hope that you get a few things from this – like how much I like Hazelcast. I’ve been doing distributed systems for what seems like a really long time – anyone still using ARCNet messaging on  MP/M? I hope not; I really do. I’ve done distributed systems before there were cool frameworks like Hazelcast and I’ve used most of the competition, too. This one is my choice – it gives performance and simplicity. You can be up and running in minutes and rolling out production quality code that looks an awful lot like your Java collections code. It’s a fun environment for programmers. A little Java gets can be all you need on the server side, then you can cut loose with Java, .NET, C++, Node.js, Python, Go or Scala – and that list is going to grow, as new languages emerge.
But, anyway – we stood up storage members and a couple clients, with a functional subset of features from Hazelcast. There was no production code here, but I think it shows the path.



