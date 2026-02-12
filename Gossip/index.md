# Gossip
- [What is the gossip protocol?](#what-is-the-gossip-protocol)
- [How does it work?](#how-does-it-work)
  - [Push](#push)
  - [Pull](#pull)
  - [Push-Pull](#push-pull)
- [Examples of of usages](#examples-of-of-usages)
- [Pros and cons](#pros-and-cons)
- [Sources](#sources)

## What is the gossip protocol?

Gossip is a multicast protocol for sharing information between nodes in a distributed system.
Just like an epidemic spreads from person to person, the gossip protocol spreads information from node to node.

## How does it work?

A node has information but that node is only aware of N amount of nodes out of total nodes in the system.
Node only update the information and redistribute it if the new information has higher version than the current one.
There are 3 ways to gossip, push, pull and push-pull, all 3 ways convergence complexity is `O(log N)`.

### Push
Node sends the information only to the N nodes it is aware of.
Those nodes spread the information to the N nodes they are aware of until the information is spread across the whole system.

![Push GIF]()

### Pull
Node periodically asks the N nodes it is aware of for the information.
Once new information is received, the node will update other nodes with the information when those nodes will pull the information for this node until the information is spread across the whole system.

![Pull GIF]()

### Push-Pull
Nodes have both push and pull capabilities.

## Examples of of usages

- **Apache Hadoop**: communication and managing member nodes.
- **Amazon DynamoDB**: node discovery and cluster managment.
- **Kubernetes**: node discovery and cluster managment.
- **Torrent**: tracking of files and peers.
- **Microsoft Teams**: presence detection and contact discovery.
- **Slack**: presence detection and messaging synchronization.
- **Cassandra**: cluster membership, propagate metadata and detect failures.
- **WhatsApp**: message routring and synchronization between devices.

## Pros and cons

### Pros

- **Scalability**: The gossip protocol is highly scalable, as it can be used to spread information between nodes in a distributed system.
- **Fault tolerance**: The gossip protocol is highly fault tolerant, as it can be used to spread information between nodes in a distributed system.
- **Simplicity**: The gossip protocol is simple to implement, as it can be used to spread information between nodes in a distributed system.

### Cons

- **Latency**: The gossip protocol is not the fastest way to spread information between nodes in a distributed system.
- **Bandwidth**: The gossip protocol can be bandwidth intensive, as it can be used to spread information between nodes in a distributed system.
- **Complexity**: The gossip protocol can be complex to implement, as it can be used to spread information between nodes in a distributed system.
- **Nodes not knowing each other**: Node might end up not knowing each other so they won't succeed in push and pull, therefor data won't spread.
- **Connection waste**: In push a node might push to another node that already has the information, in pull a node that already pulled that data will continue to pull for information periofically even when there is no new data, wasting resources.

## Sources

- [Martin Kleppmann - Distributed Systems - Broadcast algorithms](https://youtu.be/77qpCahU3fo?si=5eyEc-48jOnzZDUe&t=186)
- [Gossip protocol](https://en.wikipedia.org/wiki/Gossip_protocol)