# Scaling system

Scaling system methods from my understanding.

## Stateless API

In order for a service to be be horizontal scaling, each clone of
the server does not store user-related data on local disk or memory.
Shared state must be saved in a database or external cache.

## Load balancing

Load balancing distributes traffic to many different servers.

* Software

  * Example: Nginx, HAProxy, Linux Virtual Server, ..
  * Pros: can define balancing rule:round-robin, tcp address, least
    loaded, server down
  * TODO: try Nginx on TCP (Websocket, MySQL), try customize, try dual
    balancers for high availability.

* DNS load balancing

  * Example: `nslookup google.com` returns many IPs.
  * Pros: do not need machine or balancer software
  * Cons: DNS TTL, exposing server IPs, cannot customize

## Caching

Database server often is system bottleneck. Caching reduces number
of reads on database. Second aspect is cache help users accessing
content very fast.

* Should cache data that does not change frequently and that
  is costly to read or generate (historical records in database).
* Memory is more limited than disk, so we let caches last for
  a short time.
* Content Delivery Network is a geographically distributed group of
  servers optimized to deliver static content to users.
  (TODO: try to setup a CDN)

## Database

### Optimizing queries

When database is slow, the easiest fix is optimizing queries.
Index helps to speed up read queries.

### Database replication

* Pros: high availability, scale read queries.
* Cons: complicated setup, reducing data consistency.

### Database sharding

* Cons:
  * Need to update your application logic to work with shards.
  * Data distribution can become unbalanced.
  * Joining data from multiple shards is complex. Denormalization
    or duplication can help.

## Microservices

Microservices architecture is splitting up an application to many
services that are independent of each others to stay running, each
services handle a business feature.

* Pros:
  * Each service can have its own tech stack and dev team.
  * Update and deploy a service whenever you need without having
    to stop others.
  * If service makes a fault, it only affects itself and its
    consumers.
  * Easier to scale a service if needed.
  
* Cons:
  * Distributed communication: services have to define an
    interprocesses data format, then send them over network by
    a message queue or HTTP instead of just passing a variable.
    So performance is reduced and logic handling is more complicated.

### Message queue vs HTTP (sync vs async interservices communication)

* Pros:
  * persistance: if the server fails, the queue persist the message,
    so when the server is working again, it can handle the pending message.
  * rate limiting: servers have choices to consume request or not.
  * batching: handle many requests at a same time, so more efficient
    to insert or update a database.
* Cons:
  * Message queue can be a bottleneck.
