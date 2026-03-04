# Read Committed (RC)

## 1️⃣ What It Is

**Read Committed** is a transaction isolation level where:

> A transaction can only read data that has been committed.

That’s it.

No dirty reads.
But many other anomalies are still possible.

It is weaker than:

* Snapshot Isolation
* Serializable

It is stronger than:

* Read Uncommitted

---

## 2️⃣ What Problem It Solves

The primary issue RC solves is:

### Dirty Reads

A dirty read happens when:

* Transaction T1 writes something
* T2 reads it
* T1 later rolls back

T2 saw data that never truly existed.

Read Committed prevents this.

It ensures:

> You only read committed data.

---

## 3️⃣ How It Works

Implementation differs by database, but the principle is:

At the moment of each read:

* Only committed versions are visible.
* Uncommitted changes are invisible.

Important:

Read Committed typically gives a **fresh snapshot per statement**, not per transaction.

That means:

* Each query inside a transaction may see different data.

This is critical.

---

## 4️⃣ What It Prevents

✔ Dirty reads

That’s the only strong guarantee.

---

## 5️⃣ What It Allows

Read Committed allows:

❌ Non-repeatable reads
❌ Phantom reads
❌ Lost updates (in some implementations)
❌ Write skew
❌ Inconsistent reads across statements

Let’s go through the important ones.

---

## 6️⃣ Non-Repeatable Read

Example:

T1:

```id="rc1"
SELECT balance FROM accounts WHERE id=1;
```

T2:

```id="rc2"
UPDATE accounts SET balance=50 WHERE id=1;
COMMIT;
```

T1 again:

```id="rc3"
SELECT balance FROM accounts WHERE id=1;
```

T1 sees different values.

Because RC does not provide snapshot consistency.

Each statement sees latest committed state.

---

## 7️⃣ Phantom Reads

Example:

T1:

```id="rc4"
SELECT * FROM orders WHERE amount > 100;
```

T2 inserts new qualifying row and commits.

T1 runs same query again.
Now sees extra row.

That’s phantom read.

RC allows it.

---

## 8️⃣ Lost Update Under RC

Depending on DB behavior:

Two transactions:

T1:

```id="rc5"
SELECT balance FROM accounts WHERE id=1;
UPDATE accounts SET balance = balance + 10 WHERE id=1;
```

T2 does same.

If no write-write conflict detection:
Lost update occurs.

Some databases detect write conflicts automatically.
Others don’t.

So:
Read Committed does not guarantee protection against lost update.

---

## 9️⃣ Relationship to MVCC

Many modern databases implement Read Committed using MVCC.

How?

* Each statement gets a fresh snapshot.
* That snapshot includes only committed rows at query start time.

Important difference:

Snapshot Isolation → one snapshot per transaction
Read Committed → one snapshot per statement

That is the core conceptual difference.

---

## 🔟 Strength Hierarchy (Database Isolation)

From weakest to strongest:

```id="rc6"
Read Uncommitted
Read Committed
Snapshot Isolation / Repeatable Read
Serializable
```

Read Committed is mid-low strength.

---

## 1️⃣1️⃣ Performance Characteristics

Why is RC popular?

Because it:

* Avoids dirty reads
* Minimizes locking
* Allows high concurrency
* Rarely blocks readers

It’s a good performance compromise.

That’s why many databases default to RC.

---

## 1️⃣2️⃣ Why It Is Dangerous for Invariants

Because:

* Reads may observe changing data
* Decisions made early in transaction may become invalid
* Cross-row invariants are not protected

RC is not safe for complex business logic.

---

## 1️⃣3️⃣ Read Committed vs Snapshot Isolation

| Feature              | Read Committed | Snapshot Isolation |
| -------------------- | -------------- | ------------------ |
| Dirty reads          | ❌              | ❌                  |
| Non-repeatable reads | ✔              | ❌                  |
| Lost update          | Possible       | Prevented          |
| Write skew           | ✔              | ✔                  |
| Snapshot per txn     | ❌              | ✔                  |

This table is very important.

---

## 1️⃣4️⃣ Interview Framing

If asked:

“What is Read Committed?”

Strong answer:

> Read Committed ensures that a transaction only sees committed data. However, it does not provide a consistent snapshot across the entire transaction, so non-repeatable reads, phantom reads, and anomalies like write skew are still possible.

That shows maturity.

---

## 1️⃣5️⃣ Deep Insight

Read Committed protects against:

Physical inconsistency (dirty data).

It does NOT protect against:

Logical inconsistency (invariants).

That distinction separates junior from senior understanding.

---

## 1️⃣6️⃣ Real-World Defaults

* PostgreSQL default → Read Committed
* Oracle default → Read Committed (but with strong MVCC)
* SQL Server default → Read Committed
* MySQL InnoDB default → Repeatable Read (stronger)

Many production systems run on RC and rely on:

* Application-level checks
* Explicit locks
* Constraints

---

## 1️⃣7️⃣ When Should You Use It?

Use RC when:

* Throughput matters
* Business logic is simple
* Invariants are local to single rows
* You can tolerate anomalies

Avoid RC when:

* Cross-row invariants exist
* Financial correctness is required
* Multi-step transactional decisions exist
