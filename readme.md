# Scaling system

Scaling system methods from my understanding.

## Shared state

In order for a service to be be horizontal scaling, each clone of
the server does not store user-related data on local disk or memory.
Shared state must be saved in a database or external cache.

## Load balancing

Load balancing distributes traffic to many different servers.

* Software

  * Example: Nginx, HAProxy, Linux Virtual Server, ..
  * Pros: can define balancing rule: round-robin, TCP address, least
    loaded, server down.
  * TODO: try Nginx on TCP (Websocket, MySQL), try customize, try dual
    balancers for high availability.

* DNS load balancing

  * Example: `nslookup google.com` returns many IPs.
  * Pros:
    * Do not need machine or balancer software.
    * Can be used on top of software balancers for high availability.
  * Cons: DNS TTL, exposing server IPs, cannot be customized.

## Database

### Optimizing queries

When database is slow, the easiest fix is optimizing queries.
Understanding index helps to speed up read queries.

### Database replication

* Pros: high availability, scale read queries.
* Cons: complicated setup, reducing data consistency.

### Database sharding

* Pros: Scale write throughput (the only way?)
* Cons:
  * Need to update your application logic to work with shards.
  * Data distribution can become unbalanced.
  * Need to know how to model data for sharding.

### Data modeling

#### Pros of some DBMSes

* MySQL (Relational DBMS)
  * Natural relational modeling: start from real life objects, represent
    them as tables, normalize and assign primary keys and foreign keys
    to remove duplication and keep related data consistent. After tables
    modeled properly, you can always get the data you want (with Join
    operator or complex sub queries).
  * Reliable (ACID model, suitable for financial app).
  * Mature, the most popular free database.
* MongoDB
  * Schema-less: data is JSON-like documents that are similar to
    programming languages objects. No need to create or alter tables.
  * Powerful queries (aggregate), easy to add index on any field.
  * The most popular NoSQL.
* Cassandra
  * Writing performance is linear scalable (if data is properly 
    modeled: [query-driven data modeling](
    https://cassandra.apache.org/doc/latest/data_modeling/intro.html)).
  * Built-in auto partitioning (aka sharding or splitting data across
    many machines). Adding node without the need to reshard or reboot
    ([detail](
    https://cassandra.apache.org/doc/latest/architecture/dynamo.html)).
  * Zero downtime, easy to setup multi-master replication (contrast to
   single-master in MongoDB, take about 20s to fail-over).
  * Flexible consistency (client can determine consistency level for
  each write or read).

#### Query-driven data modeling

In contrast to relational databases that normalize data to avoid
duplication, query-driven data modeling duplicates data in multiple
tables to speed up read queries (read data from one table on in minimum
the number of shards).

If data model cannot fully integrate the complexity of
relationships between tables for a particular query, client-side joins
in application code may be used.

## Caching

Database server often is system bottleneck. Caching reduces number
of reads on database. Second aspect is cache help users accessing
content very fast.

* Should cache data that does not change frequently and that
  is costly to read or generate (historical records in database).
* Memory is more limited than disk, so we let caches last for
  a short time. Updating both origin data and cache cause application
  to be more complicated.
* Content Delivery Network is a geographically distributed group of
  servers optimized to deliver static content to users.
  (TODO: try to setup a CDN)

## Microservices

Microservices architecture is splitting up an application to many
services that are independent of each others to stay running, each
service handles a business feature.

* Pros:
  * Each service can have its own tech stack and dev team.
  * Update and deploy a service whenever you need without having
    to stop others.
  * If a service makes a fault, it only affects itself and its consumers.
  * Easier to scale the most needed features instead of the whole
    application.

* Cons:
  * Network communication: services have to define an interproces
    data format, then send them over network by a message bus or HTTP
    instead of just passing a variable. Network is not reliable, has
    a latency and a limited bandwidth, ..
  * Joins data is hard because data were split into different databases.
  * Distributed transaction is harder, instead of doing a transaction on
    one database, we need a pattern to do a distributed transaction
    among multiple databases.
    * Saga is a popular pattern. Basically, it is a sequence of local
      transactions, if any service fails to complete its local
      transaction, the others will need to do a compensation action.
    * If you are a service provider, you have to implement an undo API.

### Fault tolerant

#### Cascade failure

Cascade failure is a where the failure of a service results in the
failure of others.

Example: WebUI calls SpecialOffer service, then SpecialOffer service
spawns a thread to send HTTP to PurchaseHistory service and wait
for response.
If PurchaseHistory service is overloaded, users cannot see the result
on WebUI, they hit F5 to retry, so SpecialOffer service will spawn more
threads that waiting forever, then SpecialOffer service is slow too.

In the example, we can prevent SpecialOffer service cascade failure
by setting a timeout, or sending requests to a message bus.
Be careful to choose a good timeout.

#### Timeout

Timeout is useful as a defender of cascade failure. And a service
can choose what to do when it encounters a timeout. It can return
an error, return a default value, or retry (on idempotent API).

Too long timeout causes bad user experience. Timeout of user call is the
sum of all interservice calls, can be quite long. So for each service,
having a shorter timeout is better.

But a too short timeout can increase number of error responses from a
resource which could still be working normally. And if the caller choose
to retry on timeout, an overloaded service get even more workloads.

TODO: Why does [Hystrix](https://github.com/Netflix/Hystrix/wiki)
suggest setting a timeout slightly higher than the measured 99.5th
percentile latency.

#### Circuit breaker

A circuit breaker monitors calls to external resources with the aim
of preventing calls which are likely to fail.

* Pros:
  * Do not wait for unresponsive calls, has more time to fall back
      to other behaviour.
  * Help the external resource to have less requests.

### Message queue

* Pros:
  * Resilience: producer can add requests to the queue without waiting
    for consumer, or when consumer went offline
  * Throttling: server can decide to consume requests when they
    are ready.
* Cons:
  * Can be system bottleneck (every service sends message queue system).
  * Message queue is a distributed system and has its own problems.
* Example: Kafka, ZeroMQ, RabbitMQ, RedisPubSub, ..

TODO: try an in-memory message queue
