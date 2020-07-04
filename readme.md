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

* Pros: Scaling write (the only way?)
* Cons:
  * Need to update your application logic to work with shards.
  * Data distribution can become unbalanced.
  * Joining data from multiple shards is complex. Denormalization
    or duplication can help.

## Microservices

Microservices architecture is splitting up an application to many
services that are independent of each others to stay running, each
service handles a business feature.

* Pros:
  * Each service can have its own tech stack and dev team.
  * Update and deploy a service whenever you need without having
    to stop others.
  * If a service makes a fault, it only affects itself and its
    consumers.
  * Easier to scale a service if needed.
  
* Cons:
  * Distributed communication: services have to define an
    interprocesses data format, then send them over network by
    a message bus or HTTP instead of just passing a variable.
    So performance is reduced and logic handling is more complicated.

### Fault tolerant in microservices

#### Cascade failure

Cascade failure is a where the failure of a service results in the failure
of others.

Example: WebUI calls Offer service, then Offer service spawns a thread to
send HTTP to PurchaseHistory service and wait for response.
If PurchaseHistory service is slow, users cannot see the result on WebUI, they hit F5 to retry, so Offer service spawn more threads that very
slowly return, then Offer service is slow too.

In the example, we can prevent cascade failure by setting a timeout,
sending requests to a message bus.

#### Timeout

Timeout is useful as a defender of cascade failure. And a service
can choose what to do when it encounters a timeout. It can return an error,
return a default value, or retry (only for idempotent API).

Too long timeout causes bad user experience. Timeout of user call is the
sum of all interservice calls, can be quite long.
So for each service, having a shorter timeout is better.

But a too short timeout can increase number of error responses
from a resource which could still be working normally. And if the caller
choose to retry on timeout, an overloaded service get even more workloads.

TODO: how to get a good timeout.

#### Circuit breaker

TODO

### Message bus

* Pros:
  * persistance: if the server fails, the queue persist the message,
    so when the server is working again, it can handle the pending message.
  * rate limiting: server can decide to consume request or not.
  * batching: handle many requests at a same time, so more efficient
    to insert or update a database.
* Cons:
  * Message bus system can be a bottleneck.
