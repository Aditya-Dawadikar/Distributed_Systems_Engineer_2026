# Snapshot Isolation (SI)

## 1️⃣ What It Is

**Snapshot Isolation** is a transaction isolation level where:

> Each transaction reads from a consistent snapshot of the database taken at the start of the transaction, and write-write conflicts are detected at commit time.

Two core properties:

1. **Consistent snapshot for reads**

   * A transaction sees a stable view of the database as of its start time.
   * It does not see partial updates from concurrent transactions.

2. **First-committer-wins for write conflicts**

   * If two concurrent transactions update the same row,
     one must abort.

That’s SI in one sentence.

---

## 2️⃣ What Problem It Is Trying to Solve

Traditional locking (pessimistic locking):

* Readers block writers
* Writers block readers
* Deadlocks happen
* Performance degrades under contention

SI (usually implemented via MVCC) solves:

* Readers don’t block writers
* Writers don’t block readers
* Transactions see a stable view
* Many anomalies are prevented

It gives:

* High concurrency
* Stronger guarantees than Read Committed
* Better performance than full serializable (in many systems)

---

## 3️⃣ How Snapshot Isolation Works

Let’s break it into mechanics.

---

### 3.1 Snapshot at Transaction Start

When transaction T starts:

* It gets a timestamp (or transaction ID).
* That defines its **snapshot**.

All reads by T see:

> The database state as of that snapshot time.

Even if other transactions commit later,
T does not see those changes.

This gives:

* Repeatable reads
* Stable results within transaction

---

### 3.2 Writes Create New Versions

Using MVCC:

* Updates don’t overwrite rows.
* They create new versions.

Other transactions see versions based on their snapshot.

---

### 3.3 Write-Write Conflict Detection

If:

* T1 and T2 both modify the same row
* Both started before either committed

Then:

Only one is allowed to commit.

This prevents lost update.

Rule often used:

> First committer wins.

---

## 4️⃣ What SI Prevents

Snapshot Isolation prevents:

### 4.1 Dirty Reads

You never see uncommitted data.

### 4.2 Non-repeatable Reads

Since snapshot is fixed, repeated reads return same value.

### 4.3 Lost Update (usually)

Because concurrent writes to same row conflict.

This is important:
SI prevents lost update through write-write conflict detection.

---

## 5️⃣ What SI Allows (Critical)

Snapshot Isolation allows:

> Write Skew

This is the key weakness.

If two transactions:

* Read overlapping data
* Write different rows
* Don’t write the same row

Then:

They won’t conflict.

And invariants can break.

---

## 6️⃣ Classic Write Skew Example (Must Know)

Doctors on call example.

Rule:

> At least one doctor must be on call at all times.

Initial state:

```id="ex1"
Doctor A: on_call = true
Doctor B: on_call = true
```

Transaction T1:

* Reads A and B
* Sees both on call
* Sets A = false

Transaction T2:

* Reads A and B
* Sees both on call
* Sets B = false

Under Snapshot Isolation:

* T1 and T2 don’t update same row.
* No write-write conflict.
* Both commit.

Final state:

```id="ex2"
Doctor A: false
Doctor B: false
```

Invariant broken.

System is not serializable.

This is write skew.

This is why SI ≠ Serializable.

---

## 7️⃣ Relationship to MVCC

Important:

> MVCC is the mechanism.
> Snapshot Isolation is the policy built on top of MVCC.

MVCC gives:

* Version storage
* Snapshot visibility
* Non-blocking reads

SI adds:

* Write conflict detection rule

---

## 8️⃣ Strength Comparison

Stronger than:

* Read Committed
* Repeatable Read (in many systems)

Weaker than:

* Serializable isolation
* Strong consistency (linearizability)

So:

Serializable > Snapshot Isolation > Read Committed

---

## 9️⃣ Advantages

### 9.1 High concurrency

No read locks needed.

### 9.2 Stable snapshots

Great for long analytical queries.

### 9.3 Prevents many anomalies

Dirty reads, lost updates.

### 9.4 Practical sweet spot

Many production systems default to SI or close to it.

---

## 🔟 Limitations

### 10.1 Write Skew

Does not enforce cross-row invariants.

### 10.2 Predicate anomalies

Range constraints can be violated.

### 10.3 Long-running transactions cause bloat

Because old versions must be retained.

### 10.4 Not safe for certain constraints

Banking-like invariants need serializable.

---

## 1️⃣1️⃣ Real-World Implementations

### PostgreSQL

* Default “Repeatable Read” behaves like Snapshot Isolation.
* True serializable implemented via SSI (Serializable Snapshot Isolation).

### MySQL InnoDB

* Repeatable Read behaves similar to SI.
* Gap locks may reduce some anomalies.

### Oracle

* Uses MVCC.
* Provides consistent read semantics similar to SI.

---

## 1️⃣2️⃣ Interview Traps

### Trap 1:

“SIs are serializable.”

Wrong.

### Trap 2:

“SI prevents all anomalies.”

Wrong.
It prevents lost update but allows write skew.

### Trap 3:

Confusing write skew with lost update.

Lost update:
Same row overwritten.

Write skew:
Different rows updated but invariant violated.

---

## 1️⃣3️⃣ When Should You Use SI?

Good for:

* High-throughput OLTP
* Systems where cross-row invariants are rare
* Workloads dominated by reads
* Microservice DBs with simpler constraints

Not good for:

* Complex constraint-heavy systems
* Financial systems with multi-row invariants

---

## 1️⃣4️⃣ Interview-Ready Explanation

If asked:

“What is Snapshot Isolation?”

Strong answer:

> Snapshot Isolation is a transaction isolation level where each transaction reads from a consistent snapshot taken at its start time, and concurrent write-write conflicts are detected at commit using a first-committer-wins rule. It prevents dirty reads and lost updates, but it allows anomalies like write skew because it does not guarantee full serializability.

That’s a senior-level response.

---

## 1️⃣5️⃣ Deep Insight (Senior-Level)

Snapshot Isolation enforces:

* Version-based read consistency
* Row-level write conflict detection

It does NOT enforce:

* Predicate-level conflict detection
* Global serialization order

Serializable Snapshot Isolation (SSI) fixes this by:

* Tracking read-write dependency graph
* Aborting transactions that create dangerous structures

That’s advanced database theory.

---

## Recommended Reading

* “A Critique of ANSI SQL Isolation Levels” — Berenson et al.
* PostgreSQL SSI documentation
* Designing Data-Intensive Applications — Transactions chapter
