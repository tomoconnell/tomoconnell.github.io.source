---
title: Introduction To Hazelcast
date: 2018-07-13 00:23:17
tags:
---

# Overview
## Hazelcast, Briefly
Really briefly, Hazelcast is a clustered, in-memory data-grid that uses partitioning for data distribution and supports monitoring.

### Clustered
Clustering refers to how some network-centric software remains resilient and highly available. Because of the clustering --- i.e. the discovery process --- you can just start processes, either members or clients, as you need and they find each other and form a consistent whole. Members join, the load spreads out; members terminate and the load is absorbed by the others.

### In Memory
The sweet spot of in-memory storage --- obviously --- is purely in-memory. This is an ideal use-case for Hazelcast. You can scale the storage in a number of ways and they all work well. For the open-source edition, you pick your JVM size, based upon your own testing and tuning (there are no good portable recommendations for that). You pick your backups; you'll rarely need to go off the default (1) backup for in-memory storage. 

### Data Grid
It's a data grid. That means a lot of different things and marketing terminology can obscure this, but basically, it's data held in caches that is available for retrieval by members or clients; processing --- either in-place or in different processes --- that can support events, triggers, transformations and basically, anything you can think up and code.
The tools available in this grid are the most robust among IMDG frameworks. Vendors in this space all support some flavor of map or cache, but this tool provides a really wide array of distributed processing possibilities. There are maps, queues, lists, sets --- all extending the collections classes. There are also RingBuffers and Multimaps. Alongside queues, there are topics --- and even reliable topics --- for more messaging options. The available concurrency utilities include locks, semaphores, atomic-longs, atomic-references, the id-generator and a countdown-latch. CRDTs (conflict free replicated data-types) are being added, starting with the PN-counter. there really isn't any competition in this area.

### With Partitioning
Partitioning --- sometimes called sharding --- is a horizontal partitioning of data across multiple member processes. You can think of the Hazelcast partitions (shards) as the hash buckets of a distributed hash map. Each cluster uses a configured number of partitions --- the default is 271 and that often need not be changed. A single member "cluster" (non-clustered cluster?) would get all the partitions, without backups. As the first member joins, roughly half the partitions are transferred to that new member and backups are created at that time. As each subsequent member joins, some basically equal fraction of the partitions are transferred to that member --- both primary data and backups.
When members leave, the backups are promoted to primary partitions and new backups are created on the remaining members.

### And Monitoring
One might argue that monitoring doesn't belong here, but within distributed systems the lack of monitoring is a huge step toward failure. Headless systems are often not well understood and sometimes ignored by ops --- and who can blame them. Ops can only react to errors after they occur, where monitoring might highlight problems before.
Hazelcast uses JMX  to support the *Management Center*, so monitoring is easily available; you see issues coming in advance of major problems and --- better yet --- supports alerting thresholds that will allow the system to call for help.
You can also easily integrate this with a log file management and analysis, like the open-source ELK (elasticsearch/logstash/kibana) --- or one of the excellent commercial alternatives.

### What's it for?
It's for almost any programming task --- broadly speaking, but the three major areas are caching, distributed processing and distributed messaging. The central theme of all the applications is big, fast data. Big data is good, big fast data is awesome. Hazelcast has been around long enough to have serious adoption in various industries --- airlines, telcos, finance, crypto currency applications and on and on. Does it need to be big? Not really --- it's nice to have an infrastructure that can start small and grow enormously.
The processing capabilities have been enormously augmented by Jet --- fast stream and batch processing, built on top of the IMDG. That's a deep topic, all on its own --- and will be covered in many other places.

### Before you start
There are only a few things you need, to get going -

#### Java 8
Java 8 is probably the most widely used jdk/jre right now and is the preferred one to start with. If you have v3.10+, jdk9 is fine, too. For simplicity, though, I'll assume jdk8.

#### IDE
Eclipse, IntelliJ and NetBeans seem to make up about 97% of the IDE users and any of them work well with Hazelcast/maven. If you're not using an IDE, you may want to re-think what you're doing. The sample code in the github repo was tested in eclipse, as well as from the command line, so it's an easy import.

#### Gradle or Maven
These are popular build tools for java environments, use these, or choose your own.

#### Hazelcast Dependencies
All the dependencies for Hazelcast --- any edition --- are available on public maven repositories or from the download site. You can download the open-source code, too, for that matter.

