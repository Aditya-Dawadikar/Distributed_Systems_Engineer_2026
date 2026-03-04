# Raft Consensus Algorithm

## 1. What Raft Is

**Raft is a distributed consensus algorithm used to keep replicated systems consistent.**

It ensures that **multiple servers maintain the same ordered log of operations**, even when nodes crash or messages are delayed.

Raft was designed to be **easier to understand than Paxos** while providing the same guarantees.

Typical use cases:

* replicated databases
* distributed configuration systems
* leader election
* state machine replication

Systems using Raft include:

* etcd
* HashiCorp Consul
* RethinkDB
* CockroachDB

---

# 2. Problem Raft Solves

Distributed systems often **replicate state across multiple nodes**.

Example: replicated key-value store.

```text
Node1
Node2
Node3
Node4
Node5
```

If clients send updates to different nodes, logs may diverge.

Example:

```text
Node1: SET X = 5
Node2: SET X = 10
Node3: SET X = 5
```

Without coordination:

* nodes apply operations in different orders
* system becomes inconsistent

Raft ensures:

1. **All nodes agree on log order**
2. **Committed entries are never lost**
3. **New leaders preserve previous state**

---

# 3. Core Idea

Raft organizes the cluster into a **leader-based replication model**.

```text
Leader → Followers
```

Clients interact with the **leader**, which:

1. receives client requests
2. appends them to its log
3. replicates entries to followers

Once a **majority of nodes store the entry**, it becomes **committed**.

---

# 4. Server States

Each node in Raft can be in one of three states.

```text
Follower
Candidate
Leader
```

### Follower

* passive
* responds to leader requests
* starts election if leader fails

---

### Candidate

* starts election
* requests votes from other nodes

---

### Leader

* handles client requests
* replicates logs
* sends heartbeats

---

# 5. Concept Map

```
Distributed Consensus
│
├── Paxos
│
├── Raft
│   │
│   ├── Leader Election
│   ├── Log Replication
│   └── Safety Guarantees
│
└── PBFT
```

---

# 6. Leader Election

Raft uses **randomized timeouts**.

Each follower starts a timer.

If follower does not receive heartbeat:

```
leader considered dead
```

Follower becomes **candidate**.

---

### Election Process

Candidate:

1. increments **term number**
2. votes for itself
3. sends **RequestVote RPC**

```
candidate → all nodes
RequestVote(term)
```

If candidate receives **majority votes**, it becomes leader.

Example:

Cluster size = 5

```
majority = 3
```

---

# 7. Leader Heartbeats

Leader periodically sends:

```
AppendEntries RPC
```

Even if there are no updates.

Purpose:

* prevent elections
* maintain leadership

Example interval:

```
50–150 ms
```

---

# 8. Log Replication

Leader maintains ordered log.

Example:

```
index | command
1     | SET X=5
2     | SET Y=10
3     | SET Z=20
```

When client sends request:

```
client → leader
```

Leader:

1. appends entry locally
2. sends entry to followers

```
AppendEntries(term, prevLogIndex, entries)
```

Followers replicate the entry.

---

# 9. Commit Rule

Entry becomes **committed** when:

```
majority of nodes store the entry
```

Example:

Cluster size = 5

```
leader + 2 followers = committed
```

Once committed:

```
entry applied to state machine
```

---

# 10. Log Consistency Mechanism

Raft ensures logs never diverge using:

```
prevLogIndex
prevLogTerm
```

When follower receives AppendEntries:

1. verify previous log entry matches
2. reject if mismatch
3. leader retries with earlier index

This ensures **logs converge automatically**.

---

# 11. Example Log Replication

Cluster:

```
A (leader)
B
C
D
E
```

Client sends:

```
SET balance = 100
```

Step 1

```
A appends entry
```

Step 2

```
A → B,C,D,E
AppendEntries
```

Step 3

3 nodes accept:

```
A,B,C
```

Majority reached.

Step 4

```
entry committed
```

---

# 12. Handling Leader Failure

If leader crashes:

1. followers stop receiving heartbeat
2. election timeout triggers
3. new leader elected

Example:

```
Leader A crashes
```

Node B becomes candidate.

```
RequestVote(term=4)
```

If majority votes:

```
B becomes leader
```

---

# 13. Network Partition Example

Suppose cluster splits.

```
Partition1: A,B,C
Partition2: D,E
```

Majority side:

```
A,B,C
```

Elect leader.

Minority side cannot elect leader.

When partition heals:

```
logs synchronize automatically
```

---

# 14. Safety Guarantees

Raft guarantees:

### Leader Completeness

Committed entries must exist on future leaders.

---

### Log Matching

If two logs share an entry:

```
same index → same command
```

---

### State Machine Safety

Committed entries applied in the same order on all nodes.

---

# 15. Complexity

For each log entry:

```
messages ≈ O(N)
```

Propagation latency:

```
1–2 network round trips
```

---

# 16. Advantages

### Simpler Than Paxos

Raft separates concerns:

* election
* replication
* safety

---

### Easy Implementation

State machine approach makes it intuitive.

---

### Strong Consistency

Guarantees linearizable updates.

---

### Fault Tolerant

Works as long as majority nodes survive.

---

# 17. Limitations

### Majority Requirement

Cluster must maintain:

```
> N/2 nodes alive
```

Otherwise system stops.

---

### Leader Bottleneck

All writes go through leader.

---

### Cross-region Latency

Majority replication across regions increases delay.

---

# 18. Raft vs Paxos

| Feature           | Raft      | Paxos    |
| ----------------- | --------- | -------- |
| Understandability | High      | Low      |
| Leader model      | Explicit  | Implicit |
| Implementation    | Easier    | Complex  |
| Industry adoption | Very high | Moderate |

---

# 19. Real Systems Using Raft

| System           | Purpose                   |
| ---------------- | ------------------------- |
| etcd             | Kubernetes metadata store |
| HashiCorp Consul | service registry          |
| CockroachDB      | distributed SQL           |
| RethinkDB        | replication               |

Example:

Kubernetes uses **etcd**, which internally uses Raft.

---

# 20. Interview Talking Points

Good short explanations for interviews.

**Why leader-based replication?**

To simplify log ordering and reduce coordination complexity.

---

**Why majority quorum?**

Two majorities must intersect, preventing conflicting commits.

---

**Why randomized election timeout?**

To prevent split-vote elections.

---

**Why heartbeats?**

To maintain leader authority and detect failures.

---

# 21. Related Concepts

```
Distributed Systems
│
├── Consensus
│   ├── Paxos
│   ├── Raft
│   └── PBFT
│
├── Replication
│   └── State Machine Replication
│
└── Fault Tolerance
    ├── Leader Election
    └── Quorum Systems
```
