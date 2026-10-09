# Diffie-Hellman Key Exchange
- [The Classroom Analogy](#the-classroom-analogy)
- [Why Simple Math Fails: The Need for Asymmetry](#why-simple-math-fails-the-need-for-asymmetry)
- [How Diffie-Hellman Works](#how-diffie-hellman-works)
  - [Public Parameters](#public-parameters)
  - [The Mathematical Handshake](#the-mathematical-handshake)
- [Real-World Implementation](#real-world-implementation)
  - [One-Time Handshake and Symmetric Encryption](#one-time-handshake-and-symmetric-encryption)
  - [Authentication and Man-in-the-Middle (MITM)](#authentication-and-man-in-the-middle-mitm)
  - [Ephemeral Diffie-Hellman (PFS)](#ephemeral-diffie-hellman-pfs)
- [Sources](#sources)

## The Classroom Analogy

Imagine **Alice** and **Bob**, two close friends sitting on opposite sides of a crowded classroom.
They want to pass handwritten notes back and forth, but there is a problem: annoying kids sitting in the middle can intercept any note, read it, or even forge a fake note pretending to be the original sender.

To prevent the kids in the middle from tampering with their messages, Alice and Bob meet before class and agree on a secret number: **`5`**.

```text
[Alice] (Private: 3)                                    [Bob] (Private: 5)
   │                                                           │
   │─── Sends Note: 3 * 5 = 15 ───────────────────────────────►│
   │                                                           │
   │◄─────────────────────────────── Sends Note: 5 * 5 = 25 ───│
   │                                                           │
   ▼                                                           ▼
 Computes Bob's Number: 25 / 5 = 5           Computes Alice's Number: 15 / 5 = 3
   │                                                           │
   └────────────── Shared Secret Calculation ──────────────────┘
                    5 * (Alice's #) * (Bob's #)
                          5 * 3 * 5 = 75
```

1. **First Exchange:**
   - Alice picks her own secret number, say **`3`**, multiplies it by their shared secret (`3 * 5`), and sends note **`15`** across the classroom.
   - Bob picks his own secret number, say **`5`**, multiplies it by their shared secret (`5 * 5`), and sends note **`25`** across the classroom.
2. **Deriving the Private Numbers:**
   - Because both know the pre-agreed secret `5`, Alice divides Bob's note by 5 (`25 / 5 = 5`) to reveal Bob's number.
   - Bob divides Alice's note by 5 (`15 / 5 = 3`) to reveal Alice's number.
3. **Establishing Authentication & Signing:**
   - Both friends now compute a shared secret:
     $$\text{Secret} = 5 \times (\text{Alice's Number}) \times (\text{Bob's Number}) = 5 \times 3 \times 5 = 75$$
   - From then on, both friends can append and sign new messages using combinations of their private number and the secret multiplier (`X * 5`).
   Because only Alice and Bob know both the base secret and their friend's private number, any note carrying the correct signature is guaranteed to be authentic.

---

## Why Simple Math Fails: The Need for Asymmetry

While the classroom analogy illustrates the intuition, standard multiplication cannot protect secrets on an untrusted network:

1. **Reversibility:** Simple multiplication is trivial to undo using division, if an eavesdropper intercepts `15` and learns or guesses the multiplier `5`, finding `3` requires a single division.
2. **Ambiguity and Collisions:** Linear formulas do not guarantee uniqueness, for example, $6 \times 4 = 24$, but $2 \times 12 = 24$ and $1 \times 24 = 24$, simple factoring is ambiguous and predictable.

Real-world cryptography requires an **asymmetric, one-way trapdoor function**: a mathematical operation that is easy to compute in one direction, but practically impossible to reverse without secret knowledge.
In classical Diffie-Hellman, this is the **Discrete Logarithm Problem** over modular arithmetic (or on elliptic curves in ECDH).

---

## How Diffie-Hellman Works

The Diffie-Hellman (DH) key exchange allows two parties to establish a shared secret over an insecure channel **without ever transmitting the secret itself**.

### Public Parameters

Alice and Bob publicly agree on two non-secret numbers:
* A large prime number $p$ (the modulus).
* A primitive root modulo $p$, denoted as $g$ (the generator).

Anyone listening on the network (eavesdroppers or kids in the middle) can see both $p$ and $g$.

### The Mathematical Handshake

```text
Alice (Private: a)                                      Bob (Private: b)
  │                                                           │
  │─── Sends Public Key: A = g^a mod p ──────────────────────►│
  │                                                           │
  │◄────────────────────── Sends Public Key: B = g^b mod p ───│
  │                                                           │
  ▼                                                           ▼
Computes Shared Secret:                     Computes Shared Secret:
S = B^a mod p                               S = A^b mod p
  = (g^b)^a mod p                             = (g^a)^b mod p
  = g^(ab) mod p                              = g^(ab) mod p
```

1. **Generate Private Keys:**
   - Alice chooses a random secret integer $a$.
   - Bob chooses a random secret integer $b$.
2. **Compute Public Keys:**
   - Alice computes $A = g^a \pmod p$ and sends $A$ to Bob over the network.
   - Bob computes $B = g^b \pmod p$ and sends $B$ to Alice over the network.
3. **Compute the Shared Secret:**
   - Alice takes Bob's public key $B$ and raises it to her private key $a$:
     $$S = B^a \pmod p = (g^b)^a \pmod p = g^{ab} \pmod p$$
   - Bob takes Alice's public key $A$ and raises it to his private key $b$:
     $$S = A^b \pmod p = (g^a)^b \pmod p = g^{ab} \pmod p$$

Both arrive at the identical shared secret $S = g^{ab} \pmod p$.
An attacker who intercepts $g$, $p$, $A$, and $B$ cannot compute $S$ without solving the discrete logarithm to find $a$ or $b$, which is computationally infeasible for sufficiently large primes (e.g., 2048+ bits).

---

## Real-World Implementation

### One-Time Handshake and Symmetric Encryption

In protocols like TLS (HTTPS), SSH, and IPsec:
* **DH is only executed once:** Asymmetric math (modular exponentiation or elliptic curve point multiplication) is computationally intensive, Diffie-Hellman handshake runs only during initial connection negotiation.
* **Deriving Session Keys:** Once the handshake produces the shared secret $S$, both sides pass $S$ into a Key Derivation Function (KDF) like HKDF to produce short-lived symmetric encryption keys (e.g., AES-256-GCM or ChaCha20-Poly1305).
* **Bulk Data Transmission:** All subsequent messages are encrypted using fast symmetric ciphers.

### Authentication and Man-in-the-Middle (MITM)

A pure Diffie-Hellman exchange is **unauthenticated**.
If an active attacker sits between Alice and Bob, the attacker can negotiate one shared secret with Alice and a separate shared secret with Bob, decrypting and modifying traffic in transit.

To solve this in real production protocols:
* The server proves its identity using a **digital signature** (e.g., RSA or ECDSA signed with a trusted CA certificate).
* The server signs its public DH parameters ($g$, $p$, $A$).
* The client verifies the signature against the server's public certificate, confirming the handshake is genuinely occurring with the intended server.

### Ephemeral Diffie-Hellman (PFS)

Modern standards (such as **TLS 1.3**) mandate **Ephemeral Diffie-Hellman** (DHE or ECDHE):
* A fresh, random private key pair is generated for **each individual session** and destroyed immediately after the session key is derived.
* Provides **Perfect Forward Secrecy (PFS)**: Even if the server's long-term private signing key is stolen years later, previously recorded traffic cannot be decrypted because the ephemeral session keys no longer exist.

---

## Sources

- [Computerphile — Secret Key Exchange (Diffie-Hellman)](https://www.youtube.com/watch?v=Yjrfm_oRO0w)
- [Computerphile — Public Key Cryptography: Diffie-Hellman Key Exchange](https://www.youtube.com/watch?v=NmM9HA2MQGI)
- [RFC 2631 — Diffie-Hellman Key Agreement Method](https://datatracker.ietf.org/doc/html/rfc2631)
- [RFC 7748 — Elliptic Curves for Security (ECDH)](https://datatracker.ietf.org/doc/html/rfc7748)
