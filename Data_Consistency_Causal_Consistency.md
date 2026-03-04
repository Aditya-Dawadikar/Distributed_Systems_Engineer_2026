# Causal Consistency

# 1. What It Is

**Causal consistency** is a consistency model that guarantees that **causally related operations are observed in the same order by all processes**.

Definition:

> If one operation causally influences another, every process must observe them in that same order.

However, operations that are **not causally related** may be seen in different orders on different nodes.

This makes causal consistency **weaker than sequential consistency**, but **stronger than eventual consistency**.

---

# 2. What Problem It Solves

In distributed systems, updates propagate asynchronously.

Without ordering guarantees, users may observe updates in logically incorrect sequences.

Example problem:

1. User A posts a message:
   `"Hello"`

2. User B replies:
   `"Hi!"`

If replicas propagate updates independently, some users might see:

```id="cc1"
"Hi!"
"Hello"
```

This violates logical causality.

Causal consistency ensures:

```id="cc2"
"Hello"
"Hi!"
```

because the reply depends on the original post.

---

# 3. What Is a Causal Relationship?

Two operations are **causally related** if one could have influenced the other.

This concept is derived from **Lamport’s happens-before relation**.

Operation A causally precedes operation B if:

### Rule 1 — Program Order

Operations in the same process are causal.

Example:

```id="cc3"
write(x=1)
write(x=2)
```

All processes must observe `1` before `2`.

---

### Rule 2 — Read-Then-Write Dependency

If a process reads a value and then writes based on it.

Example:

```id="cc4"
T1: write(x=1)
T2: read(x=1)
T2: write(y=1)
```

`write(y=1)` causally depends on `write(x=1)`.

---

### Rule 3 — Transitivity

If:

```id="cc5"
A → B
B → C
```

Then:

```id="cc6"
A → C
```

Causality propagates through chains.

---

# 4. Concurrent Operations

Operations that are **not causally related** are called **concurrent**.

Example:

Two users update unrelated values simultaneously.

```id="cc7"
T1: write(x=1)
T2: write(y=1)
```

There is no causal relationship.

Different nodes may observe:

```id="cc8"
x → y
```

or

```id="cc9"
y → x
```

Both orders are valid under causal consistency.

---

# 5. Guarantees Provided by Causal Consistency

Causal consistency ensures:

✔ Read Your Writes
✔ Monotonic Reads
✔ Monotonic Writes
✔ Writes Follow Reads

These are known as **session guarantees**.

Thus:

```id="cc10"
Causal Consistency
    includes → Read Your Writes
    includes → Monotonic Reads
```

This is an important relationship in your concept map.

---

# 6. Relationship to Other Consistency Models

Consistency strength hierarchy:

```id="cc11"
Linearizability
↓
Sequential Consistency
↓
Causal Consistency
↓
Eventual Consistency
```

Interpretation:

* Linearizable systems enforce real-time order.
* Sequential consistency enforces a single global order.
* Causal consistency enforces only causal ordering.
* Eventual consistency enforces no ordering.

---

# 7. Example

Assume three operations.

```id="cc12"
A: write(x=1)
B: read(x=1)
B: write(y=1)
C: read(y=1)
```

Since `y=1` depends on `x=1`, every node must observe:

```id="cc13"
write(x=1)
before
write(y=1)
```

But unrelated operations may appear in any order.

---

# 8. How It Is Implemented

Causal consistency requires tracking dependencies.

Two common approaches are used.

---

## Vector Clocks

Each update carries a **vector clock** that records causal history.

Example:

```id="cc14"
Node A: [1,0,0]
Node B: [1,1,0]
```

Nodes compare vectors to determine ordering.

Pros:

* precise causal tracking

Cons:

* vector size grows with number of nodes

---

## Version Vectors / Dependency Tracking

Systems may track:

* dependency sets
* version histories
* causal metadata

These ensure updates are applied only when dependencies are satisfied.

---

# 9. Advantages

### Maintains Logical Order

Users observe events in a sensible order.

### More Scalable Than Strong Consistency

Does not require global coordination.

### Suitable for Collaborative Systems

Applications with user interactions benefit from causal guarantees.

---

# 10. Limitations

### Metadata Overhead

Tracking causal history requires storing dependency information.

### Implementation Complexity

Dependency tracking and propagation add system complexity.

### Does Not Guarantee Global Order

Different nodes may see concurrent events in different orders.

---

# 11. Real Systems Using Causal Consistency

Several modern distributed systems implement causal consistency.

Examples include:

### COPS

A geo-distributed key-value store designed specifically for causal consistency.

### AntidoteDB

A distributed CRDT-based database supporting causal consistency.

### MongoDB

Provides **causal consistency** within sessions.

### Dynamo-style systems

Often provide session guarantees approximating causal consistency.

---

# 12. Example Applications

Causal consistency works well in systems where **user actions depend on previous actions**.

Examples:

* social media feeds
* messaging systems
* collaborative editing
* comment threads

It prevents confusing scenarios where replies appear before original messages.

---

# 13. Interview Explanation (Concise)

A strong explanation:

> Causal consistency guarantees that operations that are causally related are observed in the same order by all nodes. However, operations that are independent may appear in different orders on different nodes. It preserves logical dependencies without requiring the global ordering of sequential consistency.

---

# 14. Key Insight

Causal consistency protects **logical ordering** rather than **real-time ordering**.

This makes it a powerful middle ground:

* stronger than eventual consistency
* far cheaper than strong consistency

---

# Recommended Reading

**Designing Data-Intensive Applications**
Chapter: Consistency Models

**Lamport — Time, Clocks, and the Ordering of Events**

[https://lamport.azurewebsites.net/pubs/time-clocks.pdf](https://lamport.azurewebsites.net/pubs/time-clocks.pdf)
