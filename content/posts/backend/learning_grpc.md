---
title: "Learning gRPC"
date: "2025-11-25"
summary: "📝 Quick notes on learning gRPC..."
tags: ["backend", "grpc"]
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

#### Communication Flow

In a Server Streaming RPC, the connection between the client and the server does not disconnect immediately after the server receives the single initial request.
1. The Client sends one request message to the Server.
2. The Server receives the request and begins processing.
3. The Server then streams back multiple response messages over the same persistent connection.
4. The stream remains active until the server is done sending responses.
5. The connection is finally closed (or returned to the connection pool) when the server finishes sending all responses and sends a special status/trailer, which the client reads as an io.EOF error.

#### An Analogy: Live News Report
```
Streaming Concept   Live News Analogy
Client Request      The client turns on the TV (sends a request for a specific news channel).
Server's Role       The news station (server) starts its continuous live broadcast.
Server Responses    The consecutive news segments (multiple responses).
Stream Termination  The client turns off the TV or the broadcast ends (stream closes).
```

### Client Streaming RPC (Many Requests, One Response)

- ClientStreamingClient[Req any, Res any]
    - Client-side
    - Send(*Req) error, CloseAndRecv() (*Res, error)
    - The client repeatedly calls Send for requests, then calls CloseAndRecv to close the stream and receive the single final response.
- ClientStreamingServer[Req any, Res any]
    - Server-side
    - Recv() (*Req, error), SendAndClose(*Res) error
    - The server handler repeatedly calls Recv for requests, then calls SendAndClose to send the final response and close the stream.

#### Communication Flow

- The client is responsible for sending a sequence of messages to the server using the streaming connection.
    - Action: The client repeatedly calls the Send(request) method on the stream object.
    - The Stream Body: These requests are sent sequentially over the single, open HTTP/2 stream.
    - Finalization: Once the client has sent all the required messages, it must explicitly call the CloseSend() method (or the CloseAndRecv() which implicitly closes the stream). This is the crucial step that signals to the server, "I am finished sending data."
    - Receiving the Final Response: Only after calling CloseSend() does the client block (wait) and call Recv() (via CloseAndRecv()) to wait for the server's single response message and the final status.
- The server's role is to receive and process the stream of data, and then provide a single summary result.
    - Action: The server handler repeatedly calls the Recv() method on the stream object (defined by the ClientStreamingServer interface).
    - Processing: As it receives each message, the server typically performs some aggregation, logging, or state update (e.g., tallying a count, accumulating data).
    - Stream End: The server continues calling Recv() until it receives the end-of-stream signal (io.EOF), which was triggered by the client's CloseSend().
    - Final Response: Once the server knows the client is done streaming (it received io.EOF), it calculates the final result and sends the single response message by calling SendAndClose(response). This simultaneously sends the response and terminates the entire RPC.

#### An Analogy: Uploading a Large File
```
Streaming Concept                   Data Upload Scenario
Client	                            A local application with a large file
Send(Request)                       The client streams the file data in small, manageable chunks to the server.
Server's Recv()                     The server reads and stores/processes each chunk as it arrives.
CloseSend()                         The client sends the final chunk and signals it's done sending.
Server's SendAndClose(Response)     The server finishes processing all chunks, calculates the final checksum/summary/status, and sends it back in one message.
```

### Bidirectional Streaming RPC (Many Requests, Many Responses)

- BidiStreamingClient[Req any, Res any]
    - Client-side
    - Send(*Req) error, Recv() (*Res, error)
    - The client can call both Send and Recv concurrently to exchange messages with the server.
- BidiStreamingServer[Req any, Res any]
    - Server-side
    - Recv() (*Req, error), Send(*Res) error
    - The server handler can call both Recv and Send concurrently to exchange messages with the client.

#### Communication Flow

1. Client's Role
- The client performs two main concurrent tasks:
    - Sending: It repeatedly calls Send(request). It can send as many or as few messages as it needs.
    - Receiving: It repeatedly calls Recv() to process the messages streaming back from the server.
    - The client signals that it has finished sending its stream by calling CloseSend(), but it must continue calling Recv() until the server also finishes its stream and sends the final status.

2. Server's Role
- The server also performs two main concurrent tasks:
    - Receiving: It repeatedly calls Recv() to process the incoming requests from the client.
    - Sending: It repeatedly calls Send(response) to push response messages back to the client.
    - The server signals the end of its stream by returning the final status when the main handler function exits.

#### An Analogy: Interactive Chat or Real-Time Game

```
Streaming Concept	Real-Time Chat Application
Client's Send()	    Sending messages you type into the chat window.
Server's Send()	    Pushing new messages from other users back to your chat window.
Client's Recv()	    Reading messages from others sent by the server.
Server's Recv()	    Reading the messages you send.
Stream Open	        Maintaining a persistent connection for the chat session.
```

## REST  vs RPC

### REST: Mobile Phone Initiates a REST API Call (HTTP/1.1)

