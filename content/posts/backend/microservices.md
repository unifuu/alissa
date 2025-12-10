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

What Consul does in microservices:
1. Service Registry
    - Each service registers itself: IP, port, health check.
2. Service Discovery
    - Other services ask Consul to find service addresses.
3. Health-based Load Balancing
    - Consul only returns healthy service instances and can distribute load.

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

## RabbitMQ

### Message Broker

- A piece of software that acts as an `intermediary` for communication between different applications, systems, or services.
- It is used to send, store, and route messages reliably between applications, even if they are written in different languages or running on different platforms.

### Basic

A message broker.
- Producer: A person sending a letter
- Exchange: The post office
- Queue: The mailbox
- Consumer: The person receiving the letter

1. Broker
    - The message middleware that stores, routes, and delivers messages.

2. Connection
- This connection is expensive to create, so you usually keep it open.
`Client <- TCP -> RabbitMQ`

3. Channel
- A lightweight “virtual connection” inside a TCP connection:
- One TCP connection can contain multiple `Channels`
- Channel handle all `Messaging actions`:
    - Publish
    - Consume
    - ACKs
    - Queue / Exchange operations
- Much cheaper than opening many TCP connections
- Channels are the real working unit in RMQ.

4. Exchange
- Producers send messages to an Exchange, not directly to a queue.
- Exchange decides which queues should receive message.
    - Direct: Exact match on routing key.
    - Topic: Wildcard match (*/#)
    - Fan-out: Broadcast to all bound queues
    - Headers: Match based on headers

5. Queue
- Queue is where messages are actually stored.
- Consumers read messages from the queue, not the exchange.

6. Binding
- A rule linking an `Exchange` to a `Queue`.
    - Direct exchange
    - Topic exchange
- Bindings define how messages are routed.

7. Producer
- A program that sends messages to an Exchange.
`Producer → Exchange → (routing) → Queue`
- The producer itself never interacts with the queue directly.

8. Consumer
- A program that receives messages from a `Queue` and processes them.
    - Push: RMQ pushes messages to the consumer (default)
    - Pull: Consumer calls `basic.get()`

9. ACK / NACK
- After a consumer process a message:
    - ACK: it means "I finished, delete this message"
    - NACK:
        - requeue = true → Put back into the queue
        - requeue = false → Drop or dead-letter
- If the consumer does not ack, RabbitMQ will not remove the message.

10. Durability & Persistence
- To avoid losing messages:
    - Durable queue: Queue survives broker restart
    - Persistent message: Message stored on disk
- Both must be enabled for full safety.

### Advance

#### Channel

- A Channel in RabbitMQ is:
    - A virtual connection inside a TCP connection
    - The real unit that does publishing, consuming, ack, confirms, flow-control
    - Concurrency-safe, because multiple goroutines may use it

#### Synchronization & Concurrency Control

``` go
type Channel struct {
	destructor sync.Once
	m          sync.Mutex // struct field mutex
	confirmM   sync.Mutex // publisher confirms state mutex
	notifyM    sync.RWMutex
    // ...
}

- `destructor`: ensures the channel is closed only once
- `m`: Global channel mutex, protects general channel state
- `confirmM`: protects publisher confirm state
- `notifyM`: protects notification listeners
```