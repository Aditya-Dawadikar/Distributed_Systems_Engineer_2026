# Compare-And-Swap (CAS)

# 1. What It Is

**Compare-And-Swap (CAS)** is an **atomic operation** used for synchronization and concurrency control.

It works like this:

> Compare the current value of a variable with an expected value.
> If they match, replace it with a new value.
> If they do not match, do nothing.

The operation returns whether the swap succeeded.

Pseudo form:

```id="cas1"
CAS(address, expected_value, new_value)
```

Meaning:

```id="cas2"
if (*address == expected_value):
    *address = new_value
    return SUCCESS
else:
    return FAILURE
```

The key property:

> The entire operation is **atomic** — it cannot be interrupted by other threads.

---

# 2. What Problem It Solves

In concurrent systems, many operations follow a pattern:

```id="cas3"
read value
modify value
write value
```

This **read-modify-write sequence** is vulnerable to race conditions.

Example:

Two threads increment a counter.

```id="cas4"
counter = 10
```

Thread A:

```id="cas5"
read 10
write 11
```

Thread B:

```id="cas6"
read 10
write 11
```

Final value becomes:

```id="cas7"
11
```

Correct value should be:

```id="cas8"
12
```

This is the **Lost Update problem**.

CAS solves this by ensuring that updates only succeed if the state has not changed since it was read.

---

# 3. How CAS Works in Practice

Typical usage:

```id="cas9"
while True:
    old = counter
    new = old + 1
    if CAS(counter, old, new):
        break
```

Steps:

1. Read current value
2. Compute new value
3. Attempt CAS
4. If CAS fails (someone changed value), retry

This loop is called a **CAS retry loop**.

---

# 4. Hardware Support

CAS is implemented directly by CPU instructions.

Examples:

| CPU Architecture | Instruction   |
| ---------------- | ------------- |
| x86              | `CMPXCHG`     |
| ARM              | `LDXR/STXR`   |
| PowerPC          | `lwarx/stwcx` |

These instructions guarantee atomicity using **cache coherence protocols**.

Without hardware support, CAS would be impossible to implement efficiently.

---

# 5. CAS in Programming Languages

Most languages expose CAS through atomic libraries.

### Java

```id="cas10"
AtomicInteger counter = new AtomicInteger(0);
counter.compareAndSet(0, 1);
```

### C++

```id="cas11"
std::atomic<int> x;
x.compare_exchange_strong(expected, new_value);
```

### Go

```id="cas12"
atomic.CompareAndSwapInt64(&x, old, new)
```

### Rust

```id="cas13"
atomic.compare_exchange(old, new)
```

---

# 6. CAS in Databases

CAS is widely used for **optimistic concurrency control**.

Example SQL pattern:

```id="cas14"
UPDATE account
SET balance = 110
WHERE id = 1 AND balance = 100;
```

If the balance has changed:

* update affects **0 rows**
* client must retry

This prevents **lost updates**.

---

# 7. CAS in Distributed Systems

CAS is also used in distributed storage systems.

Example: **etcd**

```id="cas15"
PUT key value IF version == expected_version
```

If version mismatch occurs, operation fails.

Examples of systems using CAS-like primitives:

* etcd
* ZooKeeper
* DynamoDB conditional writes
* Cassandra lightweight transactions

These operations are often called:

```id="cas16"
Conditional writes
Check-and-set
Optimistic locking
```

---

# 8. Advantages

### 1. Lock-Free Synchronization

CAS allows building **lock-free data structures**.

Benefits:

* No deadlocks
* High concurrency
* Better scalability

Examples:

* lock-free queues
* lock-free stacks
* atomic counters

---

### 2. Efficient Optimistic Concurrency

Instead of locking resources early:

* assume conflicts are rare
* retry if conflict occurs

This works well for distributed systems.

---

### 3. Prevents Lost Updates

CAS ensures a write only succeeds if the value hasn't changed.

This directly solves the **Lost Update anomaly**.

---

# 9. Limitations

### 1. ABA Problem

CAS only checks the value.

Example:

```id="cas17"
value = A
```

Thread 1 reads A.

Thread 2 changes:

```id="cas18"
A → B → A
```

Thread 1 performs CAS expecting A.

CAS succeeds even though the value changed twice.

This is the **ABA problem**.

Solutions include:

* version numbers
* tagged pointers
* hazard pointers

---

### 2. Retry Overhead

Under high contention:

* many CAS attempts fail
* threads spin and retry

This can degrade performance.

---

### 3. Complex Programming

Lock-free algorithms using CAS are difficult to implement correctly.

Subtle memory ordering bugs can occur.

---

# 10. Real-World Examples

### Redis

Atomic commands like:

```id="cas21"
INCR
SETNX
```

behave like CAS.

---

### Cassandra

Uses **lightweight transactions** built on Paxos to implement CAS semantics.

---

### DynamoDB

Supports conditional updates:

```id="cas22"
ConditionExpression
```

---

### Kubernetes

Uses **resource version CAS**:

```id="cas23"
update object IF resourceVersion matches
```

---

# 12. Interview Explanation (Concise)

Strong interview answer:

> Compare-And-Swap is an atomic primitive that updates a value only if it matches an expected value. It is used to implement optimistic concurrency control and lock-free synchronization. If another thread modifies the value before the swap occurs, the operation fails and must be retried. CAS prevents lost updates and is widely used in concurrent algorithms and distributed systems.

---

# 13. Key Insight

CAS shifts concurrency control from:

```id="cas24"
blocking (locks)
```

to

```id="cas25"
optimistic retry
```

This philosophy underlies many modern distributed systems.

---

# Recommended Reading

**Herlihy & Shavit — The Art of Multiprocessor Programming**

**Designing Data-Intensive Applications — Concurrency chapter**
