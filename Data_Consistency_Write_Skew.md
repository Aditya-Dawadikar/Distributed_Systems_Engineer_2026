# Write Skew

## 1️⃣ What It Is

**Write skew** is a concurrency anomaly where:

> Two concurrent transactions read overlapping data, make decisions based on that data, and write to different rows — resulting in a violation of a constraint or invariant.

Important:

* They **do not update the same row**
* Therefore, no write-write conflict is detected
* But the **global invariant breaks**

Write skew is the canonical anomaly allowed by **Snapshot Isolation**.

---

## 2️⃣ What Problem It Exposes

Write skew exposes:

* The difference between row-level conflict detection and logical constraint enforcement
* The fact that Snapshot Isolation ≠ Serializable
* The difficulty of enforcing cross-row invariants

It demonstrates that:

> Conflict detection on physical rows is insufficient to preserve logical constraints.

---

## 3️⃣ Classic Doctor Example (Must Know)

Rule:

> At least one doctor must be on call at all times.

Initial state:

```id="ws1"
Doctor A: on_call = true
Doctor B: on_call = true
```

Two concurrent transactions:

### Transaction T1

* Reads A and B
* Sees both are on call
* Sets A = false

### Transaction T2

* Reads A and B
* Sees both are on call
* Sets B = false

Under Snapshot Isolation:

* T1 and T2 read from same snapshot
* They do not update same row
* No write-write conflict
* Both commit

Final state:

```id="ws2"
Doctor A: false
Doctor B: false
```

Invariant broken.

That is write skew.

---

## 4️⃣ Why Snapshot Isolation Allows It

Snapshot Isolation:

* Detects write-write conflicts (same row updates)
* Does NOT detect read-write conflicts across different rows

In write skew:

* Both transactions read both rows
* But each writes a different row
* So conflict detection does not trigger

The system thinks everything is fine.

This is a **predicate-level anomaly**, not a row-level conflict.

---

## 5️⃣ Formal View (Dependency Graph Insight)

In serialization theory, write skew creates a:

> Dangerous structure (cycle in serialization graph)

T1 reads B, T2 writes B
T2 reads A, T1 writes A

This creates a dependency cycle:

T1 → T2 → T1

Which violates serializability.

Snapshot Isolation does not detect this cycle.

Serializable isolation would.

---

## 6️⃣ Difference Between Lost Update and Write Skew

This is extremely important.

| Property              | Lost Update | Write Skew      |
| --------------------- | ----------- | --------------- |
| Same row updated?     | Yes         | No              |
| Write-write conflict? | Yes         | No              |
| Prevented by SI?      | Yes         | No              |
| Violates invariant?   | Possibly    | Yes (typically) |

Lost update:
Two transactions update same row → one overwrites.

Write skew:
Transactions update different rows → invariant violated.

---

## 7️⃣ Real-World Scenarios

Write skew is not theoretical. It happens in:

### 1️⃣ Booking systems

Rule:
“At least one admin must remain active.”

Two admins deactivate themselves concurrently.

### 2️⃣ Hospital scheduling

Exactly the doctor example.

### 3️⃣ Bank reserve requirements

Two transactions check “total reserve > threshold” and withdraw separately.

### 4️⃣ Microservices

Two services read shared config state and update disjoint rows.

---

## 8️⃣ How to Prevent Write Skew

### Option 1: Serializable Isolation

Serializable isolation ensures:

> The outcome must match some serial order.

This detects cycles and aborts one transaction.

In PostgreSQL:

* SSI (Serializable Snapshot Isolation) tracks dangerous structures.

---

### Option 2: Explicit Locking

Use:

```id="ws3"
SELECT ... FOR UPDATE
```

Lock rows being read.

But:

* Must lock all rows involved in invariant.
* Hard with predicates.

---

### Option 3: Materialized Constraint Row

Instead of:

* Two independent rows

Use:

* A single “summary” row.

Example:
Maintain a counter row representing “doctors on call”.

Both transactions update that row → now write-write conflict occurs.

This converts write skew into lost update → which SI prevents.

Very practical trick.

---

### Option 4: Database Constraints

Use:

* CHECK constraints
* Triggers
* Unique constraints
* Partial indexes

But not all invariants are expressible easily.

---

## 9️⃣ Why It’s Hard in Distributed Systems

In distributed systems:

* Data is partitioned.
* Transactions may span shards.
* Conflict detection becomes expensive.
* Coordination required.

This is why many distributed databases:

* Default to Snapshot Isolation
* Accept write skew risk
* Or require explicit constraint handling

---

## 🔟 Strength Hierarchy

Serializable
⬇
Snapshot Isolation
⬇
Read Committed
⬇
Eventual Consistency

Write skew exists in:

* Snapshot Isolation
* Weaker levels

But NOT in Serializable.

---

## 1️⃣1️⃣ Interview Angle

If interviewer asks:

“What anomaly does Snapshot Isolation allow?”

Answer immediately:

> Write skew.

Then explain:

> Because SI detects write-write conflicts but not read-write dependency cycles across different rows.

That’s a very strong answer.

---

## 1️⃣2️⃣ Deep Insight (Senior-Level Understanding)

Write skew is fundamentally about:

> Predicate-based invariants vs row-based conflict detection.

Snapshot Isolation protects rows.

Serializable protects invariants.

This difference is the entire reason serializable isolation is more expensive.

---

## 1️⃣3️⃣ Visual Mental Model

Lost update:
Two cars collide head-on → detected.

Write skew:
Two cars take different roads but collapse a bridge → not detected unless you check global structure.

---

## 1️⃣4️⃣ Recommended Reading

* “A Critique of ANSI SQL Isolation Levels” — Berenson et al.
* PostgreSQL SSI docs
* Designing Data-Intensive Applications — Transactions chapter
