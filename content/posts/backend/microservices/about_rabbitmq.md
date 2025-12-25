---
title: About RabbitMQ
date: 2025-12-21
summary: 📝 Quick notes on RabbitMQ...
tags:
  - backend
  - microservices
  - messaging
  - queue
---

## Definition
RabbitMQ is an open-source message broker:
- Producer: A person sending a letter
- Exchange: The post office
- Queue: The mailbox
- Consumer: The person receiving the letter
## Components
1. Broker
	- The message middleware that stores, routes, and delivers messages.
2. Connection
	- This connection is expensive to create, so you usually keep it open.
	- `Client <- TCP -> RabbitMQ`
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
	    - Fanout: Broadcast to all bound queues
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
	- `Producer → Exchange → (routing) → Queue`
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
## Message Broker
A message broker is an intermediary system that receives, stores, routes, and delivers messages between producers and consumers in an asynchronous and decoupled way.
### Without a broker
```
Service A → (HTTP) → Service B
```
- Problems:
	- If B is down → A fails
	- If B is slow → A is blocked
	- A must know where B is
	- No buffering, no retry, no back-pressure
### With a broker
```
Service A → Message Broker → Service B
```
- Benefits:
	- A does not care if B is online
	- Messages are buffered
	- B can consume at its own speed
	- Systems are loosely coupled