##### Server
The server jar will be in one of two forms: 'hazelcast' or 'hazelcast-all' (which also includes client dependencies).

##### Client
The Hazelcast client is generally included from 'hazelcast-client', and that's the only addition to your client app build.

### Programming Models
Hazelcast is a toolkit --- that's a really important point. There are common patterns you can employ, but --- really --- it's just java. You're not constrained to any set architecture. You can design your own infrastructure to meet your needs in any way you see fit. Here are some common deployment models.

#### Embedded Member
Embedded members are really the easiest way to get started and, for some things they may be all that you need. An embedded client is a java program, where you create a `HazelcastInstance`. That will launch the framework and form a cluster with any compatibly configured members they find on the network --- that all depends upon your network and discovery configuration.

#### Dedicated Member
A dedicated member is a Hazelcast process dedicated to storage and a few other things. It won't run your code, except for server-side-specific constructs --- entry processors, Executor Tasks (Callables and Runnables), event code (Listeners and Interceptors) and persistence code (MapLoader and MapStore).
The advantage of this approach over the embedded model is that its scalability will always become more important than simplicity. With this, you can scale your storage fleet separately from your client fleet. If your storage demands soar, but the processing doesn't, you just scale these members. If you introduce new processing demands for the same, or similar, data loads, you just add clients. 
In general, this is going to be the best basic approach to almost any IMDG system.

#### Lite Member
Lite members are interesting --- they join the cluster --- unlike clients that just make a client-specific TCP connection. The do not, however, host data. They are for a small number of kinda advanced things --- they may be used for class-loading or they may be used as high-performance compute clients. You could, for example direct runnable and callable tasks to Lite members --- if they require data that's spread across the cluster for some kind of computation or processing.

#### Clients
Hazelcast clients are programs that connect to an IMDG and interacts using either REST or a native client library. These will be in your web-clients, your command-line tools or anything that iteracts with Hazelcast. Don't think, though, that because they're clients, you're going to be doing all your processing there. Well written clients will use server-side constructs --- particularly entry-processors, aggregation and executor tasks --- to delegate processing requests from (what could be) single-threaded clients onto a (what should be) massively scalable clustered storage and processing environment.
Not everything will be delegated to the back-end, of course. Many, many clients simply require extremely low-latency access to fast, big data that isn't changed too often and isn't changed (ideally) by separate clients (i.e. sticky sessions are good). For these, _near-caches_ are extremely effective. Near-caches allow each member to host --- within its process space --- potentially large sub-sets of data that are being actively managed by the cluster. These are supported in all editions of Hazelcast, but it's worth noting that the _Enterprise HD_ edition will allow off-heap near-caches, giving you low latency access to --- potentially --- many gigabytes of near-cache data in each client. This has a broad range of applications across industries; real-time inventory for e-commerce and fraud-detection for credit-card processors are two interesting ones. Note that in neither of these, is the data static --- that's not a requirement. But the data is read much more often than it's changed, making both of these ideal cases for near-caching.

### A Simple Java Cluster Member

So, finally a little code. Starting Hazelcast from Java is easy. In this example, we set up the configuration in code; we could also have let it search the classpath for an XML file (hazelcast.xml). This is done to more clearly demonstrate what's happening.

