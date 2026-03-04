# Mental Map of Consistency Concepts

## 1. Big Picture: The Three Families

```
DATA CONSISTENCY

├── Consistency Models (Distributed Systems)
│   ├── Strong Consistency
│   │   └── Linearizability
│   ├── Sequential Consistency
│   ├── Causal Consistency
│   └── Eventual Consistency
│
├── Transaction Isolation (Databases)
│   ├── Read Committed
│   ├── Snapshot Isolation
│   └── Serializable Isolation
│
└── Concurrency Problems & Mechanisms
    ├── Problems
    │   ├── Lost Update
    │   └── Write Skew
    │
    ├── Client Guarantees
    │   ├── Read Your Writes
    │   └── Monotonic Reads
    │
    └── Techniques
        ├── MVCC
        └── Compare And Swap
```

---

# 2. Concept Relationships

## Problems

```
Lost Update
Write Skew
```

These happen due to **concurrent writes without proper isolation**.

They are **caused by weak isolation**.

---

## Solutions

### MVCC

```
MVCC → enables Snapshot Isolation
```

MVCC keeps **multiple versions of data** so readers don’t block writers.

---

### CAS (Compare And Swap)

```
CAS → prevents Lost Updates
```

Used in:

* lock-free systems
* distributed systems
* atomic operations

---

# 3. Distributed Consistency Hierarchy

Strongest → Weakest

```
Linearizability
      ↓
Sequential Consistency
      ↓
Causal Consistency
      ↓
Eventual Consistency
```

### Linearizability

* Real-time order preserved
* strongest guarantee

### Sequential Consistency

* same global order
* but not real-time

### Causal Consistency

* only causally related operations ordered

### Eventual Consistency

* replicas eventually converge

---

# 4. Client Guarantees (Session Guarantees)

These exist to **make eventual consistency usable**.

```
Client Guarantees

├── Read Your Writes
│   ensures user sees their own updates
│
└── Monotonic Reads
    ensures reads never go backwards
```

Example:

```
Without Monotonic Reads
User reads value = 5
Next read → value = 3
```

---

# 5. Transaction Isolation Hierarchy

Weak → Strong

```
Read Committed
      ↓
Snapshot Isolation
      ↓
Serializable Isolation
```

### Read Committed

* cannot read uncommitted data
* still allows anomalies

---

### Snapshot Isolation

* transaction sees a snapshot
* prevents many conflicts

BUT

```
Write Skew still possible
```

---

### Serializable

* behaves as if transactions ran sequentially
* strongest isolation

---

# 6. How Everything Connects

Here is the **true mental map**.

```
                           DATA CONSISTENCY

       ┌───────────────────────────┬────────────────────────────┐
       │                           │                            │
Distributed Consistency      Transaction Isolation        Concurrency Control
       │                           │                            │
       │                           │                            │
Strong Consistency          Serializable Isolation             CAS
       │                           │                            │
Linearizability             Snapshot Isolation                MVCC
       │                           │                            │
Sequential Consistency       Read Committed              Prevents Lost Update
       │                           │
Causal Consistency          Allows Write Skew
       │
Eventual Consistency
       │
Client Guarantees
   ├─ Read Your Writes
   └─ Monotonic Reads
```

---

# 7. Quick Memory Trick

Think of it like **3 questions**:

### Q1: Replicas → "Do all machines see the same data?"

→ Consistency models

```
Linearizable
Sequential
Causal
Eventual
```

---

### Q2: Transactions → "Do concurrent transactions interfere?"

→ Isolation levels

```
Read Committed
Snapshot
Serializable
```

---

### Q3: Concurrency → "How do we implement correctness?"

→ Mechanisms

```
MVCC
CAS
```

# Examples

### Strong Consistency Systems
```
Spanner
CockroachDB
FoundationDB
TiDB
etcd
```

Usually use:
```
Raft / Paxos
```

### Eventual Consistency Systems

```
DynamoDB
Cassandra
Riak
CouchDB
```

Usually use:

```
Gossip
Vector clocks
CRDTs
```
