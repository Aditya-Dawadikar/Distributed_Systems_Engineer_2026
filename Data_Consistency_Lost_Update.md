# Lost Update

# 1️⃣ What Is a Lost Update?

A **lost update** occurs when two concurrent operations read the same value, both modify it, and one write overwrites the other — causing one update to be silently discarded.

In simple words:

> Two people update the same data at the same time.
> One update "wins", and the other disappears.

No error.
No crash.
Just incorrect final state.

That’s what makes it dangerous.

---

# 2️⃣ What Problem Does It Represent?

Lost update is not a model — it is a **consistency anomaly**.

It reveals:

* Weak isolation
* Missing concurrency control
* Race conditions
* Lack of atomic operations

It is a symptom of improper coordination.

---

# 3️⃣ Classic Example

Initial value:

```id="x1"
balance = 100
```

Two users concurrently add 10.

Process A:

```id="x2"
read balance → 100
balance = balance + 10
write 110
```

Process B:

```id="x3"
read balance → 100
balance = balance + 10
write 110
```

Final value:
110

Expected value:
120

One increment was lost.

That’s a lost update.

---

# 4️⃣ Why Does This Happen?

Because both processes:

1. Read the same old value.
2. Compute new values independently.
3. Overwrite each other’s writes.

This is a classic **read-modify-write race condition**.

The system failed to enforce:

* Mutual exclusion
* Atomicity
* Proper isolation

---

# 5️⃣ Where Does Lost Update Occur?

It can occur in:

### 1. Databases (Low Isolation Levels)

If using:

* Read Uncommitted
* Read Committed (without proper locking)
* Weak MVCC configurations

---

### 2. Distributed Systems

When:

* Multiple replicas accept writes
* No consensus
* No version checks
* No compare-and-set semantics

---

### 3. Application Layer

When:

* No transactions
* No locking
* No atomic operations
* Improper use of cache

---

# 6️⃣ Relation to Isolation Levels (Important)

In SQL isolation levels:

| Isolation Level  | Lost Update Possible? |
| ---------------- | --------------------- |
| Read Uncommitted | Yes                   |
| Read Committed   | Yes                   |
| Repeatable Read  | Depends on DB         |
| Serializable     | No                    |

Serializable isolation prevents lost updates.

Why?

Because serializable guarantees behavior equivalent to some serial execution.

And serial execution cannot lose updates.

---

# 7️⃣ How Systems Prevent Lost Updates

There are several strategies.

---

## 1️⃣ Pessimistic Locking

Before updating:

* Acquire lock
* Modify
* Release lock

Only one writer at a time.

Pros:

* Safe

Cons:

* Reduced concurrency
* Deadlock risk

---

## 2️⃣ Optimistic Concurrency Control (OCC)

Instead of locking:

* Read value + version
* Compute new value
* Write only if version unchanged

If version changed → retry

Example:

```id="x4"
UPDATE table
SET balance = 110
WHERE id = 1 AND version = 3;
```

If zero rows updated → conflict detected.

This is widely used in distributed systems.

---

## 3️⃣ Atomic Operations

Use atomic increment operations:

* SQL: `UPDATE balance = balance + 10`
* Redis: `INCR`
* CAS (compare-and-swap)

This prevents lost update by avoiding read-modify-write race.

---

## 4️⃣ Transactions with Serializable Isolation

Database guarantees serial behavior.

But performance cost increases.

---

# 8️⃣ Distributed Systems Context

Lost update is especially common in:

* Leaderless systems (Dynamo style)
* Eventual consistency systems
* Systems without quorum enforcement

Example:

Two clients write to different replicas simultaneously.

Without conflict resolution:
One write may override another.

---

# 9️⃣ Why Eventual Consistency Is Vulnerable

Eventual consistency allows:

* Concurrent writes
* No immediate coordination
* Temporary divergence

Without conflict resolution strategies (CRDT, vector clocks, etc.), lost updates can happen.

---

# 1️⃣0️⃣ Difference Between Lost Update and Write Conflict

Lost update:

* One update disappears silently.

Write conflict:

* Conflict detected and surfaced.

Lost update is worse because:
It often goes unnoticed.

---

# 1️⃣1️⃣ Real-World Scenarios

### Banking System

If increments are not atomic → catastrophic.

### Inventory Management

Two buyers purchase last item.
Inventory decremented twice.
Final stock negative or incorrect.

### Social Media Counters

Likes increment incorrectly under load.

---

# 1️⃣2️⃣ Advantages of Preventing Lost Update

* Data correctness
* Strong consistency guarantees
* Predictable business logic

---

# 1️⃣3️⃣ Cost of Preventing Lost Update

* Lock contention
* Retry overhead
* Reduced throughput
* Increased latency

You trade performance for correctness.

---

# 1️⃣4️⃣ Interview Angle

If interviewer asks:

> “What is a lost update?”

Answer:

> A lost update occurs when two concurrent transactions read the same value and update it independently, and one write overwrites the other, causing one modification to be lost. It typically happens when systems lack proper concurrency control.

Then add:

> It can be prevented using locking, optimistic concurrency control, atomic operations, or serializable isolation.

That signals depth.

---

# 1️⃣5️⃣ Deep Insight (Senior Level)

Lost update is fundamentally a violation of:

> Serializability

If your system guarantees serializable isolation,
lost update cannot occur.

Thus:

Lost update is a test for isolation strength.

---

# 1️⃣6️⃣ Recommended Reading

* Designing Data-Intensive Applications — Chapter on Transactions
* ANSI SQL Isolation Levels
* Dynamo paper (for conflict resolution)
* PostgreSQL MVCC documentation
