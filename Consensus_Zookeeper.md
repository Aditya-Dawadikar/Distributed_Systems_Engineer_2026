# Zab — ZooKeeper Atomic Broadcast

## 1. What Zab Is

**Zab (ZooKeeper Atomic Broadcast)** is the consensus and replication protocol used by Apache ZooKeeper.

Its job is to **replicate state updates across ZooKeeper servers while guaranteeing a consistent order of operations**.

Zab ensures:

* **all nodes apply updates in the same order**
* **no committed updates are lost**
* **leader handles all writes**

Zab is optimized for systems where **a single leader handles updates while followers replicate them**.

It is conceptually similar to **Raft / Multi-Paxos**, but tailored specifically for ZooKeeper’s workload.

---

# 2. Problem Zab Solves

ZooKeeper maintains **distributed coordination data**.

Examples:

```text
/config/serviceA
/locks/job1
/election/node1
```

Multiple clients may issue operations:

```text
create
delete
update
```

If replicas apply operations in different orders, the system state diverges.

Example problem:

```text
Server1:
create /lock

Server2:
delete /lock
create /lock
```

State mismatch occurs.

Zab ensures **total order broadcast**.

---

# 3. Atomic Broadcast

Atomic broadcast means:

> All servers deliver messages in the same order.

Example:

```text
operation1
operation2
operation3
```

Every node must apply them in exactly this sequence.

Properties:

### Total Order

All nodes see updates in the same order.

### Reliability

Messages are eventually delivered.

### Consistency

No server applies conflicting histories.

---

# 4. Zab Architecture

ZooKeeper cluster consists of:

```text
Leader
Followers
Observers (optional)
```

Leader:

* receives client writes
* orders updates
* broadcasts updates

Followers:

* replicate updates
* acknowledge commits

Observers:

* receive updates
* do not vote in consensus

---

# 5. Concept Map

```id="7sdgq1"
Distributed Consensus
│
├── Paxos
│
├── Raft
│
├── PBFT
│
└── Zab
     │
     ├── Leader Based Replication
     ├── Atomic Broadcast
     └── Crash Fault Tolerance
```

---

# 6. Zab Phases

Zab has **two main phases**.

```text
1 Recovery Phase
2 Broadcast Phase
```

Recovery ensures all nodes start with a consistent state.

Broadcast handles normal operation.

---

# 7. Phase 1 — Recovery Phase

When leader starts or cluster restarts, Zab performs recovery.

Goal:

```text
synchronize cluster state
```

Steps:

### Step 1 — Leader Election

ZooKeeper runs a leader election algorithm.

Node with highest:

```text
zxid
```

wins.

---

### What is zxid?

`zxid` = **ZooKeeper Transaction ID**

Structure:

```text
(epoch, counter)
```

Example:

```text
(3,45)
```

Meaning:

```text
epoch = leader term
counter = operation number
```

This ensures **global ordering of transactions**.

---

### Step 2 — State Synchronization

Leader checks follower logs.

If follower is behind:

```text
leader sends missing transactions
```

If follower has extra entries:

```text
leader truncates them
```

Goal:

```text
all replicas share same history
```

---

# 8. Phase 2 — Broadcast Phase

Normal operation begins.

Leader handles client requests.

---

### Step 1 — Client Request

Client sends write request:

```text
client → leader
setData(/node,value)
```

---

### Step 2 — Proposal

Leader creates transaction:

```text
zxid = (epoch,counter)
```

Leader broadcasts:

```text
PROPOSAL(zxid, operation)
```

---

### Step 3 — Acknowledgment

Followers receive proposal and store it.

They send:

```text
ACK(zxid)
```

to leader.

---

### Step 4 — Commit

Once leader receives **majority acknowledgments**:

```text
leader broadcasts COMMIT(zxid)
```

Followers apply transaction.

---

# 9. Example Zab Execution

Cluster:

```text
A (leader)
B
C
D
E
```

Client operation:

```text
create /service
```

---

### Step 1

Leader assigns:

```text
zxid = (5,12)
```

---

### Step 2

Leader sends:

```text
PROPOSAL(5,12)
```

to followers.

---

### Step 3

Followers reply:

```text
ACK(5,12)
```

Example:

```text
B,C,D ACK
```

Majority reached.

---

### Step 4

Leader broadcasts:

```text
COMMIT(5,12)
```

Transaction applied.

---

# 10. Leader Failure Handling

If leader crashes:

1. followers detect failure
2. leader election starts
3. new leader chosen

New leader performs **recovery phase** to ensure logs are consistent.

Example:

```text
Old leader zxid = (5,40)
New leader zxid = (5,38)
```

Leader synchronizes cluster before accepting writes.

---

# 11. Zab Guarantees

### Total Order

All nodes apply updates in same order.

---

### Durability

Committed transactions are never lost.

---

### Linearizable Writes

All writes appear to occur atomically.

---

# 12. Zab Message Complexity

Per transaction:

```text
Leader → followers proposal
Followers → leader ACK
Leader → followers commit
```

Total messages:

```text
O(N)
```

---

# 13. Zab vs Raft

| Feature         | Zab              | Raft                |
| --------------- | ---------------- | ------------------- |
| Primary use     | ZooKeeper        | General replication |
| Leader election | integrated       | integrated          |
| Log ordering    | zxid             | log index           |
| Design          | atomic broadcast | consensus           |

Raft is easier to understand.

Zab is **specialized for ZooKeeper workloads**.

---

# 14. Zab vs Paxos

| Feature    | Zab                   | Paxos             |
| ---------- | --------------------- | ----------------- |
| Model      | Leader-based          | Leader optional   |
| Complexity | Lower                 | Higher            |
| Ordering   | total order broadcast | consensus rounds  |
| Use case   | ZooKeeper             | general consensus |

---

# 15. Real System Using Zab

Primary system:

Apache ZooKeeper

ZooKeeper is used by many distributed systems:

* Apache Kafka (older versions)
* Apache HBase
* Apache Hadoop

Example uses:

```text
leader election
distributed locks
configuration management
```

---

# 16. Advantages

### Strong Ordering Guarantees

All replicas apply operations in identical order.

---

### Efficient for Leader-Based Workloads

Optimized for systems where:

```text
single leader handles writes
```

---

### Simple Replication Model

Compared to Paxos.

---

# 17. Limitations

### Leader Bottleneck

All writes go through leader.

---

### Limited Scalability

Large clusters increase coordination cost.

---

### Crash Fault Only

Zab handles **crash failures**, not Byzantine faults.

---

# 18. Concept Relationship

```id="0wgrz9"
Distributed Replication
│
├── Paxos
│
├── Raft
│
├── Zab
│   └── Atomic Broadcast
│
└── PBFT
    └── Byzantine Consensus
```

---

# 19. Interview Talking Points

**What is Zab?**

Consensus protocol used by ZooKeeper for total order broadcast.

---

**What is zxid?**

Transaction identifier used to order updates globally.

---

**Why leader-based replication?**

Simplifies ordering and reduces coordination overhead.

---

**Why recovery phase?**

Ensures all nodes start with consistent history before processing requests.

---

# 20. Related Topics

Important concepts around Zab:

```text
Atomic Broadcast
State Machine Replication
Leader Election
Distributed Logs
Quorum Consensus
```