```java

public class SimpleJavaServer
  {
    private final static Logger l = LoggerFactory.getLogger(SimpleJavaServer.class);

    public static void main(String[] args)
      {
        Object memberLock = new Object();
        Config config = new Config();
        NetworkConfig networkConfig = config.getNetworkConfig();
        JoinConfig joinConfig = networkConfig.getJoin();
        // disable multicast (the default join strategy)
        joinConfig.getMulticastConfig().setEnabled(false);
        // AWS join config is off by default, so this is only illustrative.
        joinConfig.getAwsConfig().setEnabled(false);

        TcpIpConfig tcpIpConfig = joinConfig.getTcpIpConfig();
        tcpIpConfig.setEnabled(true).addMember("127.0.0.1,127.0.0.2,127.0.0.3");
	
        HazelcastInstance instance =
            Hazelcast.newHazelcastInstance(config);
        LifecycleService service = instance.getLifecycleService();
        service.addLifecycleListener(
	    new MemberLifecycleListener(memberLock));

        synchronized (memberLock)
          {
            try
              {
                memberLock.wait(3000L);
                instance.shutdown();
              }
            catch (Exception e)
              {
                l.error("while waiting for cluster to shut down ({}): exiting",
                      e.getMessage());
              }

          }
      }

    public static class MemberLifecycleListener implements LifecycleListener
      {
        private Object memberLock;

        public MemberLifecycleListener(Object memberLock)
          {
            super();
            this.memberLock = memberLock;
          }

        @Override
        public void stateChanged(LifecycleEvent event)
          {
            LifecycleState currState = event.getState();
            switch (currState)
              {
                case SHUTTING_DOWN:
                  l.warn("cluster is shutting down");

                  break;
                case SHUTDOWN:
                  l.warn("cluster shutdown complete - will notify member");
                  synchronized (memberLock)
                    {
                      memberLock.notify();
                    }

                  break;

                default:
                  l.info("member state is now: {}", currState);
                  break;
              }

          }
      }
  }

```

Most of that code was the event listener. Without the wait/notify (or any other) mechanism, this would create the member instance --- and exit. Instead, it waits 3 sec and calls shutdown - to exercise the lifecycle listener. Notice that the listener isn't added until the member is started, so the only events you should see are the _SHUTTING_DOWN_, immediately followed by _SHUTDOWN_. To make this work with clients, remove the time from the wait.

### A Simple Client

We can do a really simple client --- without Spring boot --- just by creating a client instance. The server (above) should be running, and the client instance in this code will give you access to the all the features of the IMDG. This client will work with the above server, but make sure you change or remove the argument to `wait()`.

```java
public class SimpleClient
  {
    private final static Logger l = LoggerFactory.getLogger(SimpleClient.class);

    public static void main(String[] args)
      {
        ClientConfig config = new ClientConfig();
        config.getNetworkConfig().addAddress("127.0.0.1");

        GroupConfig groupConfig = config.getGroupConfig();
        groupConfig.setName("dev");
        groupConfig.setPassword("dev-pass");

        HazelcastInstance client = HazelcastClient.newHazelcastClient(config);

        IMap<String, String> demo = client.getMap("demo");

        String key = "someKey";

        demo.set(key, "some value");

        l.info("demo map (size {}) contains value {} for key {}", demo.size(), demo.get(key), key);
      }
  }
```

That's a very simple demo --- but, it sets up a client that looks for a server-member instance on the same host you specify, connects to it and manipulates data in a distributed map. Not bad, for that much code- IMHO.

### A Simple Spring Boot Cluster Member
So, finally a little code. All of this is on github, so you can look at the POM (pom.xml) file there. It's really basic, you need the spring-boot parent entry and just the Hazelcast dependency (from above).
Make your main class a spring boot application with the `@SpringBootApplication` annotation. 

```java
@SpringBootApplication
public class SimpleSpringBootServer implements ApplicationContextAware
  {
    <snip>
    public static void main(String[] args)
      {
        SpringApplication.run(SimpleSpringBootServer.class, args);
      }
    <snip>

  }

```
 
That's a fairly concise server. The instance it's going to use is injected. This server will terminate when the application context
is closed.
In this example, the config is wrapped in beans --- 

```java

@Configuration
@Profile({"server", "java-config"})
public class JavaServerConfig
  {

    @Bean
    public CommandLineRunner commandLineRunner()
      {
        return args -> {
          applicationContext.getBean("hazelcastInstance");
        };
      }

    @Bean
    public JoinConfig joinConfig()
      {
        JoinConfig config = new JoinConfig();
        config.getMulticastConfig().setEnabled(false);
        config.getAwsConfig().setEnabled(false);
        config.getTcpIpConfig().setEnabled(true);
        config.getTcpIpConfig().addMember("127.0.0.1,127.0.0.2,127.0.0.3");

        return config;
      }

    @Bean
    public NetworkConfig networkConfig(JoinConfig joinConfig)
      {
        NetworkConfig config = new NetworkConfig();

        config.setJoin(joinConfig);

        return config;
      }

    @Bean("config")
    public Config config(NetworkConfig networkConfig)
      {
        Config config = new Config();

        return config;
      }

    @Bean("hazelcastInstance")
    public HazelcastInstance hazelcastInstance(Config config)
      {
        HazelcastInstance instance = Hazelcast.newHazelcastInstance(config);

        return instance;
      }

    @Bean
    public CacheManager cacheManager(final HazelcastInstance instance)
      {
        CacheManager manager = new HazelcastCacheManager(instance);

        return manager;
      }

  }
```

