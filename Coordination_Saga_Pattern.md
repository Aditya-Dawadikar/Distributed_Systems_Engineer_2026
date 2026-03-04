# Saga Pattern

## 1. What the Saga Pattern Is

The **Saga Pattern** is a distributed transaction pattern used to maintain **data consistency across microservices without using distributed locks or 2PC**.

Instead of one atomic transaction across services, a saga breaks a transaction into **a sequence of smaller local transactions**.

Each step:

1. updates its local database
2. publishes an event
3. triggers the next step

If something fails, the system executes **compensating transactions** to undo previous steps.

Goal:

```text
Achieve eventual consistency across distributed services
```

Saga is widely used in **microservice architectures** where traditional distributed transactions are impractical.

---

# 2. Problem Saga Solves

In microservices, each service typically has its **own database**.

Example e-commerce system:

```text
Order Service
Payment Service
Inventory Service
Shipping Service
```

Suppose a customer places an order.

Steps:

```text
1 Create order
2 Charge payment
3 Reserve inventory
4 Ship product
```

If step 3 fails but step 2 succeeded:

```text
customer charged
inventory unavailable
```

This creates inconsistent system state.

Traditional solution:

```text
2PC
```

But 2PC is problematic in microservices because:

* high latency
* blocking
* tight coupling
* poor scalability

Saga solves this with **compensating transactions**.

---

# 3. Core Idea

A saga consists of:

```text
Sequence of local transactions
```

Each step:

```text
Ti = local transaction
Ci = compensating transaction
```

Example:

```text
T1 → create order
T2 → charge payment
T3 → reserve inventory
T4 → ship item
```

If `T3` fails:

```text
C2 → refund payment
C1 → cancel order
```

---

# 4. Concept Map

```text
Distributed Transactions
│
├── Two Phase Commit (2PC)
│      └── Strong consistency
│
├── Three Phase Commit (3PC)
│
└── Saga Pattern
       ├── Local transactions
       ├── Event driven workflow
       └── Compensating transactions
```

---

# 5. Saga Execution Flow

Example order workflow:

```text
Client
  │
  ▼
Order Service
  │
  ▼
Payment Service
  │
  ▼
Inventory Service
  │
  ▼
Shipping Service
```

Each step emits events that trigger the next service.

---

# 6. Two Saga Implementation Styles

There are **two major patterns**.

```text
1 Choreography Saga
2 Orchestration Saga
```

---

# 7. Choreography Saga

Services communicate through **events**.

No central controller.

Example flow:

```text
OrderCreated → PaymentService
PaymentCompleted → InventoryService
InventoryReserved → ShippingService
```

Each service listens to events and decides next action.

---

### Example Event Flow

```text
OrderCreated
      │
      ▼
PaymentProcessed
      │
      ▼
InventoryReserved
      │
      ▼
ShipmentCreated
```

If payment fails:

```text
PaymentFailed
      │
      ▼
OrderCancelled
```

---

### Advantages

* simple architecture
* highly decoupled services
* easy to extend

---

### Limitations

* hard to track flow
* complex event chains
* debugging difficulty

---

# 8. Orchestration Saga

A central **orchestrator** manages the workflow.

Architecture:

```text
Saga Orchestrator
│
├─ Order Service
├─ Payment Service
├─ Inventory Service
└─ Shipping Service
```

Flow:

```text
Orchestrator → CreateOrder
Orchestrator → ChargePayment
Orchestrator → ReserveInventory
Orchestrator → ShipItem
```

If failure occurs:

```text
Orchestrator triggers compensation
```

Example:

```text
RefundPayment
CancelOrder
```

---

### Advantages

* easier to understand
* centralized control
* simpler monitoring

---

### Limitations

* orchestrator becomes critical component
* less decentralized

---

# 9. Example Saga Execution

Order workflow.

### Step 1

```text
Create Order
```

Local DB transaction.

Event:

```text
OrderCreated
```

---

### Step 2

Payment service receives event.

```text
Charge Card
```

If success:

```text
PaymentCompleted
```

If failure:

```text
PaymentFailed
```

---

### Step 3

Inventory service reserves stock.

```text
InventoryReserved
```

---

### Step 4

Shipping service creates shipment.

```text
ShipmentCreated
```

---

# 10. Failure Example

Suppose inventory is unavailable.

Execution:

```text
OrderCreated
PaymentCompleted
InventoryFailed
```

System runs compensations:

```text
RefundPayment
CancelOrder
```

Final state:

```text
no order
no payment
```

Consistency restored.

---

# 11. Compensating Transactions

A compensating transaction reverses the effect of a previous step.

Examples:

| Step              | Compensating Action |
| ----------------- | ------------------- |
| Charge payment    | refund payment      |
| Reserve inventory | release inventory   |
| Create order      | cancel order        |

Important:

```text
compensation must be idempotent
```

Meaning:

```text
running twice produces same result
```

---

# 12. Message Flow Example

Choreography version.

```text
OrderService
   │
   ▼
Event: OrderCreated
   │
   ▼
PaymentService
   │
   ▼
Event: PaymentCompleted
   │
   ▼
InventoryService
   │
   ▼
Event: InventoryReserved
   │
   ▼
ShippingService
```

---

# 13. Real Systems Using Saga Pattern

Saga pattern is common in microservice architectures.

Examples include systems built with:

* Apache Kafka
* Apache Camel
* Axon Framework
* Temporal

These frameworks help coordinate saga workflows.

---

# 14. Advantages

### Avoids Distributed Locks

Each service performs **local transactions only**.

---

### High Scalability

No global transaction manager.

---

### Loose Coupling

Services communicate through events.

---

### Works Well With Event-Driven Systems

Fits naturally with message queues.

---

# 15. Limitations

### Eventual Consistency

System may temporarily be inconsistent.

---

### Complex Compensation Logic

Designing compensating transactions can be difficult.

---

### Difficult Debugging

Long event chains make failures harder to trace.

---

### Requires Careful Idempotency Handling

Duplicate messages may occur.

---

# 16. Saga vs 2PC

| Feature          | Saga                 | 2PC      |
| ---------------- | -------------------- | -------- |
| Consistency      | eventual             | strong   |
| Latency          | lower                | higher   |
| Coupling         | loose                | tight    |
| Failure handling | compensating actions | rollback |
| Scalability      | high                 | limited  |

---

# 17. Saga vs Consensus Protocols

| Feature     | Saga                      | Raft/Paxos           |
| ----------- | ------------------------- | -------------------- |
| Purpose     | transaction orchestration | replicated logs      |
| Consistency | eventual                  | strong               |
| Scope       | application level         | infrastructure level |

---

# 18. Relationship With Other Concepts

```text
Distributed Systems
│
├── Consensus
│     ├── Paxos
│     ├── Raft
│     └── PBFT
│
├── Distributed Transactions
│     ├── 2PC
│     ├── 3PC
│     └── Saga Pattern
│
└── Event Driven Architecture
```

---

# 19. Interview Talking Points

**What problem does Saga solve?**

Managing distributed transactions across microservices without blocking protocols like 2PC.

---

**What are compensating transactions?**

Operations that undo previous steps when a failure occurs.

---

**What are the two types of Saga?**

Choreography and Orchestration.

---

**When should you use Saga instead of 2PC?**

When building scalable microservices where distributed locking is impractical.

---

# 20. Example Interview Answer

If asked:

> "How would you implement distributed transactions across microservices?"

A strong answer:

```text
Use Saga pattern with event-driven architecture.
Each service performs local transaction and publishes events.
If a step fails, compensating transactions restore consistency.
```
