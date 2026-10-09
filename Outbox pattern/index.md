# Outbox pattern
- [The Dual-Write Problem](#the-dual-write-problem)
- [How the Outbox Pattern Works](#how-the-outbox-pattern-works)
- [Message Relay Strategies](#message-relay-strategies)
  - [Polling Publisher](#polling-publisher)
  - [Transaction Log Tailing (CDC / Debezium)](#transaction-log-tailing-cdc--debezium)
- [The Myth of Exactly-Once: CDC vs Outbox vs Kafka Transactions](#the-myth-of-exactly-once-cdc-vs-outbox-vs-kafka-transactions)
  - [What Kafka Transactions Actually Guarantee](#what-kafka-transactions-actually-guarantee)
  - [Why Consumer Duplicates Remain Inevitable](#why-consumer-duplicates-remain-inevitable)
  - [Defensive Idempotent Consumer Design](#defensive-idempotent-consumer-design)
- [Trade-offs & Operational Realities](#trade-offs--operational-realities)
- [Sources](#sources)

## The Dual-Write Problem

In distributed microservices, a common requirement is to mutate state in a local database and publish a corresponding event to a message broker (such as Kafka, redpanda, pubsub or RabbitMQ):

```text
       ┌────────────────────────┐
       │     Order Service      │
       └────┬──────────────┬────┘
            │ 1. UPDATE    │ 2. PUBLISH
            ▼              ▼
     ┌───────────┐   ┌───────────┐
     │ Database  │   │   Kafka   │
     └───────────┘   └───────────┘
```

This direct dual-write approach has an inherent reliability flaw:
1. **Network partition or crash after step 1:** The database commits the order, but the service crashes before publishing to Kafka, downstream services (billing, inventory) never learn about the order.
2. **Failure before step 1:** If you reverse the order and publish to Kafka first, a database rollback leaves an orphaned event in the stream that downstream services process for an order that does not exist.

Two-Phase Commit (2PC / XA transactions) across heterogeneous systems is slow, introduces distributed deadlocks, and is not supported by modern streaming platforms like Kafka.

## How the Outbox Pattern Works

The **Transactional Outbox Pattern** solves the dual-write problem by leveraging the database's native ACID transaction guarantees.
Instead of publishing directly to the message broker, the application writes both the domain entity and an outbox event record into the **same database within a single local transaction**:

```text
[ Client Request ]
       │
       ▼
┌──────────────────────────────────────────────┐
│  Single ACID Transaction                     │
│  ├── 1. INSERT INTO orders (...)             │
│  └── 2. INSERT INTO outbox_messages (...)    │
└──────────────────────────────────────────────┘
       │ (Committed atomically)
       ▼
┌──────────────────┐
│ Relational DB    │
│ ┌──────────────┐ │
│ │ orders       │ │
│ ├──────────────┤ │
│ │ outbox       │ │
│ └──────────────┘ │
└────────┬─────────┘
         │
         ▼ (Asynchronous Relay)
┌──────────────────┐
│  Message Relay   │ ───> [ Kafka / RabbitMQ ]
└──────────────────┘
```

Either both the business state change and the outbox event persist, or neither does.
Inconsistency between local state and the outbound message queue is eliminated.

## Message Relay Strategies

Once messages are safely committed to the `outbox` table, a background process must forward them to the message broker.

### Polling Publisher

A worker periodically queries the `outbox` table for unpublished records, sends them to Kafka, and marks them as processed (or deletes them):

```sql
SELECT id, aggregate_id, payload
FROM outbox_messages
WHERE status = 'PENDING'
ORDER BY created_at ASC
LIMIT 100
FOR UPDATE SKIP LOCKED;
```

* **Pros:** Simple to implement; requires no specialized database extensions or external infrastructure.
* **Cons:** Introduces polling latency (e.g., 500ms–1s delay); adds read contention to the database; requires indexes on `status` and `created_at`.

### Transaction Log Tailing (CDC / Debezium)

Modern production architectures stream database Write-Ahead Logs (WAL) using Change Data Capture tools such as **Debezium**:

* **How it works:** Debezium monitors the Postgres WAL or MySQL binlog, when a row is committed to `outbox_messages`, Debezium streams the change directly into Kafka in near real-time (sub-10ms).
* **Pros:** Zero query overhead on the primary database; no polling lag; preserves strict commit ordering.
* **Cons:** Requires running Kafka Connect and CDC connectors; schema migrations on outbox tables require careful connector coordination.

## The Myth of Exactly-Once: CDC vs Outbox vs Kafka Transactions

A common misconception is that combining the Outbox Pattern with Kafka transactions delivers magical "end-to-end exactly-once" delivery.
In real-world networks, **at-least-once delivery with idempotent consumption is the only mathematically sound model**.

### What Kafka Transactions Actually Guarantee

Kafka's transactional producer (`transactional.id`, `enable.idempotence=true`) guarantees atomic writes **within Kafka itself**:
- Messages sent across multiple topics and partitions within a transaction commit or abort as a unit.
- Downstream consumers with `isolation.level=read_committed` will never see uncommitted or aborted messages.

However, Kafka transactions **do not span outside the broker boundary**. They cannot coordinate atomically with an external consumer's database write.

### Why Consumer Duplicates Remain Inevitable

Even if the Outbox pattern guarantees that a message is placed onto Kafka, duplicate execution at the consumer side is unavoidable due to the **Two Generals Problem**:

1. Consumer pulls message offset `105`.
2. Consumer updates its database successfully.
3. Consumer attempts to commit offset `105` back to Kafka (`__consumer_offsets`).
4. A network hiccup occurs, or the consumer node dies before the commit acknowledgment arrives.
5. Kafka triggers a consumer group rebalance. Another consumer replica picks up partition ownership starting from the last committed offset (`104`) and processes message `105` a second time.

```text
[ Kafka Topic ] ─── 1. Pull Msg #105 ───> [ Consumer ]
                                                │
                                                ▼ 2. Update DB (Success)
                                                │
[ Kafka Topic ] <── 3. Commit Offset 105 ───────X (Network Timeout / Crash)

*Rebalance occurs* -> New consumer pulls Msg #105 again -> DUPLICATE!
```

### Defensive Idempotent Consumer Design

Because at-least-once delivery is the physical reality, the consumer must be designed to be completely idempotent:

1. **Unique Idempotency Keys:** Every outbox record includes a deterministic `id` or UUID.
2. **Processed Events Table:** The consumer writes the event's UUID into a `processed_events` table within the same local transaction as its state mutation:
   ```sql
   BEGIN TRANSACTION;
   INSERT INTO accounts (id, balance) VALUES ('123', 500);
   INSERT INTO processed_events (event_id, processed_at) VALUES ('evt_abc987', NOW());
   COMMIT;
   ```
   If a duplicate event arrives, the database rejects the duplicate `event_id` via a primary key conflict, discarding the duplicate safely.
3. **Natural Idempotency:** Prefer idempotent updates (e.g., `SET status = 'APPROVED'`) over relative increments (`SET balance = balance + 50`).

## Trade-offs & Operational Realities

| Consideration | Production Strategy |
| :--- | :--- |
| **Outbox Table Bloat** | High-throughput services write millions of outbox rows daily, implement partition pruning (e.g., Postgres table partitioning by day) and background truncating jobs. |
| **Ordering Guarantees** | To preserve per-entity ordering, use the entity ID (`aggregate_id`) as the Kafka message partition key, never use random keys if causal ordering matters. |
| **Relay Lag Monitoring** | Expose metrics on unread outbox rows (`COUNT(*) WHERE status = 'PENDING'`) or CDC lag against Postgres LSN (Log Sequence Number) to alert before backlogs escalate. |

## Sources

- [Transactional Outbox Pattern - Microservices.io](https://microservices.io/patterns/data/transactional-outbox.html)
- [Debezium Architecture Documentation](https://debezium.io/documentation/reference/stable/architecture.html)
- [Kafka Exactly-Once Semantics (KIP-98)](https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging)
