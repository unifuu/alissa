---
title: "About Microservices"
date: "2025-12-09"
summary: "📝 Quick notes on Microservices..."
tags: ["backend", "microservices"]
---


## Service Registry

- A Service Registry is a central database + API where all active services register themselves.
- The registry stores information such as:
    - Service name
    - Service IP
    - Port
    - Metadata (version, region, etc.)
- The registry also handles:
    - Registration (when a service starts)
    - Health checks (to remove unhealthy nodes)
    - Deregistration (when a service shuts down)

### Consul

Consul is a service discovery and service mesh tool created by HashiCorp.
- Register themselves
- Discover other services
- Perform health checks
- Do key/value configuration
- Secure service communication (service mesh)

## Service Provider
## Service Consumer

## Gateway

An API Gateway is a single entry point for all clients (mobile apps, web frontends, game clients) to access your backend services.

```
Client → API Gateway → internal microservices
```

### Roles

#### Centralized Routing

The client only talks to one endpoint:
`https://api.example.com`

Then gateway decides:
- Request to /login → user-service
- Request to /gacha/draw → gacha-service
- Request to /inventory/list → inventory-service

#### Security

The Gateway can enforce:
- Authentication (JWT/OAuth)
- Authorization
- Rate Limiting
- IP Whitelisting
- Request Validation
- Bot Detection

#### Load Balancing

```
                 ↗ service-0
Client → Gateway → service-1
                 ↘ service-2
```

#### Protocol Translation

Gateway handles REST ↔ gRPC conversion:
- Client uses REST
- Internal services use gRPC

```
Client → REST (JSON)
Gateway → gRPC (binary)
```

#### Observability (Logging / Metrics)

#### Aggregation (a big plus for mobile/game development)