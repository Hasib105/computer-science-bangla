# সার্ভিস কমিউনিকেশন

## 🎯 Communication Types

```
Synchronous (সিনক্রোনাস):
Service A → Request → Service B
Service A ← Response ← Service B
(A অপেক্ষা করে)

Asynchronous (অ্যাসিনক্রোনাস):
Service A → Message → Queue
            ↓
        Service B (পরে প্রসেস করে)
(A অপেক্ষা করে না)
```

## 📊 Synchronous Patterns

### ১. REST (HTTP)
```
Most common approach।

Service A                    Service B
    │                            │
    │── GET /users/123 ─────────→│
    │                            │
    │←── 200 OK + User Data ─────│
    │                            │

Pros:
✓ Simple, well-understood
✓ Language agnostic
✓ Caching (GET requests)

Cons:
✗ Higher latency
✗ Tight coupling
```

### ২. gRPC
```
High-performance RPC framework।

Service A                    Service B
    │                            │
    │── GetUser(123) ───────────→│
    │   (Binary Protocol)        │
    │←── User Object ────────────│
    │                            │

Pros:
✓ Faster than REST (binary)
✓ Strong typing (Protocol Buffers)
✓ Streaming support
✓ Code generation

Cons:
✗ Learning curve
✗ Browser support limited
```

### ৩. GraphQL
```
Client specifies what data it needs।

Client → GraphQL Gateway → Multiple Services

Query:
{
  user(id: 123) {
    name
    orders {
      id
      total
    }
  }
}

Pros:
✓ No over-fetching
✓ Single endpoint
✓ Flexible queries

Cons:
✗ Complexity
✗ Caching harder
```

## 📨 Asynchronous Patterns

### ১. Message Queue
```
Fire and forget।

Order Service → Queue → Payment Service
                     → Inventory Service
                     → Email Service

Pros:
✓ Decoupled services
✓ Handles spikes
✓ Retry capability

Cons:
✗ Eventual consistency
✗ Debugging harder
```

### ২. Event-Driven
```
Events represent facts।

Order Service:
  emit("order.created", {order_id: 123})

Subscribers:
  - Payment Service (on order.created)
  - Inventory Service (on order.created)
  - Analytics Service (on order.created)
```

## ⚖️ Comparison

```
┌────────────────┬──────────────┬──────────────────────┐
│    Protocol    │   Latency    │      Use Case        │
├────────────────┼──────────────┼──────────────────────┤
│ REST           │ ~100ms       │ CRUD, Web APIs       │
│ gRPC           │ ~10ms        │ Internal services    │
│ GraphQL        │ ~50ms        │ Client-facing APIs   │
│ Message Queue  │ Async        │ Background tasks     │
│ Events         │ Async        │ Event sourcing       │
└────────────────┴──────────────┴──────────────────────┘
```

## 💡 Best Practices

```
✓ Sync for queries, async for commands
✓ Timeout এবং retry strategy
✓ Circuit breaker pattern
✓ Idempotent operations
✓ Service discovery
```

## 📚 পরবর্তী টপিক

[API Gateway →](./04-api-gateway.md)
