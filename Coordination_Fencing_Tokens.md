# Token Fencing (Fencing Tokens)

## 1. What Token Fencing Is

**Token fencing** is a technique used in distributed systems to **prevent stale or zombie clients from performing operations after they lose a distributed lock**.

When a client acquires a lock, the lock service gives it a **monotonically increasing token (fencing token)**.

Every operation on the shared resource must include this token.
The resource **rejects requests with older tokens**.

Goal:

```text
Prevent stale clients from corrupting shared resources.
```

This technique is critical when **distributed locks alone are not sufficient**.

---

# 2. Problem Token Fencing Solves

Distributed locks are often implemented using **leases**.

Example:

```text
Worker1 acquires lock
lock expiry = 10 seconds
```

But suppose:

```text
Worker1 pauses (GC pause / network delay)
```

After timeout:

```text
Worker2 acquires lock
```

Now both workers may operate simultaneously.

Example timeline:

```text
t1: Worker1 gets lock
t2: Worker1 pauses
t3: lock expires
t4: Worker2 gets lock
t5: Worker1 resumes
```

Now we have **two active workers**.

This is called a **stale client problem**.

A distributed lock alone cannot prevent this.

---

# 3. Example Failure Scenario

Shared resource: file storage.

Workers:

```text
Worker1
Worker2
```

Execution:

```text
Worker1 → acquires lock
Worker1 → writing file
Worker1 → pause
lock expires
Worker2 → acquires lock
Worker2 → writes file
Worker1 → resumes and writes file again
```

Result:

```text
file corrupted
```

Token fencing prevents this.

---

# 4. Core Idea

Instead of trusting the lock alone, we attach a **fencing token** to every operation.

Each time a lock is granted:

```text
token = increasing number
```

Example:

```text
Worker1 → token 5
Worker2 → token 6
```

Resource server stores **latest valid token**.

Operations are accepted only if:

```text
token >= latest_token
```

Older tokens are rejected.

---

# 5. Concept Map

```text
Distributed Coordination
│
├── Distributed Locks
│
├── Token Fencing
│     └── Prevent stale clients
│
├── Leader Election
│
└── Consensus Protocols
      ├── Raft
      └── Paxos
```

---

# 6. How Token Fencing Works

### Step 1 — Lock Request

Client requests lock.

```text
client → lock_service
```

---

### Step 2 — Lock Granted

Lock service assigns token.

```text
token = 42
```

---

### Step 3 — Client Uses Resource

Client sends request with token.

```text
write(file, token=42)
```

---

### Step 4 — Resource Validates Token

Resource checks:

```text
if token >= last_token:
    accept
else:
    reject
```

---

# 7. Example Execution

Workers:

```text
Worker1
Worker2
```

### Step 1

Worker1 acquires lock.

```text
token = 10
```

Worker1 writes:

```text
write(token=10)
```

Accepted.

---

### Step 2

Worker1 pauses.

Lock expires.

---

### Step 3

Worker2 acquires lock.

```text
token = 11
```

Worker2 writes:

```text
write(token=11)
```

Accepted.

---

### Step 4

Worker1 resumes.

Attempts:

```text
write(token=10)
```

Server rejects because:

```text
10 < 11
```

Problem solved.

---

# 8. Algorithm

Resource server maintains:

```text
latest_token
```

Pseudo code:

```python
def process_request(token, operation):

    if token < latest_token:
        reject_request()

    latest_token = token
    execute(operation)
```

---

# 9. Why Token Fencing Works

It prevents **old clients from modifying state after losing ownership**.

Key property:

```text
tokens strictly increase
```

This guarantees ordering of operations.

---

# 10. Where Token Fencing Is Used

Token fencing appears in systems with **distributed locking and storage systems**.

Examples:

* Apache ZooKeeper
* etcd
* Google Chubby

These systems generate **monotonic sequence numbers** used as fencing tokens.

---

# 11. Relationship With Distributed Locks

Distributed locks ensure:

```text
only one client SHOULD hold lock
```

Token fencing ensures:

```text
only latest client CAN modify resource
```

Together they provide strong safety.

---

# 12. Token Generation Methods

Tokens must be **monotonically increasing**.

Typical approaches:

### Global Counter

```text
token = counter++
```

---

### Log Index

In consensus systems:

```text
token = log index
```

---

### Timestamp + Counter

Hybrid logical clocks.

---

# 13. Token Fencing vs Lock Leases

| Feature                       | Lock Lease | Token Fencing |
| ----------------------------- | ---------- | ------------- |
| protects against stale client | ❌          | ✅             |
| uses expiration               | ✅          | optional      |
| ordering guarantee            | ❌          | ✅             |

Best practice:

```text
lock + fencing token
```

---

# 14. Limitations

### Requires Resource Validation

The resource server must check tokens.

---

### Extra State

Resource must track latest token.

---

### Not Always Supported

Some storage systems cannot enforce token ordering.

---

# 15. Advantages

### Prevents Split-Brain Writes

Even if two clients believe they hold lock.

---

### Works With Lease Expiration

Protects against GC pauses and network delays.

---

### Simple and Reliable

Only requires monotonic tokens.

---

# 16. Interview Talking Points

**Why are distributed locks alone unsafe?**

Because clients may resume after lease expiration and still perform operations.

---

**What does token fencing prevent?**

Stale clients modifying shared resources.

---

**What property must tokens satisfy?**

They must be **monotonically increasing**.

---

**Where are fencing tokens used?**

Distributed lock services like ZooKeeper or etcd.

---

# 17. Relationship With Other Concepts

```text
Distributed Coordination
│
├── Distributed Locks
│
├── Token Fencing
│
├── Leader Election
│
└── Consensus
     ├── Raft
     └── Paxos
```

---

# 18. Real Interview Scenario

Interview question:

> "How do you prevent stale clients when using distributed locks?"

Strong answer:

```text
Use fencing tokens with locks.
Each lock acquisition receives a monotonically increasing token.
Resource servers reject operations with older tokens.
```
