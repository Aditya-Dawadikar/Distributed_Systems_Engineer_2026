# Distributed Locks

## 1. What Distributed Locks Are

A **distributed lock** is a mechanism that ensures **only one process across a distributed system can access a shared resource at a time**.

It is similar to a **mutex in a single-machine program**, but extended to **multiple machines communicating over a network**.

Example:

```text
multiple services → accessing same resource
```

Examples of resources:

* updating a shared database row
* processing a scheduled job
* modifying configuration data
* leader election

Distributed locks guarantee **mutual exclusion across machines**.

---

# 2. Problem Distributed Locks Solve

In distributed systems multiple nodes may attempt to perform the **same operation simultaneously**.

Example: scheduled job processing

Cluster:

```
Worker1
Worker2
Worker3
```

All workers try to run:

```
generate_monthly_report()
```

Without coordination:

```
Worker1 → runs job
Worker2 → runs job
Worker3 → runs job
```

Result:

* duplicate work
* inconsistent state
* data corruption

A distributed lock ensures:

```
only one worker executes job
```

---

# 3. Desired Properties of Distributed Locks

A correct distributed lock system should satisfy:

### Mutual Exclusion

Only one node holds the lock.

```
lock_owner = unique
```

---

### Deadlock Freedom

System eventually releases locks.

---

### Fault Tolerance

Locks should survive node crashes.

---

### Fairness (Optional)

Requests are served in order.

---

# 4. Basic Concept

Typical distributed lock workflow:

```
Client → request lock
Lock service → grant lock
Client → perform operation
Client → release lock
```

Example:

```
acquire_lock("job_lock")
run_job()
release_lock("job_lock")
```

---

# 5. Centralized Lock Service

Simplest approach: **central lock server**.

Architecture:

```
Clients → Lock Server → Resource
```

Lock server stores:

```
lock_id
owner
expiry
```

Algorithm:

```
if lock free:
    grant lock
else:
    deny request
```

---

### Advantages

* simple implementation
* easy to reason about

### Limitations

* single point of failure
* scalability bottleneck

---

# 6. Database-Based Distributed Locks

Databases can enforce locks using **transactions**.

Example:

```sql
INSERT INTO locks(lock_name, owner)
VALUES('job_lock','worker1')
```

If row exists:

```
lock already taken
```

---

Example systems:

* PostgreSQL
* MySQL

Common technique:

```
SELECT ... FOR UPDATE
```

---

### Limitation

* database contention
* poor performance at scale

---

# 7. ZooKeeper Distributed Locks

Apache ZooKeeper provides reliable distributed locks.

Mechanism:

```
ephemeral sequential nodes
```

Example:

```
/locks/job_lock/lock-00001
/locks/job_lock/lock-00002
/locks/job_lock/lock-00003
```

Steps:

1. client creates sequential node
2. client with smallest number gets lock
3. others watch previous node
4. when node deleted → next client gets lock

---

### Advantages

* fair ordering
* automatic release on crash
* strong consistency

---

# 8. Redis Distributed Locks

Redis supports distributed locking.

Basic implementation:

```
SET lock_key value NX PX 5000
```

Explanation:

```
NX → only if key doesn't exist
PX → expiration time
```

Example:

```
SET job_lock worker1 NX PX 5000
```

If success → lock acquired.

---

### Lock Release

Use unique value:

```
DEL job_lock
```

But must verify owner before deleting.

---

# 9. Redlock Algorithm

Redlock is a distributed locking algorithm for Redis clusters.

Steps:

1. acquire lock on multiple Redis nodes
2. if majority succeed → lock granted
3. compute lock validity time
4. release lock from nodes

Example cluster:

```
Redis1
Redis2
Redis3
Redis4
Redis5
```

Majority required:

```
3 nodes
```

---

# 10. Lease-Based Locks

Many distributed locks are implemented as **leases**.

Lease:

```
lock with expiration
```

Example:

```
lock expires in 10 seconds
```

Advantages:

* prevents permanent deadlocks
* automatic recovery after crash

---

# 11. Example Distributed Lock Execution

Cluster workers:

```
Worker1
Worker2
Worker3
```

Job:

```
process_queue()
```

Execution:

```
Worker1 → acquire lock
Worker2 → fail
Worker3 → fail
```

Worker1 runs job.

After completion:

```
Worker1 → release lock
```

Worker2 retries and acquires.

---

# 12. Failure Scenarios

### Client Crash

Client acquires lock but crashes.

Solution:

```
lease timeout
```

Lock automatically expires.

---

### Network Partition

Two nodes believe they hold lock.

This is called **split-brain**.

Solutions:

* quorum-based locks
* consensus systems

---

# 13. Message Complexity

Centralized lock:

```
O(1)
```

ZooKeeper lock:

```
O(log N)
```

Consensus-based locks:

```
O(N)
```

---

# 14. Real Systems Using Distributed Locks

Distributed locks appear in many infrastructure systems.

Examples:

* Apache ZooKeeper
* etcd
* Redis
* HashiCorp Consul

Use cases:

```
leader election
job scheduling
configuration updates
distributed cron jobs
```

---

# 15. Advantages

### Prevents Race Conditions

Ensures only one process modifies resource.

---

### Simplifies Coordination

Makes distributed systems easier to reason about.

---

### Enables Leader Election

Leader is simply the lock holder.

---

# 16. Limitations

### Risk of Deadlocks

Locks may remain if system fails.

---

### Network Partitions

Split-brain scenarios possible.

---

### Latency Overhead

Extra network calls required.

---

### Hard to Implement Correctly

Edge cases like:

```
clock drift
lock expiration
network delays
```

---

# 17. Distributed Lock Concept Map

```
Distributed Coordination
│
├── Distributed Locks
│   │
│   ├── Redis Locks
│   ├── ZooKeeper Locks
│   ├── Database Locks
│   └── Lease Locks
│
├── Leader Election
│
└── Consensus Protocols
    ├── Raft
    └── Paxos
```

---

# 18. Distributed Locks vs Consensus

| Feature         | Distributed Lock | Consensus         |
| --------------- | ---------------- | ----------------- |
| Purpose         | mutual exclusion | agreement         |
| Complexity      | lower            | higher            |
| fault tolerance | weaker           | stronger          |
| typical use     | job scheduling   | state replication |

---

# 19. Interview Talking Points

**What is a distributed lock?**

Mechanism ensuring only one node accesses a shared resource at a time.

---

**What problem does it solve?**

Prevents race conditions across distributed systems.

---

**How do ZooKeeper locks work?**

Using ephemeral sequential nodes and watchers.

---

**Why use leases?**

To prevent permanent locks if a client crashes.

---

# 20. Related Concepts

Distributed locks are closely connected with:

```
Distributed Coordination
│
├── Leader Election
├── Consensus
├── Quorum Systems
├── Failure Detection
└── Distributed Transactions
```