Spring Boot will create a default instance of Hazelcast --- that's fine, especially if you're using declarative configuration (XML), but
for java config, this an effective approach.


#### A Slightly Better Client
We can do more with the client code. I'm doing this all as Spring Boot, so it would make sense to have a simple executable, where the app logic is in a bean ---

```java
@SpringBootApplication
public class ClientApp implements ApplicationContextAware
  {
    <snip>
    public static void main(final String[] args)
      {
        SpringApplication.run(ClientApp.class, args);
      }

    @Bean
    public CommandLineRunner commandLineRunner(ApplicationContext context)
      {
        return args -> {
          HazelcastInstance client = (HazelcastInstance) context.getBean("clientInstance");

          IMap<String, String> demo = client.getMap("demo");

          String key = "someKey";

          demo.set(key, "some value");

          l.info("demo map (size {}) contains value {} for key {}", demo.size(), demo.get(key), key);
          System.out.println("in main --- args len: " + args.length);
        };
      }
    <snip --- boilerplate>
  }
```

This version of nearly the same code uses DI to hand us a configured Hazelcast client instance. That allows us enormous flexibility in deciding where/how to configure it and lets us use environment-specific configuration, too. In real code, the configuration would be moved out of the app class, but this is easily readable.

##### Simple Map Access
This part is easy --- a Hazelcast IMap `is a` java.util.Map --- so you can take existing code for the java collections API and just repurpose it. Not to go too far astray, but here's a little code showing how easy that can be.


```java

@Component("isAMapRunner")
public class IsAMapRunner implements CommandLineRunner, ApplicationContextAware
  {
    <snip>

    protected void populateMap(Map<String, String> map)
      {
        for (int i = 0; i < 100; i++)
          {
            String key = "k-" + i;
            String value = "Value: " + i;
            map.put(key, value);
          }
      }

    @SuppressWarnings("unchecked")
    @Override
    public void run(String... args) throws Exception
      {
        /*
         * create a java util map and do something with it -
         */
        Map<String, String> map = new HashMap<>();
        populateMap(map);
        l.info("size of the java collections map is {}", map.size());

        /*
         * now, let's talk to the IMDG --- Get a reference to an IMap, but
         * refer to it as a collections Map. Can't use distributed  functions, though.
         */
        HazelcastInstance instance = (HazelcastInstance) getApplicationContext().getBean("hazelcastInstance");
        /*
         * Use the same map declaration
         */
        map = instance.getMap("employees");

        /*
         * This may be empty --- but, it may have been populated in other
         * code --- that's the biggest difference here.
         */
        l.info("distributed map size is {} initially", map.size());
        populateMap(map);
        /*
         * The size may have changed. (If the keys were identical,
         * 'populateMap' would have just overwritten existing values and
         * not changed the size).
         */
        l.info("distributed map size is {} after being populated", map.size());
      }

    <snip>

  }
```

In that bit, there's a method that creates a map and uses it --- sort of trivially, but it's working code. 
In the second part, the only change was to use the injected Hazelcast instance (injected, via annotation), to get a reference to a distributed map in the IMDG. There's no magic; Hazelcast is designed so that you can swap it in that easily, using the --- _really better be_ --- familiar collections API. But, back to that comment for a second --- should the declaration have been IMap, not Map? It depends --- it could be, but it doesn't need to be. Hazelcast maps implement the java.util implements, so that's valid --- but maybe not useful. In a minute, we're going to use some Hazelcast specific methods on the map and to make those visible, you'd want to change the declaration. If you're just doing put, get, size, remove and all of those, then no. One interesting note on that: it's easy to forget that `put` returns the old mapping, as it inserts the new. I don't think I've ever actually seen code that used that, but, there it is. Think about that in a network environment, though --- when you do a `put`, Hazelcast --- conforming to the contract --- returns the old mapping. Over the network. Incurring serialization. For no reason. Because nobody. Ever. Looks. At. It. Hazelcast has added a `set` method, that works like `put`, save that it doesn't return the value. This may seem like small stuff, but think about a heavily utilized production environment, getting a surge of requests; you're busy and half of that flavor of network traffic is stuff you're never going to look at. Change two letters in your code and the network traffic drops --- significantly. I'd do it.
In moving from a collections map to a distributed map, keep in mind, however, that there are differences. With a distributed map, absent security configuration, other clients/other threads can use the same map. If you test the size of a new in process map, that you create in your thread, the size will be `0`. When you get a reference to a distributed collection from the IMDG, it will create it --- if required --- or return a reference to an existing collection, if it's already been created. This can be a very powerful feature --- you can pre-populate a collection, from a persistent store or any other data-source. Your client code will be smaller and simpler, because you can make assumptions about it. If you're using a map for a scratchpad cache, however, keep in mind that you may want to create unique map instances or manage data, so that your thread doesn't collide with other clients.


