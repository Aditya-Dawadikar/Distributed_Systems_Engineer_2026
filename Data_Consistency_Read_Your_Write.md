# Read-Your-Writes Consistency


## 1️⃣ What It Is

**Read-after-write / Read-your-Writes (RYW) consistency** is a *session-level consistency guarantee* that ensures:

> If a client writes a value, any subsequent reads by that same client will reflect that write (or a more recent one).

The emphasis is on:

* **Same client**
* **Subsequent reads**
* **Not global consistency**

It is not a full consistency model like linearizability.
It is a **client/session guarantee** layered on top of weaker models like eventual consistency.

---

## 2️⃣ What Problem It Solves

In eventually consistent systems:

* Writes propagate asynchronously.
* Replicas may lag.
* Load balancers may route requests to different replicas.

Without RYW, this can happen:

1. Client updates profile name → “Alice”
2. Immediately refreshes page
3. Sees old name → “Alicia”

From the user’s perspective, the system appears broken.

Read-after-write solves:

* “I just changed it — why don’t I see it?”
* Stale self-reads
* UI confusion

It preserves basic user intuition.

---

## 3️⃣ Concrete Example

Imagine a distributed database with three replicas:

* R1 (updated)
* R2 (not updated yet)
* R3 (not updated yet)

Client writes to R1.

If next read is routed to R2:

Client sees old value.

That violates read-after-write consistency.

With RYW:

The system ensures that:

* The next read is served from R1
  OR
* The read waits until replication catches up
  OR
* The read includes version constraints

---

## 4️⃣ What It Does NOT Guarantee

This is important.

Read-after-write does **not** guarantee:

* Global consistency
* That other clients see your write immediately
* No lost updates
* No write skew
* Serializability

It only guarantees:

> You will not observe your own stale write.

Other users may still see old data.

---

## 5️⃣ Relationship to Other Guarantees

Read-after-write is one of the four classic session guarantees (Terry et al., 1994):

1. Read-your-writes (RYW)
2. Monotonic reads
3. Monotonic writes
4. Writes-follow-reads

### Relationship to Monotonic Reads

Monotonic reads:
You don’t go backward.

Read-after-write:
You don’t miss your own write.

They are related but different.

You can have monotonic reads without RYW.
You can have RYW without monotonic reads.

---

## 6️⃣ Where It Fits in the Consistency Spectrum

Strength comparison:

Strong Consistency (Linearizable)
⬇
Sequential Consistency
⬇
Snapshot Isolation
⬇
Read-After-Write
⬇
Monotonic Reads
⬇
Eventual Consistency

It strengthens eventual consistency but is far weaker than strong consistency.

---

## 7️⃣ How Systems Implement Read-After-Write

There are multiple strategies.

---

### 7.1 Sticky Sessions

After a write, route the client to the same replica.

Pros:

* Simple

Cons:

* Fails if replica crashes
* Hard in global systems
* Not ideal for load balancing

---

### 7.2 Version Tracking

When client writes:

* Server returns version/timestamp

On next read:

* Client says: “Do not give me data older than version X.”

Replica either:

* Waits
* Forwards to fresher replica
* Returns updated data

This is common in distributed databases.

---

### 7.3 Quorum Reads

If system uses quorum:

* Write to majority (W)
* Read from majority (R)
* Ensure R + W > N

This can guarantee read-after-write.

But this is closer to strong consistency.

---

### 7.4 Read from Leader

In leader-based systems:

* Writes go to leader
* Reads after write are served by leader

This ensures RYW.

---

## 8️⃣ Advantages

### 8.1 Improves UX

Users see their changes immediately.

### 8.2 Low Cost

Cheaper than strong consistency.

### 8.3 Good Enough for Many Systems

Social media, profiles, settings, dashboards.

---

## 9️⃣ Limitations

### 9.1 Per-Client Only

Other users may see stale data.

### 9.2 Does Not Prevent Anomalies

Lost update and write skew can still happen.

### 9.3 Requires Session Awareness

Stateless systems must track metadata.

---

## 🔟 Real-World Systems

Read-after-write consistency is common in:

* Dynamo-style systems (with session stickiness)
* Cassandra (tunable consistency)
* MongoDB with causal sessions
* CDN-backed apps
* User-facing web systems

Often provided as:

* “Session consistency”
* “Causal consistency” (which includes RYW)

---

## 1️⃣1️⃣ Common Interview Mistakes

### Mistake 1:

Equating RYW with strong consistency.

Wrong.
RYW is per-client.

Strong consistency is global.

---

### Mistake 2:

Thinking RYW prevents stale reads.

It prevents stale self-reads only.

---

### Mistake 3:

Ignoring implementation details.

Mentioning sticky sessions or version tracking shows depth.

---

## 1️⃣2️⃣ Interview-Ready Explanation

If asked:

“What is read-after-write consistency?”

Answer:

> Read-after-write consistency guarantees that once a client performs a write, any subsequent reads by that same client will reflect that write. It is a session-level guarantee commonly implemented in eventually consistent systems to prevent users from seeing stale versions of their own updates.

If senior-level, add:

> It does not guarantee global ordering or prevent anomalies like write skew; it only ensures self-consistency.

---

## 1️⃣3️⃣ Deeper Insight

Read-after-write is a minimal form of causality.

If:

* You wrote X,
* And then you read,

The read causally depends on your write.

RYW ensures that causal dependency is preserved for that client.

That’s why:

Causal consistency ⊃ Read-after-write
