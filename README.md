# Distributed Systems Revision Kit — System Design Kit

This repository is a compact system design kit for distributed systems concepts.

## Basic Concepts
- [Concept_Availability_vs_Reliability.md](Concept_Availability_vs_Reliability.md#L1) — tradeoffs between availability and reliability.
- [Concept_CAP_Theorem.md](Concept_CAP_Theorem.md#L1) — overview of consistency, availability, and partition tolerance tradeoffs.
- [Concept_Determinism.md](Concept_Determinism.md#L1) — determinism in distributed systems and why it matters.
- [Concept_Durability.md](Concept_Durability.md#L1) — storage durability guarantees and write-ahead logs.
- [Concept_Idempotency.md](Concept_Idempotency.md#L1) — idempotent operations and safe retry strategies.
- [Concept_Latency_vs_Throughput.md](Concept_Latency_vs_Throughput.md#L1) — tradeoffs between latency and throughput.
- [Concept_Linearizability.md](Concept_Linearizability.md#L1) — definition and implications of linearizable consistency.
- [Concept_Majority_Write-Read.md](Concept_Majority_Write-Read.md#L1) — quorum-based majority read/write strategies.
- [Concept_PACELC_Theorem.md](Concept_PACELC_Theorem.md#L1) — PACELC theorem and its guidance for system design.
- [Concept_Quorum.md](Concept_Quorum.md#L1) — quorum systems and failure tolerance calculations.
- [Concept_Write_Ahead_log.md](Concept_Write_Ahead_log.md#L1) — write-ahead logging for durability and crash recovery.

## Caching
- [Caching_Aside.md](Caching_Aside.md#L1) — supplementary caching considerations.
- [Caching_CDN.md](Caching_CDN.md#L1) — CDN patterns and edge caching.
- [Caching_Invalidation.md](Caching_Invalidation.md#L1) — cache invalidation strategies.
- [Caching_TTL.md](Caching_TTL.md#L1) — TTL and expiration policies.
- [Caching_Write_Back.md](Caching_Write_Back.md#L1) — write-back caching behavior.
- [Caching_Write_Through.md](Caching_Write_Through.md#L1) — write-through caching behavior.
- [Causal_Consistency.md](Causal_Consistency.md#L1) — explanation of causal ordering and guarantees.

## Consensus
- [Consensus_Gossip_Protocol.md](Consensus_Gossip_Protocol.md#L1) — gossip-based consensus patterns and behavior.
- [Consensus_Paxos.md](Consensus_Paxos.md#L1) — Paxos algorithm summary and roles.
- [Consensus_PBFT.md](Consensus_PBFT.md#L1) — Practical Byzantine Fault Tolerance overview.
- [Consensus_RAFT.md](Consensus_RAFT.md#L1) — Raft consensus algorithm and leader election.
- [Consensus_Zookeeper.md](Consensus_Zookeeper.md#L1) — Zookeeper's consensus and coordination primitives.
- [Consistency_Sequential.md](Consistency_Sequential.md#L1) — sequential consistency model.
- [Consistency_Strong_vs_Eventual.md](Consistency_Strong_vs_Eventual.md#L1) — comparison of strong and eventual consistency.

## Coordination
- [Coordination_2Phase_Commit.md](Coordination_2Phase_Commit.md#L1) — two-phase commit protocol for distributed transactions.
- [Coordination_3Phase_Commit.md](Coordination_3Phase_Commit.md#L1) — three-phase commit and its failure modes.
- [Coordination_Distributed_locks.md](Coordination_Distributed_locks.md#L1) — patterns for distributed locking.
- [Coordination_Fencing_Tokens.md](Coordination_Fencing_Tokens.md#L1) — fencing tokens to prevent split-brain and stale leaders.
- [Coordination_Leader_Election.md](Coordination_Leader_Election.md#L1) — leader election algorithms and considerations.
- [Coordination_Saga_Pattern.md](Coordination_Saga_Pattern.md#L1) — long-running transactions using sagas and compensating actions.
- [Coordination_XA_Transaction.md](Coordination_XA_Transaction.md#L1) — XA transactions and two-phase commit interoperability.

## Data Consistency

- [Concept_Map_DataConsistency.md](Concept_Map_DataConsistency.md)
- [Data_Consistency_Lost_Update.md](Data_Consistency_Lost_Update.md#L1) — lost update anomalies and mitigation.
- [Data_Consistency_Monotonic_Reads.md](Data_Consistency_Monotonic_Reads.md#L1) — monotonic read guarantees and enforcement.
- [Data_Consistency_MVCC.md](Data_Consistency_MVCC.md#L1) — multi-version concurrency control basics.
- [Data_Consistency_Read_After_Write.md](Data_Consistency_Read_After_Write.md#L1) — read-after-write guarantees and patterns.
- [Data_Consistency_Snapshot_Isolation.md](Data_Consistency_Snapshot_Isolation.md#L1) — snapshot isolation semantics.
- [Data_Consistency_Write_Skew.md](Data_Consistency_Write_Skew.md#L1) — write skew anomalies and prevention.
- [Data_Consistency_Causal_Consistency.md](Data_Consistency_Causal_Consistency.md#L1)
- [Data_Consistency_Compare_and_Swap.md](Data_Consistency_Compare_and_Swap.md#L1)
- [Data_Consistency_Linearizability.md](Data_Consistency_Linearizability.md#L1)
- [Data_Consistency_Read_Committed.md](Data_Consistency_Read_Committed.md#L1)
- [Data_Consistency_Serializable_Isolation.md](Data_Consistency_Serializable_Isolation.md#L1)
- [Data_Consistency_Sequential_Consistency.md](Data_Consistency_Sequential_Consistency.md#L1)

## Data Parititioning
- [Data_Partitioning_Consistent_hashing.md](Data_Partitioning_Consistent_hashing.md#L1) — consistent hashing and low-movement rebalancing.
- [Data_Partitioning_Hash_partitioning.md](Data_Partitioning_Hash_partitioning.md#L1) — hash partitioning for even distribution.
- [Data_Partitioning_Range_partitioning.md](Data_Partitioning_Range_partitioning.md#L1) — range-based partitions and locality.
- [Data_Partitioning_Rebalancing.md](Data_Partitioning_Rebalancing.md#L1) — rebalancing strategies and costs.
- [Data_Partitioning_Sharding.md](Data_Partitioning_Sharding.md#L1) — sharding fundamentals and routing.
- [Data_Partitioning_Virtual_nodes.md](Data_Partitioning_Virtual_nodes.md#L1) — virtual nodes (vnodes) for finer-grained balance.

## Failures
- [Failure_Byzantine.md](Failure_Byzantine.md#L1) — Byzantine faults and defensive strategies.
- [Failure_Crash.md](Failure_Crash.md#L1) — crash failure semantics and handling.
- [Failure_Network_Partition.md](Failure_Network_Partition.md#L1) — network partition behaviors and system responses.
- [Failure_Omission.md](Failure_Omission.md#L1) — omission failures (dropped messages) and mitigation.
- [Failure_Partial.md](Failure_Partial.md#L1) — partial failures and partial progress concerns.
- [Failure_Split_Brain.md](Failure_Split_Brain.md#L1) — split-brain scenarios and prevention techniques.

## Indexing
- [Indexing_B+_Tree.md](Indexing_B+_Tree.md#L1) — B+ tree indexing structure and uses.
- [Indexing_Compaction.md](Indexing_Compaction.md#L1) — compaction strategies for storage engines.
- [Indexing_LSM_Trees.md](Indexing_LSM_Trees.md#L1) — LSM tree design and write-optimized storage.
- [Indexing_SSTables.md](Indexing_SSTables.md#L1) — SSTables and immutable segment files.

## Load Balancing
- [LoadBalancing_Consistent_Hashing.md](LoadBalancing_Consistent_Hashing.md#L1) — consistent hashing in load balancers.
- [LoadBalancing_Health_Checks.md](LoadBalancing_Health_Checks.md#L1) — health check patterns for LB.
- [LoadBalancing_Least_Connections.md](LoadBalancing_Least_Connections.md#L1) — least-connections strategy.
- [LoadBalancing_RoundRobin.md](LoadBalancing_RoundRobin.md#L1) — round-robin distribution.
- [LoadBalancing_Sticky_Sessions.md](LoadBalancing_Sticky_Sessions.md#L1) — sticky session techniques and tradeoffs.

## Message Queus
- [Messaging_At_Least_Once.md](Messaging_At_Least_Once.md#L1) — at-least-once delivery semantics.
- [Messaging_At_Most_Once.md](Messaging_At_Most_Once.md#L1) — at-most-once delivery semantics.
- [Messaging_BackPressure.md](Messaging_BackPressure.md#L1) — backpressure mechanisms for messaging.
- [Messaging_Consumer_Groups.md](Messaging_Consumer_Groups.md#L1) — consumer group patterns (parallel consumption).
- [Messaging_Exactly_Once.md](Messaging_Exactly_Once.md#L1) — exactly-once delivery considerations.
- [Messaging_Idempotent_Producer.md](Messaging_Idempotent_Producer.md#L1) — idempotent producers and dedup.
- [Messaging_Kafka_Architecture.md](Messaging_Kafka_Architecture.md#L1) — Kafka architecture and components.
- [Messaging_Offsets.md](Messaging_Offsets.md#L1) — offset management and semantics.
- [Messaging_Partitioning_Strategy.md](Messaging_Partitioning_Strategy.md#L1) — partitioning strategies for topics.
- [Messaging_PubSub.md](Messaging_PubSub.md#L1) — publish/subscribe basics.
- [Messaging_Queue.md](Messaging_Queue.md#L1) — queue semantics and patterns.

## Ordering
- [Ordering_Causal.md](Ordering_Causal.md#L1) — causal ordering and vector/lamport clocks.
- [Ordering_Happens-Before_relation.md](Ordering_Happens-Before_relation.md#L1) — happens-before relation explanation.
- [Ordering_Total_vs_Partial.md](Ordering_Total_vs_Partial.md#L1) — total vs partial ordering tradeoffs.

## Rate Limiting
- [Rate_Limiting_Fixed_Window_Counter.md](Rate_Limiting_Fixed_Window_Counter.md#L1) — fixed-window rate limiting.
- [Rate_Limiting_Leaky_Bucket.md](Rate_Limiting_Leaky_Bucket.md#L1) — leaky-bucket algorithm for smoothing traffic.
- [Rate_Limiting_Sliding_Window_Log.md](Rate_Limiting_Sliding_Window_Log.md#L1) — sliding-window log approach.
- [Rate_Limiting_Token_Bucket.md](Rate_Limiting_Token_Bucket.md#L1) — token-bucket rate limiting.

## Reliability
- [Reliability_Patterns_BulkHead.md](Reliability_Patterns_BulkHead.md#L1) — bulkhead isolation pattern.
- [Reliability_Patterns_Circuit_Breaker.md](Reliability_Patterns_Circuit_Breaker.md#L1) — circuit breaker pattern.
- [Reliability_Patterns_Dead_Letter_Queue.md](Reliability_Patterns_Dead_Letter_Queue.md#L1) — DLQ for failed messages.
- [Reliability_Patterns_Graceful_Degradation.md](Reliability_Patterns_Graceful_Degradation.md#L1) — graceful degradation strategies.
- [Reliability_Patterns_Health_Checks.md](Reliability_Patterns_Health_Checks.md#L1) — health check best practices.
- [Reliability_Patterns_Observability.md](Reliability_Patterns_Observability.md#L1) — observability and monitoring patterns.
- [Reliability_Patterns_Retry_With_Backoff.md](Reliability_Patterns_Retry_With_Backoff.md#L1) — retries with exponential backoff.

## Replication
- [Replication_Anti_Entropy.md](Replication_Anti_Entropy.md#L1) — anti-entropy for eventual consistency.
- [Replication_Asynchronous.md](Replication_Asynchronous.md#L1) — asynchronous replication tradeoffs.
- [Replication_Hint_Handoff.md](Replication_Hint_Handoff.md#L1) — hint handoff technique for eventual replication.
- [Replication_Leaderless.md](Replication_Leaderless.md#L1) — leaderless replication patterns.
- [Replication_Leader_Based.md](Replication_Leader_Based.md#L1) — leader-based replication patterns and costs.
- [Replication_Log.md](Replication_Log.md#L1) — replication logs and ordering.
- [Replication_Multi_Leader.md](Replication_Multi_Leader.md#L1) — multi-leader replication and anti-entropy.
- [Replication_Read_Repair.md](Replication_Read_Repair.md#L1) — read-repair mechanisms.
- [Replication_Synchronous.md](Replication_Synchronous.md#L1) — synchronous replication guarantees.

## Security
- [Security_API_Gateway.md](Security_API_Gateway.md#L1) — API gateway security patterns.
- [Security_JWT.md](Security_JWT.md#L1) — JWT tokens and best practices.
- [Security_mTLS.md](Security_mTLS.md#L1) — mutual TLS for service-to-service auth.
- [Security_OAuth.md](Security_OAuth.md#L1) — OAuth flows for authentication.
- [Security_TLS.md](Security_TLS.md#L1) — TLS basics for transport security.
- [Security_Zero_Trust.md](Security_Zero_Trust.md#L1) — zero-trust security principles.

## Time
- [Time_Clock_Skew.md](Time_Clock_Skew.md#L1) — clock skew issues and mitigation (NTP, hybrid clocks).
- [Time_Hybrid_Logical_Clocks.md](Time_Hybrid_Logical_Clocks.md#L1) — hybrid logical clocks overview.
- [Time_Lamport_Clocks.md](Time_Lamport_Clocks.md#L1) — Lamport clocks and logical time.
- [Time_NTP.md](Time_NTP.md#L1) — NTP and synchronizing physical clocks.
- [Time_Physical_vs_Logical.md](Time_Physical_vs_Logical.md#L1) — comparing physical and logical clocks.
- [Time_Vector_Clocks.md](Time_Vector_Clocks.md#L1) — vector clocks for causal tracking.

