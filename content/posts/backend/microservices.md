---
title: "About Microservices"
date: "2025-12-09"
summary: "📝 Quick notes on Microservices..."
tags: ["backend", "microservices"]
---

## Service Discovery

### Service Registry

A Service Registry is a central database + API where all active services register themselves.

| Registry                  | Notes                            |
| ------------------------- | -------------------------------- |
| Consul                    | Very popular; strong feature set |
| etcd                      | Used by Kubernetes internally    |
| Eureka                    | Netflix OSS (Java ecosystem)     |
| Zookeeper                 | Old but still used (Kafka, etc.) |
| Kubernetes API Server     | Acts as the registry in K8s      |

The registry stores information such as:
- Service name
- Service IP
- Port
- Metadata (version, region, etc.)

The registry also handles:
- Registration (when a service starts)
- Health checks (to remove unhealthy nodes)
- Deregistration (when a service shuts down)

### Service Provider
### Service Consumer