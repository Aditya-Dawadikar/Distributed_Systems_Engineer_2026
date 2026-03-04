# Linearizability

## 1. What It Is

**Linearizability** is the strongest consistency model for single-object operations in distributed systems.

A system is **linearizable** if:

> Every operation appears to take effect instantaneously at some point between its invocation and its response.

This means the system behaves **as if all operations occurred in a single global timeline**, and the order respects **real-world time**.

Two requirements define linearizability:

1. **Operations appear atomic** (instantaneous).
2. **Real-time ordering is preserved**.

If operation **A completes before operation B begins**, then **A must appear before B in the system’s history**.

This property makes the system behave like a **single-threaded machine**, even if internally it is distributed.

---

# 2. What Problem It Solves

Distributed systems suffer from:

* Replication delays
* Network partitions
* Concurrent writes
* Stale reads

Without strong guarantees, clients may observe:

* Out-of-order updates
* Old values after newer ones
* Conflicting states

Linearizability solves this by ensuring:

* **All clients observe updates in the same order**
* **Operations respect real-time causality**

This gives the illusion of a **single, globally consistent copy of the data**.

---

# 3. Core Idea (Intuition)

Imagine a distributed key-value store.

Two clients perform:

```
T1: write(x = 10)
T2: read(x)
```

If T1 finishes before T2 starts, then T2 **must return 10**.

It cannot return an older value like `5`.

This guarantee ensures that **completed writes are immediately visible to subsequent reads**.

The system behaves as though every operation happens at a precise instant called the **linearization point**.

---

# 4. Linearization Point

Every operation must appear to occur at one specific instant between:

```
request_time ---- linearization_point ---- response_time
```

Even if internally the system:

* replicates data
* sends messages
* runs consensus

Externally it must look like the operation occurred **atomically at that moment**.

This is how distributed systems emulate a **single logical timeline**.

---

# 5. Example

Assume a replicated database storing `x`.

Initial value:

```
x = 0
```

Two operations occur.

Client A:

```
write(x = 1)
```

Client B:

```
read(x)
```

If the write completes before the read starts, the read **must return 1**.

Any system returning `0` would violate linearizability.

---

# 6. Relationship to Sequential Consistency

These two are often confused.

### Sequential Consistency

Guarantees:

* All operations appear in a single order
* The order respects each client’s program order

But it **does not respect real-time ordering**.

### Linearizability

Adds an extra rule:

* Operations must respect **real-time order**

Example:

```
T1 completes
T2 starts
```

Sequential consistency may still reorder them.

Linearizability cannot.

Therefore:

```
Linearizability ⊂ Sequential Consistency
```

Linearizability is stronger.

---

# 7. Relationship to Serializable Isolation

These two apply to different layers.

| Property        | Domain            |
| --------------- | ----------------- |
| Linearizability | Single operations |
| Serializable    | Transactions      |

Serializable guarantees that **transactions appear in serial order**, but that order may not match real-world time.

Linearizability enforces **real-time order**.

A system can be:

* Serializable but not linearizable
* Linearizable but not transactional

---

# 8. How Linearizability Is Implemented

Linearizability requires **coordination between replicas**.

Common approaches:

### 1. Consensus Protocols

Systems like:

* Paxos
* Raft

ensure all replicas agree on operation order.

Examples:

* etcd
* ZooKeeper
* Spanner metadata

---

### 2. Leader-Based Replication

A leader orders writes.

Process:

1. Client sends write to leader
2. Leader replicates to followers
3. Majority confirms
4. Operation commits

Reads must go to:

* leader
* quorum

to remain linearizable.

---

### 3. Quorum Systems

Using read/write quorums:

```
R + W > N
```

Where:

* R = read quorum
* W = write quorum
* N = replicas

This ensures overlap between reads and writes.

---

# 9. Advantages

### Strongest Consistency Guarantee

Clients always observe the most recent state.

### Intuitive Behavior

Programmers can reason as if the system were single-threaded.

### Safe for Critical Systems

Used for:

* financial ledgers
* coordination services
* distributed locks

---

# 10. Limitations

### High Latency

Requires coordination across nodes.

### Reduced Availability

During network partitions, the system may reject operations.

This comes from the **CAP theorem**:

```
Consistency + Availability cannot both hold during partitions
```

Linearizable systems prioritize **consistency**.

---

### Poor Scalability for Global Systems

Cross-region consensus increases latency dramatically.

For example:

```
US → Europe → Asia consensus
```

This can add 100–300 ms latency.

---

# 11. Real Systems That Use Linearizability

### etcd

Used by Kubernetes for cluster state.

### ZooKeeper

Provides strongly consistent coordination primitives.

### Google Spanner

Uses TrueTime to provide **external consistency**, a form of linearizability.

### CockroachDB

Provides linearizable reads with consensus.

---

# 12. Systems That Do NOT Use Linearizability

Many high-scale systems relax consistency.

Examples:

* Cassandra
* DynamoDB (eventual by default)
* Riak

They trade strict ordering for availability and performance.

---

# 13. Relationship to Other Concepts in Your Map

Linearizability connects with your previous topics.

```
Linearizability
    stronger than → Sequential Consistency
    stronger than → Eventual Consistency
    guarantees → Read-After-Write
    prevents → Lost Update
    prevents → Write Skew (when using transactions)
```

It sits near the **top of the consistency hierarchy**.

---

# 14. When to Use Linearizability

Use when:

* operations must reflect real-time order
* correctness is critical
* coordination primitives are needed

Examples:

* distributed locks
* leader election
* configuration services

Avoid when:

* ultra-low latency required
* global geo-distribution needed
* availability prioritized over strict ordering

---

# 15. Interview Explanation (Concise)

A strong interview answer:

> Linearizability is a consistency model that guarantees operations appear to occur atomically in a single global timeline that respects real-time ordering. If one operation completes before another begins, the system must reflect that order. It provides the illusion of a single copy of the data but typically requires coordination protocols such as Paxos or Raft.

---

# Recommended Resources

**Designing Data-Intensive Applications**
Chapter: Consistency and Consensus

**Herlihy & Wing (1990)**
Linearizability: A Correctness Condition for Concurrent Objects

[https://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf](https://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)
