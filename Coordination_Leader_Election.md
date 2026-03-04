# Leader Election

## 1. What Leader Election Is

**Leader election** is the process by which a distributed system chooses **one node to act as the coordinator (leader)** among multiple nodes.

The leader is responsible for:

* coordinating operations
* ordering requests
* managing shared resources
* maintaining system state

Example cluster:

```text
Node1
Node2
Node3
Node4
Node5
```

After election:

```text
Leader → Node3
Followers → others
```

Leader election simplifies distributed coordination.

---

# 2. Problem Leader Election Solves

Many distributed systems require a **single authority** to avoid conflicts.

Example: replicated database.

If multiple nodes accept writes simultaneously:

```text
Node1 → write X=5
Node2 → write X=10
```

This causes inconsistent state.

Solution:

```text
only leader handles writes
```

Followers replicate state.

This is the model used by systems like:

* etcd
* Apache ZooKeeper
* Apache Kafka

---

# 3. Leader Election Requirements

A correct leader election algorithm should satisfy:

### Safety

At most **one leader exists at a time**.

```text
leaders ≤ 1
```

---

### Liveness

A leader must eventually be elected.

---

### Fault Tolerance

Leader election should work even if nodes fail.

---

### Stability

Frequent leader changes should be avoided.

---

# 4. Concept Map

```text
Distributed Coordination
│
├── Leader Election
│
├── Distributed Locks
│
├── Consensus Protocols
│     ├── Raft
│     ├── Paxos
│     └── Zab
│
└── Failure Detection
```

Leader election is often implemented using **consensus algorithms**.

---

# 5. Basic Leader Election Algorithm

Simplest approach: **highest node ID wins**.

Example cluster:

```text
Node1
Node2
Node3
Node4
Node5
```

If leader fails:

```text
highest ID = Node5
```

Node5 becomes leader.

This approach is used in early distributed algorithms.

---

# 6. Bully Algorithm

The **Bully Algorithm** is a classic leader election protocol.

Idea:

```text
highest node ID becomes leader
```

Steps:

1. Node detects leader failure.
2. Node sends election message to higher-ID nodes.
3. Higher node responds and takes over election.

Example:

Nodes:

```text
1 2 3 4 5
```

Leader (5) crashes.

Node 2 detects failure.

```text
2 → election → 3,4,5
```

Node 4 responds and becomes leader.

---

### Limitation

* high message overhead
* not suitable for large clusters

---

# 7. Ring Election Algorithm

Nodes are organized in a logical ring.

Example:

```text
1 → 2 → 3 → 4 → 5 → 1
```

Election message circulates:

```text
ELECTION(id)
```

Each node forwards the highest ID seen.

When message returns to origin:

```text
highest ID becomes leader
```

---

### Advantages

* deterministic
* simple

---

### Limitations

* slow
* requires ring topology

---

# 8. Leader Election in Raft

Modern distributed systems use **consensus-based election**.

Example: Raft.

Election flow:

1. leader fails
2. followers start election timers
3. first node whose timer expires becomes candidate
4. candidate requests votes

```text
RequestVote RPC
```

If candidate receives **majority votes**, it becomes leader.

Example cluster:

```text
5 nodes
majority = 3
```

---

# 9. ZooKeeper Leader Election

Apache ZooKeeper implements election using **ephemeral sequential nodes**.

Example:

Clients create nodes:

```text
/election/node-0001
/election/node-0002
/election/node-0003
```

The node with the **lowest sequence number becomes leader**.

Others watch the preceding node.

If leader dies:

```text
next node becomes leader
```

This guarantees **fair ordering**.

---

# 10. Leader Election with Distributed Locks

Some systems elect leaders by acquiring a **distributed lock**.

Example:

```text
lock = "leader"
```

Nodes attempt:

```text
acquire_lock("leader")
```

The node that succeeds becomes leader.

Example systems:

* Redis
* etcd

---

# 11. Failure Detection

Leader election depends on **failure detection**.

Nodes detect leader failure using:

```text
heartbeats
timeouts
```

Example:

Leader sends heartbeat every:

```text
100 ms
```

If follower misses heartbeat:

```text
leader considered dead
```

Election begins.

---

# 12. Split-Brain Problem

Network partitions can cause **multiple leaders**.

Example:

Cluster splits:

```text
Partition A → Node1 Node2
Partition B → Node3 Node4 Node5
```

Each partition may elect a leader.

Solution:

```text
majority quorum
```

Only partition with majority can elect leader.

---

# 13. Message Complexity

Leader election algorithms vary.

| Algorithm | Complexity |
| --------- | ---------- |
| Bully     | O(N²)      |
| Ring      | O(N)       |
| Raft      | O(N)       |

Modern systems use **O(N)** approaches.

---

# 14. Real Systems Using Leader Election

Leader election is used in many distributed platforms.

Examples:

* etcd
* Apache ZooKeeper
* Apache Kafka
* HashiCorp Consul

Typical uses:

```text
metadata management
replicated logs
cluster coordination
task scheduling
```

---

# 15. Advantages

### Simplifies Coordination

Only one node handles critical operations.

---

### Prevents Conflicting Updates

Leader serializes operations.

---

### Improves Consistency

All followers replicate leader state.

---

# 16. Limitations

### Leader Bottleneck

All operations may go through leader.

---

### Failover Delay

Election takes time after leader crash.

---

### Split-Brain Risk

Network partitions may produce multiple leaders.

---

# 17. Relationship with Consensus

Leader election is often integrated with **consensus algorithms**.

Example:

```text
Raft → built-in leader election
Paxos → leader emerges implicitly
Zab → leader coordinates broadcast
```

Consensus ensures:

```text
only one leader is valid
```

---

# 18. Distributed System Concept Relationship

```text
Distributed Systems
│
├── Coordination
│     ├── Distributed Locks
│     └── Leader Election
│
├── Consensus
│     ├── Paxos
│     ├── Raft
│     └── PBFT
│
└── Transactions
      ├── 2PC
      └── Saga Pattern
```

---

# 19. Interview Talking Points

**Why do distributed systems use leader election?**

To coordinate operations and avoid conflicting updates.

---

**What happens when leader crashes?**

Followers detect failure and start election.

---

**How do modern systems prevent split-brain?**

Using majority quorum.

---

**Where is leader election used?**

In distributed databases, coordination systems, and message brokers.

---

# 20. Example Interview Answer

If asked:

> "How does leader election work in distributed systems?"

Strong answer:

```text
Nodes detect leader failure via heartbeat timeout.
A node becomes candidate and requests votes from others.
If it receives majority votes, it becomes leader.
Consensus protocols like Raft ensure only one leader exists.
```
