---
title: "About Consul"
date: "2025-12-15"
summary: "📝 Quick notes on Consul..."
tags: ["backend", "microservices", "consul"]
---

Consul is a service discovery and service mesh tool created by HashiCorp.
- Register themselves
- Discover other services
- Perform health checks
- Do key/value configuration
- Secure service communication (service mesh)

What Consul does in microservices:
1. Service Registry
    - Each service registers itself: IP, port, health check.
2. Service Discovery
    - Other services ask Consul to find service addresses.
3. Health-based Load Balancing
    - Consul only returns healthy service instances and can distribute load.

``` go
// Direct:
http.Get("http://player-service:8001/players/1")

// Consul Discovery:
url, _ := serviceDiscovery.DiscoverService("player-service")
http.Get(url + "/players/1")
// url will be: http://172.20.0.5:8001 (discovered from Consul)

// Benefits:
// - If player-service scales to 3 instances, Consul knows about all of them
// - Random load balancing between instances
// - Only calls healthy instances
// - Services can move without code changes
```