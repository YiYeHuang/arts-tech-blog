---
layout: post
title:  "Spring Boot Cloud 2.X Update With Hikari"
date:   2020-03-20 22:59:53 -0700
categories: tips
tag: [spring boot, JDBC]
---
 
### Tips
After Spring Boot Cloud 2.X Tomcat connection pool is replaced by HikariCP JDBC connection pool. While Hikari is dead fast, there are some config issues.
 
Probably the first thing that upgrade will do is remove the tomcat connection config.
 
```yml
tomcat:
     initial-size: 5 # starting number of connections
     max-wait: 5000 # ms to wait before throwing an exception if no connection is available
     max-active: 50 # max number of active connections that can be allocated from this pool
     test-on-borrow: true
```
after this, things just working?!?! 
meanwhile Hikari connection pool config is hidden and the default config could cause some serious performance problem.
 
When launching spring boot application, set
```yml
logging:
     level.com.zaxxer.hikari.HikariConfig: DEBUG
     level.com.zaxxer.hikari: TRACE
```
upon launching you will see
```
HikariPool-1-configuration
allowPoolSuspension.............false
autoCommit......................true
catalog.........................none
connectionInitSql...............none
onnectionTestQuery.............none
connectionTimeout...............30000
dataSource......................none
dataSourceClassName.............none
dataSourceJNDI..................none
dataSourceProperties............{password=<masked>}
driverClassName................."com.mysql.cj.jdbc.Driver"
healthCheckProperties...........{}
healthCheckRegistry.............none
idleTimeout.....................600000
initializationFailTimeout.......1
isolateInternalQueries..........false
jdbcUrl.........................jdbc:mysql://localhost:3306/XXXXXX
leakDetectionThreshold..........0
maxLifetime.....................1800000
maximumPoolSize.................10
metricRegistry..................none
metricsTrackerFactory...........none
minimumIdle.....................10
password........................<masked>
poolName........................"HikariPool-1"
readOnly........................false
registerMbeans..................false
scheduledExecutor...............none
schema..........................none
threadFactory...................internal
transactionIsolation............default
username........................"demo"
validationTimeout...............5000
```
The default `maximumPoolSize` is `10`, with a connection timeout `30000` for 5 minutes. 
10 connections can be very easily taken by a number of operations. Make sure to config the connection number to a certain size based on your application.
 
### HikariCP Design
Hikari is a very lightweight and fast connection pool. (total compiled size is only 130k) 
Some core design for the amazingly fast performance are listed as follows:
 
[FastList](https://github.com/brettwooldridge/HikariCP/blob/d0cb0f8dab5e3edaea7c4d3c5807f006d3695880/src/main/java/com/zaxxer/hikari/util/FastList.java)
 - a very light weight implementation of list, remove tuns of list support, and simplify the exception handle


[ConcurrentBag](https://github.com/brettwooldridge/HikariCP/blob/17447199f507e5b7d2fe984487e2e9d431b9215f/src/main/java/com/zaxxer/hikari/util/ConcurrentBag.java)
 - main data structure of connection pools
 - use lockless design CopyOnWriteArrayList to read and right new poolEntry
 - contains a ThreadLocal to get the current thread to avoid lock competition
 - add new connection
    - CopyOnWriteArrayList add the connection
    - spin until a thread takes it or none are waiting
 - get connection: borrow a BagEntry from the bag, blocking for the specified timeout if none are available.
   - Try the thread-local list first to get a bagEntry
   - if thread-local not found scan the CopyOnWriteArrayList get the idle connection in `STATE_NOT_IN_USE`
     - here `compareAndSet` is used to change the connection state
     - per entry is a `PoolEntry implements IConcurrentBagEntry`
     - per entry is using `AtomicIntegerFieldUpdater<PoolEntry> stateUpdater` to make sure each state is updated in atomic fashion and lock free

[HikariCP Reference](https://github.com/brettwooldridge/HikariCP)