##### Near Cache Access
Hazelcast supports second-level (edge) caching in client processes and refers to it as "near caching". Near caches are almost transparent to your code --- although there are things you need to be aware of.
Each mapping in the near cache is, fundamentally, managed by the IMDG member that owns the "master" copy of the data. It may be cached on multiple clients --- in multiple apps --- and each caching client app may define their own policy for managing updates. The data in your near cache may be stale --- you probably shouldn't cache things for overly long times (you set the expiry interval in config). You should be careful about using near caches for things that are updated frequently and very careful about using them for things that are updated from multiple points. For a web application with sticky sessions, you should be able to count on certain objects being in only one client process --- that's a good scenario. 

#### Simple Query Operations
##### SQL Queries
Hazelcast is not a SQL database or --- _really_ --- a SQL query tool, but it provides a very workable, robust subset of SQL query functionality. It's very accessible for developers. If you have a SQL background, this is nothing; if you don't, it's still pretty intuitive. The SqlPredicate encapsulates the "where" clause of a query. For IMDG data extraction, that's probably the part you really care about. Having said that it's not a relational database, I should point out that, since you're generally dealing with purely in memory data, this is going to be very fast.

I think that shows how easy it is to bridge a SQL background, with the IMDG SQL-like query. The big caveat here is joins; out of the box, the IMDG is not a really good tool for joins, because of the nature of the data. We split it up because it's big data; because it's big data, joining it back together is a more specialized thing. Hazelcast IMDG can certainly do joins, but Hazelcast Jet is a more appropriate tool.


##### Predicate Queries --- Criteria API
For Java developers who really never liked SQL (lots of us) there's also a pure java approach to querying the IMDG: the criteria API.

```java
@Component("predicateQueryRunner")
public class ClientPredicateQueryRunner implements CommandLineRunner, ApplicationContextAware
  {
    <snip>
    @Override
    public void run(String... args) throws Exception
      {
        HazelcastInstance client = (HazelcastInstance) applicationContext.getBean("clientInstance");

        IMap<Long, Employee> employees = client.getMap("employees");
        Employee a = new Employee(Long.valueOf(1), "Jane", "Doe");
        Employee b = new Employee(Long.valueOf(2), "John", "Smith");
        Employee c = new Employee(Long.valueOf(3), "Tom", "Notmyrealname");
        Employee d = new Employee(Long.valueOf(4), "Tom", "FakeName");
        Employee e = new Employee(Long.valueOf(5), "Tom", "ActualName");
        employees.set(a.getEmpId(), a);
        employees.set(b.getEmpId(), b);
        employees.set(c.getEmpId(), c);
        employees.set(d.getEmpId(), d);
        employees.set(e.getEmpId(), e);

        Predicate<String, String> fnamePredicate = equal("firstName", "Tom");
        Collection<Employee> employeesNamedTom = employees.values(fnamePredicate);

        l.info("found {} employees named 'Tom'", employeesNamedTom.size());

        Predicate<String, String> lnamePredicate = equal("lastName", "ActualName");
        Predicate actNamePredicate = and(fnamePredicate, lnamePredicate);

        l.info("{} emploees match first and last name", employees.values(actNamePredicate).size());
      }
      <snip - boilerplate>
  }

```

