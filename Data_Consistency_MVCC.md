# MVCC (Multi-Version Concurrency Control)

## 1) What it is

**MVCC** is a database concurrency-control mechanism where the system keeps **multiple versions** of the same logical row (or record). Instead of writers blocking readers (or readers blocking writers), MVCC lets readers see a **consistent snapshot** while writers create **new versions**.

The mental model:

* A write does **not** overwrite a row in-place.
* It creates a new version.
* Each transaction reads from a snapshot based on time/version rules.

This is why MVCC is often associated with “non-blocking reads”.

---

## 2) What problem it’s trying to solve

Classic locking approaches (pessimistic locking) have a huge pain:

* Readers take shared locks, writers take exclusive locks.
* Under load, readers and writers **block each other**.
* Long reads (analytics, scans) can stall write throughput.
* Long writes can stall read throughput.

MVCC solves:

### a) Reader–Writer contention

Readers should not block writers, and writers should not block readers.

### b) Consistent reads without holding locks

A transaction should see a stable view of the DB while it runs (depending on isolation level).

### c) High concurrency at scale

Many OLTP systems need lots of concurrent reads/writes with minimal lock wait.

---

## 3) How it works (core mechanism)

Different DBs implement MVCC differently, but the core pieces are consistent.

### 3.1 Versions and timestamps (or transaction IDs)

Each row version is tagged with metadata like:

* `created_by_txn` (xmin)
* `deleted_by_txn` (xmax) or a tombstone
* commit timestamp / commit sequence number

A transaction has a **snapshot** that defines which transactions are visible.

So: a reader decides visibility like:

> “Show me the newest version whose creator committed before my snapshot, and that isn’t deleted before my snapshot.”

That’s the MVCC visibility rule in plain English.

---

### 3.2 Reads: “time travel to your snapshot”

When transaction `T` reads row `R`, the DB finds the row version that is visible under `T`’s snapshot.

This is why MVCC gives you:

* **Repeatable reads** (in many DBs)
* stable results inside a transaction

even while other transactions are committing new versions.

---

### 3.3 Writes: create new versions (copy-on-write)

When transaction `T` updates a row:

* it creates a **new version** of that row
* marks the old version as deleted/expired for future snapshots

So updates are effectively:

* append new version
* old version remains for readers that need it

This is also why MVCC needs garbage collection (more on that).

---

## 4) What MVCC guarantees (and what it doesn’t)

### MVCC guarantees (as a mechanism)

MVCC itself guarantees:

* **Readers don’t block writers** (in general)
* Transactions can read a **consistent snapshot**
* You can implement isolation levels like **Snapshot Isolation** or **Repeatable Read** efficiently

### MVCC does NOT automatically guarantee serializability

This is the big trap.

MVCC often implements **Snapshot Isolation (SI)** by default, which:

* prevents many anomalies (like dirty reads)
* **still allows write skew** unless you add extra checks (SSI, predicate locks, etc.)

So the correct framing is:

> MVCC is a mechanism. Isolation level is the policy built on top of it.

---

## 5) MVCC and isolation levels (crucial interview link)

### Read Committed + MVCC

Each statement may see a newer snapshot.

* Two reads in the same transaction can see different committed data.

### Repeatable Read / Snapshot Isolation + MVCC

Transaction sees one consistent snapshot for the whole transaction.

This is where:

* “repeatable reads” come from
* phantom handling depends on DB & implementation

### Serializable on MVCC

Possible, but you need additional machinery:

* PostgreSQL: Serializable Snapshot Isolation (SSI)
* Others: predicate locks / range locks / conflict graph validation

---

## 6) Advantages

### 6.1 High concurrency, low blocking

Reads are usually lock-free (or very lightweight) and don’t stall writes.

### 6.2 Great for mixed workloads

OLTP with many reads + writes benefits hugely.

### 6.3 Predictable read behavior inside a transaction

Snapshot-based reads simplify reasoning and reduce weirdness.

---

## 7) Limitations / costs

### 7.1 Storage overhead

Multiple versions accumulate. Updates are more expensive than in-place overwrite.

### 7.2 Garbage collection / vacuum complexity

Old versions must be cleaned up.

* PostgreSQL: VACUUM reclaims dead tuples
* MySQL InnoDB: undo logs + purge threads
* SQL Server: version store in tempdb

If cleanup lags:

* storage bloat grows
* performance degrades

### 7.3 Write amplification

Updates create new versions and index changes, causing extra IO.

### 7.4 Long-running transactions are toxic

A long-running reader holds back cleanup because old versions must stay around to satisfy its snapshot.

This causes:

* bloat
* slower vacuum
* increased IO
* potential wraparound issues (Postgres transaction IDs)

---

## 8) Common anomalies with MVCC systems

### Lost Update

Depends on isolation and write conflict rules.

* Many MVCC systems detect write-write conflicts (two txns update same row) and abort one.
* But “lost update” can still happen in app-level patterns if you do unsafe read-modify-write outside proper constraints.

### Write Skew (big one)

Classic: Snapshot Isolation allows two txns to read overlapping constraints and write disjoint rows, violating an invariant.

This is the textbook “MVCC ≠ serializable” case.

If interviewer asks “what anomaly does SI allow?”:
**Write skew** is the answer.

---

## 9) Real-world implementations (what to mention)

### PostgreSQL

* MVCC with row versions (tuples)
* Needs VACUUM
* Serializable via SSI (not via strict locking)

### MySQL InnoDB

* MVCC via undo logs
* Consistent reads use undo to reconstruct older versions
* Purge threads remove old undo

### Oracle

* Classic MVCC with undo segments
* Very strong historical consistency story

### SQL Server (optional modes)

* Snapshot isolation / read committed snapshot use version store

---

## 10) Interview traps (don’t fall for these)

### Trap 1: “MVCC means no locks”

Wrong.
There are still locks, especially for:

* writes
* constraint checks
* indexes
* metadata
  MVCC reduces read locking, not eliminate locking.

### Trap 2: “MVCC guarantees serializable”

Wrong. MVCC often defaults to SI/Repeatable Read, which allows write skew.

### Trap 3: “MVCC makes writes cheap”

No. Writes are often more expensive due to versioning + indexes + GC.

---

## 11) How to explain MVCC in an interview (tight but deep)

Say:

> MVCC is a concurrency control technique where the database keeps multiple versions of rows. Readers use a snapshot to see a consistent version without blocking writers, while writers create new versions instead of overwriting in place. This improves concurrency significantly, but it introduces storage/cleanup overhead and, depending on the isolation level (often snapshot isolation), it may still allow anomalies like write skew unless the DB adds serializable mechanisms.

That’s a strong answer.
