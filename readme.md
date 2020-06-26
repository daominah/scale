# Scaling system

Scaling system methods from my understanding.

## Load balancing

* **DNS load balancing**

  * Example: `nslookup google.com` returns many IPs.
  * Cons: DNS TTL, exposing server IPs, a server breaks

* **Software**

  * Example: Nginx, HAProxy, Linux Virtual Server, ..
  * TODO:
    * try on Websocket, MySQL
    * try types: least loaded, Layer 4 (tcp address)
    * multiple load balancers for more availability
