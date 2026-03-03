# Sequential Consistency

## 1. What Is Sequential Consistency?

Sequential consistency (SC) is a **consistency model for shared memory or distributed systems** that defines how operations (reads and writes) appear to execute when multiple processes access shared state concurrently.

It was formally defined by Leslie Lamport in 1979 in the paper:

> *“How to Make a Multiprocessor Computer That Correctly Executes Multiprocess Programs”*
> (Highly recommended reading.)

The formal definition:

> The result of any execution is the same as if the operations of all processes were executed in some sequential order, and the operations of each individual process appear in this sequence in the order specified by its program.

This definition has two critical components:

### (1) Global Sequential Order Exists

There must exist **a single total order** of all operations across all processes.

### (2) Program Order Is Preserved

Within that global order, the operations of each process must appear in the exact order written in its code.

What SC does **not** require:

* It does not require real-time ordering.
* It does not require that operations complete in wall-clock order.

That difference is extremely important.

---

## 2. What Problem Is It Trying to Solve?

When multiple processes execute concurrently and share data:

* Writes from different processes can interleave unpredictably.
* Reads may observe intermediate states.
* Hardware and networks may reorder operations.
* Caches may delay propagation.

Without a defined consistency model, reasoning about correctness becomes impossible.

Sequential consistency solves:

* How can programmers reason about concurrent execution?
* Can we think about concurrent programs as if they ran one instruction at a time?

SC gives a powerful mental model:

> Even though execution is physically concurrent, we can pretend everything happened in some single sequential order.

That abstraction dramatically simplifies reasoning about concurrent correctness.

---

## 3. Intuition Through Example

Let’s take a canonical example.

Initial state:

```
x = 0
y = 0
```

Process A:

```
x = 1
r1 = y
```

Process B:

```
y = 1
r2 = x
```

Question:
Is it possible that:

```
r1 = 0
r2 = 0
```

Under sequential consistency, the answer is: **No.**

Why?

To allow both reads to see 0, we would need:

* A reads y before B writes y.
* B reads x before A writes x.

But that implies:

* A’s write to x must come after B’s read of x.
* B’s write to y must come after A’s read of y.

This creates a cyclic dependency. There is no single total ordering that preserves both program orders and yields both reads as 0.

Therefore:

Sequential consistency forbids that outcome.

Now contrast that with weaker memory models (like modern CPU memory models), where that outcome *can* happen due to reordering.

This example demonstrates why SC is considered strong.

---

## 4. How Sequential Consistency Works Conceptually

Sequential consistency does not prescribe a specific implementation. It defines an observable property.

To achieve SC in practice, systems usually need:

* A mechanism to totally order writes.
* A way to prevent reordering of operations within a process.
* A way to ensure reads see a consistent prefix of that global order.

In distributed systems, that typically implies:

* A centralized coordinator
* Or consensus protocols
* Or atomic broadcast mechanisms

In shared-memory multiprocessors, it requires:

* Strong memory barriers
* Strict cache coherence enforcement

These mechanisms introduce performance costs.

---

## 5. Relationship to Other Consistency Models

Sequential consistency sits between weaker and stronger models.

### Stronger: Linearizability

Linearizability requires:

* A total order
* Program order preservation
* Real-time order preservation

Sequential consistency does **not** require real-time ordering.

That means:

An operation that finishes later in wall-clock time can appear earlier in the global order under SC.

This makes SC strictly weaker than linearizability.

---

### Weaker: Causal Consistency

Causal consistency only preserves causally related operations. Independent operations may be observed in different orders by different processes.

Sequential consistency forces a **single global order for everyone.**

Thus:

Linearizability > Sequential Consistency > Causal Consistency

---

## 6. Advantages of Sequential Consistency

### 1. Intuitive Reasoning

You can reason about concurrent programs as if they executed one instruction at a time.

That dramatically simplifies correctness proofs.

### 2. Strong Ordering Guarantees

It eliminates many weird interleavings possible in weaker models.

### 3. Deterministic Reasoning Model

Even if execution is nondeterministic, outcomes must correspond to some sequential ordering.

---

## 7. Limitations and Costs

### 1. Performance Overhead

To maintain a global total order, coordination is required.

Coordination implies:

* Increased latency
* Reduced scalability
* Lower throughput

### 2. Poor Fit for Geo-Distributed Systems

Maintaining a single global ordering across continents introduces massive delays.

Large-scale systems (e.g., Dynamo-style systems) avoid SC for this reason.

### 3. Does Not Guarantee Real-Time Order

Developers often assume that if A finishes before B starts, then A must appear before B globally.

SC does not guarantee that.

This is the biggest misconception.

---

## 8. Where Sequential Consistency Is Used

Sequential consistency is common in:

* Academic models of distributed systems
* Some shared memory models
* Certain coordination services (when using atomic broadcast)
* Systems implementing strict locking protocols

Most modern distributed databases prefer:

* Linearizability (for critical metadata)
* Eventual consistency (for scale)
* Causal consistency (for social/media systems)

---

## 9. Implementation Insight (Interview-Level Depth)

To implement SC in distributed systems, you typically need:

### Option 1: Central Sequencer

All operations go through a central node that assigns sequence numbers.

Pros:

* Simple reasoning

Cons:

* Single point of failure
* Bottleneck

---

### Option 2: Consensus-Based Ordering

Use consensus protocols (Raft, Paxos) to agree on operation order.

Pros:

* Fault-tolerant

Cons:

* Expensive
* Latency increases with replication

---

### Option 3: Atomic Broadcast

Total-order broadcast ensures all replicas apply operations in the same order.

This is logically equivalent to consensus.

Important theoretical result:

> Consensus and total-order broadcast are equivalent problems.

---

## 10. Common Interview Mistakes

### Mistake 1:

Confusing SC with linearizability.

If you mention real-time ordering when defining SC, that is incorrect.

### Mistake 2:

Thinking SC prevents stale reads.

It prevents inconsistent ordering, not necessarily stale reads in real-time sense.

### Mistake 3:

Believing SC is practical at massive scale.

In large geo-distributed systems, SC is usually too expensive.

---

## 11. How To Explain It Cleanly in an Interview

If asked:

“What is sequential consistency?”

A strong answer:

> Sequential consistency guarantees that all operations across processes appear as if they were executed in some global sequential order, while preserving the program order of each individual process. However, unlike linearizability, it does not require that this order respect real-time ordering. Achieving it in distributed systems typically requires total-order broadcast or consensus, which introduces coordination costs.

---

## 12. Recommended Resources

1. Leslie Lamport (1979)
   “How to Make a Multiprocessor Computer That Correctly Executes Multiprocess Programs”

2. Designing Data-Intensive Applications (Martin Kleppmann)
   Chapter on consistency models

3. MIT 6.824 Distributed Systems lectures
   (Excellent for understanding ordering and consensus)

4. Jepsen consistency model documentation
   Useful for practical system behavior