It's probably obvious --- but let me talk about what that's doing. From a client, you're setting up arbitrarily complex criteria --- in both the SQL and the criteria APIs --- and dispatching the evaluation to the grid. This is very powerful, because the queries are distributed across hardware. I've seen people do things like bring the entire map to a client, to iterate over it, for reasons. Bad reasons. With Hazelcast, you describe what you want back and dispatch the evaluation and extraction to your entire server fleet, seeing only correct results. This is a reporting or analysis kind of thing --- you wouldn't look up everyone whose last name starts with "D", to do any kind of processing where the employee object would be updated, of course.
Does the query capability make the IMDG a database? Nope. Does it give you highly leveraged tools for fast access to big data? Yep. 

One thing that, looking at this, really needs to be mentioned, if only briefly, is memory in the query client. I've described Hazelcast as a fast big-data framework, so clearly, the query operations can quickly retrieve very large volumes of data. If you had, say 20 storage members and queried the cluster and brought back 1GB, or so, of data from each, you'd be looking at 20GB of data from one query. You may, or may not, have that much memory available in your client. The fix for that is paging-predicates. These predicates wrap (logically subsume) other predicates, so that you get the same logical comparisons --- the same filtering --- wrapped in a container that lets you bring results back in batch sizes that you specify --- it's as though you're reading a printed book and seeing one page at a time.


##### Entry Processors
Entry processors are really pretty cool --- you can do highly efficient in-place processing with a minimum of locking. Consider what people often end up doing to work with remote objects --- lock a key, fetch the value, mutate the value, put it back then (in a finally block, I hope) unlock the key. That's four network calls, to start with --- three if you're only looking at the data and not updating the central source. Your objects may be large and incur significant cost in terms of CPU and network for serialization and transport. They may not, but I'm trying to make a point.
Entry processors allow you to dispatch a key-based 'task' object across the LAN --- directly to the member owning a key, where it is executed in a lock-free, thread-safe fashion. Too good to be true? Not really --- Hazelcast has an interesting threading model that allows this to happen. That's a topic worth its own discussion, too, so take my word for it, for now.

Here's a brain dead simple entry processor example --- but it's still a really useful approach:

```java
@Component
@Profile("client")
public class EntryProcessorRunner implements CommandLineRunner, ApplicationContextAware
  {
    private ApplicationContext applicationContext;

    @Override
    public void run(String... args) throws Exception
      {

        HazelcastInstance instance = (HazelcastInstance) applicationContext.getBean("hazelcastInstance");
        IMap<String, String> demo = instance.getMap("demo");

        String key = "someKey";
        demo.set(key, "Just a String value...");

        demo.executeOnKey(key, new DemoEntryProcessor());

        EntryProcessor<String, String> asyncProcessor = new DemoOffloadableEntryProcessor();

        demo.submitToKey(key, asyncProcessor);
        ExecutionCallback<String> callback = new AsynchCallbackDemo();

        demo.submitToKey(key, asyncProcessor, callback);
      }

    <snip>
  }
```

Here's the entry processor that is called in this:

```java
public class DemoEntryProcessor extends AbstractEntryProcessor<String, String>
  {
    <snip>
    private static String FMT = "This value (%s) was modified at -- %s";
    
    @Override
    public Object process(Entry<String, String> entry)
      {
        String key = entry.getKey();
        String value = entry.getValue();
        l.info("in-place processing called for {}::{}", key, value);

        SimpleDateFormat sdf = new SimpleDateFormat(dtFormat);
	String dt = sdf.format(new Date())
        entry.setValue(String.format(FMT, dt, value));
        return null;
      }
    <snip>
  }
```


I'm describing the simplest case, but as simple as it is, this is a very powerful tool to have. From your client, you can dispatch computation to your servers, no explicit locking, no worrying about thread-safety, efficient user of the LAN. But because you're synchronously waiting for the result to be returned, it may not seem like a really big deal. Hazelcast also has a number of options to run synchronously, asynchronously and on multiple entries.

This is a really useful set of calls. The first one, `executeOnKey`, does exactly that; it makes one direct call to the key owner and executes synchronously on that entry. The next two execute asynchronously --- your client code doesn't need to wait for long running operations. A word of warning, though: long running entry-processors can be truly evil --- use off-loadable in your code, by annotation, to tell Hazelcast to move the operation off of the default threading structure.

Here's the async processor, similar in function to the first:

