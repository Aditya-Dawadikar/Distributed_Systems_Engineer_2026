# PBFT — Practical Byzantine Fault Tolerance

## 1. What PBFT Is

**PBFT is a consensus algorithm designed to tolerate Byzantine failures in distributed systems.**

A **Byzantine failure** means nodes may behave arbitrarily:

* crash
* send incorrect messages
* send conflicting messages
* act maliciously

PBFT ensures that **correct nodes reach agreement even when some nodes behave maliciously**.

It was introduced in **1999 by Miguel Castro and Barbara Liskov**.

PBFT is widely used in:

* **permissioned blockchains**
* **distributed ledgers**
* **secure distributed systems**

---

# 2. Problem PBFT Solves

Traditional consensus algorithms like **Paxos and Raft assume crash failures**.

Example failure:

```id="p0bft1"
Node crashes
Node stops responding
```

But in many environments nodes may behave maliciously.

Example:

```id="p0bft2"
Node A sends value = 10 to B
Node A sends value = 20 to C
```

This creates **inconsistent system state**.

PBFT solves the **Byzantine Generals Problem**.

---

# 3. Byzantine Generals Problem

Imagine generals coordinating an attack.

```id="p0bft3"
General A
General B
General C
General D
```

Some generals may be traitors.

Goal:

```id="p0bft4"
All loyal generals must agree on the same decision.
```

Even if traitors send false messages.

---

# 4. Fault Tolerance Requirement

PBFT requires:

```id="p0bft5"
N ≥ 3f + 1
```

Where:

```id="p0bft6"
f = number of Byzantine nodes
```

Example:

| Nodes | Byzantine tolerated |
| ----- | ------------------- |
| 4     | 1                   |
| 7     | 2                   |
| 10    | 3                   |

Reason:

At least **2f + 1 honest nodes must exist**.

---

# 5. PBFT Architecture

Nodes are divided into:

```id="p0bft7"
1 Primary (Leader)
N-1 Replicas
```

The **primary coordinates the request**.

If primary fails or behaves maliciously:

```id="p0bft8"
view change occurs
```

New primary is elected.

---

# 6. PBFT Phases

PBFT operates in **three main phases**.

```id="p0bft9"
1. Pre-Prepare
2. Prepare
3. Commit
```

These phases ensure that all honest nodes agree.

---

# 7. Step-by-Step PBFT Protocol

Assume **4 nodes**:

```id="p0bft10"
N1 (Primary)
N2
N3
N4
```

Client wants to perform operation:

```id="p0bft11"
SET X = 5
```

---

# Phase 1 — Pre-Prepare

Client sends request to primary.

```id="p0bft12"
Client → Primary
request(operation)
```

Primary assigns sequence number:

```id="p0bft13"
seq = 101
```

Primary sends message:

```id="p0bft14"
PRE-PREPARE(view, seq, operation)
```

to all replicas.

---

# Phase 2 — Prepare

Replicas verify:

* message format
* signature
* correct sequence number

If valid they broadcast:

```id="p0bft15"
PREPARE(view, seq, digest)
```

to all nodes.

Each node collects prepares.

Condition:

```id="p0bft16"
2f prepares received
```

Then node enters **prepared state**.

---

# Phase 3 — Commit

Nodes broadcast:

```id="p0bft17"
COMMIT(view, seq)
```

When node receives:

```id="p0bft18"
2f + 1 commit messages
```

It executes operation.

Example:

```id="p0bft19"
SET X = 5 applied
```

---

# 8. Message Flow Example

Cluster of **4 nodes (f=1)**.

### Step 1

Client request:

```id="p0bft20"
Client → Primary
```

---

### Step 2

Primary broadcasts:

```id="p0bft21"
PRE-PREPARE
```

---

### Step 3

Replicas broadcast:

```id="p0bft22"
PREPARE
```

---

### Step 4

Replicas broadcast:

```id="p0bft23"
COMMIT
```

---

### Step 5

Operation executed.

---

# 9. View Change (Leader Replacement)

If primary behaves incorrectly:

* sends conflicting messages
* stops responding

Replicas trigger:

```id="p0bft24"
VIEW CHANGE
```

New primary is selected.

Primary selection rule:

```id="p0bft25"
primary = view_number mod N
```

---

# 10. Why PBFT Needs 3f + 1 Nodes

Example:

```id="p0bft26"
N = 4
f = 1 Byzantine node
```

Honest nodes:

```id="p0bft27"
3 nodes
```

Majority ensures malicious nodes cannot control consensus.

---

# 11. Complexity

PBFT communication complexity:

```id="p0bft28"
O(N²)
```

Reason:

Each node communicates with every other node.

---

# 12. Example Systems Using PBFT

### Hyperledger Fabric

Permissioned blockchain.

---

### Tendermint

Byzantine consensus engine.

---

### Zilliqa

Blockchain consensus.

---

### Libra / Diem (Meta)

Modified BFT protocol.

---

# 13. Advantages

### 1 Strong Security

Handles **malicious nodes**.

---

### 2 Deterministic Finality

Once consensus reached:

```id="p0bft29"
transaction cannot be reversed
```

---

### 3 Low Latency (for small clusters)

Faster than proof-of-work.

---

### 4 Works Without Mining

No energy waste.

---

# 14. Limitations

### 1 Poor Scalability

Communication:

```id="p0bft30"
O(N²)
```

Large clusters become expensive.

---

### 2 Works Best in Permissioned Networks

Nodes must be **known and authenticated**.

---

### 3 Complex View Change

Leader changes add overhead.

---

### 4 Not Suitable for Large Open Networks

Public blockchains often prefer:

```id="p0bft31"
PoW
PoS
```

---

# 15. PBFT vs Paxos

| Feature         | Paxos   | PBFT      |
| --------------- | ------- | --------- |
| Failure model   | Crash   | Byzantine |
| Fault tolerance | f < N/2 | f < N/3   |
| Complexity      | O(N)    | O(N²)     |
| Security        | medium  | high      |

---

# 16. PBFT Concept Map

```id="p0bft32"
Distributed Consensus
│
├── Crash Fault Tolerance
│     ├── Paxos
│     └── Raft
│
└── Byzantine Fault Tolerance
      ├── PBFT
      ├── Tendermint
      └── HotStuff
```

---

# 17. Relationship With Other Concepts

```id="p0bft33"
Distributed Systems
│
├── Consensus
│     ├── Paxos
│     ├── Raft
│     └── PBFT
│
├── Fault Models
│     ├── Crash Failures
│     └── Byzantine Failures
│
└── Replication
      └── State Machine Replication
```

---

# 18. Interview Talking Points

**Why PBFT needs 3f+1 nodes?**

Because with f malicious nodes we still need **2f+1 honest majority**.

---

**Why multiple phases?**

To ensure malicious nodes cannot trick replicas.

---

**Why communication is O(N²)?**

Each node broadcasts messages to all others.

---

**Where is PBFT used?**

Permissioned blockchains like **Hyperledger Fabric and Tendermint**.

---

# 19. Related Algorithms

Important algorithms near PBFT:

```id="p0bft34"
Byzantine Consensus
│
├── PBFT
├── HotStuff
├── Tendermint
├── ZAB
└── Byzantine Paxos
```
