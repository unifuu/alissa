---
title: "About Kong"
date: "2025-12-14"
summary: "📝 Quick notes on Kong..."
tags: ["backend", "api", "kong"]
---

## Definition

- An API Gateway.
- A reverse proxy (Based on Nginx + OpenResty)
- A traffic control layer for microservices

```
Client (Browser / Mobile / Game)
        |
        v
┌─────────────────────────┐
│        API Gateway      │
│  (Routing / Auth / CORS)│
└─────────────────────────┘
        |
        v
┌───────────────┬───────────────┐
│ player-service│ battle-service│
└───────────────┴───────────────┘
        |
        v
┌─────────────────────────┐
│   Database / Cache      │
└─────────────────────────┘
```

## Concepts

### Service

Represents an upstream backend service.

```
Service:    player-service
URL:        http://player-service:8080
```

One backend service = usually one Kong Service.

### Route

Defines how requests are matched and forwarded to a Service.

Routes can match by:
- Path (/api/players)
- Host (api.example.com)
- Method (GET, POST)
- Headers (Route → Service)

### Plugin

Adds functionality without touching backend code.

Common plugins:
- Authentication (JWT, Key Auth)
- CORS
- Rate Limiting
- Request / Response Transformation
- Logging & Metrics

Plugins can be attached to:
- Global
- Service
- Route
- Consumer

### Consumer