```java
public class DemoOffloadableEntryProcessor extends AbstractEntryProcessor<String, String> implements Offloadable
  {
    <snip>
    private static String FMT = "This value (%s) was modified at -- %s";
    public final static String  OFFLOADABLE_EXECUTOR  = "default";

    @Override
    public Object process(Entry<String, String> entry)
      {
        String key = entry.getKey();
        String value = entry.getValue();
        l.info("in-place processing called for {}::{}", key, value);

        SimpleDateFormat sdf = new SimpleDateFormat(dtFormat);
	String dt = sdf.format(new Date())
	
        entry.setValue(String.format(FMT, dt, value));
        entry.setValue(newValue);

        return newValue;
      }

    @Override
    public String getExecutorName()
      {
        return OFFLOADABLE_EXECUTOR;
      }

  }
```


```java


```


I like this one: submit the processing, tag it with a callback and be on your way. Where would you use that? Broad question --- it really depends --- but if you have a stream of data requests that need to be initiated without the caller needing to wait for a result, this may be perfect. If you're going to do this, be sure to read ahead to the section on back-pressure.

The execution callback executes in the caller's process space, so it's your notification that the invocation is complete. This one just logs, like this:


#####  Back Pressure

Back-pressure is a topic worth some consideration, if you're using asynchronous operations. The settings to consider are: the number of simultaneous async operations, the client timeout and the _syncwindow_.

When you have a need and are ready to test, configuring it can be done with system properties, like this:

```java
    -Dhazelcast.backpressure.enabled=true
    -Dhazelcast.backpressure.backoff.timeout.millis=60000
    -Dhazelcast.backpressure.max.concurrent.invocations.per.partition=100
    -Dhazelcast.backpressure.syncwindow=1000
    
```
First, we enable backpressure - this should always be done for async.
Hazelcast is using its threads to execute these, so there's a limit to how many invocations can be in flight at any one point in time. The absolute number doesn't matter --- that would depend upon the size of your cluster and the number of CPUs/cores/physical threads. What's  likely to be interesting is how many can be queued up for one partition --- by default, mutating entry-processors operate on a "partition thread". Hazelcast will examine the hardware resources available and dynamically allocate thread pools at runtime. You rarely need to configure threading for your IMDG.

