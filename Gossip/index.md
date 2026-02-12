# Gossip
- [What is the gossip protocol?](#what-is-the-gossip-protocol)
- [How does it work?](#how-does-it-work)
  - [Push](#push)
  - [Pull](#pull)
  - [Push-Pull](#push-pull)
- [Gossip Variants](#gossip-variants)
  - [Anti-Entropy](#anti-entropy)
  - [Rumor-Mongering](#rumor-mongering)
- [Real-world Examples](#real-world-examples)
- [Pros and Cons](#pros-and-cons)
- [Sources](#sources)

## What is the gossip protocol?

Gossip is a decentralized, peer-to-peer multicast protocol for sharing information between nodes in a distributed system. 
Just like an epidemic spreads from person to person, the gossip protocol spreads information from node to node through random peer selection. It is designed to ensure **eventual consistency** and high **fault tolerance**.

## How does it work?

A node has information but that node is only aware of N amount of nodes out of total nodes in the system.
Node only update the information and redistribute it if the new information has higher version than the current one.
There are 3 ways to gossip, push, pull and push-pull, all 3 ways convergence complexity is `O(log N)`.

### Push
A node with new information actively sends (pushes) it to its N known peers.
- **Early Phase**: Highly efficient; information spreads exponentially fast.
- **Late Phase**: Inefficient; nodes keep pushing to peers who likely already have the data, wasting bandwidth.

![Push GIF](https://raw.githubusercontent.com/barakadax/blog/refs/heads/Master/Gossip/push.gif)

### Pull
A node periodically asks (pulls) its N known peers for updates.
- **Early Phase**: Inefficient; many pulls return no new data if few nodes are "infected".
- **Late Phase**: Highly efficient; uninformed nodes actively seek out the remaining updates, solving the "blind spot" problem.

![Pull GIF](https://raw.githubusercontent.com/barakadax/blog/refs/heads/Master/Gossip/pull.gif)

### Push-Pull
Nodes combine both capabilities. This is the **optimal** approach for rapid dissemination.
Leveraging the speed of Push in the early phase and the reliability of Pull in the late phase.

## Gossip Variants

### Anti-Entropy
Used for state reconciliation (full data synchronization).
Nodes exchange their entire dataset or summaries to identify and fix inconsistencies. This is a "safety net" for eventual consistency.

### Rumor-Mongering
Used for rapid propagation of new events or updates.
When a node receives a "rumor", it gossips it frequently.
To prevent infinite loops, nodes use a "cooling off" period (SIR model: Susceptible, Infected, Removed) where they stop gossiping after a message has spread sufficiently.

## Real-world Examples

- **Apache Cassandra**: Uses gossip for cluster membership, metadata propagation, and failure detection via the **Phi Accrual Failure Detector** (calculating a suspicion level rather than a fixed timeout).
- **Amazon DynamoDB**: Employs gossip for node discovery and cluster management, utilizing **Seed Nodes** to prevent network partitions.
- **Kubernetes**: Used by various components (like Calico or memberlist) for node discovery and health checks.
- **Apache Hadoop**: Communication and management of member nodes across large clusters.
- **BitTorrent**: Tracking files and peers in a decentralized manner.
- **Messaging (Slack/WhatsApp)**: Presence detection and message routing synchronization between clusters and devices.
- **Microsoft Teams**: presence detection and contact discovery.

## Pros and Cons

### Pros
- **Scalability**: Nodes only talk to a small subset of peers, so the load on any single node remains constant even as the cluster grows.
- **Fault Tolerance**: No central point of failure; if a node goes down, information naturally routes around it.
- **Simplicity**: No complex leader election or consensus algorithms (like Paxos/Raft) are required for basic state sharing.

### Cons
- **Latency**: Since it relies on random rounds, it is not "real-time". Information takes time to converge across the whole system.
- **Bandwidth Overhead**: Redundant messages are common, especially in large clusters or pure Push models.
- **Eventual (Not Strong) Consistency**: Data might be stale on some nodes for a short period.
- **Connectivity Risks**: If nodes don't know enough peers, "islands" can form where data never reaches certain partitions.

## Sources

- [Martin Kleppmann - Distributed Systems - Broadcast algorithms](https://youtu.be/77qpCahU3fo?si=5eyEc-48jOnzZDUe&t=186)
- [Gossip protocol](https://en.wikipedia.org/wiki/Gossip_protocol)
