# Consistent Hashing
- [The Modulo Hashing Bottleneck](#the-modulo-hashing-bottleneck)
- [Consistent Hashing and the Hash Ring](#consistent-hashing-and-the-hash-ring)
  - [Virtual Nodes](#virtual-nodes)
- [Application-Level Routing in Kubernetes](#application-level-routing-in-kubernetes)
  - [Pod-Managed Ring Architecture](#pod-managed-ring-architecture)
  - [Out-of-the-Box with Microsoft Orleans in C#](#out-of-the-box-with-microsoft-orleans-in-c)
- [Multi-Pod Workflow: Hashing, Peer Search, and Fallback](#multi-pod-workflow-hashing-peer-search-and-fallback)
  - [Step 1: Ingress & Ring Lookup](#step-1-ingress--ring-lookup)
  - [Step 2: Peer-to-Peer Routing](#step-2-peer-to-peer-routing)
  - [Step 3: Cache Miss & Last-Pod Fallback](#step-3-cache-miss--last-pod-fallback)
- [Summary Table](#summary-table)
- [Sources](#sources)

## The Modulo Hashing Bottleneck

In distributed caching, the simplest partitioning method is **Modulo Hashing**:

```text
server_index = hash(key) % N
```

While effective in static clusters, this strategy breaks down when $N$ changes:
* **Mass Invalidation:** Adding or removing a single node changes the modulo for almost every key, up to **$N / (N + 1)$** of all keys (e.g., 80% when scaling from 4 to 5 servers) instantly remap to the wrong node.
* **Cache Stampede:** The cluster drops its cache hit rate near zero, overloading primary databases.

## Consistent Hashing and the Hash Ring

**Consistent Hashing** maps both **data keys** and **node identifiers** onto a shared circular 2^{32}-1 (or 2^{64}-1) unique collection called the **Hash Ring**.

```text
               Node A (Hash: 100)
                  /         \
                 /           \
  Node D (Hash: 900)       Node B (Hash: 400)
                 \           /
                  \         /
               Node C (Hash: 700)
```

* **Placement:** A hash function (e.g., MurmurHash3 or xxHash) places nodes and incoming data keys at specific coordinates along the ring.
* **Routing:** A key is assigned to the first node encountered moving **clockwise** ($\text{node\_hash} \ge \text{key\_hash}$), if a key's hash exceeds all nodes, it wraps around past $0$ to the first node.
* **Scale Invariance:** Adding or removing a node only impacts keys between the target node and its immediate predecessor. On average, only **$K / N$** keys move ($K$ total keys, $N$ nodes), leaving the remaining $1 - 1/N$ of keys intact.

### Virtual Nodes

With few physical nodes, non-uniform hash spacing causes **hot spots**, where one server absorbs disproportionate traffic.

**Solution:** Assign each physical node multiple virtual coordinates across the ring (e.g., `pod-1#1`, `pod-1#2`, ..., `pod-1#200`). This ensures:
1. Uniform key distribution across all members.
2. Even load offloading: when a node leaves, its traffic redistributes evenly across *all* remaining nodes rather than overwhelming its single physical neighbor.

## Application-Level Routing in Kubernetes

In a Kubernetes deployment, the underlying network layer doesn't need to know anything about your application keys.
Kubernetes can distribute external traffic randomly or via round-robin to **any** available pod.

The pods themselves execute the consistent hashing logic and route requests internally.

### Pod-Managed Ring Architecture

1. **Ingress Agnostic:** Any pod in the cluster can accept an incoming request.
2. **Ring Awareness:** Each pod maintains an in-memory view of the cluster's active members and their coordinates on the hash ring.
3. **Decentralized Forwarding:** The receiving pod hashes the requested key, checks the ring, and decides whether to serve the data locally or forward the request directly to the responsible peer pod.

### Out-of-the-Box with Microsoft Orleans in C#

Implementing peer discovery and ring management manually is non-trivial.
**Microsoft Orleans** (the virtual actor framework for .NET) provides this architecture out of the box using **consistent hashing**:

* **Consistent Hash Ring DHT:** Orleans manages its **Distributed Grain Directory** as a Distributed Hash Table (DHT) backed by a consistent hash ring, when a message targets a virtual actor (Grain), Orleans hashes the grain ID to locate the exact Silo (pod) holding that actor's state or directory entry.
* **Auto-Magic Kubernetes Discovery:** Simply enable Kubernetes hosting in C#:

```csharp
siloHostBuilder.UseKubernetesHosting();
```

* **Zero-Plumbing Cluster Management:** Orleans queries the Kubernetes API directly to detect active Silo pods, pod IPs, and health states, as pods scale up via the Horizontal Pod Autoscaler (HPA) or terminate, Orleans dynamically adjusts its internal consistent hash ring and balances grain routing with zero manual network configuration.

## Multi-Pod Workflow: Hashing, Peer Search, and Fallback

Here is the lifecycle of a request entering a Kubernetes cache cluster (`Pod-A`, `Pod-B`, `Pod-C`) backed by a database:

```text
Client Request
      │
      ▼
┌──────────────┐      Hash(Key) maps to Pod-B       ┌──────────────┐
│    Pod-A     │ ─────────────────────────────────► │    Pod-B     │
│ (Ingress Pod)│      (Internal Peer Forward)       │(Primary Owner)│
└──────────────┘                                    └──────────────┘
                                                           │
                                          Data cached?     │
                                         ┌─────────────────┤
                                   [Yes] │                 │ [No / Miss]
                                         ▼                 ▼
                                  Return Cache     Pod-B Fallback:
                                                   - Fetch from DB
                                                   - Cache locally
                                                   - Return to caller
```

### Step 1: Ingress & Ring Lookup
1. An incoming request (`GET /data?key=user:9821`) lands on `Pod-A` via standard Kubernetes load balancing.
2. `Pod-A` computes `hash("user:9821")` and performs a binary search against its local hash ring.
3. The lookup determines that **`Pod-B`** is the designated owner for this key.

### Step 2: Peer-to-Peer Routing
1. If `Pod-A` is the owner, it serves the data directly from its local cache.
2. Since `Pod-B` is the owner, `Pod-A` forwards an internal peer request directly to `Pod-B`'s pod IP.
3. `Pod-B` inspects its local cache. If the key exists, it returns the cached data immediately to `Pod-A`.

### Step 3: Cache Miss & Last-Pod Fallback
If the data is missing, expired, or was never loaded:
1. `Pod-B` searches its local store (and any secondary replicas on the ring).
2. **The Fallback Guarantee:** If none of the evaluated pods have the data, **the last pod to receive the request (`Pod-B`) takes ownership**:
   * It queries the primary database or upstream service.
   * It stores the result in its local cache partition for subsequent requests.
   * It returns the freshly fetched data back along the chain to answer the client.

This ensures that only a single, ring-designated pod ever queries the backing database for a given key, completely avoiding duplicate work and database stampedes.

## Summary Table

| Approach | Scaling Rebalance Impact | Distribution Quality | Topology Management | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **Modulo Hashing (`hash % N`)** | High ($~N/(N+1)$ keys moved) | Good (if hash is uniform) | Static IP lists | Immutable, fixed node clusters |
| **Consistent Hashing (No Vnodes)** | Low ($1/N$ keys moved) | Poor (susceptible to clustering) | Manual discovery | Simple, non-critical sharding |
| **Consistent Hashing + Vnodes** | Low ($1/N$ keys moved) | Optimal (near-equal distribution)| Pod discovery | Production caches, distributed KV stores |
| **Microsoft Orleans (C#)** | Low ($1/N$ keys moved) | Optimal (automatic DHT ring) | K8s API (`UseKubernetesHosting`) | Stateful actor systems, automatic pod clusters |

## Sources

- [Consistent Hashing (Wikipedia)](https://en.wikipedia.org/wiki/Consistent_hashing)
- [Consistent Hashing and Random Trees (Karger et al., 1997)](https://www.cs.princeton.edu/courses/archive/fall09/cos518/papers/chash.pdf)
- [Microsoft Orleans Overview](https://learn.microsoft.com/en-us/dotnet/orleans/overview)
- [Microsoft Orleans Hosting on Kubernetes](https://learn.microsoft.com/en-us/dotnet/orleans/deployment/kubernetes)
- [Containerization](https://barakadax.github.io/blog?article=Containerization)
- [Microservice Ingress patterns](https://barakadax.github.io/blog?article=Microservice%20Ingress%20patterns)
