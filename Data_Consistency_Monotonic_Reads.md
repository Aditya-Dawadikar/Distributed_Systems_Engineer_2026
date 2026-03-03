# Monotonic Reads

# 1️⃣ What Is Monotonic Reads?

Monotonic reads is a **consistency guarantee for reads**.

Definition:

> If a process reads a value of a data item, any subsequent read by that same process will return the same value or a more recent value.

In simple words:

> Once you’ve seen a version of data, you will never see an older version later.

The key phrase is:

**“same client / same session”**

This is a *session guarantee*, not a global guarantee.

---

# 2️⃣ What Problem Is It Trying to Solve?

In eventually consistent systems:

* Replicas update asynchronously.
* Clients may be routed to different replicas.
* Some replicas may lag.

Without monotonic reads:

You could see:

1. Version 5
2. Then version 3
3. Then version 6

That breaks user intuition.

It feels like time went backward.

Monotonic reads prevents this backward movement.

---

# 3️⃣ Concrete Example

Imagine a social media system.

Post initially:

```id="a1"
"Version 1"
```

Then edited:

```id="a2"
"Version 2"
```

Then edited again:

```id="a3"
"Version 3"
```

Now assume three replicas:

* Replica A → Version 3
* Replica B → Version 1
* Replica C → Version 2

Without monotonic reads:

User:

1. Reads from Replica A → sees Version 3
2. Next request routed to Replica B → sees Version 1

From user’s perspective:
The system went backward in time.

That is allowed in eventual consistency.

With monotonic reads:

After seeing Version 3,
user will never be served Version 1 or 2 again.

---

# 4️⃣ Important Clarification

Monotonic reads does NOT guarantee:

* You always see the latest value.
* Global consistency.
* Strong consistency.

It only guarantees:

> Your reads never regress.

Other users may see different values.

---

# 5️⃣ Relationship to Other Guarantees

There are four classical session guarantees (Terry et al., 1994):

1. Read-your-writes
2. Monotonic reads
3. Monotonic writes
4. Writes-follow-reads

Monotonic reads is just one of them.

---

## Difference from Read-Your-Writes

Read-your-writes:
After you write something, you must see your own write.

Monotonic reads:
After you read something, you must not see an older value later.

They are related but different.

You can have one without the other.

---

# 6️⃣ How Systems Implement Monotonic Reads

To enforce monotonic reads, systems typically:

### Option 1: Sticky Sessions

Route the client to the same replica.

Simple but not fault-tolerant.

---

### Option 2: Track Version Numbers

Each read returns a version (timestamp, vector clock, etc.).

Client includes:

“Do not give me data older than version X.”

Replica ensures:
It serves at least that version.

---

### Option 3: Use Majority Reads

If read quorum intersects with write quorum,
older versions are less likely to be returned.

This does not strictly guarantee monotonic reads unless carefully enforced.

---

# 7️⃣ Advantages

### 1. Prevents Confusing User Experience

No backward time travel.

### 2. Improves Application-Level Predictability

UI won’t flicker between versions.

### 3. Low Cost Compared to Strong Consistency

Does not require global coordination.

---

# 8️⃣ Limitations

### 1. Per-Session Guarantee Only

Other users can see different states.

### 2. Requires Client Metadata

Must track last-seen version.

### 3. Not a Full Consistency Model

It doesn’t prevent:

* Lost updates
* Write skew
* Stale reads (forward stale still possible)

You may still see old data initially.
You just won’t see something older than what you already saw.

---

# 9️⃣ Real-World Usage

Common in:

* Dynamo-style systems
* Cassandra
* MongoDB with session consistency
* Distributed caches
* Edge replication systems

Often implemented in:

* Mobile apps
* Web session layers
* CDN-backed systems

---

# 🔟 Comparison Table

| Guarantee            | What It Ensures         |
| -------------------- | ----------------------- |
| Strong consistency   | Always latest value     |
| Eventual consistency | Eventually converges    |
| Read-your-writes     | You see your own writes |
| Monotonic reads      | You don’t go backward   |

---

# 1️⃣1️⃣ Interview Framing

If interviewer asks:

“What is monotonic read consistency?”

Good answer:

> Monotonic reads guarantee that once a client has read a particular version of data, it will never see an older version in subsequent reads. It is a session-level consistency guarantee often used in eventually consistent systems to prevent time-travel anomalies.

Then add:

> It does not guarantee freshness — only non-regression.

That shows precision.

---

# 1️⃣2️⃣ Deep Insight (Senior-Level Understanding)

Monotonic reads ensures:

> A client observes a non-decreasing sequence of versions.

Formally:

If client observes version v1, then later version v2 must satisfy:

v2 ≥ v1

This is essentially enforcing:

Per-client causal ordering.

It is weaker than:

* Causal consistency
* Sequential consistency
* Linearizability

But stronger than pure eventual consistency.

---

# 1️⃣3️⃣ Recommended Reading

* Terry et al., 1994 — Session Guarantees for Weakly Consistent Replicated Data
* Designing Data-Intensive Applications — Consistency chapter
* Dynamo paper — Amazon

---

# ⚠️ Important Interview Insight

Monotonic reads is often enough for:

* User-facing systems
* Social feeds
* Comments
* Profile updates

You don’t always need strong consistency.

Choosing monotonic reads shows you understand tradeoffs.
