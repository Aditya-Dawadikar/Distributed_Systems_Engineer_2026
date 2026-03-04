# Three-Phase Commit (3PC)

## 1. What 3PC Is

**Three-Phase Commit (3PC)** is a distributed transaction protocol designed to improve on **Two-Phase Commit (2PC)** by **eliminating the blocking problem**.

Like 2PC, it ensures **atomic commitment across multiple distributed participants**, but it introduces an extra phase to allow participants to **make progress even if the coordinator crashes**.

Goal:

```text
Either all participants commit
OR
all participants abort
```

3PC is primarily a **theoretical improvement** to 2PC. In practice, it is rarely used because most modern systems rely on **consensus protocols** instead.

---

# 2. Problem 3PC Solves

Recall the main weakness of **Two-Phase Commit (2PC)**:

If the **coordinator crashes after the prepare phase**, participants are stuck.

Example:

```
Coordinator: crashed
Participant A: prepared
Participant B: prepared
```

Participants cannot decide whether to:

```
commit or abort
```

So they must **wait indefinitely**, which locks resources.

This is called the **blocking problem**.

3PC solves this by ensuring:

```
participants always know a safe action
```

---

# 3. Key Idea Behind 3PC

3PC introduces an intermediate phase so that the system always remains in a **non-blocking state**.

The protocol guarantees that:

```
participants never remain in an uncertain state
```

Even if the coordinator fails.

This requires additional assumptions:

* bounded message delay
* bounded node failure detection

This means the system must be **partially synchronous**.

---

# 4. Architecture

Similar to 2PC.

Roles:

```
Coordinator
Participants
```

Coordinator:

* manages transaction
* collects votes
* issues commit decision

Participants:

* execute operations
* respond with readiness

---

# 5. Concept Map

```
Distributed Transactions
│
├── Two Phase Commit (2PC)
│     └── Blocking protocol
│
├── Three Phase Commit (3PC)
│     └── Non-blocking protocol
│
└── Saga Pattern
```

---

# 6. Phases of 3PC

3PC introduces **three phases**:

```
1. CanCommit (Voting Phase)
2. PreCommit (Preparation Phase)
3. DoCommit (Commit Phase)
```

This separates **agreement** from **final commit**.

---

# 7. Phase 1 — CanCommit

Coordinator asks participants if they **can commit**.

Message:

```
Coordinator → Participants
CAN_COMMIT?
```

Participants check if transaction can proceed.

Possible responses:

```
YES
NO
```

If any participant votes **NO**, coordinator aborts.

If all vote **YES**, protocol continues.

---

# 8. Phase 2 — PreCommit

Coordinator sends:

```
PRE_COMMIT
```

Participants:

* prepare transaction
* write logs
* lock resources

Then reply:

```
ACK
```

Participants now know:

```
commit is very likely
```

But transaction is **not committed yet**.

---

# 9. Phase 3 — DoCommit

Coordinator sends:

```
DO_COMMIT
```

Participants finalize the transaction.

Example:

```
UPDATE accounts SET balance=...
```

Transaction becomes permanent.

---

# 10. Example Execution

Suppose a distributed money transfer.

Nodes:

```
Coordinator
BankA
BankB
```

Transaction:

```
Transfer $100
```

---

### Phase 1

Coordinator asks:

```
CAN_COMMIT?
```

Participants respond:

```
BankA → YES
BankB → YES
```

---

### Phase 2

Coordinator sends:

```
PRE_COMMIT
```

Participants prepare transaction and acknowledge.

```
ACK
```

---

### Phase 3

Coordinator sends:

```
DO_COMMIT
```

Participants commit transaction.

---

# 11. Why 3PC Is Non-Blocking

The **PreCommit phase** ensures participants know the system state.

If coordinator crashes:

Participants can determine safe action.

Example:

```
Coordinator crashes after PRE_COMMIT
```

Participants know:

```
transaction must commit
```

So they can safely finish commit.

If crash occurs earlier:

Participants can abort.

Thus **no indefinite waiting**.

---

# 12. Failure Scenarios

### Case 1 — Participant Failure

If participant crashes before responding:

Coordinator aborts transaction.

---

### Case 2 — Coordinator Failure (Before PreCommit)

Participants time out and **abort transaction**.

---

### Case 3 — Coordinator Failure (After PreCommit)

Participants detect that commit is inevitable and **commit transaction**.

---

# 13. Message Flow

```
Client
   │
   ▼
Coordinator
   │
   ├── CAN_COMMIT → A
   ├── CAN_COMMIT → B
   │
   ◄── YES
   ◄── YES
   │
   ├── PRE_COMMIT → A
   ├── PRE_COMMIT → B
   │
   ◄── ACK
   ◄── ACK
   │
   ├── DO_COMMIT → A
   └── DO_COMMIT → B
```

---

# 14. Message Complexity

For **N participants**:

```
CAN_COMMIT messages  = N
Votes                 = N
PRE_COMMIT            = N
ACK                   = N
DO_COMMIT             = N
```

Total complexity:

```
O(N)
```

But **more round trips than 2PC**.

---

# 15. Advantages

### Non-Blocking Protocol

Participants can always reach a decision.

---

### Better Failure Handling

Handles coordinator crashes better than 2PC.

---

### Clear Transaction States

PreCommit state reduces ambiguity.

---

# 16. Limitations

### Requires Synchronous Network

3PC assumes bounded delays.

Real distributed systems cannot guarantee this.

---

### Higher Latency

Extra phase increases communication overhead.

---

### Rarely Used In Practice

Modern systems prefer:

* consensus algorithms
* saga patterns

---

# 17. 2PC vs 3PC

| Feature             | 2PC          | 3PC                   |
| ------------------- | ------------ | --------------------- |
| Phases              | 2            | 3                     |
| Blocking            | Yes          | No                    |
| Latency             | Lower        | Higher                |
| Network assumptions | asynchronous | partially synchronous |
| Adoption            | Very common  | Rare                  |

---

# 18. 3PC vs Consensus Protocols

| Feature         | 3PC           | Paxos/Raft            |
| --------------- | ------------- | --------------------- |
| Problem solved  | atomic commit | distributed consensus |
| Blocking        | No            | No                    |
| Fault tolerance | limited       | stronger              |
| Real-world use  | rare          | extremely common      |

---

# 19. Real-World Use

3PC is mostly **academic**.

Modern distributed systems prefer:

* consensus protocols (Raft/Paxos)
* compensating transactions (Saga)

---

# 20. Relationship With Other Concepts

```
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

---

# 21. Interview Talking Points

**Why was 3PC created?**

To eliminate the blocking problem of 2PC.

---

**Why is 3PC rarely used?**

Because it assumes synchronous networks, which real systems cannot guarantee.

---

**What replaced 3PC in modern systems?**

Consensus algorithms and saga-based transactions.
