# ALOHAnet Protocol
- [What is ALOHA?](#what-is-aloha)
- [ALOHA](#aloha)
  - [Pure ALOHA](#pure-aloha)
  - [Slotted ALOHA](#slotted-aloha)
- [Collision Handling and Backoff](#collision-handling-and-backoff)
- [Pure ALOHA vs Slotted ALOHA](#pure-aloha-vs-slotted-aloha)
- [Legacy and Influence](#legacy-and-influence)
- [Sources](#sources)

## What is ALOHA?

ALOHA (**Additive Links On-line Hawaii Area**) is one of the earliest **random-access protocols** for shared communication channels, developed by **Norman Abramson** at the **University of Hawaii** in **1970** as part of the **ALOHAnet** project.

The problem was simple: multiple computers spread across the Hawaiian islands needed to communicate over a single shared radio channel, without a central controller assigning turns. The solution was to let any station transmit whenever it had data and deal with collisions after the fact.

> [!NOTE]
> The word "ALOHA" isn't used as a message in this protocol, it's a common misconception.
> Though this expansion was rarely used. The name is primarily the Hawaiian word for hello/goodbye. It is not a message or signal within the protocol itself.

## ALOHA

**Frame** is a chunk of data wrapped with a header (addressing info) and a trailer (error detection), essentially a packet wrapped for transmission over a shared channel.

### Pure ALOHA

In Pure ALOHA, when data is available for a station that station transmits a frame, without checking if the channel is free.
If two transmissions overlap even partially, a **collision** occurs and both frames are destroyed.

The sender waits for an **acknowledgment (ACK)** from the receiver. If no ACK arrives within a timeout, the sender assumes a collision happened and retransmits after a **random backoff**.

The **vulnerable period** for a frame of duration `T` is `2T`, any other transmission starting within `T` before or during the frame causes a collision.

Maximum throughput formula:

$S = G \cdot e^{-2G}$

Where:
- $S$ = **throughput**, the fraction of successful (collision-free) transmissions.
- $G$ = **offered load**, the average number of transmission attempts per frame time (including retransmissions).
- $e$ = **Euler's number**, a mathematical constant approximately equal to **2.718**.

This peaks at $G = 0.5$, giving a maximum channel utilization of **~18.4%** ($\frac{1}{2e}$).

### Slotted ALOHA

In **1972**, **Lawrence Roberts** proposed an improvement: divide time into discrete **slots** of fixed duration `T`. Stations may only begin transmitting at the **start of a slot**.

This simple constraint reduces the vulnerable period from `2T` to just `T`, because collisions can only happen when two or more stations choose the **same slot**.

Maximum throughput formula:

$S = G \cdot e^{-G}$

This peaks at $G = 1$, giving a maximum channel utilization of **~36.8%** ($\frac{1}{e}$), exactly **double** Pure ALOHA.

> [!NOTE]
> Slotted ALOHA requires all stations to be synchronized to a common clock, adding complexity but doubling efficiency.

## Collision Handling and Backoff

When a collision is detected (no ACK received), the station doesn't retransmit immediately, it waits a **random delay** to avoid repeated collisions with the same station.

The most common strategy is **Binary Exponential Backoff**:
after the $n$-th collision, pick a random delay from $[0, 2^n - 1]$ slots.

```
delay = random(0, 2^attempt - 1) * slot_time
```

As congestion increases, the backoff window grows exponentially, spreading retransmissions over a wider time range.
This same backoff strategy was later adopted by **Ethernet (IEEE 802.3)**.

## Pure ALOHA vs Slotted ALOHA

| Feature | Pure ALOHA | Slotted ALOHA |
| :--- | :--- | :--- |
| Time division | None (continuous) | Fixed time slots |
| Transmission rule | Send anytime | Send at slot start only |
| Vulnerable period | 2T | T |
| Max throughput | ~18.4% (1/2e) | ~36.8% (1/e) |
| Synchronization | Not required | Required (shared clock) |
| Complexity | Simpler | Slightly more complex |

## Legacy and Influence

Despite its low throughput by modern standards, ALOHA's impact on networking is immense:

- **Ethernet**: Bob Metcalfe visited the University of Hawaii to study ALOHAnet during his Harvard PhD, before developing Ethernet at **Xerox PARC** in 1973. Ethernet's **CSMA/CD** (Carrier Sense Multiple Access with Collision Detection) is essentially ALOHA with "listen before talking" and "detect collisions mid-transmission" added on top.
- **Wi-Fi (IEEE 802.11)**: Uses **CSMA/CA** (Collision Avoidance), another direct descendant of ALOHA's random-access approach.
- **Cellular Networks**: Modern random access schemes in **LTE** and **5G** still use slotted ALOHA variants for initial device-to-base-station communication.
- **Satellite Communication**: Slotted ALOHA remains widely used in satellite systems where propagation delays make centralized scheduling impractical.

ALOHA's core idea, let stations transmit freely and deal with collisions after the fact, remains foundational to modern networking.

## Sources

- [ALOHAnet - Wikipedia](https://en.wikipedia.org/wiki/ALOHAnet)
- [Computerphile - The Aloha Protocol](https://www.youtube.com/watch?v=oKrUGRVwFBI)
- [IEEE Milestones Program](https://ieeemilestones.ethw.org/Milestone-Proposal:ALOHANET_(aka_ALOHA_System))
