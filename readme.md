# Scaling system

Scaling system methods from my understanding.

## Stateless

In order for a service to be be horizontal scaling, each clone of the server
does not store user-related data on local disk or memory. Shared state must be saved in a database or external cache.

## Load balancing

* **DNS load balancing**

  * Example: `nslookup google.com` returns many IPs.
  * Cons: DNS TTL, exposing server IPs, a server breaks

* **Software**

  * Example: Nginx, HAProxy, Linux Virtual Server, ..
  * TODO:
    * try on Websocket, MySQL
    * try types: least loaded, Layer 4 (tcp address)
    * multiple load balancers for availability

## Caching

* Should cache data that does not need to frequently change and that
  is costly to read or generate. Ex: historical data.
* Memory is more limited than disk, so we need to configure
  a cache invalidation job.

## Database replication

Place holder

## Database sharding

* Cons:
  * Need to update your application logic to work with shards.
  * Data distribution can become unbalanced.
  * Joining data from multiple shards is complex. Denormalization
  or duplication can help.
