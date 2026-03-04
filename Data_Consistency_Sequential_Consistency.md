# Sequential Consistency

# 1. What It Is

**Sequential consistency** is a memory and distributed system consistency model that guarantees:

> The result of execution is the same as if all operations from all processes were executed in some single sequential order, and the operations of each individual process appear in that order in the sequence.

This definition was introduced by **Leslie Lamport (1979)**.

Two important requirements define sequential consistency:

1. **All operations appear in a single global order**
2. **Each process’s operations appear in that order according to program order**

However, sequential consistency **does not require the global order to match real-world time**.

This is the key difference from **linearizability**.

---

# 2. What Problem It Solves

In distributed or multi-core systems, operations occur concurrently across multiple nodes or threads.

Without ordering guarantees, different processes may observe inconsistent sequences of updates.

Example problem:

Two threads update shared variables.

Thread A:

```id="sc1"
x = 1
y = 1
```

Thread B:

```id="sc2"
read y
read x
```

Without ordering guarantees, thread B might observe:

```id="sc3"
y = 1
x = 0
```

This violates logical expectations because the write to `x` happened before the write to `y`.

Sequential consistency ensures that **all processes observe operations in the same global order**.

---

# 3. Core Idea

Sequential consistency gives the illusion that:

```id="sc4"
all operations from all processes
execute one-by-one
on a single global timeline
```

But internally the system may still execute operations concurrently.

The system just needs to produce results that **could be explained by some sequential ordering**.

---

# 4. Example

Consider two processes.

Process P1:

```id="sc5"
write(x = 1)
```

Process P2:

```id="sc6"
write(y = 1)
```

Two readers observe values.

Reader R1 might see:

```id="sc7"
x = 1
y = 1
```

Reader R2 might see:

```id="sc8"
y = 1
x = 1
```

Both observations are valid because there is **no causal relationship between the writes**.

Sequential consistency allows multiple valid global orders.

---

# 5. Example Showing Allowed Reordering

Assume:

Process A:

```id="sc9"
write(x = 1)
write(y = 1)
```

Process B:

```id="sc10"
read(y)
read(x)
```

Sequential consistency requires:

```id="sc11"
write(x=1) happens before write(y=1)
```

because program order must be preserved.

But **other processes may see unrelated operations in different orders**.

---

# 6. What Sequential Consistency Guarantees

Sequential consistency ensures:

✔ All processes see **the same order of operations**
✔ Each process’s **program order is preserved**

But it does **not guarantee real-time ordering**.

---

# 7. What Sequential Consistency Does NOT Guarantee

Sequential consistency allows:

❌ Real-time violations
❌ Stale reads (in some scenarios)
❌ Delayed visibility of writes

Example:

```id="sc12"
T1 completes write(x=1)
T2 starts read(x)
```

Sequential consistency may still return:

```id="sc13"
x = 0
```

because the system may reorder operations in the global sequence.

This would be illegal under **linearizability**.

---

# 8. Sequential Consistency vs Linearizability

This is a very common interview comparison.

| Property                  | Sequential Consistency | Linearizability |
| ------------------------- | ---------------------- | --------------- |
| Global ordering           | Yes                    | Yes             |
| Program order preserved   | Yes                    | Yes             |
| Real-time order preserved | No                     | Yes             |
| Strength                  | Weaker                 | Stronger        |

Linearizability adds the rule:

```id="sc14"
If operation A completes before B begins,
A must appear before B.
```

Sequential consistency does not enforce this.

---

# 9. Relationship to Causal Consistency

Consistency hierarchy:

```id="sc15"
Linearizability
↓
Sequential Consistency
↓
Causal Consistency
↓
Eventual Consistency
```

Sequential consistency requires **a single global order**, while causal consistency only enforces **causal ordering**.

---

# 10. Implementation Approaches

Sequential consistency can be implemented using several techniques.

---

## Centralized Ordering

One node acts as an **operation sequencer**.

All operations pass through it and are ordered.

Advantages:

* simple

Disadvantages:

* bottleneck
* single point of failure

---

## Total Order Broadcast

Systems may use protocols that enforce total ordering of messages.

Examples:

* Paxos
* Raft
* Zab (ZooKeeper)

These ensure every node applies operations in the same order.

---

## Shared Memory Models

Sequential consistency is also used in **multiprocessor memory systems**.

Processors must ensure memory operations appear globally ordered.

This often requires:

* memory barriers
* cache coherence protocols

---

# 11. Advantages

### Intuitive Programming Model

Developers can reason about execution as if operations occurred sequentially.

### Stronger than Many Distributed Models

Provides predictable ordering guarantees.

### Simpler Reasoning

Compared to weaker models like eventual consistency.

---

# 12. Limitations

### Performance Overhead

Maintaining a global order requires coordination.

### Scalability Issues

Global ordering becomes expensive at large scale.

### No Real-Time Guarantees

Even if an operation completes earlier, sequential consistency may reorder it.

This can cause confusing behaviors.

---

# 13. Systems That Approximate Sequential Consistency

Examples include:

### ZooKeeper

Uses total order broadcast to ensure operations are applied in the same order.

### Kafka

Within a partition, operations are sequentially ordered.

### Some shared-memory multiprocessors

Use sequentially consistent memory models.

---

# 14. Applications

Sequential consistency is useful when:

* operations must appear globally ordered
* deterministic execution is important
* coordination systems require strong ordering

Examples:

* distributed logs
* configuration systems
* replicated state machines

---

# 15. Interview Explanation (Concise)

A good interview answer:

> Sequential consistency guarantees that all operations appear in some single global order that respects the program order of each process. However, the order does not necessarily reflect real-time ordering. It provides a consistent global view of operations but is weaker than linearizability.

---

# 16. Key Insight

Sequential consistency ensures **logical global ordering**, but not **real-world ordering**.

That is why it sits between:

```id="sc16"
Linearizability (strong)
↓
Sequential Consistency
↓
Causal Consistency
```

---

# Recommended Reading

**Lamport — How to Make a Multiprocessor Computer That Correctly Executes Multiprocess Programs**

[https://lamport.azurewebsites.net/pubs/lamport-how-to.pdf](https://lamport.azurewebsites.net/pubs/lamport-how-to.pdf)

**Designing Data-Intensive Applications — Consistency Models**
