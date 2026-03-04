# Paxos

## 1. What Paxos Is

**Paxos is a consensus algorithm used in distributed systems to ensure that multiple nodes agree on a single value even in the presence of failures.**

Consensus means:

> All non-faulty nodes eventually agree on the same value.

Typical uses:

* electing a leader
* committing a log entry
* choosing configuration values
* agreeing on distributed state

Paxos guarantees **safety even when nodes crash or messages are lost.**

It was introduced by **Leslie Lamport (1998)**.

---

# 2. Problem Paxos Solves

In distributed systems we often need **agreement**.

Example:

Suppose a distributed database must commit a transaction.

We have nodes:

```id="zckfr3"
A  B  C  D  E
```

If nodes commit **different values**, the system becomes inconsistent.

Example problem:

```id="f60ctv"
A commits transaction
B rejects transaction
C times out
```

Now cluster state diverges.

We need a protocol that guarantees:

### Consensus Properties

1. **Agreement**

All nodes decide on the same value.

2. **Validity**

Chosen value must be proposed by some node.

3. **Termination**

System eventually reaches a decision.

4. **Fault tolerance**

Works even if some nodes crash.

---

# 3. Assumptions Paxos Makes

Paxos assumes a **partially synchronous system**:

Nodes may:

* crash
* restart
* experience message delays

But:

* messages are not corrupted
* nodes do not act maliciously

This is known as the **crash failure model**.

---

# 4. Roles in Paxos

Paxos has three logical roles.

### Proposer

Proposes a value.

Example:

```id="v5y7z4"
propose(transaction_commit)
```

---

### Acceptor

Votes on proposals.

Acceptors decide which value is chosen.

---

### Learner

Learners learn the chosen value.

Example:

```id="yk02k5"
replicas update database state
```

---

## Typical Deployment

Nodes often play **all roles simultaneously**.

Example cluster:

```id="nlsutk"
Node1 → proposer + acceptor + learner
Node2 → proposer + acceptor + learner
Node3 → proposer + acceptor + learner
```

---

# 5. Core Idea of Paxos

A value is chosen when:

```id="2qvmbc"
majority of acceptors accept the same proposal
```

Example:

Cluster of **5 nodes**.

Majority:

```id="w74b0n"
3 nodes
```

Once **3 nodes accept a value**, it becomes the decision.

---

# 6. Paxos Algorithm Overview

Paxos proceeds in **two main phases**.

```id="4c6hxn"
Phase 1: Prepare
Phase 2: Accept
```

---

# 7. Phase 1 — Prepare

The proposer sends a **Prepare request** with a proposal number.

Example:

```id="nhpmac"
prepare(n=10)
```

Proposal numbers must be **unique and increasing**.

---

### Step 1

Proposer sends prepare request.

```id="f3fth9"
Proposer → Acceptors
prepare(10)
```

---

### Step 2

Acceptors respond with:

* promise not to accept lower proposal numbers
* previously accepted proposal (if any)

Example:

```id="j7asif"
acceptor reply:
promise(n ≥ 10)
last_accepted = (8, value=X)
```

---

# 8. Phase 2 — Accept

After receiving majority promises, proposer sends **Accept request**.

```id="rqj7i6"
accept(10, value=Y)
```

Acceptors then vote.

---

### Step 3

Acceptors accept proposal if:

```id="0qhqns"
proposal_number ≥ promised_number
```

Then reply:

```id="inr65j"
accepted(10, Y)
```

---

### Step 4

Once majority accepts:

```id="p6sttq"
value Y is chosen
```

Learners are notified.

---

# 9. Example Paxos Execution

Cluster:

```id="gptay5"
5 nodes: A B C D E
```

### Step 1

Proposer A sends:

```id="9g9hfp"
prepare(1)
```

---

### Step 2

Acceptors respond:

```id="8k1d4k"
B → promise
C → promise
D → promise
```

Majority reached.

---

### Step 3

Proposer sends:

```id="1ejn22"
accept(1, value=transaction_commit)
```

---

### Step 4

Acceptors vote:

```id="72vxf3"
B accepted
C accepted
D accepted
```

Majority.

Consensus reached.

---

# 10. Handling Conflicts

Suppose two proposers compete.

Example:

```id="uvhplv"
P1 propose value = X
P2 propose value = Y
```

Each proposal has a number.

Example:

```id="rtybng"
proposal 5
proposal 7
```

Higher proposal number wins.

Acceptors ignore lower proposals once promised.

---

# 11. Paxos Safety Guarantee

Paxos guarantees:

> Once a value is chosen, no different value can be chosen later.

This is achieved using:

```id="3aqnq3"
majority intersection
```

Example:

```id="kce1jn"
majority1 = A B C
majority2 = C D E
```

They overlap at **C**, preserving consistency.

---

# 12. Multi-Paxos

Basic Paxos agrees on **one value**.

Real systems require a **log of values**.

Example:

```id="e7lpqk"
entry1
entry2
entry3
```

Multi-Paxos optimizes this by:

* electing a stable leader
* skipping repeated prepare phases

Result:

```id="dtf75u"
lower latency
```

Multi-Paxos essentially becomes similar to **Raft log replication**.

---

# 13. Real Systems Using Paxos

### Google Chubby

Distributed lock service.

---

### Google Spanner

Uses Paxos groups for replication.

---

### Apache Cassandra (older versions)

Used Paxos for **lightweight transactions**.

---

### ZooKeeper (ZAB inspired by Paxos)

Consensus protocol for metadata.

---

# 14. Advantages

### 1. Fault Tolerant

Works with **node failures and message loss**.

---

### 2. Strong Consistency

Guarantees **single agreed value**.

---

### 3. Proven Correct

Mathematically verified.

---

### 4. Works in Asynchronous Networks

No assumption of perfect clocks.

---

# 15. Limitations

### 1. Hard to Understand

Paxos is notoriously complex.

---

### 2. Many Message Exchanges

Basic Paxos requires multiple network round trips.

---

### 3. Liveness Depends on Leader

If proposers compete heavily, progress slows.

---

### 4. Difficult to Implement

Implementations often have subtle bugs.

---

# 16. Paxos Message Complexity

For each decision:

```id="0kqzti"
prepare phase → N messages
accept phase → N messages
```

Total:

```id="rxcmel"
O(N)
```

---

# 17. Paxos Concept Map

```id="bks9y4"
Distributed Consensus
│
├── Paxos
│     ├── Prepare Phase
│     ├── Accept Phase
│     └── Majority Voting
│
├── Multi-Paxos
│     └── Log Replication
│
├── Raft
│     └── Understandable Paxos Variant
│
└── Byzantine Consensus
      └── PBFT
```

---

# 18. Paxos vs Raft

| Feature           | Paxos          | Raft     |
| ----------------- | -------------- | -------- |
| Complexity        | Very high      | Easier   |
| Leader election   | implicit       | explicit |
| Understandability | difficult      | easier   |
| Adoption          | Google systems | industry |

---

# 19. Interview Talking Points

Good answers for interviews:

**Why majority quorum?**

Because two majorities must overlap.

---

**Why proposal numbers?**

To order competing proposals.

---

**Why two phases?**

Phase 1 prevents conflicting decisions.

---

**Why Multi-Paxos?**

To avoid repeated prepare phases and improve performance.

---

# 20. Related Concepts

Important topics around Paxos:

```id="qbbjnf"
Distributed Consensus
│
├── Paxos
├── Raft
├── Zab
├── Viewstamped Replication
├── Leader Election
└── Quorum Systems
```
