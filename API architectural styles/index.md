# API Architectural Styles
- [What is an API](#what-is-an-api)
- [What is a Socket](#what-is-a-socket)
- [Resource-Oriented](#resource-oriented)
- [Query-Based / Data-Graph](#query-based--data-graph)
- [Remote Procedure Call (RPC)](#remote-procedure-call-rpc)
- [Persistent Stream / Real-Time](#persistent-stream--real-time)
- [Asynchronous Event-Driven (Pull) / Message-Oriented](#asynchronous-event-driven-pull--message-oriented)
- [Reverse HTTP (Event-Driven Push)](#reverse-http-event-driven-push)
- [Sources](#sources)

## What is an API

An Application Programming Interface (API) is a set of rules, protocols, and definitions that allows different software applications to communicate with each other.
It acts as a contract, enabling one system to request data or trigger functionality in another system.
APIs abstract the underlying implementation details, exposing only the necessary interface to consumers.

## What is a Socket

A socket is an endpoint of a two-way communication link between two programs running over a network.
It is defined by an IP address and a port number.
While an API defines *what* data can be exchanged and the logical rules of interaction, a socket provides the low-level *how*—the channel through which raw data bytes flow.

## Resource-Oriented

Resource-oriented architectures center around resources, which represent individual entities or data objects.

### REST (Representational State Transfer)
REST is an architectural style designed around resources identified by URLs.
It leverages standard, stateless HTTP methods (GET, HEAD, CONNECT, POST, PUT, DELETE, PATCH, TRACE, QUERY, OPTIONS) and standard HTTP status codes.
REST typically uses JSON or XML to transfer resource representations.
It is highly scalable, loose-coupled, and is the most common paradigm for public web APIs.

## Query-Based / Data-Graph

Query-based architectures treat backend data as an interconnected graph, allowing clients to query exact data shapes.

### GraphQL
GraphQL is a query language and server-side runtime that allows clients to request only the specific fields they need.
Instead of hitting multiple endpoints, clients send queries to a single endpoint.
It prevents over-fetching and under-fetching of data.
It relies on a strongly-typed schema to define the queries, mutations, and types available.

## Remote Procedure Call (RPC)

RPC architectures focus on actions or behaviors, allowing client applications to invoke subroutines on a remote server as if they were local calls.

### gRPC
gRPC is a high-performance, open-source RPC framework developed by Google.
It uses Protocol Buffers (protobuf) as its interface definition language and serialization format.
It runs over HTTP/2, enabling features like bi-directional streaming, multiplexing, and header compression.
It is ideal for low-latency, high-throughput microservice communication.

### JSON-RPC / XML-RPC
JSON-RPC and XML-RPC are simple, lightweight RPC protocols encoded in JSON and XML respectively.
A client sends a POST request containing a method name and parameters, and the server returns a result or an error.
They are transport-agnostic, simple to implement, but lack formal schemas or advanced streaming features.

### SOAP (Simple Object Access Protocol)
SOAP is a highly structured, XML-based protocol for exchanging information.
It relies on strict XML schemas (WSDL) and offers built-in standards for security (WS-Security) and transaction compliance (ACID support).
It is formal, verbose, and mostly used in legacy enterprise systems and financial sectors.

## Persistent Stream / Real-Time

Persistent stream architectures maintain long-lived connections for real-time, low-latency communication.

### WebSockets
WebSockets provide full-duplex, bi-directional communication channels over a single TCP connection.
Communication starts with an HTTP handshake that upgrades the connection to the WebSocket protocol.
It uses lightweight framing to transmit text or binary data with minimal overhead, making it ideal for real-time chat, multiplayer gaming, and financial trackers.

### SSE (Server-Sent Events)
SSE is a standard HTTP-based unidirectional streaming technology.
The server holds a persistent HTTP connection open to push updates to the client in real-time as text streams.
Unlike WebSockets, communication is strictly one-way (server-to-client) and does not require protocol upgrades.
It is natively supported in web browsers via the EventSource API.

## Asynchronous Event-Driven (Pull) / Message-Oriented

Message-oriented architectures rely on message queues or streaming logs to decouple services asynchronously.

### Pub/Sub Messaging
Publishers send messages to an intermediary message broker without knowing who the subscribers are.
Subscribers register interest in specific topics or queues to receive messages asynchronously.
- **AMQP:** Advanced Message Queuing Protocol, used by RabbitMQ for robust, feature-rich enterprise messaging.
- **MQTT:** Message Queuing Telemetry Transport, a lightweight protocol designed for IoT and low-bandwidth networks.
- **NATS:** A simple, high-performance, cloud-native messaging system.

### Log-Based Event Streaming
Messages are appended sequentially to a distributed, immutable commit log.
Consumers read from the log at their own pace, maintaining their own offsets.
This allows for massive scale, high throughput, and the ability to replay historical events.
- **Kafka:** The industry standard distributed event streaming platform.
- **Redpanda:** A modern, C++-written, Kafka-compatible alternative with no JVM dependencies.

### Durable Execution
Durable execution platforms orchestrate complex, stateful workflows.
They guarantee that code execution will run to completion, surviving process crashes, network failures, or server restarts.
- **Temporal:** Achieves durable execution by recording all workflow execution events in an event-sourced log, allowing the runner to reconstruct the program state at any point.

## Reverse HTTP (Event-Driven Push)

Reverse HTTP architectures reverse the client-server relationship, letting the server push events to the client.

### Webhooks
Webhooks are user-defined HTTP callbacks triggered by specific events on a server.
Instead of the client polling the server for updates, the server sends a POST request containing the event payload to the client's registered URL.
It is widely used for integrating third-party services like Stripe payment notifications or GitHub commit alerts.

## Sources

- [Gossip](https://barakadax.github.io/blog?article=Gossip)
- [C.R.U.D](https://barakadax.github.io/blog?article=C.R.U.D)
- [Architectural Styles and the Design of Network-based Software Architectures (Roy Fielding)](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
- [WebSockets vs. Server-Sent Events (SSE)](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)
- [Introduction to Kafka (Apache Kafka)](https://kafka.apache.org/documentation/#introduction)
