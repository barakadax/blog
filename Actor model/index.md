# Actor model
1. [What is the Actor Model](#what-is-the-actor-model)
2. [Why We Need It](#why-we-need-it)
3. [Traditional Actor Model](#traditional-actor-model)
4. [Virtual Actor Model](#virtual-actor-model)
5. [Traditional vs. Virtual](#traditional-vs-virtual)
6. [Technical Showcase: OrleansMapReduce](#technical-showcase-orleansmapreduce)
7. [Sources](#sources)

## What is the Actor Model

The Actor Model is a mathematical model for concurrent computation that treats "actors" as the universal primitives.
Each actor is an isolated entity that communicates exclusively through asynchronous message passing. Upon receiving a message, an actor can:
- Send a finite number of messages to other actors.
- Create a finite number of new actors.
- Designate the behavior to be used for the next message it receives.

Isolation is absolute; actors do not share memory, eliminating the need for locks.

## Why We Need It

Traditional concurrency models relying on shared state and threads often lead to:
- **Shared State Contention:** Managing access to shared memory via locks is complex and error-prone.
- **Deadlocks:** Circular dependencies between locks can freeze applications.
- **Scalability Issues:** Thread-per-connection models do not scale to millions of concurrent operations.
- **Fault Tolerance:** Distributed systems need a way to isolate failures so they don't cascade.

The Actor Model solves these by providing a clear boundary for state and logic, enabling massive horizontal scalability and "let it crash" error handling.

## Traditional Actor Model

In the traditional model, actors have an explicit lifecycle. Developers are responsible for creating, supervising, and terminating actors.
- **Implementations:** Erlang, Elixir, Akka (JVM), Pekko, Actix (Rust), CAF (C++), Nact (Node.js), Bastion, Comedy.
- **Lifecycle:** Actors must be started manually. If a system crashes, the supervision tree must decide how to restart them.

## Virtual Actor Model

The Virtual Actor Model, pioneered by Microsoft Orleans, introduces a higher level of abstraction where actors (called "Grains") are "always-on."
- **Automatic Activation:** A grain is automatically instantiated when a message is sent to its unique ID.
- **Passivation:** Grains are automatically removed from memory when idle to save resources, but their state remains persistent.
- **Location Transparency:** The runtime handles where a grain lives in a cluster; the caller never needs to know the physical address.

## Traditional vs. Virtual

| Feature | Traditional Actor Model | Virtual Actor Model |
| :--- | :--- | :--- |
| **Lifecycle** | Explicit (Create/Stop) | Managed (Always-on) |
| **Placement** | Manual/Supervised | Automatic/Transparent |
| **State** | In-memory (Manual Persistence) | Automatic Persistence/Activation |
| **Examples** | Akka, Pekko, Erlang, Actix | Orleans, Dapr, Proto.Actor, Darlean |

## Virtual Actor Examples

- **Microsoft Orleans:** The original virtual actor framework for .NET.
- **Proto.Actor:** A cross-platform actor model (Go/C#) supporting both traditional and virtual patterns.
- **Dapr:** Uses the virtual actor pattern via its Actors building block to provide multi-language support.

## Technical Showcase: OrleansMapReduce

The [OrleansMapReduce](https://github.com/barakadax/OrleansMapReduce) repository demonstrates the power of the Virtual Actor model in simplifying distributed data processing.
By leveraging Orleans Grains:
- **Simplified Distribution:** The Map and Reduce phases are distributed across a cluster without manual orchestration.
- **Resilience:** If a node failing during a Map task, the Virtual Actor runtime ensures the task can be resumed or retried seamlessly.
- **Scalability:** It abstracts the complexity of coordination, allowing developers to focus on the business logic of the MapReduce algorithm.

## Sources

- [Microsoft Orleans Documentation](https://learn.microsoft.com/en-us/dotnet/orleans/)
- [The Actor Model (Wikipedia)](https://en.wikipedia.org/wiki/Actor_model)
- [Dapr Actors Building Block](https://docs.dapr.io/developing-applications/building-blocks/actors/actors-overview/)
- [Akka Documentation](https://akka.io/docs/)
- [Proto.actor](https://asynkron.se/docs/protoactor/what-is-protoactor/)
- [Actix](https://actix.rs/docs/actix/actor/)
- [Pekko](https://pekko.apache.org/)
- [Darlean](https://darlean.io/)
