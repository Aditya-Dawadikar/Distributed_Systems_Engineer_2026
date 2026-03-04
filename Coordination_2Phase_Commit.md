# Two-Phase Commit (2PC)

## 1. What 2PC Is

**Two-Phase Commit (2PC)** is a distributed protocol used to ensure **atomic transactions across multiple systems**.

Atomic means:

> Either **all participants commit** the transaction or **none commit**.

2PC is commonly used when a transaction spans **multiple databases or services**.

Example:

```text
Transfer $100 from Account A → Account B
```

If the systems storing A and B are on different machines, both must succeed or both must rollback.

Typical systems using distributed transactions:

* PostgreSQL (distributed transactions)
* MySQL
* Oracle Database
* Apache Kafka (transaction coordinator)

---

# 2. Problem 2PC Solves

Consider a distributed money transfer.

```text
Node A → deduct money
Node B → add money
```

If a failure happens between operations:

```text
A: money deducted
B: money not added
```

The system becomes **inconsistent**.

We need a protocol ensuring:

```text
all nodes commit
OR
all nodes rollback
```

This is called **distributed atomic commitment**.

---

# 3. Architecture

2PC involves two roles.

```text
Coordinator
Participants
```

### Coordinator

* manages the transaction
* collects votes
* decides commit/abort

### Participants

* execute the transaction
* report readiness

Example:

```text
Coordinator → Transaction Manager

Participants → Databases / Services
```

---

# 4. Concept Map

```text
Distributed Transactions
│
├── Two Phase Commit (2PC)
│     ├── Prepare Phase
│     └── Commit Phase
│
├── Three Phase Commit (3PC)
│
└── Saga Pattern
```

---

# 5. 2PC Phases

The protocol has **two phases**.

```text
1 Prepare Phase (Voting)
2 Commit Phase (Decision)
```

---

# 6. Phase 1 — Prepare (Voting Phase)

Coordinator asks participants if they can commit.

Step:

```text
Coordinator → Participants
PREPARE
```

Participants attempt transaction locally.

Possible responses:

```text
VOTE_COMMIT
VOTE_ABORT
```

Participants **lock resources** during this phase.

Example:

```text
BankA: deduct $100 (locked)
BankB: add $100 (locked)
```

But changes are **not committed yet**.

---

# 7. Phase 2 — Commit (Decision Phase)

Coordinator collects votes.

### Case 1: All participants vote commit

Coordinator sends:

```text
COMMIT
```

Participants finalize transaction.

---

### Case 2: Any participant votes abort

Coordinator sends:

```text
ROLLBACK
```

Participants undo operations.

---

# 8. Example Execution

Suppose transfer across two banks.

Nodes:

```text
Coordinator
BankA
BankB
```

Transaction:

```text
Transfer $100
```

---

### Phase 1

Coordinator sends:

```text
PREPARE
```

Participants respond:

```text
BankA → VOTE_COMMIT
BankB → VOTE_COMMIT
```

---

### Phase 2

Coordinator sends:

```text
COMMIT
```

Participants finalize transaction.

---

# 9. Message Flow

```text
Client
   │
   ▼
Coordinator
   │
   ├── PREPARE → NodeA
   ├── PREPARE → NodeB
   │
   ◄── VOTE_COMMIT
   ◄── VOTE_COMMIT
   │
   ├── COMMIT → NodeA
   └── COMMIT → NodeB
```

---

# 10. Failure Scenarios

## Case 1 — Participant Failure

Participant crashes before voting.

Coordinator receives no vote.

Result:

```text
transaction aborted
```

---

## Case 2 — Coordinator Failure

Coordinator crashes **after prepare phase**.

Participants remain in uncertain state.

Example:

```text
NodeA → prepared
NodeB → prepared
Coordinator → crashed
```

Participants cannot decide.

They remain **blocked**.

This is called the **blocking problem**.

---

# 11. Blocking Problem

2PC is a **blocking protocol**.

If coordinator crashes after prepare phase:

```text
participants wait indefinitely
```

Resources remain locked.

Example:

```text
rows locked
transactions stalled
```

This is the major weakness of 2PC.

---

# 12. Logging for Recovery

Nodes store decisions in **write-ahead logs**.

Example log entry:

```text
transaction_id
state
```

Possible states:

```text
INIT
PREPARED
COMMITTED
ABORTED
```

If node crashes:

```text
recover from logs
```

---

# 13. Message Complexity

For **N participants**.

Messages:

```text
Prepare → N
Votes → N
Commit → N
```

Total:

```text
O(N)
```

Latency:

```text
2 network round trips
```

---

# 14. Real Systems Using 2PC

### Distributed Databases

* PostgreSQL
* MySQL
* Oracle Database

---

### Distributed Transaction Managers

Examples:

* Apache Kafka transactions
* Google Spanner (with Paxos)

---

# 15. Advantages

### Strong Consistency

Guarantees atomic transactions.

---

### Simple Design

Conceptually easy to implement.

---

### Works With Existing Databases

Most relational databases support 2PC.

---

# 16. Limitations

### Blocking Problem

Coordinator failure blocks participants.

---

### High Latency

Requires multiple network round trips.

---

### Locks Resources

Participants hold locks during prepare phase.

This reduces concurrency.

---

### Poor Scalability

Not suitable for very large distributed systems.

---

# 17. 2PC vs 3PC

| Feature       | 2PC         | 3PC    |
| ------------- | ----------- | ------ |
| Phases        | 2           | 3      |
| Blocking      | Yes         | No     |
| Complexity    | Low         | Higher |
| Practical use | Very common | Rare   |

3PC attempts to solve the blocking problem.

---

# 18. 2PC vs Consensus Protocols

| Feature         | 2PC                 | Raft/Paxos          |
| --------------- | ------------------- | ------------------- |
| Goal            | Atomic transactions | Replicated log      |
| Fault tolerance | weak                | stronger            |
| Blocking        | possible            | no                  |
| Use case        | databases           | distributed storage |

---

# 19. Interview Talking Points

**What problem does 2PC solve?**

Ensures atomic commit across multiple distributed participants.

---

**Why is 2PC blocking?**

Participants cannot decide commit/abort if coordinator fails after prepare.

---

**Why does 2PC lock resources?**

Participants must prevent conflicting updates while transaction is pending.

---

**Where is 2PC used?**

Distributed databases and transaction managers.

---

# 20. Relationship With Other Concepts

```text
Distributed Systems
│
├── Consensus
│     ├── Paxos
│     ├── Raft
│     └── PBFT
│
├── Replication
│     └── Zab
│
└── Distributed Transactions
      ├── Two Phase Commit
      ├── Three Phase Commit
      └── Saga Pattern
```
