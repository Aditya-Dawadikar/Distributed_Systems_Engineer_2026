# Strong Consistency vs Eventual Consistency

This comparison is fundamental to distributed systems. Almost every system design interview will force you to choose between them.

---

# 1️⃣ What Is “Strong Consistency”?

⚠️ First correction:
“Strong consistency” is not a formally precise term.

In interviews, when someone says “strong consistency”, they usually mean:

> Linearizability (also called atomic consistency).

So for clarity:

* Strong consistency ≈ Linearizability (in most interview contexts)

---

## What Is Linearizability?

A system is linearizable if:

1. All operations appear to occur in a single global order.
2. That order respects real-time ordering.
3. Reads always see the most recent completed write.

If write W finishes before read R starts, R must see W (or a later write).

---

## What Problem Strong Consistency Solves

In distributed systems:

* Replicas may lag.
* Writes propagate asynchronously.
* Network delays cause inconsistency.

Strong consistency guarantees:

* No stale reads.
* Single-copy illusion.
* Predictable behavior.

You can think of the system as one single correct copy.

---

## Example

User updates password.
Immediately logs in.
Under strong consistency:

* Login must use the new password.

There is no window where old data is visible.

---

## How Strong Consistency Is Achieved

It requires coordination.

Common mechanisms:

* Consensus (Raft, Paxos)
* Synchronous replication
* Majority quorum (read + write quorum rules)
* Leader-based replication

All of these introduce:

* Latency
* Reduced availability under partitions

You pay for coordination.

---

## Advantages of Strong Consistency

### 1. Simpler reasoning

No edge-case state.

### 2. Correctness-critical systems

Payments, inventory, banking.

### 3. Prevents anomalies

No lost updates, no stale reads.

---

## Limitations

### 1. Higher latency

Must coordinate replicas.

### 2. Lower availability (CAP theorem)

Under network partition:
You must choose consistency over availability.

### 3. Lower throughput

Coordination bottlenecks.

---

# 2️⃣ What Is Eventual Consistency?

Eventual consistency is much weaker.

Definition:

> If no new updates are made to a data item, eventually all replicas will converge to the same value.

Key insight:

It does NOT guarantee:

* Immediate consistency
* Read-after-write consistency
* Monotonic reads
* Real-time order

It only guarantees convergence.

---

## What Problem It Solves

Strong consistency does not scale globally.

Eventual consistency allows:

* Low latency
* High availability
* Partition tolerance
* Massive horizontal scale

It avoids coordination on the critical path.

---

## Example

User posts a tweet.

In one region:
You see it immediately.

In another region:
It appears 500ms later.

That’s acceptable in social media.

Not acceptable in banking.

---

## How Eventual Consistency Is Achieved

Common techniques:

* Asynchronous replication
* Leaderless replication (Dynamo style)
* Gossip protocols
* Conflict resolution (last-write-wins, CRDTs)
* Read repair
* Anti-entropy mechanisms

Writes don’t wait for all replicas.

Replication happens later.

---

## Advantages of Eventual Consistency

### 1. Low latency

Writes complete quickly.

### 2. High availability

System continues under partition.

### 3. Massive scalability

Works well in global systems.

---

## Limitations

### 1. Stale reads

User may see old data.

### 2. Write conflicts

Concurrent writes may clash.

### 3. Complex reasoning

Developers must handle anomalies.

---

# 3️⃣ Core Differences

| Property                     | Strong Consistency | Eventual Consistency |
| ---------------------------- | ------------------ | -------------------- |
| Global ordering              | Yes                | Not required         |
| Real-time order preserved    | Yes                | No                   |
| Stale reads possible         | No                 | Yes                  |
| Coordination required        | Yes                | No (on write path)   |
| Latency                      | Higher             | Lower                |
| Availability under partition | Lower              | Higher               |
| Scalability                  | Moderate           | Very high            |

---

# 4️⃣ CAP Theorem Context

Under partition (P):

You must choose:

* C (strong consistency)
* A (availability)

Strong consistency → CP system
Eventual consistency → AP system

Examples:

* ZooKeeper → CP
* Cassandra (default) → AP
* Dynamo → AP
* Spanner → CP

---

# 5️⃣ Important Subtleties (Interview Depth)

### Eventual consistency does NOT mean “randomly inconsistent”

It still guarantees:

* Convergence
* Deterministic conflict resolution (usually)

---

### Strong consistency does NOT mean “no replication”

It means:
Replication is coordinated.

---

### Eventual consistency can be tuned

In systems like Cassandra:

* R + W > N gives strong consistency (quorum)
* R = 1, W = 1 gives eventual

Consistency is sometimes configurable.

---

# 6️⃣ Real-World Applications

## Strong Consistency Needed For:

* Financial transactions
* Inventory management
* Authentication systems
* Configuration systems

## Eventual Consistency Works For:

* Social feeds
* Caching layers
* Analytics
* Logging systems
* CDN edge content

---

# 7️⃣ Common Interview Mistakes

### Mistake 1:

“Eventual consistency means data will be wrong sometimes.”

Wrong.
It means temporarily inconsistent, but eventually convergent.

---

### Mistake 2:

“Strong consistency always means single server.”

Wrong.
It can be replicated using consensus.

---

### Mistake 3:

Not mentioning tradeoffs.

If you don’t talk about latency, availability, and coordination cost, your answer is shallow.

---

# 8️⃣ How To Answer in Interview (Clean Framing)

If asked:

“What’s the difference?”

A good answer:

> Strong consistency guarantees that reads always return the most recent write, respecting real-time ordering, typically requiring coordination like consensus. Eventual consistency allows temporary divergence across replicas but guarantees that replicas will converge if no new updates occur, enabling lower latency and higher availability.

Then add:

> The choice depends on whether correctness or availability/latency is more critical.

That shows maturity.

---

# 9️⃣ Deeper Insight

Strong consistency is expensive because:

Coordination requires:

* Majority agreement
* Network round trips
* Failure handling

Eventual consistency is cheap because:

No coordination on critical path.

That is the core tradeoff.

---

# Recommended Reading

* CAP Theorem — Eric Brewer
* Designing Data-Intensive Applications — Chapter on Replication
* Dynamo Paper (Amazon)
* Spanner Paper (Google)