First, we just just enable backpressure --- we could do that and just take all the defaults or we can configure more about it. The second property (timeout) is the max wait (millis) for an invocation space to be available. The third controls the number of pending operations per partition. With asynch operations, Hazelcast will periodically convert one operation from async to sync --- and the last property controls that. In the above example, about 1 in 1,000 operations (it's randomized +- 25%) will be converted to a synch call and that caller waits while the queue --- if any --- of operations is drained.


The client-side analogue of back pressure are concurrent invocations (which can be managed by the system property `hazelcast.client.max.concurrent.invocations`). This limits the pressure that a client can generate on the server. Use of asynchronous client calls --- `getAsync()`, `putAsync()`, `setAsync()` (and many others) ---  will be regulated by this.


##### Runnable Tasks
These are simply java runnable objects that are dispatched to one or more members. Keep in mind --- the salient part of the signature is `public void run()` --- i.e. nothing is returned. The way that they're dispatched to the members is very flexible --- it can be one member, all members, a member that owns a key, or you can select members by an attribute you have set on them.
Here's an example of running something on all members ---

```java
@Component("runnableDemo")
@Profile("client")
public class RunnableDemoCaller implements CommandLineRunner, ApplicationContextAware
  {
    private ApplicationContext applicationContext;

    @Override
    public void run(String... args) throws Exception
      {
        HazelcastInstance client = (HazelcastInstance) applicationContext.getBean("clientInstance");
        IExecutorService executorService = client.getExecutorService("default");
        executorService.executeOnAllMembers(new LoggingRunnable());
      }
    <snip>

  }
```


The Runnable object is pretty ordinary, with the caveat that it needs to be serializable. This is an interesting topic and I'm going to touch on it in a minute. This runnable is also `HazelcastInstanceAware`, so that, when it's set up on the target node, the Hazelcast framework will inject the correct instance, so that it can communicate with the cluster.

```java

@Component("loggingRunnable")
public class LoggingRunnable implements Runnable, HazelcastInstanceAware
  {
    <snip>
    private HazelcastInstance  hazelcastInstance;

    @Override
    public void run()
      {
        l.info("into run, cluster size: {}", getHazelcastInstance().getCluster().getMembers().size());
      }
      
    <snip>
  }
```


This code wasn't particularly profound, but there's one cool aspect to it --- you can direct processing to a member that owns a key (or other members) and process that key and or other keys in multiple maps. So, complex manipulation may be performed outside your client, eliminating multiple network round-trips. They data need not come all from one member, either --- there are no restrictions on that. It can be a significant performance boost to design your data, so that related items are all within one node --- then this kind of task will tend not to make network calls but will not be restricted from doing so.


##### Callable Tasks
As with runnable, callable tasks are dispatched to one or more members, but offer more options for things like bringing back data.
Here's a really simple callable that will be dispatched to a member, log some noise to show it ran, and return the partition count. There are better ways to monitor or manage partitions, but this should just show how you get a value --- easily --- from a member.

```java
@Component("runnableDemo")
@Profile("client")
public class RunnableDemoCaller implements CommandLineRunner, ApplicationContextAware
  {
    private ApplicationContext applicationContext;

    @Override
    public void run(String... args) throws Exception
      {
        HazelcastInstance client = (HazelcastInstance) applicationContext.getBean("clientInstance");
        IExecutorService executorService = client.getExecutorService("default");
        executorService.executeOnAllMembers(new LoggingRunnable());
      }

    <snip>
  }

```

That is the client code that is executing a `LoggingRunnable` on each member --- here's the runnable:

```java
@Component("loggingRunnable")
public class LoggingRunnable implements Runnable, HazelcastInstanceAware
  {
    <snip>

    @Override
    public void run()
      {
        l.info("into run, cluster size: {}", getHazelcastInstance().getCluster().getMembers().size());
      }

    <snip --- boilerplate>
  }
```


That's just looking at the member, which you might want to do. Importantly, If you wanted to get/set/remove data, run a query, or any other Hazelcast operation, you can do that from that code.
The call is easy --- as with the collections, it draws heavily from the Java Executors API.

Callable Tasks are similar, but allow a value to be returned from each member. If more than one member is involved, a collection of results is returned. Here's a sample call ---



```java
public class CallableDemoCaller implements CommandLineRunner, ApplicationContextAware
  {
    <snip>
    @Override
    public void run(String... args) throws Exception
      {
        HazelcastInstance client = (HazelcastInstance) applicationContext.getBean("clientInstance");
        IExecutorService executorService = client.getExecutorService("default");

        Callable<String> demoCallable = new DemoCallable();
        l.info("Submitting this one to all members");
        Map<Member, Future<String>> results = executorService.submitToAllMembers(demoCallable);
        for (Future<String> result : results.values())
          {
            l.info("Result from '{}'", result.get());
          }

        executorService.executeOnAllMembers(new LoggingRunnable());
      }
    <snip>
  }
```

The callable task just a java class that, at a minimum, implements `Callable` ---

```java
public class DemoCallable implements Callable<String>, HazelcastInstanceAware, Serializable
  {
    <snip>
    @Override
    public String call() throws Exception
      {
        InetSocketAddress socketAddress = hazelcastInstance.getCluster().getLocalMember().getSocketAddress();
        String id = String
          .format("Running in member '%s::%d", socketAddress.getHostName(), socketAddress.getPort());

        return id;
      }
    <snip>
  }
```

### What Just Happened?
Hard to say, but if you were following along, you've created clients and members that start Hazelcast and do useful things. This may be a little more than a simple intro, but it's also just a little of what you can do with Hazelcast. I hope that you get a few things from this --- like how much I like Hazelcast. I've been doing distributed systems for what seems like a really long time --- anyone still using ARCNet messaging on  MP/M? I hope not; I really do. I've done distributed systems before there were cool frameworks like Hazelcast and I've used most of the competition, too. This one is my choice --- it gives performance and simplicity. You can be up and running in minutes and rolling out production quality code that looks an awful lot like your Java collections code. It's a fun environment for programmers. A little Java gets can be all you need on the server side, then you can cut loose with Java, .NET, C++, Node.js, Python, Go or Scala --- and that list is going to grow, as new languages emerge.
But, anyway --- we stood up storage members and a couple clients, with a functional subset of features from Hazelcast. There was no production code here, but I think it shows the path.