Assume the mobile app makes a request to fetch user data, for example: GET https://api.example.com/users/123.

| Phase | Step | Detail | Protocol/Data Format |
| ----- | ---- | ------ | --------------- |
| Application Layer | 1. Function Call | The App calls a method in the HTTP Client Library (e.g., Retrofit/Alamofire) | Programming Language Function |
| | 2. Request Construction | The client library assembles the HTTP Request: Method (GET), URL, and Headers (Accept: application/json) | HTTP Request Format (Plain Text) |
| Protocol Layer | 3. DNS Resolution | The system resolves the domain name api.example.com to the server's IP address | DNS Protocol |
| | 4. Connection Establishment | The App initiates a TCP connection with the server's IP, usually followed by a TLS/SSL Handshake (for HTTPS). | TCP/IP, TLS/SSL |
| Transport Layer | 5. Request Transmission | The HTTP Request (in plain text) is packaged by the transport layer and sent to the server over the TCP connection | TCP/IP |
| | 6. Server Receive | The server receives the TCP/IP packets and reassembles the complete HTTP Request | |
| | 7. Business Logic | The server application executes the business logic | |
| Response | 8. Response Construction | The server packages the data into a JSON formatted text, setting the HTTP Status Code and headers | JSON (Plain Text) |
| | 9. Response Transmission | The HTTP Response is sent back to the phone over the TCP connection | TCP/IP |
| Client Processing | 10. Receive & Parse | The mobile app receives the response, and the HTTP Client Library parses the HTTP headers and status code | |
| | 11. Deserialization | The Client Library deserializes the JSON text into a programming language object | JSON Parser (High CPU usage) |
| | 12. Result Presentation | The App receives the User object and updates the User Interface | Programming Language Function |

### RPC: Mobile Phone Initiates a gRPC Call

Assume the App calls a gRPC service method, such as UserService.GetUser(id: 123).

| Phase | Step | Detail | Protocol/Data Format |
| ----- | ---- | ------ | --------------- |
| Application Layer | 1. Stub Call | The App calls a function on the locally generated gRPC Client Stub, passing the parameters | Programming Language Function |
| | 2. Serialization (Protobuf) | The Stub uses Protocol Buffers to efficiently serialize the parameters into a highly compact binary format (the Payload) | Protocol Buffers (Binary) |
| Protocol Layer (HTTP/2) | 3. HTTP/2 Frame | The gRPC framework encapsulates the serialized binary Payload into an HTTP/2 Data Frame, and the service method name is carried in the Header Frame | HTTP/2 (Binary) |
| | 4. Connection Management | gRPC typically uses a single, persistent HTTP/2 connection. If not established, DNS resolution and TLS handshake occur | TCP/IP, TLS/SSL |
| Transport Layer | 5. Request Transmission | The HTTP/2 frames are sent to the server over the TCP connection. Multiple requests/responses can be processed in parallel on the same connection | TCP/IP |
| | 6. Server Deserialization | The server receives the HTTP/2 frames, and the gRPC Server Library very quickly deserializes the Protobuf binary Payload back into an object in the server's language | Protocol Buffers (Efficient CPU usage) |
| | 7. Business Logic | The server executes the business logic | |
| Response Construction | 8. Response Serialization | The server serializes the result object back into a binary Payload using Protobuf | Protocol Buffers (Binary) |
| | 9. Response Transmission | The server encapsulates the Protobuf Payload into HTTP/2 frames and sends them back to the client | HTTP/2 (Binary) |
| Client Processing | 10. Receive & Deserialization | The mobile App receives the response, and the Stub very quickly deserializes the Protobuf Payload into the result object | Protocol Buffers (Efficient CPU usage) |
| | 11. Result Presentation	| The App receives the result object and updates the UI | Programming Language Function |

### Key Differences Comparison

| Feature | REST API (HTTP/1.1 & JSON) | gRPC (HTTP/2 & Protobuf) |
| ------- | -------------------------- | ------------------------ |
| Underlying Protocol | HTTP/1.1 (Common) | HTTP/2 (Mandatory) |
| Connection Efficiency | Prone to "Head-of-Line Blocking," typically short-lived connections. | Supports Multiplexing, typically long-lived connections |
| Data Format | JSON/XML (Plain Text, Human-Readable) | Protocol Buffers (Binary, Machine-Efficient) |
| Serialization/Deserialization | Slower, high CPU overhead (text parsing) | Extremely Fast, low CPU overhead (binary encoding/decoding) |
| Transmission Size | Larger (redundant text headers, verbose JSON keys) | Extremely Small (compressed headers, compact binary payload) |
| Abstraction Level | Focuses on Resources (/users/123) and Standard Actions (GET/POST) | Focuses on Operations/Functions |
| Interface Contract | Loose, relies on documentation | Strict, enforced via Protobuf definition files |
| Real-time Comms | Requires external solutions (WebSockets) | Built-in Streaming, handles bidirectional real-time communication |