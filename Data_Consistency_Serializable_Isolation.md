# Serializable Isolation

## 1️⃣ What It Is

**Serializable isolation** is the strongest transaction isolation level.

Definition:

> A concurrent execution of transactions is serializable if its result is equivalent to some serial (one-at-a-time) execution of those same transactions.

Important:

* It does NOT mean transactions literally run one at a time.
* It means the final result must be explainable as if they had.

This is about **correctness equivalence**, not execution order.

---

## 2️⃣ What Problem It Solves

Lower isolation levels allow anomalies:

* Lost update
* Write skew
* Dirty reads
* Non-repeatable reads
* Phantom reads

Serializable isolation guarantees:

> None of those anomalies can occur.

If an invariant holds under serial execution,
it will hold under serializable isolation.

It preserves **application-level correctness**.

---

## 3️⃣ What Does “Equivalent to Serial” Mean?

Suppose you have:

Transaction T1
Transaction T2

Serializable isolation ensures:

The outcome must match either:

T1 → T2
or
T2 → T1

If you cannot find a serial order that produces the same result,
the execution is not serializable.

This is the key test.

---

## 4️⃣ Why Snapshot Isolation Is Not Serializable

Snapshot Isolation:

* Detects write-write conflicts
* Prevents lost updates
* Provides stable snapshots

But it allows **write skew**.

Write skew produces results that are not equivalent to any serial order.

Therefore:

Snapshot Isolation ≠ Serializable.

This distinction is extremely important in interviews.

---

## 5️⃣ How Serializable Isolation Is Implemented

There are three major approaches.

---

### 5.1 Strict Two-Phase Locking (2PL)

Traditional method.

Rules:

1. Acquire locks when reading/writing.
2. Do not release locks until commit.

This ensures:

* No cycles in conflict graph.
* Serial execution equivalence.

Downside:

* Blocking.
* Deadlocks.
* Lower concurrency.

---

### 5.2 Serializable Snapshot Isolation (SSI)

Used by PostgreSQL.

Instead of blocking, SSI:

* Runs transactions under snapshot isolation.
* Tracks read-write dependencies.
* Detects dangerous dependency cycles.
* Aborts one transaction if cycle forms.

Key idea:

> Allow concurrency, but abort if serializability would be violated.

This is modern and elegant.

---

### 5.3 Timestamp Ordering

Each transaction gets a timestamp.

Rules enforce ordering based on timestamps.

If an operation violates timestamp order:

* Abort.

Used in some database engines and distributed systems.

---

## 6️⃣ What Serializable Prevents

Serializable prevents:

✔ Lost Update
✔ Write Skew
✔ Dirty Read
✔ Non-repeatable Read
✔ Phantom Read

If it can’t find a serial ordering,
it aborts one of the transactions.

---

## 7️⃣ Costs of Serializable Isolation

Serializable is expensive.

### 7.1 More Abort Rates

Conflicting transactions are aborted more often.

### 7.2 Reduced Concurrency

More coordination required.

### 7.3 Higher Latency

May need locking or dependency tracking.

### 7.4 More Memory Overhead

Must track dependencies or locks.

This is why many systems default to Snapshot Isolation instead.

---

## 8️⃣ Serializable vs Linearizability (Critical Distinction)

These are often confused.

Serializable:

* About multi-operation transactions.
* Guarantees equivalence to serial transaction order.
* Does NOT guarantee real-time ordering.

Linearizable:

* About single operations.
* Guarantees real-time order.

Serializable ≠ Linearizable.

You can have:

* Serializable but not linearizable.
* Linearizable operations but not serializable transactions.

They solve different problems.

---

## 9️⃣ Relationship to Your Other Concepts

Serializable is:

Stronger than:

* Snapshot Isolation
* Sequential Consistency (transactional context)
* Eventual Consistency

Prevents:

* Write Skew
* Lost Update

Does NOT require:

* Real-time ordering

---

## 🔟 Real-World Systems

### PostgreSQL

* Default: Snapshot Isolation (Repeatable Read)
* Serializable: SSI implementation

### MySQL InnoDB

* Uses locking to approximate serializable

### Spanner

* Provides serializable transactions
* Uses TrueTime for external consistency (linearizable + serializable)

---

## 1️⃣1️⃣ Interview Framing

If asked:

“What is Serializable Isolation?”

Answer:

> Serializable isolation guarantees that concurrent transactions produce results equivalent to some serial execution order. It prevents all classic anomalies such as lost updates and write skew, but may require locking or dependency tracking, leading to reduced concurrency or higher abort rates.

That’s clean and strong.

---

## 1️⃣2️⃣ Deep Insight

Serializable isolation protects:

* Application-level invariants.

Snapshot Isolation protects:

* Row-level conflicts.

This is the core difference.

Serializable detects dependency cycles.
Snapshot Isolation does not.

That’s the conceptual leap.

---

## 1️⃣3️⃣ When Should You Use Serializable?

Use when:

* Business invariants span multiple rows.
* Financial correctness is critical.
* You cannot tolerate subtle constraint violations.

Avoid when:

* High throughput is primary goal.
* Invariants are simple.
* Application handles consistency.

---

## Recommended Reading

* “A Critique of ANSI SQL Isolation Levels” — Berenson et al.
* PostgreSQL SSI documentation
* Designing Data-Intensive Applications — Transactions chapter
