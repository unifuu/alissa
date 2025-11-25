---
title: "Learning gRPC"
date: "2025-11-25"
summary: "📝 Quick notes on learning gRPC..."
tags: ["grpc", "backend"]
---

## stream_interfaces.go

### Server Streaming RPC (One Request, Many Responses)

- ServerStreamingClient[Res any]
    - Client-side
    - Recv() (*Res, error)
    - The client repeatedly calls Recv to read response messages from the server until io.EOF is returned (stream terminated successfully).
- ServerStreamingServer[Res any]
    - Server-side
    - Send(*Res) error
    - The server handler repeatedly calls Send to write response messages back to the client.

#### Connection and Disconnection

In a Server Streaming RPC, the connection between the client and the server does not disconnect immediately after the server receives the single initial request.
```
1. The Client sends one request message to the Server.
2. The Server receives the request and begins processing.
3. The Server then streams back multiple response messages over the same persistent connection.
4. The stream remains active until the server is done sending responses.
5. The connection is finally closed (or returned to the connection pool) when the server finishes sending all responses and sends a special status/trailer, which the client reads as an io.EOF error.
```

Analogy: Live News Report
```
Streaming Concept   Live News Analogy
Client Request      The client turns on the TV (sends a request for a specific news channel).
Server's Role       The news station (server) starts its continuous live broadcast.
Server Responses    The consecutive news segments (multiple responses).
Stream Termination  The client turns off the TV or the broadcast ends (stream closes).
```