# Orleans vs Temporal
- [Core Paradigm Difference](#core-paradigm-difference)
- [Microsoft Orleans: High-Throughput Stateful Entities](#microsoft-orleans-high-throughput-stateful-entities)
- [Temporal: Durable Multi-Step Orchestration](#temporal-durable-multi-step-orchestration)
- [Decision Tree: When to Use What](#decision-tree-when-to-use-what)
- [Can They Be Used Together?](#can-they-be-used-together)
- [Comparison Matrix](#comparison-matrix)
- [Sources](#sources)

## Core Paradigm Difference

While both Microsoft Orleans and Temporal manage state across distributed systems, they solve fundamentally different problems:

* **Microsoft Orleans** models **active, stateful entities (nouns)**, It is a Virtual Actor framework optimized for high-concurrency, sub-millisecond, in-memory computation.
* **Temporal** models **durable, multi-step processes (verbs)**, It is a Durable Execution engine optimized for long-running workflows, distributed sagas, and guaranteed completion across infrastructure failures.

```text
                  Microsoft Orleans                       Temporal
         ┌─────────────────────────────────┐   ┌─────────────────────────────┐
Focus    │ High-frequency Stateful Entity  │   │ Multi-step Resilient Process│
Speed    │ Sub-millisecond (In-Memory RAM) │   │ 10ms - 100ms+ (Event Log)   │
Lifetime │ Ephemeral lifecycle (in-memory) │   │ Seconds, days, or months    │
Unit     │ Virtual Actor (Grain)           │   │ Workflow & Activity         │
         └─────────────────────────────────┘   └─────────────────────────────┘
```

## Microsoft Orleans: High-Throughput Stateful Entities

Microsoft Orleans implements the **Virtual Actor Pattern**.
The core unit of computation is a **Grain**: an isolated, stateful object with a unique identity.

### In-Memory Execution with Backing Retention
Orleans keeps active state in memory for maximum speed while supporting persistent backing stores (e.g., PostgreSQL, SQL Server, Redis, Azure Tables):
1. **Activation:** On receiving a message, Orleans transparently loads the Grain's state from the database into RAM.
2. **In-Memory Operations:** Successive calls run against in-memory state without database round trips.
3. **Persistence:** State changes persist explicitly using `WriteStateAsync()`.
4. **Passivization:** Idle Grains are quietly garbage-collected from RAM to free resources.

### Threading & Multi-Core Execution
Unlike runtimes constrained by a global lock, Orleans runs on .NET, taking full advantage of all CPU cores:
* **Multi-Core Cluster Execution:** Orleans utilizes the .NET ThreadPool and its own cooperative work-stealing scheduler to run thousands of distinct grains concurrently across all hardware cores.
* **Turn-Based Concurrency per Grain:** A single Grain instance processes incoming messages sequentially. This provides deterministic, single-threaded semantics per entity, completely eliminating race conditions, mutexes, and deadlocks.

## Temporal: Durable Multi-Step Orchestration

Temporal provides **Durable Execution**, guaranteeing that code will execute to completion regardless of server crashes, network partitions, or worker restarts.

### Event-Sourced Deterministic Replay
A Temporal application is split into two primitives:
* **Workflows:** Orchestration logic written in standard sequential code. Workflows must be deterministic because Temporal tracks their execution via an append-only event history log. If a worker crashes mid-workflow, a new worker replays the event history to restore the exact stack trace and continue execution.
* **Activities:** Units of work that execute non-deterministic actions and side effects (network I/O, third-party API calls, database writes).

### Built-in Resilience & Sagas
* **Transparent Durability:** A workflow can `await Task.Delay(TimeSpan.FromDays(30))` without consuming thread resources or risking loss if servers reboot.
* **Automatic Retries:** Built-in exponential backoff policies for transient network or external API failures.
* **Saga Orchestration:** Compensating transactions execute cleanly when downstream steps fail (e.g., refunding a payment if shipment creation fails).

## Decision Tree: When to Use What

```text
                       What are you designing?
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
A Stateful Entity / Digital Twin                  A Multi-Step Process / Saga
(Player, Device, Chat Room, Cart)                 (Checkout, Onboarding, Billing)
         │                                                 │
         ▼                                                 ▼
What is the latency budget?                       What are the reliability needs?
         │                                                 │
 ┌───────┴───────┐                                 ┌───────┴───────┐
 ▼               ▼                                 ▼               ▼
Sub-millisecond  Standard CRUD                     Must not fail,  Simple fire-and-forget
to low-ms RAM    low-concurrency                   runs over time  asynchronous task
 │               │                                 │               │
 ▼               ▼                                 ▼               ▼
USE ORLEANS      Stateless API                     USE TEMPORAL    Message Queue
(Virtual Actors) + Database                        (Durable Workflows) (Kafka, RabbitMQ)
```

## Can They Be Used Together?

Orleans and Temporal are complementary. A common distributed architecture combines them into an edge-and-orchestration pipeline:

```text
[ Client Request ]
       │  (WebSocket / HTTP)
       ▼
┌────────────────────────────────────────┐
│     Microsoft Orleans (Edge Layer)     │  <-- Sub-millisecond state mutations,
│   PlayerGrain / CartGrain (In-Memory)  │      validations, and fast session updates
└────────────────────────────────────────┘
       │
       │  Trigger checkout / multi-step saga
       ▼
┌────────────────────────────────────────┐
│      Temporal (Orchestration Layer)    │  <-- Durable payment processing, inventory
│   OrderCheckoutWorkflow (Durable)      │      reservation, third-party APIs, retries
└────────────────────────────────────────┘
       │
       │  Signal completion
       ▼
┌────────────────────────────────────────┐
│     Orleans Grain Reactivated/Notified │  <-- Push real-time notification to client
└────────────────────────────────────────┘
```

1. **Orleans at the edge:** Handles fast, concurrent user interactions in RAM without overwhelming backing databases.
2. **Temporal in the core:** Takes over when a business transaction demands transactional durability across multiple external boundaries.

## Comparison Matrix

| Feature | Microsoft Orleans | Temporal |
| :--- | :--- | :--- |
| **Architectural Role** | In-memory distributed entity state | Distributed workflow and saga orchestration |
| **Core Abstraction** | Grains (Virtual Actors) | Workflows & Activities |
| **State Storage** | Primary state in RAM; backed by DB (PostgreSQL, Redis, etc.) | Append-only event history log in database |
| **Execution Latency** | Microseconds to low milliseconds | Tens of milliseconds to seconds |
| **Crash Semantics** | Re-activates on healthy node from last saved DB state | Replays event history to resume exact line of code |
| **Concurrency Rule** | Turn-based single-threaded execution per Grain | Deterministic workflow execution with concurrent activities |
| **Multi core Scaling** | Fully parallel across all CPU cores via .NET ThreadPool | Polyglot worker pools horizontally scaled |
| **Language Support** | C# / .NET | Polyglot (C#, Go, TypeScript, Java, Python) |

## Sources

- [Microsoft Orleans Overview](https://learn.microsoft.com/en-us/dotnet/orleans/overview)
- [Temporal Documentation & Architecture](https://docs.temporal.io/temporal)
- [The Actor Model (Wikipedia)](https://en.wikipedia.org/wiki/Actor_model)
- [Actor model](https://barakadax.github.io/blog?article=Actor%20model)
- [Consistent hashing](https://barakadax.github.io/blog?article=Consistent%20hashing)
