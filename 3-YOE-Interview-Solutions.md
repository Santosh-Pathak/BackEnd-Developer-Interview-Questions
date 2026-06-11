# ✅ Solutions — Day 4: 3 Years Experience (3 YOE)

> **Series:** Backend Interview Prep · Fresher → 10 Years of Experience  
> **Level:** Mid-Level Developer — 3 Years of Experience  
> **Questions File:** [day-04-3yoe.md](./day-04-3yoe.md)  
> **Total Answers:** 60  

> 💡 **Tip:** At 3 YOE, interviewers expect you to understand distributed systems trade-offs, know when to use which tool, and have opinions backed by production experience.

---

## 📚 Table of Contents

1. [Microservices Architecture](#1-microservices-architecture)
2. [Message Brokers & Event-Driven Systems](#2-message-brokers--event-driven-systems)
3. [CI/CD Pipelines](#3-cicd-pipelines)
4. [Kubernetes Basics](#4-kubernetes-basics)
5. [Advanced Database Concepts](#5-advanced-database-concepts)
6. [Distributed Systems Fundamentals](#6-distributed-systems-fundamentals)
7. [Observability — Logs, Metrics, Traces](#7-observability--logs-metrics-traces)
8. [Advanced API Design](#8-advanced-api-design)
9. [Cloud Fundamentals](#9-cloud-fundamentals)
10. [Engineering Practices & Team Collaboration](#10-engineering-practices--team-collaboration)

---

## 1. Microservices Architecture

---

**Q1. What is a microservices architecture? How does it differ from a monolith?**

**Answer:**

A **monolith** packages all features (auth, orders, payments, notifications) into one deployable unit. A **microservices architecture** splits the application into small, independently deployable services, each owning a specific business capability and its own database.

```
Monolith:
┌────────────────────────────────────────────────────────┐
│  UserModule  │  OrderModule  │  PaymentModule  │  NotifyModule  │
│                  Single Process, Single Deploy                   │
│                  Single Database                                 │
└────────────────────────────────────────────────────────┘

Microservices:
┌────────────┐  ┌────────────┐  ┌─────────────┐  ┌──────────────┐
│  User Svc  │  │  Order Svc │  │ Payment Svc │  │  Notify Svc  │
│  :3001     │  │  :3002     │  │  :3003      │  │  :3004       │
│  users DB  │  │  orders DB │  │payments DB  │  │ notify DB    │
└────────────┘  └────────────┘  └─────────────┘  └──────────────┘
  (Node.js)       (Go)           (Java)             (Python)
```

**Key differences:**

| | Monolith | Microservices |
|--|----------|--------------|
| Deployment | All-or-nothing | Per-service, independent |
| Scaling | Scale entire app | Scale only bottleneck service |
| Tech stack | One language/framework | Polyglot per service |
| Team structure | One big team | Small teams own each service |
| Failure isolation | One bug can crash everything | Failure isolated to one service |
| Complexity | Simple initially | High operational complexity |
| Data | Single shared DB | DB per service |
| Testing | Easier (everything local) | Integration testing is harder |

> 📖 Reference: [Microservices — Martin Fowler](https://martinfowler.com/articles/microservices.html)

---

**Q2. What are the key benefits and drawbacks of microservices?**

**Answer:**

**Benefits:**

```
1. Independent deployability
   → Team A deploys Order Service 10 times/day without waiting for Team B
   → Smaller deploys = smaller blast radius

2. Independent scalability
   → Only the Order Service is under load during Black Friday?
   → Scale just Order Service from 3 pods to 50 pods
   → Don't waste money scaling the rarely-used Notification Service

3. Technology heterogeneity
   → Use Go for the high-throughput Image Service
   → Use Python for the ML Recommendation Service
   → Use Node.js for the real-time Chat Service
   → Best tool for each job

4. Fault isolation
   → Notification Service crashes → Orders still work
   → Monolith: one bug in notifications crashes everything

5. Team autonomy
   → Each team owns a service end-to-end: design, build, deploy, on-call
   → Faster delivery, less coordination overhead
```

**Drawbacks:**

```js
// 1. Network overhead and latency
// Monolith: function call = nanoseconds
getUserOrders(userId); // in-process call

// Microservices: HTTP/gRPC call = milliseconds + failure risk
const orders = await fetch('http://order-service/users/${userId}/orders');
// Network can fail, timeout, be slow

// 2. Distributed data management (no joins across services!)
// ❌ Can't do this in microservices:
SELECT u.name, o.total FROM users u JOIN orders o ON u.id = o.user_id;
// Users are in User Service DB, Orders in Order Service DB

// ✅ Must aggregate in application code (slower, more complex):
const user  = await userService.getUser(userId);
const orders = await orderService.getOrdersByUser(userId);
const result = { ...user, orders };

// 3. Distributed tracing complexity
// A single user request spans 5 services — which one caused the bug?
// Need: correlation IDs, distributed tracing (Jaeger, Zipkin)

// 4. Operational overhead
// 1 monolith → 1 deployment, 1 log stream, 1 metric dashboard
// 20 microservices → 20 deployments, 20 log streams, 20 dashboards
// Need: service mesh, centralized logging, distributed tracing, API gateway

// 5. Testing complexity
// Unit tests: easy
// Integration tests: need all 5 services running locally → Docker Compose hell
// Contract tests: need Pact or similar framework

// 6. "Microservices tax" — overhead before you write business code:
// - Set up service template (Dockerfile, CI/CD, health check, logging, tracing)
// - Configure service discovery
// - Set up API gateway route
// - Configure authentication
// → New monolith feature: days. New microservice: weeks.
```

> 📖 Reference: [Microservices Trade-offs — Martin Fowler](https://martinfowler.com/articles/microservice-trade-offs.html)

---

**Q3. What is service discovery? What is the difference between client-side and server-side discovery?**

**Answer:**

In microservices, service instances start and stop dynamically (scaling, failures, deployments). **Service discovery** lets services find each other's current network addresses without hardcoding IPs.

**Client-Side Discovery:**
The client queries a service registry directly, gets the list of available instances, and picks one using a load-balancing algorithm.

```
Client → Service Registry (Consul/Eureka) → "Order Service is at 10.0.1.5:3002"
Client → Directly calls 10.0.1.5:3002
```

```js
// Client-side discovery with Consul
const consul = require('consul')();

async function getOrderServiceUrl() {
  const services = await consul.health.service({
    service: 'order-service',
    passing: true   // only healthy instances
  });

  // Client picks one (round-robin, random, least-connections)
  const instance = services[Math.floor(Math.random() * services.length)];
  return `http://${instance.Service.Address}:${instance.Service.Port}`;
}

// Register this service on startup
await consul.agent.service.register({
  name: 'user-service',
  address: process.env.POD_IP,
  port: 3001,
  check: { http: 'http://localhost:3001/health', interval: '10s' }
});
```

**Server-Side Discovery:**
The client sends the request to a load balancer (or API gateway). The load balancer queries the registry and forwards to the right instance. Client is oblivious.

```
Client → Load Balancer/API Gateway → Service Registry
                                    → "Order Service" → 10.0.1.5:3002
                                    → Forwards request
```

```
# Kubernetes uses server-side discovery natively:
# Service DNS: order-service.default.svc.cluster.local
# kube-proxy handles routing to the correct pod

# Client just calls:
fetch('http://order-service/orders')
# Kubernetes resolves → routes to a healthy pod automatically
```

| | Client-Side | Server-Side |
|--|------------|------------|
| Discovery logic | In the client | In the load balancer |
| Client complexity | High (registry SDK needed) | Low (just call LB URL) |
| Language support | Must implement per language | Language-agnostic |
| Examples | Netflix Eureka + Ribbon | AWS ALB, Kubernetes Services, NGINX |

> 📖 Reference: [Service Discovery — NGINX](https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/)

---

**Q4. What is an API gateway? What responsibilities does it typically handle?**

**Answer:**

An **API gateway** is the single entry point for all client traffic to your microservices. Instead of clients knowing about dozens of internal services, they talk only to the gateway.

```
Clients → API Gateway → User Service
                     → Order Service
                     → Payment Service
                     → Product Service
```

**Responsibilities:**

```js
// A real API gateway (Express + custom middleware) handles:

// 1. Request routing — forward to the right service
app.use('/api/users',    proxy('http://user-service:3001'));
app.use('/api/orders',   proxy('http://order-service:3002'));
app.use('/api/payments', proxy('http://payment-service:3003'));

// 2. Authentication — verify JWT before any service sees the request
app.use(async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Unauthorized' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch { res.status(401).json({ error: 'Invalid token' }); }
});
// Services don't need to implement auth — gateway handles it ✅

// 3. Rate limiting
app.use(rateLimit({ windowMs: 60000, max: 100 }));

// 4. Request/response transformation
app.use('/api/v2/orders', (req, res, next) => {
  // Transform v2 request format to v1 that the service understands
  req.body = transformV2ToV1(req.body);
  next();
});

// 5. SSL termination (handle HTTPS at gateway, forward HTTP internally)

// 6. Logging & correlation IDs
app.use((req, res, next) => {
  req.correlationId = req.headers['x-correlation-id'] || crypto.randomUUID();
  res.setHeader('X-Correlation-ID', req.correlationId);
  logger.info('Request', { method: req.method, path: req.path, correlationId: req.correlationId });
  next();
});

// 7. Load balancing between service instances

// 8. Circuit breaking — stop sending to unhealthy services

// Popular gateways: Kong, AWS API Gateway, NGINX, Traefik, Envoy
```

> 📖 Reference: [API Gateway Pattern — Microsoft](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway)

---

**Q5. How do microservices communicate? What is the difference between synchronous and asynchronous inter-service communication?**

**Answer:**

**Synchronous (request/response):** Caller waits for the response before continuing.

```js
// HTTP/REST — simple, widely used
const response = await fetch('http://order-service/orders/42');
const order = await response.json();
// Caller WAITS here — if order-service is slow, caller is slow too

// gRPC — binary, typed, faster for internal services
const order = await orderServiceClient.getOrder({ id: '42' });
// Still synchronous — caller waits

// Problem: Cascading failures
// If Order Service calls Payment Service calls Fraud Service:
// Fraud Service slow → Payment Service slow → Order Service slow → User sees slowness
// Solution: Circuit breaker, timeouts
```

**Asynchronous (event/message-based):** Caller sends a message and continues without waiting.

```js
// Kafka / RabbitMQ — fire and forget
await kafka.producer().send({
  topic: 'order-events',
  messages: [{ value: JSON.stringify({ type: 'ORDER_PLACED', orderId: 42 }) }]
});
// Caller continues immediately — doesn't wait for consumers to process ✅

// Benefits:
// - Decoupling: Order Service doesn't need to know about Notification Service
// - Resilience: Notification Service down? Messages queue up, processed when it recovers
// - Scalability: Consumers scale independently

// Drawbacks:
// - Eventual consistency: effects happen "later", not immediately
// - Harder to debug: no synchronous call stack
// - Requires message broker infrastructure
```

**Decision guide:**

| Use Synchronous when | Use Asynchronous when |
|---------------------|----------------------|
| Response needed immediately (user is waiting) | Work can happen later (background processing) |
| Query for data | Notify other services of events |
| Simple request-response | Fan-out to multiple consumers |
| Read operations | Long-running operations |
| Example: "Get user profile" | Example: "User signed up → send email + update analytics + assign trial" |

> 📖 Reference: [Microservice Communication — Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/communication-in-microservice-architecture)

---

**Q6. What is the Circuit Breaker pattern? How does it prevent cascading failures?**

**Answer:**

The **Circuit Breaker** wraps calls to a failing service. After a threshold of failures, it "opens" (stops sending requests) for a cooldown period, then "half-opens" to test recovery.

**Three states:**

```
CLOSED (normal):     Requests pass through normally, failures counted
OPEN (tripped):      Requests immediately fail-fast (no network call made)
                     Protects downstream service from more load
HALF-OPEN (testing): Let a few requests through to test if service recovered
                     Success → CLOSED, Failure → OPEN again
```

**Real implementation:**

```js
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.failureThreshold  = options.failureThreshold  || 5;   // open after 5 failures
    this.successThreshold  = options.successThreshold  || 2;   // close after 2 successes
    this.timeout           = options.timeout           || 10000; // 10s cooldown
    this.state             = 'CLOSED';
    this.failureCount      = 0;
    this.successCount      = 0;
    this.nextAttempt       = Date.now();
  }

  async call(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        // Fail immediately — don't even try
        throw new Error(`Circuit OPEN — service unavailable. Retry after ${new Date(this.nextAttempt).toISOString()}`);
      }
      // Cooldown elapsed → try half-open
      this.state = 'HALF_OPEN';
      console.log('Circuit HALF-OPEN — testing service recovery');
    }

    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (err) {
      this.onFailure();
      throw err;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    if (this.state === 'HALF_OPEN') {
      this.successCount++;
      if (this.successCount >= this.successThreshold) {
        this.state = 'CLOSED';
        this.successCount = 0;
        console.log('Circuit CLOSED — service recovered ✅');
      }
    }
  }

  onFailure() {
    this.failureCount++;
    this.successCount = 0;
    if (this.state === 'HALF_OPEN' || this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.log(`Circuit OPEN — ${this.failureCount} failures. Cooling down for ${this.timeout/1000}s`);
    }
  }
}

// Usage: wrap the external call
const paymentBreaker = new CircuitBreaker(
  (orderId, amount) => paymentService.charge(orderId, amount),
  { failureThreshold: 5, timeout: 15000 }
);

app.post('/orders/:id/pay', async (req, res) => {
  try {
    const result = await paymentBreaker.call(req.params.id, req.body.amount);
    res.json({ success: true, result });
  } catch (err) {
    if (err.message.includes('Circuit OPEN')) {
      // Graceful degradation: queue the payment for later retry
      await paymentQueue.add('retry-payment', { orderId: req.params.id, amount: req.body.amount });
      res.json({ success: true, status: 'payment_queued', message: 'Payment will be processed shortly' });
    } else {
      res.status(500).json({ error: err.message });
    }
  }
});

// Popular libraries: opossum (Node.js), resilience4j (Java), polly (.NET)
const { CircuitBreaker: Opossum } = require('opossum');
const breaker = new Opossum(paymentService.charge, {
  timeout:           3000,  // 3s timeout per call
  errorThresholdPercentage: 50,   // open if >50% fail
  resetTimeout:      30000  // try again after 30s
});
```

> 📖 Reference: [Circuit Breaker — Martin Fowler](https://martinfowler.com/bliki/CircuitBreaker.html)

---

**Q7. What is the Saga pattern? Compare orchestration vs choreography.**

**Answer:**

When a business transaction spans multiple services (each with its own DB), traditional ACID transactions don't work. A **Saga** is a sequence of local transactions where each step publishes an event or message to trigger the next — with **compensating transactions** to undo completed steps on failure.

**Example: Place order saga**
```
1. Order Service:    Create order (PENDING)
2. Inventory Service: Reserve items
3. Payment Service:  Charge card
4. Shipping Service: Schedule delivery
5. Order Service:    Mark order CONFIRMED

If step 3 (Payment) fails:
→ Compensate step 2: Release reserved inventory
→ Compensate step 1: Cancel/reject order
```

**Orchestration (Conductor model):**
A central **orchestrator** coordinates all steps and knows the full flow.

```js
// Orchestrator: Order Orchestrator Service
class PlaceOrderSaga {
  async execute(orderData) {
    const sagaId = crypto.randomUUID();

    try {
      // Step 1: Create order
      const order = await orderService.createOrder({ ...orderData, sagaId });

      // Step 2: Reserve inventory
      const reservation = await inventoryService.reserve({
        items: orderData.items, sagaId
      });

      // Step 3: Charge payment
      const payment = await paymentService.charge({
        userId: orderData.userId,
        amount: order.total,
        sagaId
      });

      // Step 4: Schedule shipping
      await shippingService.schedule({ orderId: order.id, sagaId });

      // All succeeded!
      await orderService.confirm(order.id);
      return { success: true, orderId: order.id };

    } catch (err) {
      // Compensate in reverse order
      await this.compensate(sagaId, err);
      throw err;
    }
  }

  async compensate(sagaId, failedAt) {
    // Undo steps that succeeded before the failure
    if (failedAt.step >= 3) await inventoryService.releaseReservation(sagaId);
    if (failedAt.step >= 1) await orderService.cancelOrder(sagaId);
  }
}
```

**Choreography (Event-driven model):**
No central orchestrator. Each service listens to events, reacts, and publishes its own events.

```js
// Each service listens to and emits events independently

// Order Service: publishes ORDER_CREATED
eventBus.publish('ORDER_CREATED', { orderId, items, userId, total });

// Inventory Service: listens to ORDER_CREATED, publishes INVENTORY_RESERVED or INVENTORY_FAILED
eventBus.subscribe('ORDER_CREATED', async ({ orderId, items }) => {
  try {
    await reserveItems(items);
    eventBus.publish('INVENTORY_RESERVED', { orderId });
  } catch {
    eventBus.publish('INVENTORY_FAILED', { orderId });
  }
});

// Payment Service: listens to INVENTORY_RESERVED
eventBus.subscribe('INVENTORY_RESERVED', async ({ orderId }) => {
  try {
    const order = await getOrder(orderId);
    await chargeCard(order.userId, order.total);
    eventBus.publish('PAYMENT_COMPLETED', { orderId });
  } catch {
    eventBus.publish('PAYMENT_FAILED', { orderId });
    // Compensate: release inventory
    eventBus.publish('RELEASE_INVENTORY', { orderId });
  }
});
```

| | Orchestration | Choreography |
|--|--------------|-------------|
| Coordination | Central orchestrator knows all steps | Each service knows only its own step |
| Visibility | Easy to see full flow in one place | Flow is implicit — spread across services |
| Coupling | Services coupled to orchestrator | Services only coupled to events |
| Debugging | Easier — one place to check state | Harder — trace events across services |
| Single point of failure | Orchestrator can become bottleneck | No single bottleneck |
| Best for | Complex workflows, strict ordering | Loose workflows, many consumers |

> 📖 Reference: [Saga Pattern — Microsoft](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)

---

**Q8. What is the Outbox pattern? How does it solve the dual-write problem?**

**Answer:**

The **dual-write problem**: when you write to the DB AND publish a message, one can succeed while the other fails — leaving systems out of sync.

```js
// ❌ Dual-write problem
await db.query('INSERT INTO orders ...'); // succeeds
await kafka.publish('ORDER_CREATED', ...); // FAILS (Kafka down)
// DB has the order but nobody knows it was created
// OR:
await kafka.publish('ORDER_CREATED', ...); // succeeds
await db.query('INSERT INTO orders ...'); // FAILS
// Event published but order doesn't exist in DB!
```

The **Outbox pattern** fixes this by writing to an outbox table in the SAME database transaction as the main write. A separate process then reliably publishes outbox records to the message broker.

```js
// ✅ Outbox pattern — atomic write to DB + outbox in one transaction

// Step 1: Write order AND outbox entry in ONE transaction
app.post('/orders', async (req, res) => {
  await db.query('BEGIN');
  try {
    // Create the order
    const order = await db.query(
      'INSERT INTO orders (user_id, total, status) VALUES ($1, $2, $3) RETURNING *',
      [req.body.userId, req.body.total, 'pending']
    );

    // Write to outbox in the SAME transaction (atomic!)
    await db.query(
      `INSERT INTO outbox (aggregate_type, aggregate_id, event_type, payload, created_at)
       VALUES ($1, $2, $3, $4, NOW())`,
      ['Order', order.rows[0].id, 'ORDER_CREATED', JSON.stringify({
        orderId: order.rows[0].id,
        userId:  req.body.userId,
        total:   req.body.total,
        items:   req.body.items
      })]
    );

    await db.query('COMMIT');
    // If Kafka is down: order exists in DB, outbox record is there, will be published later ✅
    // If DB fails: ROLLBACK — neither order nor outbox record exists ✅
    res.status(201).json(order.rows[0]);
  } catch (err) {
    await db.query('ROLLBACK');
    throw err;
  }
});

// Step 2: Outbox relay — reads outbox and publishes to Kafka (separate process)
async function outboxRelay() {
  while (true) {
    // Fetch unpublished events
    const events = await db.query(
      `SELECT * FROM outbox WHERE published_at IS NULL ORDER BY created_at ASC LIMIT 100`
    );

    for (const event of events.rows) {
      try {
        await kafka.producer().send({
          topic: `${event.aggregate_type.toLowerCase()}-events`,
          messages: [{
            key:   event.aggregate_id.toString(),
            value: event.payload
          }]
        });

        // Mark as published
        await db.query(
          'UPDATE outbox SET published_at = NOW() WHERE id = $1',
          [event.id]
        );
      } catch (err) {
        console.error('Failed to publish event', event.id, err.message);
        // Will retry on next iteration
      }
    }

    await sleep(1000); // poll every second
  }
}

// Step 3: Use Debezium for production (CDC-based outbox relay)
// Debezium watches the outbox table in PostgreSQL WAL
// Automatically publishes new rows to Kafka
// Zero-polling, near-real-time, highly reliable
```

**Outbox table schema:**

```sql
CREATE TABLE outbox (
  id             BIGSERIAL PRIMARY KEY,
  aggregate_type VARCHAR(255) NOT NULL,  -- 'Order', 'User', etc.
  aggregate_id   VARCHAR(255) NOT NULL,  -- the entity's ID
  event_type     VARCHAR(255) NOT NULL,  -- 'ORDER_CREATED', 'USER_REGISTERED'
  payload        JSONB        NOT NULL,  -- event data
  created_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
  published_at   TIMESTAMPTZ  NULL       -- NULL = not yet published
);
CREATE INDEX idx_outbox_unpublished ON outbox(created_at) WHERE published_at IS NULL;
```

> 📖 Reference: [Outbox Pattern — Debezium](https://debezium.io/blog/2019/02/19/reliable-microservices-data-exchange-with-the-outbox-pattern/)

---

**Q9. What is a distributed transaction? Why is it hard in microservices?**

**Answer:**

A **distributed transaction** is a transaction that spans multiple services/databases and must maintain ACID properties across all of them — either all steps commit or all roll back.

**Why it's hard:**

```
Traditional ACID (single DB):
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- atomic, guaranteed

Distributed (Order + Payment service with separate DBs):
Order Service DB:   INSERT INTO orders ...
Payment Service DB: INSERT INTO payments ...
               ↑                    ↑
    Separate DBs, separate processes, separate networks
    No single transaction coordinator!

Problems:
1. Partial failure: Order DB write succeeds, Payment DB fails
   → Order exists, payment doesn't → inconsistent state!

2. Network partitions: Payment service unreachable mid-transaction
   → Did the payment happen? Did it not?

3. Two-Phase Commit (2PC) solution exists but:
   - Requires a coordinator that knows about all participants
   - If coordinator crashes mid-commit → all participants BLOCK waiting
   - Performance bottleneck: all participants must hold locks until coordinator confirms
   - Not supported natively by most microservice databases

4. Tight coupling: services need to know about each other's transaction protocol
```

**What to do instead of distributed transactions:**

```js
// Option 1: Saga pattern (see Q7) — compensating transactions
// Accept eventual consistency; undo completed steps on failure

// Option 2: Design to avoid cross-service transactions
// Restructure your domain: maybe Order + Payment belong in the SAME service

// Option 3: Outbox + idempotency (see Q8)
// Write locally, publish events, handle duplicates at consumers

// Option 4: "Eventual consistency is acceptable"
// For most use cases, 50ms of inconsistency doesn't matter
// Bank account balance can show $0 briefly before payment confirms
// This is actually what all real payment systems (Stripe, PayPal) do

// The fundamental insight:
// Distributed systems can't have ACID across network boundaries without massive coordination cost
// Accept eventual consistency + design compensations for failure cases
```

> 📖 Reference: [Distributed Transactions — Chris Richardson](https://microservices.io/patterns/data/saga.html)

---

**Q10. What is the Sidecar pattern? Give a practical example.**

**Answer:**

The **Sidecar** deploys a helper container alongside the main application container in the same pod/host. The sidecar augments or extends the main container's functionality without modifying its code.

```
Pod:
┌──────────────────────────────────────────┐
│  Main Container (Node.js API)             │
│  :3000 → business logic                   │
│                                           │
│  Sidecar Container (Envoy proxy)          │
│  :15001 → handles TLS, metrics, tracing  │
│                                           │
│  Shared: network namespace, volumes       │
└──────────────────────────────────────────┘
```

**Real-world examples:**

```yaml
# Example 1: Log shipping sidecar
# Main app writes logs to a shared volume
# Sidecar (Fluentd) reads and ships to Elasticsearch
apiVersion: v1
kind: Pod
metadata:
  name: my-api
spec:
  containers:
  - name: api                          # Main container
    image: my-api:1.0
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  - name: log-shipper                  # Sidecar
    image: fluent/fluentd:v1.16
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app          # reads same logs
    env:
    - name: ELASTICSEARCH_HOST
      value: "elasticsearch:9200"

  volumes:
  - name: log-volume
    emptyDir: {}                        # shared between main + sidecar

---
# Example 2: Service mesh sidecar (Istio/Envoy)
# Envoy proxy is injected as a sidecar to every pod
# Handles: mTLS, retries, circuit breaking, distributed tracing
# The Node.js app has ZERO knowledge of this — it's transparent

# Traffic flow:
# Incoming: Network → Envoy (sidecar) → Node.js app
# Outgoing: Node.js app → Envoy (sidecar) → Network

# Node.js app code (unaware of proxy):
fetch('http://order-service/orders');
# Envoy intercepts this, adds tracing headers, handles mTLS, retries on 503

---
# Example 3: Config reloader sidecar
# Main app reads config from a file
# Sidecar watches Consul/Vault and updates the config file when it changes
```

**Benefits of the Sidecar:**
- Cross-cutting concerns (logging, security, tracing) handled without modifying app code.
- Works with any language — sidecar is language-agnostic.
- Update sidecar independently (e.g., update Envoy version without touching your app).

> 📖 Reference: [Sidecar Pattern — Microsoft](https://learn.microsoft.com/en-us/azure/architecture/patterns/sidecar)

---

## 2. Message Brokers & Event-Driven Systems

---

**Q11. What is Apache Kafka? How does it differ from a traditional message queue like RabbitMQ?**

**Answer:**

**Apache Kafka** is a distributed, fault-tolerant event streaming platform. Unlike traditional queues, Kafka persists messages on disk for configurable retention periods and allows multiple consumers to independently replay the event log.

**Core architectural difference:**

```
RabbitMQ (Traditional Queue):
Producer → [Queue] → Consumer (message deleted after ACK)
                  → Consumer 2 gets DIFFERENT message (round-robin)
"Messages are consumed once and deleted"

Kafka (Event Log):
Producer → [Topic/Partition: append-only log]
         offset: 0,1,2,3,4,5,6,7,...
Consumer Group A reads from offset 5 →
Consumer Group B ALSO reads from offset 0 → (independent)
Consumer Group C reads from offset 3 →
"Events are retained — multiple consumers read independently"
```

**Key differences:**

| | RabbitMQ | Kafka |
|--|----------|-------|
| Model | Push (broker pushes to consumer) | Pull (consumer polls broker) |
| Message retention | Deleted after ACK | Retained (default 7 days, configurable) |
| Ordering | Per-queue ordering | Per-partition ordering |
| Multiple consumers | Competing (one message, one consumer) | Independent (all consumer groups read all messages) |
| Throughput | ~50K msg/sec | ~1M+ msg/sec |
| Replay | ❌ No — once consumed, gone | ✅ Yes — rewind offset to re-process |
| Use case | Task queues, RPC, complex routing | Event streaming, audit log, data pipelines |

```js
// RabbitMQ: Order processing (competing consumers)
// One order → one worker processes it
ch.consume('orders', (msg) => {
  processOrder(JSON.parse(msg.content));
  ch.ack(msg); // message gone
});

// Kafka: Order event (multiple independent consumers)
// One ORDER_PLACED event → billing reads it → analytics reads it → email reads it
// All independently, at their own pace, can replay from the start
const consumer1 = kafka.consumer({ groupId: 'billing-service' });
const consumer2 = kafka.consumer({ groupId: 'analytics-service' });
const consumer3 = kafka.consumer({ groupId: 'email-service' });

await consumer1.subscribe({ topic: 'order-events' });
// Billing processes from offset 0 at billing's speed
// Analytics processes from offset 0 at analytics' speed — completely independent
```

> 📖 Reference: [Kafka vs RabbitMQ — Confluent](https://www.confluent.io/blog/kafka-vs-rabbitmq/)

---

**Q12. What are Kafka topics, partitions, and consumer groups? How do they enable scalability?**

**Answer:**

```
Topic: Logical channel for a category of events
       e.g., "order-events", "user-events", "payment-events"

Partition: A topic is split into N ordered, immutable logs
       Partitions enable parallelism:
       - Producer writes are distributed across partitions
       - Multiple consumers can read in parallel (1 consumer per partition)
       - More partitions = higher throughput

Consumer Group: A set of consumers that jointly consume a topic
       Kafka guarantees each partition is read by exactly ONE consumer in a group
       → Load balancing within the group
```

**Visual:**

```
Topic "order-events" with 3 partitions:

Partition 0: [msg0, msg3, msg6, msg9...]   ← Consumer Instance A
Partition 1: [msg1, msg4, msg7, msg10...]  ← Consumer Instance B
Partition 2: [msg2, msg5, msg8, msg11...]  ← Consumer Instance C

Consumer Group "order-processor" has 3 instances → each reads 1 partition
→ 3x parallelism ✅

If you add a 4th consumer instance → one instance is idle (more consumers than partitions)
If you have only 1 consumer → it reads all 3 partitions sequentially
```

**Scalability in practice:**

```js
const { Kafka } = require('kafkajs');
const kafka = new Kafka({ brokers: ['kafka1:9092', 'kafka2:9092', 'kafka3:9092'] });

// Producer: use partition key for ordering
const producer = kafka.producer();
await producer.connect();
await producer.send({
  topic: 'order-events',
  messages: [{
    key:   order.userId.toString(), // same user → same partition → ordered events per user
    value: JSON.stringify({ type: 'ORDER_PLACED', orderId: order.id, userId: order.userId })
  }]
});

// Consumer: 3 instances in same group → automatic partition assignment
const consumer = kafka.consumer({ groupId: 'order-processor' });
await consumer.connect();
await consumer.subscribe({ topic: 'order-events', fromBeginning: false });

await consumer.run({
  eachMessage: async ({ topic, partition, message }) => {
    const event = JSON.parse(message.value.toString());
    console.log(`Processing from partition ${partition}:`, event.type);
    await processOrderEvent(event);
  }
});

// Scale: run 3 instances of this consumer process
// Each automatically gets assigned 1 partition (with 3-partition topic)
// → 3x throughput without any code changes
```

> 📖 Reference: [Kafka Core Concepts — Confluent](https://developer.confluent.io/courses/apache-kafka/events/)

---

**Q13. What is the difference between a queue and a pub/sub model?**

**Answer:**

| | Queue (Point-to-Point) | Pub/Sub |
|--|----------------------|--------|
| Consumers | One consumer per message | ALL subscribers receive the message |
| Use case | Task distribution, work queues | Event broadcasting, notifications |
| Coupling | Consumer knows the queue | Publisher doesn't know who subscribes |
| Retention | Message deleted after consumption | Message can be retained (Kafka) or ephemeral (Redis Pub/Sub) |

```
Queue:
Producer → [Queue] → Consumer A gets msg1
                   → Consumer B gets msg2
                   → Consumer A gets msg3  (load balanced)

Pub/Sub:
Publisher → [Topic] → Subscriber A receives msg ← SAME message
                    → Subscriber B receives msg ←
                    → Subscriber C receives msg ←
```

**Code examples:**

```js
// ── Queue (RabbitMQ — competing consumers) ────────────────────────────
// Use case: distribute video encoding jobs
// Each job processed by exactly one worker

// Producer:
ch.sendToQueue('video-encoding', Buffer.from(JSON.stringify({ videoId: 42, format: 'mp4' })));

// Workers (3 instances, each gets different jobs):
ch.consume('video-encoding', (msg) => {
  const job = JSON.parse(msg.content.toString());
  encodeVideo(job.videoId, job.format);
  ch.ack(msg); // message removed from queue
});

// ── Pub/Sub (Redis) — all subscribers get the message ─────────────────
// Use case: real-time notifications to ALL connected clients

// Publisher: order service publishes to ALL notification handlers
await redis.publish('notifications:user:42',
  JSON.stringify({ type: 'ORDER_SHIPPED', orderId: 99 })
);

// Subscribers: multiple handlers interested in user 42's events
const sub1 = redis.duplicate();
await sub1.subscribe('notifications:user:42');
sub1.on('message', (ch, msg) => sendWebSocketNotification(JSON.parse(msg)));

const sub2 = redis.duplicate();
await sub2.subscribe('notifications:user:42');
sub2.on('message', (ch, msg) => logToAnalytics(JSON.parse(msg)));

// Both sub1 AND sub2 receive the same message

// ── Kafka as durable Pub/Sub ─────────────────────────────────────────
// ORDER_PLACED → billing-service consumer group reads it
// ORDER_PLACED → analytics-service consumer group ALSO reads it
// ORDER_PLACED → email-service consumer group ALSO reads it
// Each group independently, can replay from beginning
// This is Kafka's killer feature over traditional pub/sub
```

> 📖 Reference: [Queue vs Pub/Sub — Google Cloud](https://cloud.google.com/pubsub/docs/overview)

---

**Q14. What is an event-driven architecture? What are its advantages and challenges?**

**Answer:**

**Event-Driven Architecture (EDA)** is a design pattern where components communicate by producing and consuming **events** — records of things that happened — rather than calling each other directly.

```
Traditional (request-driven):
Order Service → calls → Inventory Service (direct HTTP)
Order Service → calls → Notification Service (direct HTTP)
Order Service → calls → Analytics Service (direct HTTP)
→ Tight coupling: Order Service must know all 3 services

Event-Driven:
Order Service → publishes → ORDER_PLACED event
                     ↓
              [Event Bus (Kafka)]
                     ↓           ↓           ↓
          Inventory Service  Notification  Analytics
          (subscribes to     (subscribes)  (subscribes)
          ORDER_PLACED)
→ Loose coupling: Order Service doesn't know who listens
```

**Advantages:**

```js
// 1. Loose coupling — producer doesn't know consumers
// Add new consumer without changing producer:
// Week 1: Order Service publishes ORDER_PLACED
// Week 3: Add Fraud Detection Service — subscribes to ORDER_PLACED
// → Zero changes to Order Service ✅

// 2. Scalability — consumers scale independently
// Analytics processes at its own pace, doesn't slow down Orders

// 3. Resilience — consumer downtime doesn't affect producer
// Notification Service is down for 2 hours
// → Events queue up in Kafka
// → When Notification Service recovers, it processes backlog
// → Order Service never knew there was a problem ✅

// 4. Audit log — events are a natural audit trail
// Replay all ORDER_PLACED events from January → rebuild analytics from scratch
// Debug a bug: "what exactly happened to order 42?" → replay events
```

**Challenges:**

```js
// 1. Eventual consistency — effects happen asynchronously
// User places order → "Order confirmed!" shown immediately
// Inventory update happens 100ms later (after event processed)
// "Is inventory actually reserved?" → not immediately guaranteed

// 2. Complex error handling — what if consumer fails?
// Order placed → event published → Inventory Service crashes mid-processing
// Item appears available but order is created → oversell!
// Solution: idempotency + dead letter queues + saga compensations

// 3. Event schema evolution — hard to change event structure
// ORDER_PLACED v1: { orderId, userId, total }
// ORDER_PLACED v2: { orderId, userId, total, currency } ← added field
// Consumers processing v1 events will break on v2 structure
// Solution: schema registry (Confluent Schema Registry + Avro)

// 4. Distributed debugging — "what happened to this request?"
// A request triggers events → events trigger more events → chain of async events
// No synchronous call stack to trace
// Solution: correlation IDs, distributed tracing (Jaeger, Zipkin, OpenTelemetry)

// 5. Ordering guarantees — hard to guarantee global order
// ORDER_PLACED then ORDER_CANCELLED — must process in that order!
// Solution: use Kafka partition key to keep related events ordered
await producer.send({
  topic: 'order-events',
  messages: [{ key: orderId.toString(), value: event }] // same orderId → same partition
});
```

> 📖 Reference: [Event-Driven Architecture — AWS](https://aws.amazon.com/event-driven-architecture/)

---

**Q15. What is event sourcing? How is it different from storing just the current state?**

**Answer:**

**Traditional state storage:** Store only the current value. History is lost.

```sql
-- Current state: just the latest values
UPDATE orders SET status = 'shipped', updated_at = NOW() WHERE id = 42;
-- All we know: order is currently "shipped". What happened before? Unknown.
```

**Event Sourcing:** Never update — only append events. Current state is derived by replaying events.

```js
// ── Event Store (append-only) ────────────────────────────────────────
// Events table: id, aggregate_id, event_type, payload, timestamp

// Order lifecycle events:
// 1. ORDER_CREATED   { userId: 1, items: [...], total: 150 }
// 2. PAYMENT_PROCESSED { amount: 150, transactionId: 'pi_xyz' }
// 3. ORDER_CONFIRMED {}
// 4. SHIPMENT_CREATED  { trackingNumber: 'UPS123' }
// 5. ORDER_DELIVERED   { deliveredAt: '2024-06-01T14:30:00' }

// Rebuild current state by replaying events:
async function getOrder(orderId) {
  const events = await db.query(
    'SELECT * FROM events WHERE aggregate_id = $1 ORDER BY timestamp ASC',
    [orderId]
  );

  return events.rows.reduce((state, event) => {
    switch (event.event_type) {
      case 'ORDER_CREATED':
        return { ...state, ...event.payload, status: 'created' };
      case 'PAYMENT_PROCESSED':
        return { ...state, paymentId: event.payload.transactionId, status: 'paid' };
      case 'ORDER_CONFIRMED':
        return { ...state, status: 'confirmed' };
      case 'SHIPMENT_CREATED':
        return { ...state, trackingNumber: event.payload.trackingNumber, status: 'shipped' };
      case 'ORDER_DELIVERED':
        return { ...state, deliveredAt: event.payload.deliveredAt, status: 'delivered' };
      default: return state;
    }
  }, {});
}

// Store new event:
async function applyEvent(orderId, eventType, payload) {
  await db.query(
    'INSERT INTO events (aggregate_id, event_type, payload, timestamp) VALUES ($1, $2, $3, NOW())',
    [orderId, eventType, JSON.stringify(payload)]
  );
  // Optionally update a snapshot for performance (don't replay 1000 events every time)
}
```

**Benefits of Event Sourcing:**

```js
// 1. Complete audit trail — answer "what happened?" with full history
// Financial regulation: "show me every state change to account 42 in March"

// 2. Time travel — reconstruct state at any point in time
const orderStateAtMarch1st = replay(events.filter(e => e.timestamp <= '2024-03-01'));

// 3. Bug recovery — replay events with fixed logic to get correct state
// "Our shipping logic had a bug that miscalculated costs. Replay with fixed code."

// 4. Event-driven integration — events are first-class, can feed Kafka directly
// Every event written to event store → published to Kafka → other services react

// 5. CQRS natural fit — separate write model (events) from read model (projections)
```

**Drawbacks:**
- Querying "current state" requires replaying (use snapshots/projections for performance).
- Event schema changes are hard (can't alter past events).
- Conceptually complex for teams used to CRUD.

> 📖 Reference: [Event Sourcing — Martin Fowler](https://martinfowler.com/eaaDev/EventSourcing.html)

---

**Q16. What is CQRS (Command Query Responsibility Segregation)? When is it a good fit?**

**Answer:**

**CQRS** separates **write operations (Commands)** from **read operations (Queries)** into distinct models, often with separate data stores optimized for each purpose.

```
Traditional CRUD (same model for read + write):
PUT /orders/42    → same Order object → same DB
GET /orders?userId=1 → same Order object → same DB
(one model does everything)

CQRS:
Write side (Commands):
  POST /orders   → OrderCommand model → Write DB (normalized, ACID)
  PUT /orders/42 → OrderCommand model → Write DB

Read side (Queries):
  GET /orders?userId=1 → OrderQuery model → Read DB (denormalized, optimized for reads)
  GET /dashboard       → DashboardQuery   → Read DB (pre-aggregated)
```

**Implementation example:**

```js
// ── Write side: command handler ────────────────────────────────────────
class PlaceOrderCommand {
  constructor({ userId, items, shippingAddress }) {
    this.userId = userId;
    this.items = items;
    this.shippingAddress = shippingAddress;
  }
}

class PlaceOrderHandler {
  async handle(command) {
    // Validate, enforce business rules
    const user = await userRepo.findById(command.userId);
    if (!user.isActive) throw new Error('Inactive user cannot place orders');

    // Write to normalized write DB
    const order = await orderWriteRepo.create({
      userId: command.userId,
      items: command.items,
      total: calculateTotal(command.items),
      status: 'pending'
    });

    // Publish event for read side to update its projection
    await eventBus.publish('ORDER_PLACED', {
      orderId:  order.id,
      userId:   command.userId,
      items:    command.items,
      total:    order.total,
      placedAt: new Date()
    });

    return order.id;
  }
}

// ── Read side: query handler (pre-computed projection) ─────────────────
// Read DB has a denormalized orders table with user name embedded
// Updated asynchronously when ORDER_PLACED event arrives

// Event handler updates read model
eventBus.subscribe('ORDER_PLACED', async (event) => {
  const user = await userReadRepo.findById(event.userId); // pre-fetched

  // Upsert into read-optimized "order_summaries" table
  await orderReadRepo.upsert({
    orderId:      event.orderId,
    userId:       event.userId,
    userName:     user.name,        // denormalized — no JOIN needed at query time
    userEmail:    user.email,       // denormalized
    total:        event.total,
    itemCount:    event.items.length,
    status:       'pending',
    placedAt:     event.placedAt
  });
});

// Query: ultra-fast — no JOINs, pre-computed
class GetUserOrdersQuery {
  async execute({ userId, page = 1, limit = 20 }) {
    return orderReadDB.query(
      `SELECT order_id, user_name, total, item_count, status, placed_at
       FROM order_summaries
       WHERE user_id = $1
       ORDER BY placed_at DESC
       LIMIT $2 OFFSET $3`,
      [userId, limit, (page - 1) * limit]
    );
    // No JOINs, denormalized, indexed → extremely fast ✅
  }
}
```

**When CQRS is a good fit:**
- Read and write workloads have very different scaling needs.
- Complex domain with many business rules on writes.
- Need different data shapes for reads vs writes (dashboard vs edit form).
- Event sourcing (natural CQRS companion).

**When NOT to use it:**
- Simple CRUD apps — CQRS adds significant complexity.
- Small teams — maintaining two models is expensive.
- Data must be immediately consistent — CQRS has eventual consistency between write and read models.

> 📖 Reference: [CQRS — Martin Fowler](https://martinfowler.com/bliki/CQRS.html)

---

**Q17. What is Kafka consumer lag? How do you monitor and fix it?**

**Answer:**

**Consumer lag** is the difference between the latest message in a Kafka partition (the "end offset") and the offset your consumer has processed. Lag = how far behind your consumer is.

```
Partition 0:
Produced messages: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]  ← end offset: 9
Consumer processed: [0, 1, 2, 3, 4, 5]               ← committed offset: 5
LAG = 9 - 5 = 4 messages behind ⚠️
```

**Monitoring:**

```bash
# Kafka built-in CLI
kafka-consumer-groups.sh \
  --bootstrap-server kafka:9092 \
  --describe \
  --group order-processor

# Output:
# TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG  CONSUMER-ID
# order-events  0          1250            1350            100  consumer-1
# order-events  1          890             890             0    consumer-2
# order-events  2          1100            1500            400  consumer-3 ⚠️

# Prometheus metric: kafka_consumer_group_lag
# Alert if lag > threshold (e.g., > 1000 messages or > 5 minutes of lag)
```

```js
// Monitor lag programmatically (KafkaJS)
const admin = kafka.admin();
await admin.connect();

const offsets = await admin.fetchOffsets({
  groupId: 'order-processor',
  topics: ['order-events']
});

const topicOffsets = await admin.fetchTopicOffsets('order-events');

for (const partitionOffset of offsets.topics[0].partitions) {
  const { partition, offset: committedOffset } = partitionOffset;
  const endOffset = topicOffsets.find(t => t.partition === partition).offset;
  const lag = parseInt(endOffset) - parseInt(committedOffset);

  if (lag > 1000) {
    await alertOncall(`High consumer lag on partition ${partition}: ${lag} messages`);
  }
}
```

**Fixing consumer lag:**

```js
// Cause 1: Consumer too slow (processing takes too long)
// Fix: Increase concurrency within consumer
const consumer = kafka.consumer({ groupId: 'order-processor' });
await consumer.run({
  partitionsConsumedConcurrently: 3,  // process 3 partitions concurrently
  eachMessage: async ({ message }) => {
    await processOrderEvent(JSON.parse(message.value.toString()));
  }
});

// Cause 2: Not enough consumer instances (more partitions than consumers)
// Fix: Scale up consumer instances
// If topic has 6 partitions and you have 2 consumers → add 4 more
// Each consumer reads 1 partition → 6x parallelism ✅

// Cause 3: Message processing is CPU-bound
// Fix: Batch process instead of one-by-one
await consumer.run({
  eachBatch: async ({ batch }) => {
    const messages = batch.messages.map(m => JSON.parse(m.value.toString()));
    await db.query('INSERT INTO events VALUES ...', [messages]); // bulk insert
    // 1 DB call for 100 messages vs 100 DB calls ✅
  }
});

// Cause 4: Topic has too few partitions → can't parallelize enough
// Fix: Increase partition count (requires rebalancing)
await admin.createTopics({
  topics: [{ topic: 'order-events-v2', numPartitions: 12 }] // was 3
});
```

> 📖 Reference: [Consumer Lag — Confluent](https://docs.confluent.io/platform/current/kafka/monitoring.html)

---

## 3. CI/CD Pipelines

---

**Q18. What is Continuous Integration (CI)? What does a good CI pipeline include?**

**Answer:**

**Continuous Integration (CI)** is the practice of frequently merging developer code changes into a shared repository where automated builds and tests run on every push — catching integration issues early.

**What a good CI pipeline includes:**

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  ci:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env: { POSTGRES_DB: test, POSTGRES_USER: test, POSTGRES_PASSWORD: test }
        ports: ['5432:5432']
      redis:
        image: redis:7
        ports: ['6379:6379']

    steps:
      # 1. Checkout
      - uses: actions/checkout@v4

      # 2. Setup environment
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }

      # 3. Install dependencies (reproducible)
      - run: npm ci

      # 4. Lint (code style, formatting)
      - run: npm run lint
      # Fails if: console.log left in, unused imports, style violations

      # 5. Type check (TypeScript)
      - run: npm run typecheck
      # Catches type errors before runtime

      # 6. Unit tests (fast, isolated)
      - run: npm run test:unit -- --coverage
      # Must pass, coverage > 80%

      # 7. Integration tests (real DB, real Redis)
      - run: npm run test:integration
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/test
          REDIS_URL:    redis://localhost:6379

      # 8. Security audit
      - run: npm audit --audit-level=high
      # Fails if HIGH or CRITICAL vulnerabilities found

      # 9. Build (compile TypeScript, bundle)
      - run: npm run build
      # Catches compile errors

      # 10. Docker build (catches Dockerfile issues)
      - run: docker build -t my-app:${{ github.sha }} .

      # 11. Push (only on main branch merges)
      - if: github.ref == 'refs/heads/main'
        run: |
          docker push registry.io/my-app:${{ github.sha }}
          docker push registry.io/my-app:latest
```

**Golden rule of CI:** The pipeline should be fast (< 10 minutes) and reliable. Flaky tests erode trust in CI.

> 📖 Reference: [CI — Martin Fowler](https://martinfowler.com/articles/continuousIntegration.html)

---

**Q19. What is Continuous Delivery vs Continuous Deployment? What is the difference?**

**Answer:**

```
Continuous Integration (CI):
  Code → Automated tests → Build artifact
  Always: every push is tested automatically

Continuous Delivery (CD):
  CI + Automated release to staging
  Artifact is READY to deploy to production
  Manual approval required for production deploy
  "We COULD deploy anytime, but a human clicks the button"

Continuous Deployment (CD):
  CI + Automatic deploy to PRODUCTION
  Every passing commit goes live automatically
  No human approval — fully automated
  "If tests pass, it's live"
```

```
           Push     CI passes    Manual approval   Deploy to prod
Dev ──────► CI ───►  ✅        ──────────────────► Production
                                 ^
                                 |
                      Continuous Delivery stops here (human decides)

           Push     CI passes                      Auto deploy to prod
Dev ──────► CI ───►  ✅  ────────────────────────► Production
                                                    ^
                                                    |
                         Continuous Deployment (fully automated)
```

**In practice:**

```yaml
# Continuous Delivery: manual production approval gate
jobs:
  deploy-staging:
    needs: ci
    runs-on: ubuntu-latest
    steps:
      - run: kubectl set image deployment/api api=my-app:${{ github.sha }} -n staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://api.myapp.com
    # 'environment: production' requires manual approval in GitHub ✅
    steps:
      - run: kubectl set image deployment/api api=my-app:${{ github.sha }} -n production

# Continuous Deployment: no approval gate, auto-deploy to prod
  deploy-production:
    needs: [ci, deploy-staging, run-smoke-tests]
    if: github.ref == 'refs/heads/main'
    # No environment approval required — deploys automatically
    steps:
      - run: kubectl set image deployment/api api=my-app:${{ github.sha }} -n production
```

**Which to choose:**
- **Continuous Delivery:** regulated industries (finance, healthcare), complex deployments, when manual QA is required.
- **Continuous Deployment:** high-confidence test suite, small/frequent changes, teams that do multiple deploys per day (Netflix, Amazon).

> 📖 Reference: [CD vs CD — Atlassian](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)

---

**Q20. What is a blue-green deployment? What are its advantages?**

**Answer:**

**Blue-green deployment** maintains two identical production environments (blue = current live, green = new version). Traffic is switched from blue to green all at once when the new version is ready.

```
Normal state:
Load Balancer → Blue (v1.0) ← 100% traffic
                Green (idle)

Deploying v1.1:
1. Deploy v1.1 to Green (blue still serves all traffic)
2. Run smoke tests on Green
3. Switch LB: Blue → Green (instant traffic switch)
Load Balancer → Blue (v1.0) ← 0% traffic (kept for rollback)
                Green (v1.1) ← 100% traffic

If problem detected:
4. Rollback: switch LB back to Blue (instant!)
Load Balancer → Blue (v1.0) ← 100% traffic
                Green (v1.1) ← 0% (can redeploy)
```

**Implementation:**

```bash
# Kubernetes blue-green deployment

# Blue deployment (current production)
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-blue
spec:
  replicas: 5
  selector:
    matchLabels: { app: api, version: blue }
  template:
    metadata:
      labels: { app: api, version: blue }
    spec:
      containers:
      - name: api
        image: my-app:1.0.0
EOF

# Service pointing to blue
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
    version: blue    # ← points to blue
  ports: [{ port: 80, targetPort: 3000 }]
EOF

# Deploy new version to green (blue still serves traffic)
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-green
spec:
  replicas: 5
  template:
    metadata:
      labels: { app: api, version: green }
    spec:
      containers:
      - name: api
        image: my-app:1.1.0   # ← new version
EOF

# Run smoke tests against green directly
kubectl port-forward deployment/api-green 3001:3000
curl http://localhost:3001/health

# Switch traffic to green (zero downtime!)
kubectl patch service api-service -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback if needed (instant!)
kubectl patch service api-service -p '{"spec":{"selector":{"version":"blue"}}}'
```

**Advantages:**
- Zero-downtime deployment (traffic switch is instant).
- Instant rollback (just switch back to blue).
- Green environment fully tested before receiving traffic.
- Reduces deployment risk significantly.

**Disadvantage:** Requires 2x infrastructure cost (two full environments running simultaneously).

> 📖 Reference: [Blue-Green Deployment — Martin Fowler](https://martinfowler.com/bliki/BlueGreenDeployment.html)

---

**Q21. What is a canary deployment? How does it reduce deployment risk?**

**Answer:**

**Canary deployment** gradually rolls out a new version to a small percentage of users first. If metrics look healthy, the rollout continues. If not, roll back — affecting only the small canary group.

```
Phase 1: 5% of traffic → v1.1 (canary)
         95% of traffic → v1.0 (stable)
         Monitor: error rate, latency, business metrics

Phase 2: 25% → v1.1, 75% → v1.0 (if healthy)
Phase 3: 50% → v1.1, 50% → v1.0
Phase 4: 100% → v1.1 (full rollout complete)
         Decommission v1.0

Rollback at any phase: set canary weight back to 0% instantly
```

**Kubernetes + Argo Rollouts:**

```yaml
# argo-rollout.yaml — automated canary with metrics analysis
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: api-rollout
spec:
  replicas: 10
  strategy:
    canary:
      steps:
      - setWeight: 5          # 5% of traffic to canary
      - pause: { duration: 5m }  # wait 5 minutes
      - analysis:             # automated health check
          templates:
          - templateName: success-rate
          args:
          - name: service-name
            value: api-canary
      - setWeight: 25         # if analysis passes, increase to 25%
      - pause: { duration: 10m }
      - setWeight: 50
      - pause: { duration: 10m }
      - setWeight: 100        # full rollout
  template:
    spec:
      containers:
      - name: api
        image: my-app:{{.NewVersion}}
```

```yaml
# Analysis template — auto-rollback if error rate spikes
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  metrics:
  - name: success-rate
    interval: 1m
    successCondition: result[0] >= 0.95    # 95% success rate required
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus:9090
        query: |
          sum(rate(http_requests_total{job="api", status!~"5.."}[1m]))
          /
          sum(rate(http_requests_total{job="api"}[1m]))
# If success rate drops below 95%, auto-rollback triggers!
```

**Canary vs Blue-Green:**
- Canary: gradual rollout, real user traffic validates, smaller blast radius.
- Blue-Green: instant full switch, full parallel environment, easier to manage.

> 📖 Reference: [Canary Deployment — Martin Fowler](https://martinfowler.com/bliki/CanaryRelease.html)

---

**Q22. What is a feature flag (feature toggle)? How does it decouple deployment from release?**

**Answer:**

A **feature flag** (toggle) is a conditional that controls whether a feature is active — independently of whether the code is deployed. You deploy code with the flag OFF, then turn it ON when ready.

```
Without feature flags:
Code ready → must wait for "release" meeting → deploy + release together
"Big bang" release: all or nothing, no gradual rollout

With feature flags:
Code ready → deploy (flag OFF) → production has new code but feature is invisible
→ Turn flag ON for 5% of users → monitor
→ Turn ON for 50% → full release
→ If problem: turn flag OFF instantly (no redeploy needed!)
```

**Types of flags:**

```js
const { LaunchDarkly } = require('@launchdarkly/node-server-sdk');
const ldClient = LaunchDarkly.init(process.env.LAUNCHDARKLY_SDK_KEY);

// 1. Release toggle — control feature rollout
app.get('/dashboard', requireAuth, async (req, res) => {
  const useNewDashboard = await ldClient.variation(
    'new-dashboard-v2',
    { key: req.user.id, email: req.user.email },
    false  // default (off)
  );

  if (useNewDashboard) {
    return res.json(await newDashboardService.getData(req.user.id));
  } else {
    return res.json(await legacyDashboardService.getData(req.user.id));
  }
});

// 2. Ops toggle — emergency kill switch
app.post('/payments', requireAuth, async (req, res) => {
  const paymentsEnabled = await ldClient.variation('payments-enabled', {}, true);

  if (!paymentsEnabled) {
    return res.status(503).json({
      error: 'Payments temporarily unavailable. Please try again later.'
    });
  }

  // Process payment...
});

// 3. Experiment toggle — A/B testing
app.get('/checkout', requireAuth, async (req, res) => {
  const checkoutVariant = await ldClient.variation(
    'checkout-experiment',
    { key: req.user.id },
    'control'   // 'control', 'variant-a', 'variant-b'
  );

  const checkout = await checkoutService.getCheckout(req.user.id, checkoutVariant);
  res.json({ ...checkout, variant: checkoutVariant }); // log variant for analytics
});

// 4. Permission toggle — specific users/accounts
const betaAccess = await ldClient.variation('beta-feature', {
  key: req.user.id,
  custom: { plan: req.user.plan, company: req.user.companyId }
}, false);
// Can target: enterprise users, specific companies, internal employees, beta testers
```

**Simple in-house implementation:**

```js
// Redis-backed feature flags (no external dependency)
class FeatureFlags {
  async isEnabled(flagName, userId = null) {
    const flag = await redis.hgetall(`flag:${flagName}`);
    if (!flag) return false;
    if (flag.enabled === 'false') return false;
    if (flag.percentage) {
      // Hash user ID to consistent percentage (0-100)
      const hash = parseInt(crypto.createHash('md5').update(`${flagName}:${userId}`).digest('hex').slice(0, 8), 16);
      return (hash % 100) < parseInt(flag.percentage);
    }
    return true;
  }

  async setFlag(name, { enabled, percentage }) {
    await redis.hmset(`flag:${name}`, { enabled: enabled.toString(), percentage: percentage || 100 });
  }
}
```

> 📖 Reference: [Feature Toggles — Martin Fowler](https://martinfowler.com/articles/feature-toggles.html)

---

**Q23. What is Infrastructure as Code (IaC)? Name two tools used for it.**

**Answer:**

**Infrastructure as Code (IaC)** means defining and managing infrastructure (servers, databases, networks, load balancers) using code files — not manual clicks in a UI. Infrastructure becomes versioned, reproducible, and automated.

**Without IaC:**
- Click through AWS console to create an RDS instance.
- Write down the settings somewhere.
- Hope someone remembers what was configured when you need to recreate it.
- Manually repeat for staging, production, disaster recovery.

**With IaC:**
```bash
# One command to create identical infrastructure in any environment
terraform apply -var="env=production"
# Creates: VPC, subnets, RDS, Redis, ECS cluster, load balancer, security groups
# Idempotent: run again = no change (already exists)
# Version control: track every infrastructure change in git
```

**Tool 1: Terraform (HCL — declarative)**

```hcl
# infrastructure/rds.tf
resource "aws_db_instance" "main" {
  identifier        = "myapp-${var.environment}"
  engine            = "postgres"
  engine_version    = "15.4"
  instance_class    = var.environment == "production" ? "db.r6g.xlarge" : "db.t3.medium"
  allocated_storage = 100
  db_name           = "myapp"
  username          = "app"
  password          = var.db_password   # from secrets

  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name

  backup_retention_period = 7
  deletion_protection     = var.environment == "production"

  tags = { Environment = var.environment, ManagedBy = "terraform" }
}

output "db_endpoint" {
  value = aws_db_instance.main.endpoint
}
```

```bash
terraform plan    # preview changes
terraform apply   # apply changes
terraform destroy # tear down (use with caution!)
```

**Tool 2: AWS CDK (Python/TypeScript — imperative)**

```typescript
// infrastructure/stack.ts
import * as cdk from 'aws-cdk-lib';
import * as rds from 'aws-cdk-lib/aws-rds';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

export class AppStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: cdk.StackProps) {
    super(scope, id, props);

    const vpc = new ec2.Vpc(this, 'AppVpc', { maxAzs: 3 });

    const database = new rds.DatabaseInstance(this, 'AppDB', {
      engine: rds.DatabaseInstanceEngine.postgres({ version: rds.PostgresEngineVersion.VER_15_4 }),
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.R6G, ec2.InstanceSize.XLARGE),
      vpc,
      multiAz: true,
      backupRetention: cdk.Duration.days(7),
    });
  }
}
```

> 📖 Reference: [IaC — HashiCorp](https://www.hashicorp.com/resources/what-is-infrastructure-as-code)

---

**Q24. What is a rollback strategy in deployments? How do you design for it?**

**Answer:**

A **rollback strategy** defines how to revert to the previous version when a deployment causes problems. Fast, reliable rollback is as important as the deployment itself.

**Rollback strategies by deployment type:**

```bash
# ── Kubernetes: immediate rollback ────────────────────────────────────
kubectl rollout undo deployment/api
# Reverts to previous ReplicaSet (keeps history by default)

kubectl rollout history deployment/api
# REVISION  CHANGE-CAUSE
# 1         Initial deployment v1.0
# 2         Update to v1.1
# 3         Update to v1.2 ← current (problematic)

kubectl rollout undo deployment/api --to-revision=2  # roll back to v1.1
# Kubernetes starts old pods, terminates new ones (rolling)

# ── Blue-green: instant traffic switch ───────────────────────────────
kubectl patch service api-service -p '{"spec":{"selector":{"version":"blue"}}}'
# Traffic immediately returns to blue (old version)
# Green stays running temporarily for analysis

# ── Database: the hardest part ────────────────────────────────────────
# Code rollback is easy. DB schema rollback is hard.

# ❌ Bad migration (irreversible):
ALTER TABLE users DROP COLUMN phone_number;
# If you roll back the code, old code expects phone_number → crashes!

# ✅ Good migration (expand-contract / backward-compatible):
-- Deploy step 1: Add new column (old code ignores it, new code uses it)
ALTER TABLE users ADD COLUMN phone_number VARCHAR(20);

-- Deploy step 2: new code writes to BOTH old and new columns

-- Deploy step 3: backfill, then drop old column
-- If you rollback between steps 1-2: no problem, old code runs fine

# ✅ Never make column NOT NULL in same migration as adding it:
-- ❌ This breaks rollback:
ALTER TABLE orders ADD COLUMN currency VARCHAR(3) NOT NULL DEFAULT 'USD';

-- ✅ Do it in steps:
-- Step 1: Add nullable
ALTER TABLE orders ADD COLUMN currency VARCHAR(3) DEFAULT 'USD';
-- Step 2: Backfill existing rows
UPDATE orders SET currency = 'USD' WHERE currency IS NULL;
-- Step 3 (later): Make NOT NULL
ALTER TABLE orders ALTER COLUMN currency SET NOT NULL;
```

**Automated rollback triggers:**

```yaml
# Argo Rollouts: auto-rollback on error rate spike
analysis:
  metrics:
  - name: error-rate
    interval: 30s
    failureCondition: result[0] > 0.05   # auto-rollback if error rate > 5%
    provider:
      prometheus:
        query: rate(http_requests_total{status=~"5.."}[1m]) / rate(http_requests_total[1m])
```

> 📖 Reference: [Rollback Strategies — Google SRE Book](https://sre.google/sre-book/release-engineering/)

---

## 4. Kubernetes Basics

---

**Q25. What is Kubernetes? What problem does it solve that Docker alone doesn't?**

**Answer:**

**Docker** packages apps into containers. **Kubernetes (K8s)** orchestrates those containers at scale — scheduling, healing, scaling, and networking them across a cluster of machines.

```
Docker alone:
- Run a container on ONE machine: docker run my-app
- Container crashes → stays dead (you must restart manually)
- Need more capacity → manually SSH, docker run more
- No load balancing, no service discovery, no rolling updates
- Works for 1 container. Fails for 100 containers across 10 servers.

Kubernetes:
- Schedules containers across a CLUSTER of nodes
- Container crashes → K8s restarts it automatically (self-healing)
- CPU too high → K8s adds more pods (autoscaling)
- Deploy new version → K8s rolls it out without downtime
- Service discovery built-in (DNS for every service)
- Secret management, config management, volume management
```

**Docker vs Kubernetes comparison:**

| | Docker | Kubernetes |
|--|--------|-----------|
| Runs on | 1 machine | Cluster of machines |
| Self-healing | ❌ Manual restart | ✅ Automatic pod restart |
| Scaling | ❌ Manual | ✅ HPA autoscales on CPU/memory |
| Load balancing | ❌ Need NGINX manually | ✅ Built-in Service abstraction |
| Rolling updates | ❌ Manual | ✅ Deployment rolling update |
| Service discovery | ❌ None | ✅ DNS-based (service-name.namespace) |
| Config management | Manual env files | ConfigMaps + Secrets |
| Storage | Volumes per container | PersistentVolumes across cluster |

> 📖 Reference: [Kubernetes Overview — Kubernetes Docs](https://kubernetes.io/docs/concepts/overview/)

---

**Q26. What are Pods, Deployments, and Services in Kubernetes?**

**Answer:**

**Pod:** The smallest deployable unit. A pod wraps one or more containers that share network and storage. You almost never create pods directly — Deployments manage them.

**Deployment:** Manages a set of identical pods. Handles rolling updates, scaling, and self-healing.

**Service:** Stable network endpoint that load-balances traffic to pods. Pods come and go; Service provides a consistent DNS name and IP.

```yaml
# ── Pod (rarely created directly) ─────────────────────────────────────
apiVersion: v1
kind: Pod
metadata:
  name: my-api-pod
spec:
  containers:
  - name: api
    image: my-app:1.0
    ports: [{ containerPort: 3000 }]

# ── Deployment (use this instead of raw pods) ──────────────────────────
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3                    # keep 3 pods running at all times
  selector:
    matchLabels: { app: api }
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1          # at most 1 pod down during update
      maxSurge:       1          # at most 1 extra pod during update
  template:
    metadata:
      labels: { app: api }
    spec:
      containers:
      - name: api
        image: my-app:1.0
        ports: [{ containerPort: 3000 }]
        resources:
          requests: { cpu: "100m", memory: "256Mi" }
          limits:   { cpu: "500m", memory: "512Mi" }
        readinessProbe:
          httpGet: { path: /health, port: 3000 }
          initialDelaySeconds: 5
          periodSeconds: 10

# ── Service (stable endpoint to reach pods) ────────────────────────────
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector: { app: api }         # routes to pods with label app=api
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP                # internal only

# Other service types:
# type: LoadBalancer   → external AWS/GCP load balancer (internet-facing)
# type: NodePort       → expose on each node's IP:port (dev use)
# type: ClusterIP      → internal cluster traffic only (default)
```

```bash
# Operations
kubectl get pods              # list running pods
kubectl get deployments       # list deployments
kubectl get services          # list services
kubectl describe pod api-xyz  # detailed info + events

kubectl scale deployment api --replicas=10  # scale to 10 pods
kubectl set image deployment/api api=my-app:1.1  # rolling update

# Access service from within cluster:
# http://api-service.default.svc.cluster.local:80
# Or just: http://api-service (within same namespace)
```

> 📖 Reference: [Kubernetes Workloads — Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/)

---

**Q27. What is a Kubernetes ConfigMap vs a Secret?**

**Answer:**

| | ConfigMap | Secret |
|--|-----------|--------|
| Purpose | Non-sensitive configuration | Sensitive data (passwords, keys, certs) |
| Encoding | Plain text | Base64 encoded (NOT encrypted by default) |
| Access | Env vars, volume mounts, CLI | Same, but access controlled by RBAC |
| Example | App log level, feature flags, DB host | DB password, API keys, TLS certificates |

```yaml
# ── ConfigMap ─────────────────────────────────────────────────────────
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL:    "info"
  PORT:         "3000"
  DB_HOST:      "postgres.default.svc.cluster.local"
  DB_PORT:      "5432"
  CACHE_TTL:    "300"
  APP_ENV:      "production"

# ── Secret ────────────────────────────────────────────────────────────
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  # Values must be base64-encoded
  # echo -n 'mysecretpassword' | base64
  DB_PASSWORD:    bXlzZWNyZXRwYXNzd29yZA==
  JWT_SECRET:     c3VwZXJzZWNyZXRrZXkxMjM=
  STRIPE_API_KEY: c2tfdGVzdF94eHh4eHg=

# ── Use in Deployment ─────────────────────────────────────────────────
spec:
  containers:
  - name: api
    image: my-app:1.0
    envFrom:
    - configMapRef:
        name: app-config          # load all ConfigMap entries as env vars
    - secretRef:
        name: app-secrets         # load all Secret entries as env vars
    # Or individually:
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: DB_PASSWORD
```

```bash
# Create from files (don't commit secrets to git!)
kubectl create secret generic app-secrets \
  --from-literal=DB_PASSWORD=mysecretpassword \
  --from-literal=JWT_SECRET=supersecretkey

# Or from a .env file
kubectl create secret generic app-secrets --from-env-file=.env.production

# In production: use external secret manager (AWS Secrets Manager + External Secrets Operator)
# Syncs secrets from AWS → Kubernetes Secret automatically
```

> 📖 Reference: [ConfigMaps and Secrets — Kubernetes Docs](https://kubernetes.io/docs/concepts/configuration/)

---

**Q28. What is horizontal pod autoscaling (HPA) in Kubernetes?**

**Answer:**

**HPA** automatically scales the number of pod replicas based on observed CPU utilization, memory, or custom metrics (like requests per second, Kafka consumer lag).

```
Normal load: 3 pods at 30% CPU each
Traffic spike: CPU rises to 80%
HPA: "Above 50% CPU threshold → scale up"
HPA scales to 6 pods → CPU drops to 40% ✅

Traffic drops:
CPU at 10% per pod
HPA: "Below threshold → scale down"
HPA scales back to 3 pods → cost optimized ✅
```

```yaml
# ── HPA based on CPU ──────────────────────────────────────────────────
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50   # scale when avg CPU > 50%

# ── HPA based on memory ───────────────────────────────────────────────
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 400Mi     # scale when avg memory > 400Mi

# ── HPA based on custom metric (requests per second) ──────────────────
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"     # scale when avg RPS per pod > 100

# ── HPA based on Kafka consumer lag (KEDA) ────────────────────────────
# KEDA extends HPA with external metrics
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: kafka-consumer-scaler
spec:
  scaleTargetRef:
    name: order-processor
  minReplicaCount: 1
  maxReplicaCount: 30
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka:9092
      consumerGroup:    order-processor
      topic:            order-events
      lagThreshold:     "100"   # 1 pod per 100 lagging messages
```

```bash
kubectl get hpa
# NAME      REFERENCE   TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
# api-hpa   Deployment  45%/50%         3         20        5          10m

kubectl describe hpa api-hpa  # shows scaling events history
```

**Important:** Pods must have resource requests set (CPU/memory) for HPA to work. HPA looks at requests, not limits.

> 📖 Reference: [HPA — Kubernetes Docs](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

---

**Q29. What is a liveness probe vs a readiness probe in Kubernetes?**

**Answer:**

| | Liveness Probe | Readiness Probe |
|--|---------------|----------------|
| Question | "Is this container alive?" | "Is this container ready to serve traffic?" |
| Action on failure | Kill container + restart | Remove from Service load balancer |
| When to use | Detect deadlock, infinite loop, hung process | Waiting for DB connection, startup, high load |
| Effect | Pod restarts | Pod stays running but gets no traffic |

```yaml
spec:
  containers:
  - name: api
    image: my-app:1.0

    # ── Liveness: is the container healthy? ───────────────────────────
    # Fails → Kubernetes kills + restarts the container
    livenessProbe:
      httpGet:
        path: /healthz      # simple endpoint: just returns 200 (no DB check!)
        port: 3000
      initialDelaySeconds: 30  # wait 30s before first check (give app time to start)
      periodSeconds: 10        # check every 10 seconds
      timeoutSeconds: 5        # timeout after 5 seconds
      failureThreshold: 3      # restart after 3 consecutive failures

    # ── Readiness: is the container ready for traffic? ─────────────────
    # Fails → removed from Service endpoints (no traffic), NOT restarted
    readinessProbe:
      httpGet:
        path: /ready        # deep check: verifies DB + Redis connectivity
        port: 3000
      initialDelaySeconds: 5
      periodSeconds: 5
      successThreshold: 1      # 1 success = mark ready
      failureThreshold: 3      # 3 failures = remove from rotation
```

```js
// Liveness endpoint: simple (don't check dependencies!)
app.get('/healthz', (req, res) => {
  // Just check the process is alive and event loop is running
  res.status(200).json({ status: 'alive' });
  // Don't check DB here — if DB is down, K8s would restart infinitely!
});

// Readiness endpoint: thorough (check all dependencies)
app.get('/ready', async (req, res) => {
  const checks = {};

  try {
    await db.query('SELECT 1');
    checks.database = 'ok';
  } catch {
    checks.database = 'error';
  }

  try {
    await redis.ping();
    checks.redis = 'ok';
  } catch {
    checks.redis = 'error';
  }

  const ready = Object.values(checks).every(v => v === 'ok');
  res.status(ready ? 200 : 503).json({ checks });
  // 503 → K8s removes pod from load balancer (no traffic)
  // Pod stays running, keeps trying to reconnect
  // Once DB recovers → probe passes → traffic resumes ✅
});
```

> 📖 Reference: [Probes — Kubernetes Docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

---

**Q30. What is a Kubernetes namespace? Why is it used?**

**Answer:**

A **namespace** is a virtual cluster within a Kubernetes cluster — a way to partition resources, apply policies, and isolate teams or environments within the same physical cluster.

```bash
# Default namespaces:
# default     → where resources go if you don't specify a namespace
# kube-system → Kubernetes internal components (CoreDNS, kube-proxy)
# kube-public → publicly readable resources
```

```yaml
# Create namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production

---
# Deploy to a specific namespace
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: production   # ← deploys to production namespace
```

```bash
kubectl get pods -n production         # list pods in production namespace
kubectl get pods -n staging            # list pods in staging namespace
kubectl get pods --all-namespaces      # list all pods in all namespaces

# Apply resource quotas per namespace (prevent one team from using all cluster resources)
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-payments
spec:
  hard:
    requests.cpu: "10"          # team can request at most 10 CPUs total
    requests.memory: 20Gi       # at most 20GB RAM
    limits.cpu: "20"
    pods: "50"                  # at most 50 pods
EOF

# Network policy: namespace isolation
# By default, all pods in ALL namespaces can talk to each other
# Add NetworkPolicy to restrict:
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-cross-namespace
  namespace: production
spec:
  podSelector: {}
  policyTypes: [Ingress]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production    # only allow traffic from production namespace
```

**Common patterns:**
- `production`, `staging`, `development` — environment isolation in one cluster.
- `team-frontend`, `team-backend`, `team-payments` — team-based isolation.
- Per-feature-branch namespaces in CI for isolated testing.

> 📖 Reference: [Namespaces — Kubernetes Docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

---

## 5. Advanced Database Concepts

---

**Q31. What is database sharding? What are the common sharding strategies?**

**Answer:**

**Sharding** is horizontal partitioning of a database across multiple servers (shards). Each shard holds a subset of the data and is independently queryable.

```
Without sharding:
All 100M users in one DB server → slow queries, disk full, bottleneck

With sharding (4 shards):
Shard 0: users 0-25M
Shard 1: users 25M-50M
Shard 2: users 50M-75M
Shard 3: users 75M-100M
→ Each shard handles 25% of queries → 4x throughput
```

**Common sharding strategies:**

```js
// ── 1. Range-based sharding ───────────────────────────────────────────
// Route by value range of the shard key
function getShardByUserId(userId) {
  if (userId < 25_000_000) return shards[0];
  if (userId < 50_000_000) return shards[1];
  if (userId < 75_000_000) return shards[2];
  return shards[3];
}
// Pro: simple, good for range queries ("users 1-1000")
// Con: hotspots (new users always go to last shard)

// ── 2. Hash-based sharding ────────────────────────────────────────────
// Distribute evenly using hash function
function getShardByUserIdHash(userId) {
  return shards[userId % shards.length]; // evenly distributed
}
// Pro: even distribution, no hotspots
// Con: range queries span all shards, resharding is painful

// ── 3. Directory-based sharding ──────────────────────────────────────
// Lookup table maps entity → shard
const directory = new Map([
  ['user:1', 'shard-1'],
  ['user:2', 'shard-3'],
  // ...
]);
function getShardByLookup(userId) {
  return directory.get(`user:${userId}`) || assignToLeastLoadedShard(userId);
}
// Pro: flexible, can rebalance without rehashing
// Con: directory itself can become bottleneck

// ── Application-level sharding example ───────────────────────────────
const shards = [
  new Pool({ host: 'shard-0.db.internal' }),
  new Pool({ host: 'shard-1.db.internal' }),
  new Pool({ host: 'shard-2.db.internal' }),
  new Pool({ host: 'shard-3.db.internal' }),
];

async function getUserById(userId) {
  const shardIndex = userId % shards.length;  // hash sharding
  const shard = shards[shardIndex];
  return shard.query('SELECT * FROM users WHERE id = $1', [userId]);
}

async function getUsersByIds(userIds) {
  // Group by shard
  const groups = {};
  for (const id of userIds) {
    const shardIndex = id % shards.length;
    (groups[shardIndex] = groups[shardIndex] || []).push(id);
  }

  // Query each shard in parallel
  const results = await Promise.all(
    Object.entries(groups).map(([shardIndex, ids]) =>
      shards[shardIndex].query('SELECT * FROM users WHERE id = ANY($1)', [ids])
    )
  );
  return results.flat().map(r => r.rows).flat();
}
```

**Sharding challenges:**
- Cross-shard joins are expensive or impossible — must denormalize.
- Resharding (adding shards) is complex.
- Distributed transactions across shards are very hard.
- Most teams should exhaust vertical scaling and read replicas before sharding.

> 📖 Reference: [Database Sharding — MongoDB](https://www.mongodb.com/features/database-sharding-explained)

---

**Q32. What is database replication? Explain primary-replica (master-slave) replication.**

**Answer:**

**Replication** copies data from one database server (primary) to one or more servers (replicas) to provide high availability, read scaling, and disaster recovery.

```
Primary-Replica Architecture:

              ┌────────────────┐
Write → ──────►  Primary DB    ├──► Replica 1 (read-only)
              │  (read+write)  ├──► Replica 2 (read-only)
              └────────────────┘──► Replica 3 (read-only, different region)

Reads → routed to any replica
Writes → always go to primary
```

**How it works (PostgreSQL streaming replication):**

```
1. Application writes to primary DB
2. Primary writes change to WAL (Write-Ahead Log) — a sequential log of all changes
3. WAL records streamed to replica in real-time
4. Replica applies WAL records to its own data → stays in sync
5. Replication lag: how far behind the replica is (typically 0-100ms)
```

```yaml
# PostgreSQL replication config (postgresql.conf on primary)
wal_level = replica             # enable WAL for replication
max_wal_senders = 5             # allow 5 replica connections
wal_keep_size = 512MB           # keep 512MB of WAL for replicas to catch up

# pg_hba.conf on primary (allow replica to connect)
host replication replicator 10.0.1.20/32 md5

# On replica (recovery.conf or postgresql.conf)
primary_conninfo = 'host=primary.db.internal user=replicator password=xxx'
hot_standby = on   # allow read queries on replica
```

```js
// Application-level: route reads to replicas
const primary = new Pool({ host: 'primary.db.internal' });
const replica = new Pool({ host: 'replica.db.internal' });

// Write: must go to primary
async function createUser(data) {
  return primary.query('INSERT INTO users ... RETURNING *', [data]);
}

// Read: can go to replica (eventual consistency acceptable)
async function getUserById(id) {
  return replica.query('SELECT * FROM users WHERE id = $1', [id]);
}

// Read-your-own-writes: use primary for immediate consistency
async function register(data) {
  const user = await primary.query('INSERT INTO users ... RETURNING *', [data]);
  return primary.query('SELECT * FROM users WHERE id = $1', [user.rows[0].id]);
  // Use primary for the read that immediately follows the write
}
```

**Failover:**
When primary goes down, a replica is promoted to primary. This can be automated with tools like Patroni (PostgreSQL), AWS RDS Multi-AZ (automatic), or orchestrated manually.

> 📖 Reference: [DB Replication — PostgreSQL Docs](https://www.postgresql.org/docs/current/high-availability.html)

---

**Q33. What is a write-ahead log (WAL)? How does it ensure durability?**

**Answer:**

The **Write-Ahead Log (WAL)** is a sequential, append-only log file where every database change is written **before** it's applied to the actual data files. It is the backbone of durability (the "D" in ACID) and replication.

**Why it ensures durability:**

```
Without WAL:
1. Transaction begins
2. Data files updated in memory (buffer pool)
3. If server crashes HERE → changes in memory are lost! Data file partially updated!
4. Database is corrupt on restart

With WAL:
1. Transaction begins
2. Changes written to WAL on disk (sequential, fast)
   fsync: WAL is physically durable before step 3
3. COMMIT returned to client ← "Your data is safe" (WAL is durable)
4. Data files updated asynchronously from WAL
5. If server crashes between 3 and 4: WAL is intact → replay WAL on restart → no data loss ✅
```

```sql
-- PostgreSQL WAL configuration
-- postgresql.conf
wal_level = replica               -- minimal, replica, or logical
synchronous_commit = on           -- wait for WAL flush before COMMIT returns (safe)
fsync = on                        -- force WAL to disk (never turn off in production!)
checkpoint_completion_target = 0.9 -- spread checkpoint I/O over 90% of checkpoint interval

-- WAL files: /var/lib/postgresql/data/pg_wal/
-- Each file is 16MB by default
-- Named: 000000010000000000000001, 000000010000000000000002, ...

-- Check WAL position
SELECT pg_current_wal_lsn();   -- current WAL Log Sequence Number
SELECT pg_walfile_name(pg_current_wal_lsn()); -- current WAL file name

-- Monitor WAL generation rate (helps right-size disk)
SELECT * FROM pg_stat_bgwriter;
```

**WAL uses beyond durability:**
1. **Replication:** Replicas receive WAL stream from primary → stay in sync.
2. **Point-in-time recovery:** Replay WAL from a backup to any specific moment.
3. **Logical decoding:** CDC tools (Debezium) read WAL to stream changes to Kafka.
4. **Crash recovery:** On startup after crash, replay WAL from last checkpoint.

> 📖 Reference: [WAL — PostgreSQL Docs](https://www.postgresql.org/docs/current/wal-intro.html)

---

**Q34. What is the difference between synchronous and asynchronous replication?**

**Answer:**

| | Synchronous Replication | Asynchronous Replication |
|--|------------------------|------------------------|
| When primary commits | After replica confirms WAL written | Immediately (doesn't wait for replica) |
| Write latency | Higher (waits for replica ACK) | Lower (no waiting) |
| Data loss on primary failure | Zero (replica is up-to-date) | Possible (in-flight changes lost) |
| Replica availability required | Yes (primary blocks if replica down) | No |
| Use case | Financial data, anything where data loss is unacceptable | Read scaling, analytics, DR with RPO > 0 |

```sql
-- PostgreSQL: configure synchronous replication
-- postgresql.conf on primary:
synchronous_commit = on               -- default: async
synchronous_standby_names = 'replica1' -- wait for replica1 to confirm

-- Per-transaction control (flexibility):
BEGIN;
SET LOCAL synchronous_commit = ON;    -- this transaction: synchronous (safe)
INSERT INTO payments ...;
COMMIT;

BEGIN;
SET LOCAL synchronous_commit = OFF;   -- this transaction: async (faster)
INSERT INTO audit_logs ...;           -- audit logs can lag slightly
COMMIT;
```

```
Synchronous flow:
Client → BEGIN; INSERT; COMMIT → Primary writes WAL → Sends WAL to Replica
                                                        Replica confirms WAL written
                                                                    ↓
Client ← "COMMIT" ← Primary ← Replica ACK
(Client doesn't get COMMIT until replica has the data)
Latency: primary + 1 network round trip to replica

Asynchronous flow:
Client → BEGIN; INSERT; COMMIT → Primary writes WAL → "COMMIT" → Client
                                Primary sends WAL to Replica asynchronously
                                (Client doesn't wait for replica)
Latency: primary only (much faster)
Risk: If primary crashes before replica receives WAL → last few transactions lost (RPO > 0)
```

**AWS RDS Multi-AZ:**
- Uses synchronous replication within an AZ pair.
- Automatic failover in 60-120 seconds.
- Zero data loss (RPO = 0) because standby is always in sync.

> 📖 Reference: [Sync vs Async Replication — AWS](https://aws.amazon.com/rds/features/multi-az/)

---

**Q35. What is a time-series database? When would you choose one over a relational DB?**

**Answer:**

A **time-series database (TSDB)** is optimized for storing and querying data that is indexed primarily by time — metrics, sensor readings, financial ticks, IoT telemetry.

```
Time-series data shape:
timestamp          | metric            | value | tags
2024-06-01 12:00:00 | cpu.utilization   | 45.2  | host=web-01, env=prod
2024-06-01 12:00:05 | cpu.utilization   | 47.8  | host=web-01, env=prod
2024-06-01 12:00:10 | cpu.utilization   | 43.1  | host=web-01, env=prod
```

**Why relational DBs are bad for time-series at scale:**

```sql
-- Naive PostgreSQL approach: millions of rows per day
-- cpu_metrics table: timestamp, host, metric, value
-- Problem 1: index bloat — B-tree on timestamp becomes huge
-- Problem 2: old data never queried but takes space — need range partitioning
-- Problem 3: no built-in downsampling ("average per hour" of "per-second" data)
-- Problem 4: no compression for similar values (45.2, 45.3, 45.1 stored 3 times)
```

**InfluxDB example:**

```js
const { InfluxDB, Point } = require('@influxdata/influxdb-client');
const influx = new InfluxDB({ url: 'http://influxdb:8086', token: process.env.INFLUX_TOKEN });

const writeApi = influx.getWriteApi('my-org', 'metrics');

// Write a metric
const point = new Point('cpu_utilization')
  .tag('host', 'web-01')
  .tag('env', 'production')
  .floatField('value', 45.2)
  .timestamp(new Date());

writeApi.writePoint(point);
await writeApi.flush();

// Query: InfluxDB Flux language
const queryApi = influx.getQueryApi('my-org');
const fluxQuery = `
  from(bucket: "metrics")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "cpu_utilization")
    |> filter(fn: (r) => r.host == "web-01")
    |> aggregateWindow(every: 5m, fn: mean)   // 5-minute averages
    |> yield(name: "mean")
`;
const rows = await queryApi.collectRows(fluxQuery);
```

**TSDB-specific features (absent in relational DBs):**
- **Automatic downsampling:** Store raw 1-second data for 1 week, then aggregate to 1-minute for 1 year.
- **Compression:** delta encoding, run-length encoding for similar timestamps/values → 90% size reduction.
- **Time-range queries:** `WHERE time > NOW() - 1h` optimized with time-partitioned storage.
- **Functions:** `rate()`, `increase()`, `rolling_average()` built-in.

**When to choose a TSDB:**
- Monitoring/metrics (Prometheus → InfluxDB/TimescaleDB/VictoriaMetrics).
- IoT sensor data (millions of data points per second).
- Financial tick data.
- Application performance monitoring.

> 📖 Reference: [Time-Series Databases — InfluxData](https://www.influxdata.com/time-series-database/)

---

## 6. Distributed Systems Fundamentals

---

**Q36. What is the two-generals problem? What does it tell us about distributed systems?**

**Answer:**

**The Two Generals Problem** is a classic thought experiment showing that it is **impossible to achieve reliable consensus** over an unreliable communication channel.

**The scenario:**
Two armies (General A and General B) must attack a city simultaneously to win. They can only communicate via messengers through enemy territory. Any messenger might be captured.

```
General A → "Attack at dawn?" → messenger → General B
            (might be captured)

General B → "Agreed, attack at dawn!" → messenger → General A
            (might be captured)

General A needs to know B got the message before attacking.
But B needs confirmation that A received B's confirmation.
But A needs confirmation that B received A's confirmation... (infinite regress!)

No matter how many acknowledgments they exchange:
The last message might not arrive → neither can be 100% certain the other will attack
```

**What it tells us about distributed systems:**

```
It proves that perfect reliability over unreliable networks is IMPOSSIBLE.

Real implications for backends:

1. "Did the payment go through?"
   Client sends payment → server processes → server sends response → NETWORK DROPS
   Was payment charged? Did the client get the response?
   You can never be 100% certain without idempotency keys + retry logic.

2. "Did the database write succeed?"
   App writes to DB → DB commits → "COMMIT" response → NETWORK DROPS
   Data IS in the DB, but the app doesn't know!
   Next retry → duplicate write → two orders created!
   Solution: idempotency keys, unique constraints, optimistic locking.

3. Distributed consensus is hard
   This is why Raft and Paxos are complex algorithms.
   They don't "solve" the two-generals problem — they work around it with:
   - Quorums (majority agreement)
   - Leader election
   - Bounded message loss assumptions

// Practical takeaways:
// 1. Design every external call to be idempotent (safe to retry)
// 2. Use idempotency keys for operations with side effects (payments, emails)
// 3. Accept eventual consistency — "maybe" is the fundamental state of distributed systems
// 4. Build retry logic everywhere, but make handlers idempotent first
```

> 📖 Reference: [Two Generals Problem — Wikipedia](https://en.wikipedia.org/wiki/Two_Generals%27_Problem)

---

**Q37. What is a distributed lock? How would you implement one using Redis?**

**Answer:**

A **distributed lock** ensures that only ONE process (across multiple servers/pods) can execute a critical section at a time — preventing race conditions in distributed systems.

```
Problem without distributed lock:
Pod 1: checks "is job running?" → No → starts job
Pod 2: checks "is job running?" → No (Pod 1 just started, not yet marked!)
→ Both pods run the same job simultaneously → duplicate processing!

With distributed lock:
Pod 1: acquires lock "job:invoice-gen" → starts job
Pod 2: tries to acquire "job:invoice-gen" → fails → waits/skips
→ Only one pod runs the job ✅
```

**Redis-based distributed lock:**

```js
const redis = require('ioredis');
const client = new redis();

class DistributedLock {
  constructor(client) {
    this.client = client;
  }

  async acquire(lockKey, ttlMs = 30000) {
    const lockValue = crypto.randomUUID(); // unique value to identify this lock owner

    // SET NX EX: set only if Not eXists, with EXpiry
    // Atomic operation — either we get the lock or we don't
    const result = await this.client.set(
      `lock:${lockKey}`,
      lockValue,
      'NX',           // only set if key doesn't exist
      'PX',           // expiry in milliseconds
      ttlMs           // auto-release after ttlMs (prevents deadlock if process dies)
    );

    if (result === 'OK') {
      return lockValue; // return our unique value (needed to release safely)
    }
    return null; // lock not acquired — someone else has it
  }

  async release(lockKey, lockValue) {
    // Use Lua script for atomic check-and-delete
    // Never delete a lock you don't own!
    const luaScript = `
      if redis.call("GET", KEYS[1]) == ARGV[1] then
        return redis.call("DEL", KEYS[1])
      else
        return 0
      end
    `;
    // This atomically: checks the lock value, deletes ONLY if it's ours
    await this.client.eval(luaScript, 1, `lock:${lockKey}`, lockValue);
  }

  async withLock(lockKey, fn, options = {}) {
    const { ttlMs = 30000, retryMs = 100, maxRetries = 10 } = options;

    for (let attempt = 0; attempt < maxRetries; attempt++) {
      const lockValue = await this.acquire(lockKey, ttlMs);

      if (lockValue) {
        try {
          return await fn(); // execute the critical section
        } finally {
          await this.release(lockKey, lockValue); // always release
        }
      }

      // Lock not available — wait and retry
      await new Promise(r => setTimeout(r, retryMs * (attempt + 1)));
    }

    throw new Error(`Could not acquire lock on ${lockKey} after ${maxRetries} attempts`);
  }
}

const lock = new DistributedLock(client);

// Usage: only one pod generates the invoice at a time
app.post('/billing/generate', async (req, res) => {
  const { userId, month } = req.body;

  await lock.withLock(`invoice:${userId}:${month}`, async () => {
    // This block runs on exactly ONE pod at a time
    const existing = await Invoice.findOne({ userId, month });
    if (existing) return existing; // already generated

    return Invoice.create({ userId, month, amount: calculateBill(userId, month) });
  }, { ttlMs: 60000, maxRetries: 3 });

  res.json({ success: true });
});

// For production: use Redlock algorithm (multi-node Redis for fault tolerance)
// https://redis.io/docs/manual/patterns/distributed-locks/#the-redlock-algorithm
const Redlock = require('redlock');
const redlock = new Redlock([client1, client2, client3]); // 3 Redis nodes
const lock = await redlock.acquire(['lock:invoice-gen'], 30000);
try { /* critical section */ }
finally { await lock.release(); }
```

> 📖 Reference: [Redlock Algorithm — Redis Docs](https://redis.io/docs/manual/patterns/distributed-locks/)

---

**Q38. What is consistent hashing? How is it used in distributed caching?**

**Answer:**

**Consistent hashing** is an algorithm for distributing data across nodes in a way that minimizes redistribution when nodes are added or removed.

**Problem with naive hash-based distribution:**

```
5 cache nodes. Key → node using: hash(key) % 5

Add a 6th node:
hash(key) % 5 → hash(key) % 6 → DIFFERENT RESULT for almost every key!
→ ~83% of keys map to wrong node → massive cache miss storm → DB hammered
```

**Consistent hashing solution:**

```
Imagine a ring (0 to 2^32).
Each node is placed at multiple positions on the ring (virtual nodes).
Each key is hashed to a position on the ring.
Key → find the nearest node clockwise.

           Node A (12 o'clock)
          /                   \
Node D                         Node B
(9 o'clock)                    (3 o'clock)
          \                   /
           Node C (6 o'clock)

Key X hashes to 1:30 position → assigned to Node B (nearest clockwise)

Adding Node E at 2 o'clock:
Only keys that were between 12 o'clock and 2 o'clock move from B to E
→ Only ~25% of keys remapped instead of ~83% ✅
```

```js
// Consistent hashing implementation
const crypto = require('crypto');

class ConsistentHashRing {
  constructor(nodes, virtualNodes = 150) {
    this.ring = new Map();       // position → node
    this.sortedKeys = [];        // sorted ring positions
    this.virtualNodes = virtualNodes;

    for (const node of nodes) this.addNode(node);
  }

  hash(key) {
    return parseInt(
      crypto.createHash('md5').update(key).digest('hex').slice(0, 8),
      16
    );
  }

  addNode(node) {
    for (let i = 0; i < this.virtualNodes; i++) {
      const position = this.hash(`${node}:${i}`);
      this.ring.set(position, node);
      this.sortedKeys.push(position);
    }
    this.sortedKeys.sort((a, b) => a - b);
  }

  removeNode(node) {
    for (let i = 0; i < this.virtualNodes; i++) {
      const position = this.hash(`${node}:${i}`);
      this.ring.delete(position);
    }
    this.sortedKeys = this.sortedKeys.filter(k => this.ring.has(k));
  }

  getNode(key) {
    const hash = this.hash(key);
    // Find first node position >= hash (clockwise)
    const idx = this.sortedKeys.findIndex(pos => pos >= hash);
    const pos = idx === -1 ? this.sortedKeys[0] : this.sortedKeys[idx];
    return this.ring.get(pos);
  }
}

// Usage: distributed cache routing
const ring = new ConsistentHashRing(['redis-1:6379', 'redis-2:6379', 'redis-3:6379']);

async function cacheGet(key) {
  const node = ring.getNode(key);
  const redis = redisConnections[node];
  return redis.get(key);
}

async function cacheSet(key, value, ttl) {
  const node = ring.getNode(key);
  const redis = redisConnections[node];
  return redis.setex(key, ttl, value);
}

// 'user:42' → always maps to redis-2 (consistent!)
// Add redis-4: only ~25% of keys remapped → minimal disruption
```

> 📖 Reference: [Consistent Hashing — Tom White](https://tom-e-white.com/2007/11/consistent-hashing.html)

---

**Q39. What is a quorum in distributed systems? How does it relate to consistency?**

**Answer:**

A **quorum** is the minimum number of nodes that must participate in an operation for it to be considered valid. Quorum-based systems ensure that any two operations share at least one common node — guaranteeing consistency.

**Formula:** For N nodes, quorum = ⌈N/2⌉ + 1 (majority)

```
3 nodes: quorum = 2 (majority of 3)
5 nodes: quorum = 3 (majority of 5)
7 nodes: quorum = 4 (majority of 7)
```

**Why quorum ensures consistency:**

```
5-node Cassandra cluster. Write quorum = 3. Read quorum = 3.

Write: client writes to 3 of 5 nodes
  Node1 ✅, Node2 ✅, Node3 ✅, Node4 (not written), Node5 (not written)
  Write succeeds when 3 respond ✅

Read: client reads from 3 of 5 nodes
  Node1 ✅, Node2 ✅, Node4 ✅ (at least 2 of these have the latest value)
  Read returns the latest value ✅

Why? Write quorum (3) + Read quorum (3) > N (5)
     → They MUST overlap by at least 1 node
     → That node has the latest data
     → Consistency guaranteed ✅
```

**Cassandra quorum levels:**

```js
const cassandra = require('cassandra-driver');
const consistencies = cassandra.types.consistencies;

// ONE: fastest, lowest consistency (any 1 replica responds)
await client.execute(query, params, { consistency: consistencies.one });

// QUORUM: majority of replicas (balanced speed/consistency)
await client.execute(query, params, { consistency: consistencies.quorum });

// ALL: all replicas must respond (slowest, highest consistency)
await client.execute(query, params, { consistency: consistencies.all });

// LOCAL_QUORUM: quorum within local datacenter only (good for multi-region)
await client.execute(query, params, { consistency: consistencies.localQuorum });
```

**Trade-off (Quorum vs ONE):**

```
quorum read + quorum write = strong consistency (always get latest write)
one read + one write = eventual consistency (may get stale data, but fastest)

Tunable consistency: choose per-operation based on business need
- Bank balance: QUORUM (must be accurate)
- "Online users" indicator: ONE (slight staleness ok)
```

> 📖 Reference: [Quorum — Martin Fowler](https://martinfowler.com/articles/patterns-of-distributed-systems/quorum.html)

---

**Q40. What is the difference between a synchronous and an asynchronous distributed system?**

**Answer:**

| | Synchronous | Asynchronous |
|--|------------|-------------|
| Message timing | Messages arrive within known bounded time | Messages may arrive at any time (unbounded delay) |
| Clock assumptions | Clocks synchronized, bounded drift | No clock assumptions |
| Failure detection | Reliable (timeout = failure) | Unreliable (timeout ≠ failure, just slow) |
| Consensus | Easier (FLP doesn't apply with timing bounds) | Impossible (FLP impossibility theorem) |
| Real-world example | Idealized model (rare in practice) | The actual internet |

**In practice — the real internet is asynchronous:**

```js
// Asynchronous system problem: timeout ≠ failure
async function callPaymentService(orderId, amount) {
  try {
    const result = await fetch('http://payment-service/charge', {
      method: 'POST',
      body: JSON.stringify({ orderId, amount }),
      signal: AbortSignal.timeout(5000) // 5 second timeout
    });
    return await result.json();
  } catch (err) {
    if (err.name === 'TimeoutError') {
      // ❌ We DON'T know if the payment went through or not!
      // The request may have been received and processed AFTER our timeout
      // Retrying without idempotency → double charge!
      throw new Error('Payment service timed out — status unknown');
    }
  }
}

// ✅ Correct handling in asynchronous systems:
async function callPaymentServiceIdempotent(orderId, amount) {
  const idempotencyKey = `payment:${orderId}`;  // deterministic key

  try {
    return await fetch('http://payment-service/charge', {
      method: 'POST',
      headers: { 'Idempotency-Key': idempotencyKey },
      body: JSON.stringify({ orderId, amount }),
      signal: AbortSignal.timeout(5000)
    }).then(r => r.json());
  } catch (err) {
    // Safe to retry with the same idempotency key
    // If payment went through: service returns same response
    // If it didn't: service processes it now
    return await retryWithBackoff(() =>
      fetch('http://payment-service/charge', {
        headers: { 'Idempotency-Key': idempotencyKey },
        ...
      })
    );
  }
}
```

**Key lesson:** In real distributed systems (which are always partially asynchronous):
- Timeouts don't mean failure — just "we don't know yet."
- Design for idempotency and retries.
- Accept that you cannot achieve perfect consistency without coordination overhead.
- This is why Kafka at-least-once delivery + idempotent consumers is the standard pattern.

> 📖 Reference: [Distributed Systems — Kleppmann Book](https://dataintensive.net/)

---

## 7. Observability — Logs, Metrics, Traces

---

**Q41. What are the three pillars of observability? Explain logs, metrics, and traces.**

**Answer:**

**Observability** is the ability to understand the internal state of a system from its external outputs. The three pillars provide complementary views:

```
Logs:   What happened?        → Discrete events with context
Metrics: How much/many/fast? → Aggregated numbers over time
Traces:  Where did it happen? → Request journey across services
```

**1. Logs — structured event records:**

```js
// Structured logs (JSON) — queryable in production
logger.info('Payment processed', {
  orderId:       'ord-123',
  userId:        42,
  amount:        150.00,
  currency:      'USD',
  processorId:   'stripe',
  duration_ms:   245,
  requestId:     'req-abc-xyz',
  timestamp:     '2024-06-01T12:00:00.123Z'
});
// Query in Datadog: @orderId:ord-123 AND @amount:>100 AND @status:failed
```

**2. Metrics — time-series numbers:**

```js
// Prometheus metrics
const prometheus = require('prom-client');

// Counter: monotonically increasing (never decreases)
const httpRequestsTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Histogram: distribution of values (latency, request size)
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'route'],
  buckets: [0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5]
});

// Gauge: can go up or down (memory, active connections)
const activeConnections = new prometheus.Gauge({
  name: 'active_db_connections',
  help: 'Active database connections'
});

app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer({ method: req.method, route: req.route?.path });
  res.on('finish', () => {
    end(); // record duration
    httpRequestsTotal.inc({ method: req.method, route: req.route?.path, status_code: res.statusCode });
  });
  next();
});
```

**3. Traces — request journey across services:**

```js
// OpenTelemetry distributed tracing
const { trace } = require('@opentelemetry/api');
const tracer = trace.getTracer('order-service');

app.post('/orders', async (req, res) => {
  // Automatically: if incoming request has trace headers, continue that trace
  const span = tracer.startSpan('create-order');

  try {
    span.setAttribute('user.id', req.user.id);
    span.setAttribute('order.items_count', req.body.items.length);

    // Child span for DB operation
    const dbSpan = tracer.startSpan('db.insert-order', { parent: span });
    const order = await db.createOrder(req.body);
    dbSpan.end();

    // Child span for external service call (payment)
    const paySpan = tracer.startSpan('payment.charge', { parent: span });
    await paymentService.charge(order.id, order.total);
    paySpan.end();

    span.setAttribute('order.id', order.id);
    span.setStatus({ code: SpanStatusCode.OK });
    res.status(201).json(order);
  } catch (err) {
    span.recordException(err);
    span.setStatus({ code: SpanStatusCode.ERROR });
    throw err;
  } finally {
    span.end();
  }
});
// Trace shows: API Gateway (10ms) → Order Service (45ms) → DB (5ms) + Payment (38ms)
// In Jaeger/Zipkin: see entire request timeline as a Gantt chart
```

> 📖 Reference: [Three Pillars of Observability — Honeycomb](https://www.honeycomb.io/blog/three-pillars-of-observability)

---

**Q42. What is distributed tracing? What is a trace ID and a span?**

**Answer:**

**Distributed tracing** tracks a single request as it flows through multiple services, building a complete picture of what happened and how long each step took.

```
User request → API Gateway → Order Service → DB
                          → Payment Service → Stripe API
                          → Notification Service → SendGrid

Without tracing: "The request took 2 seconds. Which service caused it?"
With tracing:   "API Gateway: 5ms, Order Service: 180ms (DB: 5ms, Payment: 175ms!)"
→ Immediately identify Payment Service is the bottleneck
```

**Trace ID and Span:**

```
Trace ID: A unique identifier for the ENTIRE request journey across ALL services
Span:     A single unit of work within the trace (one service operation, one DB call)
Parent Span: A span that contains child spans (hierarchical)

Example trace:
Trace ID: abc-123-xyz

Span 1 (root):   POST /orders             duration: 200ms  service: api-gateway
  Span 2:        create-order             duration: 195ms  service: order-service
    Span 3:      db.insert-order          duration:   8ms  service: order-service (DB)
    Span 4:      payment.charge           duration: 185ms  service: order-service
      Span 5:    stripe.charge-card       duration: 180ms  service: payment-service
        Span 6:  http.post stripe-api     duration: 175ms  service: payment-service (external)
    Span 7:      notify.send-confirmation duration:   2ms  service: order-service (async)
```

```js
// Propagation: trace headers passed between services
// Order Service calls Payment Service:
const response = await fetch('http://payment-service/charge', {
  headers: {
    'traceparent': `00-${traceId}-${spanId}-01`,  // W3C trace context standard
    // Payment Service receives this → continues the same trace → adds its own spans
  }
});

// OpenTelemetry auto-instruments HTTP clients:
// @opentelemetry/instrumentation-http automatically adds trace headers to all outgoing requests
// No manual code needed for propagation ✅
```

> 📖 Reference: [Distributed Tracing — OpenTelemetry](https://opentelemetry.io/docs/concepts/observability-primer/)

---

**Q43. What is Prometheus? How does it scrape and store metrics?**

**Answer:**

**Prometheus** is an open-source monitoring and alerting system that collects metrics by **pulling** (scraping) HTTP endpoints from your services at regular intervals and stores them in a time-series database.

```
Architecture:

Your App (exposes /metrics)   ← Prometheus scrapes every 15s
Your DB (postgres_exporter)   ← Prometheus scrapes every 15s
Your K8s nodes (node-exporter)← Prometheus scrapes every 15s

All scraped metrics → Prometheus TSDB (local storage, 15-day default retention)
                    → Grafana queries via PromQL for dashboards
                    → AlertManager evaluates rules → PagerDuty/Slack alerts
```

```js
// Exposing metrics from Node.js app
const prometheus = require('prom-client');
prometheus.collectDefaultMetrics(); // CPU, memory, event loop, GC, etc.

// Custom business metrics
const ordersCreated = new prometheus.Counter({
  name: 'orders_created_total',
  help: 'Total number of orders created',
  labelNames: ['status', 'payment_method']
});

const orderValue = new prometheus.Histogram({
  name: 'order_value_dollars',
  help: 'Order value distribution',
  buckets: [10, 50, 100, 500, 1000, 5000]
});

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', prometheus.register.contentType);
  res.end(await prometheus.register.metrics());
});

// Use metrics
app.post('/orders', async (req, res) => {
  const order = await createOrder(req.body);
  ordersCreated.inc({ status: 'success', payment_method: order.paymentMethod });
  orderValue.observe(order.total);
  res.json(order);
});
```

```yaml
# Prometheus config (prometheus.yml)
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'api-service'
    static_configs:
      - targets: ['api-service:3000']  # scrapes /metrics endpoint

  # Kubernetes auto-discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: 'true'  # scrape pods annotated with prometheus.io/scrape: "true"
```

```promql
# PromQL queries:
# Request rate (per second, 5-min window)
rate(http_requests_total[5m])

# Error rate (percentage)
sum(rate(http_requests_total{status_code=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))

# p95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Alert: error rate > 5%
ALERT HighErrorRate
  IF (rate(http_requests_total{status="500"}[5m]) / rate(http_requests_total[5m])) > 0.05
  FOR 2m
  LABELS { severity = "critical" }
```

> 📖 Reference: [Prometheus Overview — Prometheus Docs](https://prometheus.io/docs/introduction/overview/)

---

**Q44. What is Grafana? How does it complement Prometheus?**

**Answer:**

**Grafana** is an open-source visualization and dashboarding platform. It queries data sources (Prometheus, Loki, InfluxDB, Elasticsearch, etc.) and renders beautiful dashboards, graphs, and alerts.

```
Prometheus: stores metrics, powerful querying (PromQL), alerting
Grafana:    visualizes metrics, creates dashboards, routes alerts to channels

They complement each other:
Prometheus = data collection + storage + PromQL engine
Grafana    = visualization + dashboard + multi-source correlation
```

**Practical setup:**

```yaml
# docker-compose.yml — local monitoring stack
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports: ['9090:9090']

  grafana:
    image: grafana/grafana:latest
    ports: ['3000:3000']
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources

  # Log aggregation
  loki:
    image: grafana/loki:latest
    ports: ['3100:3100']

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log
      - ./promtail-config.yml:/etc/promtail/config.yml
```

```json
// Grafana dashboard JSON (provisioned from code)
{
  "title": "API Service Overview",
  "panels": [
    {
      "title": "Request Rate",
      "type": "graph",
      "targets": [{
        "expr": "sum(rate(http_requests_total[1m])) by (route)",
        "legendFormat": "{{route}}"
      }]
    },
    {
      "title": "Error Rate",
      "type": "stat",
      "targets": [{
        "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100",
        "legendFormat": "Error %"
      }],
      "thresholds": { "steps": [
        { "value": 0, "color": "green" },
        { "value": 1, "color": "yellow" },
        { "value": 5, "color": "red" }
      ]}
    },
    {
      "title": "p95 Latency",
      "targets": [{
        "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
      }]
    }
  ]
}
```

**Grafana + Loki (logs):**
```
Grafana can correlate:
- See a spike in error rate in Prometheus graph
- Click the spike time → automatically query Loki for logs at that exact time
- See the error logs that caused the spike
→ One tool for metrics + logs + traces (Grafana Tempo)
```

> 📖 Reference: [Grafana Docs](https://grafana.com/docs/grafana/latest/introduction/)

---

**Q45. What is an SLI, SLO, and SLA? How do they differ?**

**Answer:**

These three terms define service reliability commitments at different levels:

| Term | Full Name | What it is | Example |
|------|-----------|-----------|---------|
| **SLI** | Service Level Indicator | A metric that measures service behavior | 99.2% of requests succeeded in the last 30 days |
| **SLO** | Service Level Objective | An internal target for an SLI | 99.9% of requests must succeed (our goal) |
| **SLA** | Service Level Agreement | A contract with external consequences | 99.9% uptime or customer gets a refund |

```
SLI ← the actual measured number
SLO ← the internal target we aim to meet
SLA ← the external commitment with financial consequences

SLO should be stricter than SLA:
SLO: 99.95% availability (internal goal)
SLA: 99.9% availability (external commitment)
Buffer: 0.05% → room to miss SLO without breaching SLA
```

**Real implementation:**

```js
// SLIs as Prometheus metrics and PromQL queries

// SLI 1: Availability (success rate)
// "Fraction of successful HTTP requests over all requests"
const availabilitySLI = `
  sum(rate(http_requests_total{status_code!~"5.."}[30d]))
  /
  sum(rate(http_requests_total[30d]))
`;
// Current: 99.92% → SLO: 99.9% → ✅ within SLO

// SLI 2: Latency
// "Fraction of requests that complete within 200ms"
const latencySLI = `
  sum(rate(http_request_duration_seconds_bucket{le="0.2"}[30d]))
  /
  sum(rate(http_request_duration_seconds_count[30d]))
`;
// Current: 94.3% → SLO: 95% → ❌ violating SLO!

// SLI 3: Error rate
// "Fraction of requests returning 5xx errors"
const errorRateSLI = `
  sum(rate(http_requests_total{status_code=~"5.."}[30d]))
  /
  sum(rate(http_requests_total[30d]))
`;
```

**Error budget:** `1 - SLO`

```
SLO: 99.9% availability for 30 days
Error budget: 0.1% = 43.8 minutes of downtime allowed per month

Spending error budget:
- Incident: 15 minutes down → 15 min spent from 43.8 min budget
- Remaining: 28.8 minutes this month

If error budget depleted:
→ Freeze new feature deployments until next period
→ Focus on reliability improvements
→ This creates natural incentive to balance velocity vs reliability
```

> 📖 Reference: [SLI/SLO/SLA — Google SRE Book](https://sre.google/sre-book/service-level-objectives/)

---

**Q46. What is an alert fatigue? How do you design a good alerting strategy?**

**Answer:**

**Alert fatigue** happens when the on-call engineer receives so many alerts (many of them false positives, noisy, or non-actionable) that they start ignoring them — including real critical ones.

```
Signs of alert fatigue:
- "I just acknowledge and silence it, it usually resolves itself"
- Hundreds of alerts per day, most not requiring action
- On-call is woken up at 3 AM for things that auto-recovered
- Team ignores alerts → real incident missed → major outage
```

**Designing a good alerting strategy:**

```yaml
# ── Rule 1: Alert on symptoms, not causes ─────────────────────────────
# BAD: "CPU > 80%" — so what? Maybe it's fine. Users aren't impacted yet.
# GOOD: "Error rate > 1%" — users are experiencing errors right now

# BAD: "DB connection pool at 80% capacity"
# GOOD: "API p95 latency > 500ms" (which may be caused by DB pool exhaustion)

# ── Rule 2: Every alert must be actionable ────────────────────────────
# If you can't DO something about it, don't page
# Either fix it or add it to a dashboard for passive monitoring

# ── Rule 3: Alerts should have clear urgency levels ────────────────────
# P1 (page immediately):  User-facing outage, revenue impact, data loss
# P2 (page during hours): Degraded performance, non-critical failure
# P3 (ticket only):       Warning threshold crossed, investigate tomorrow

# ── Prometheus alerting rules ─────────────────────────────────────────
groups:
- name: api-service
  rules:

  # P1: Error rate spike (wake someone up NOW)
  - alert: HighErrorRate
    expr: |
      sum(rate(http_requests_total{status_code=~"5.."}[5m]))
      /
      sum(rate(http_requests_total[5m])) > 0.05
    for: 2m          # sustained for 2 minutes (not a blip)
    labels:
      severity: critical
    annotations:
      summary: "High error rate on {{ $labels.service }}"
      description: "Error rate is {{ $value | humanizePercentage }} (threshold: 5%)"
      runbook: "https://runbooks.company.com/high-error-rate"

  # P2: Elevated latency (investigate soon)
  - alert: HighLatency
    expr: |
      histogram_quantile(0.95,
        rate(http_request_duration_seconds_bucket[5m])
      ) > 1.0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "p95 latency above 1s"

  # P3: Error budget burn rate (plan for reliability work)
  - alert: ErrorBudgetBurning
    expr: |
      (
        1 - (
          sum(rate(http_requests_total{status_code!~"5.."}[1h]))
          / sum(rate(http_requests_total[1h]))
        )
      ) > (0.001 * 14.4)   # burning 14.4x faster than budget allows
    labels:
      severity: warning     # don't page — just create a ticket
```

**Alert checklist:**
- [ ] Does this alert indicate a user is being impacted right now?
- [ ] Is there a specific action to take when this fires?
- [ ] Is the `for` duration long enough to avoid false positives?
- [ ] Does the alert link to a runbook?
- [ ] Who is responsible for triaging this alert?

> 📖 Reference: [Alerting — Google SRE Book](https://sre.google/sre-book/monitoring-distributed-systems/)

---

## 8. Advanced API Design

---

**Q47. What is API backward compatibility? How do you make a breaking change safely?**

**Answer:**

**API backward compatibility** means existing clients continue to work unchanged after an API update. A **breaking change** removes, renames, or fundamentally changes behavior in a way that breaks existing clients.

**Breaking vs non-breaking changes:**

```
✅ Non-breaking (safe to deploy):
  - Add a new optional field to response
  - Add a new endpoint
  - Add a new optional query parameter
  - Add a new enum value (if clients handle unknown values)

❌ Breaking changes (require versioning):
  - Rename/remove a field from response
  - Change field type (string → integer)
  - Change URL structure
  - Make optional parameter required
  - Remove an endpoint
  - Change authentication method
```

**Safe breaking change process:**

```js
// ── Expand-contract pattern ───────────────────────────────────────────

// CURRENT API (v1):
// GET /users/:id → { id, name, email, created_at }

// DESIRED CHANGE: rename "name" → "fullName", add "firstName", "lastName"

// ❌ BAD: Just rename it
// { id, fullName, email } ← breaks all clients using "name" field

// ✅ GOOD: Three-phase migration

// Phase 1: Expand — return BOTH old and new fields simultaneously
app.get('/v1/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json({
    id:         user.id,
    name:       user.fullName,    // OLD field (kept for backward compat)
    fullName:   user.fullName,    // NEW field (clients can start using)
    firstName:  user.firstName,   // NEW field
    lastName:   user.lastName,    // NEW field
    email:      user.email,
    created_at: user.createdAt,
  });
  // All old clients still work (name is there)
  // New clients can use fullName/firstName/lastName
});

// Phase 2: Migrate — communicate deprecation, update all known clients
// Add deprecation headers:
res.set('Deprecation', 'true');
res.set('Sunset', 'Sat, 31 Dec 2025 23:59:59 GMT');
res.set('Link', '</v2/users/:id>; rel="successor-version"');

// Phase 3: Contract — remove old field (after Sunset date)
app.get('/v2/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json({
    id:        user.id,
    fullName:  user.fullName,  // "name" field removed
    firstName: user.firstName,
    lastName:  user.lastName,
    email:     user.email,
    createdAt: user.createdAt,
  });
});
```

> 📖 Reference: [API Compatibility — Stripe Blog](https://stripe.com/blog/api-versioning)

---

**Q48. What is long polling? How does it compare to WebSockets and SSE (Server-Sent Events)?**

**Answer:**

All three solve the problem of pushing real-time updates to clients, but with different trade-offs:

| | Long Polling | SSE | WebSocket |
|--|-------------|-----|----------|
| Connection | HTTP (open and close per update) | HTTP (persistent, one-way) | TCP (persistent, bidirectional) |
| Direction | Client initiates; server holds | Server → Client only | Full duplex |
| Protocol | HTTP/1.1 | HTTP/1.1 or HTTP/2 | WS (ws:// or wss://) |
| Reconnect | Must re-request | Browser auto-reconnects | Manual reconnect logic |
| Server complexity | Low | Low | Medium |
| Scaling | Hard (many held connections) | Medium | Hard (stateful, sticky sessions) |
| Use case | Simple notifications, legacy | Live feeds, notifications | Chat, games, collaborative editing |

**Long Polling:**
```js
// Client keeps asking "any updates?" — server holds the connection open until there are some

// Server: hold connection until event or timeout
app.get('/poll/notifications', requireAuth, async (req, res) => {
  const timeout = 30000; // 30 seconds max hold
  const startTime = Date.now();

  while (Date.now() - startTime < timeout) {
    const notifications = await getNewNotifications(req.user.id, req.query.since);
    if (notifications.length > 0) {
      return res.json({ notifications });
    }
    await sleep(1000); // check every 1s
  }
  res.json({ notifications: [] }); // timeout, client will re-request
});

// Client: immediately re-request after getting a response
async function poll(since = Date.now()) {
  const response = await fetch(`/poll/notifications?since=${since}`);
  const { notifications } = await response.json();
  processNotifications(notifications);
  poll(Date.now()); // immediately request again
}
```

**Server-Sent Events (SSE):**
```js
// Server: stream events to client over persistent connection
app.get('/events/notifications', requireAuth, (req, res) => {
  res.writeHead(200, {
    'Content-Type':  'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection':    'keep-alive',
  });

  // Subscribe to Redis pub/sub
  const subscriber = redis.duplicate();
  subscriber.subscribe(`user:${req.user.id}:notifications`);

  subscriber.on('message', (channel, message) => {
    res.write(`data: ${message}\n\n`); // SSE format
  });

  // Heartbeat to keep connection alive
  const heartbeat = setInterval(() => res.write(': heartbeat\n\n'), 30000);

  req.on('close', () => {
    clearInterval(heartbeat);
    subscriber.unsubscribe();
    subscriber.disconnect();
  });
});

// Client:
const eventSource = new EventSource('/events/notifications');
eventSource.onmessage = (e) => showNotification(JSON.parse(e.data));
// Browser auto-reconnects if connection drops ✅
```

**WebSocket:**
```js
// Server: bidirectional real-time communication
const WebSocket = require('ws');
const wss = new WebSocket.Server({ server });

wss.on('connection', (ws, req) => {
  const userId = getUserFromCookie(req);

  ws.on('message', (data) => {
    const message = JSON.parse(data);
    if (message.type === 'CHAT') {
      broadcastToRoom(message.roomId, message);
    }
  });

  // Push updates to client
  redis.subscribe(`user:${userId}`, (msg) => ws.send(msg));

  ws.on('close', () => cleanup(userId));
});

// Client: full two-way communication
const ws = new WebSocket('wss://api.myapp.com/ws');
ws.send(JSON.stringify({ type: 'JOIN_ROOM', roomId: 'general' }));
ws.onmessage = (e) => renderMessage(JSON.parse(e.data));
```

> 📖 Reference: [Long Polling vs WebSockets — Ably](https://ably.com/topic/long-polling)

---

**Q49. What is an API contract? What is contract testing?**

**Answer:**

An **API contract** is a formal agreement between a provider (service exposing the API) and consumer (service calling it) about what the request and response look like.

**Contract testing** verifies both sides honor the contract, independently — without needing both services running simultaneously.

```
Without contract testing:
Order Service assumes:
  GET /users/:id returns { id, name, email, role }

User Service changes to:
  GET /users/:id returns { id, fullName, email, permissions }
  (removed "name", renamed to "fullName", changed "role" to "permissions")

Order Service breaks in production when it calls User Service!
The change wasn't coordinated — contract was broken silently.

With contract testing:
Consumer test: "I expect GET /users/:id to return { name, email, role }"
This contract is uploaded to Pact Broker.
When User Service changes: "Does this change break any consumer contracts?"
Pact Broker: "YES — Order Service expects 'name' field, you removed it"
→ Caught BEFORE deployment ✅
```

**Pact contract testing example:**

```js
// ── Consumer (Order Service) — defines what it expects ──────────────
const { Pact } = require('@pact-foundation/pact');

const provider = new Pact({
  consumer: 'order-service',
  provider: 'user-service',
  port: 1234,
});

describe('User Service contract', () => {
  before(() => provider.setup());
  after(() => provider.finalize());

  it('returns user by ID', async () => {
    // Define the expected interaction
    await provider.addInteraction({
      state: 'user 42 exists',
      uponReceiving: 'a request for user 42',
      withRequest: {
        method: 'GET',
        path: '/users/42',
        headers: { Authorization: like('Bearer token') }
      },
      willRespondWith: {
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: {
          id:    42,
          name:  like('Alice'),           // type matching (any string)
          email: like('alice@example.com'),
          role:  term({ generate: 'user', matcher: 'user|admin' })
        }
      }
    });

    // Run actual consumer code against mock server
    const user = await UserServiceClient.getUser(42);
    expect(user.name).toBeDefined();
    expect(user.role).toMatch(/user|admin/);
  });
});
// This generates a "pact" file (contract) uploaded to Pact Broker

// ── Provider (User Service) — verifies it satisfies contracts ──────
const { Verifier } = require('@pact-foundation/pact');

describe('User Service provider verification', () => {
  it('verifies against consumer contracts', () => {
    return new Verifier({
      provider:            'user-service',
      providerBaseUrl:     'http://localhost:3001',
      pactBrokerUrl:       'https://pact-broker.company.com',
      publishVerificationResult: true,
      // Sets up "user 42 exists" state before running tests
      stateHandlers: {
        'user 42 exists': async () => {
          await db.createTestUser({ id: 42, name: 'Alice', email: 'alice@test.com', role: 'user' });
        }
      }
    }).verifyProvider();
  });
});
// If User Service changes response shape → verification fails → blocked from deploy ✅
```

> 📖 Reference: [Contract Testing — Pact Docs](https://docs.pact.io/)

---

**Q50. What is an OpenAPI / Swagger specification? Why is it important?**

**Answer:**

**OpenAPI (formerly Swagger)** is a language-agnostic standard for describing REST APIs in a machine-readable YAML/JSON format. It documents endpoints, request/response schemas, authentication, and more.

```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title:   Order Service API
  version: 1.0.0
  description: Manage customer orders

servers:
  - url: https://api.myapp.com/v1

paths:
  /orders:
    post:
      summary: Create a new order
      operationId: createOrder
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
            example:
              userId: 42
              items:
                - productId: 1, quantity: 2
              shippingAddress:
                street: "123 Main St"
                city: "New York"
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    Order:
      type: object
      required: [id, status, total, createdAt]
      properties:
        id:        { type: integer, example: 99 }
        status:    { type: string, enum: [pending, confirmed, shipped, delivered] }
        total:     { type: number, format: float, example: 150.00 }
        createdAt: { type: string, format: date-time }

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

**Why it's important:**

```js
// 1. Auto-generate documentation → Swagger UI
// swagger-ui-express: serve interactive docs
const swaggerUi = require('swagger-ui-express');
const spec = require('./openapi.yaml');
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(spec));
// Visit /api-docs → interactive UI, try requests in browser ✅

// 2. Generate client SDKs automatically
// npx @openapitools/openapi-generator-cli generate \
//   -i openapi.yaml -g typescript-fetch -o ./sdk/
// → TypeScript client with all types and methods generated ✅

// 3. Request validation (middleware)
const OpenApiValidator = require('express-openapi-validator');
app.use(OpenApiValidator.middleware({
  apiSpec: './openapi.yaml',
  validateRequests: true,   // validate all incoming requests against spec
  validateResponses: true   // validate all outgoing responses against spec
}));
// Any request not matching spec → 400 error automatically ✅
// Any response not matching spec → 500 error (catch schema bugs)

// 4. Contract testing
// Generate Pact tests from OpenAPI spec
// Test server-side API against the spec
// Multiple teams aligned on the same contract
```

> 📖 Reference: [OpenAPI Specification — Swagger](https://swagger.io/specification/)

---

## 9. Cloud Fundamentals

---

**Q51. What is the difference between IaaS, PaaS, and SaaS?**

**Answer:**

These define how much the cloud provider manages vs. how much you manage:

```
On-Premises: You manage EVERYTHING
IaaS: Cloud manages hardware + networking. You manage OS, runtime, app, data.
PaaS: Cloud manages hardware + OS + runtime. You manage app + data.
SaaS: Cloud manages everything. You just use the software.
```

| | You Manage | Provider Manages | Example |
|--|-----------|-----------------|---------|
| **On-Prem** | Everything | Nothing | Your own data center |
| **IaaS** | OS, Runtime, App, Data | Hardware, Network, Virtualization | AWS EC2, Azure VMs, GCP Compute Engine |
| **PaaS** | App, Data | OS, Runtime, Middleware, Hardware | Heroku, AWS Elastic Beanstalk, Google App Engine |
| **SaaS** | Just data/configuration | Everything | Salesforce, Gmail, Slack, GitHub |

**Backend developer perspective:**

```js
// ── IaaS (EC2): You manage the OS and runtime ──────────────────────
// SSH into EC2 instance, install Node.js, configure nginx, manage certificates,
// apply OS patches, set up monitoring, configure firewall...
// Lots of control, lots of work.

// ── PaaS (Heroku): Just push code ─────────────────────────────────
// git push heroku main
// → Heroku detects Node.js, installs deps, starts app, handles routing
// → Add Heroku Postgres add-on (managed DB)
// → No SSH, no OS config, no nginx setup
// → But: less control, can't customize OS

// ── Container PaaS (AWS ECS/GKE): push Docker image ────────────────
// docker build + push → AWS ECS runs the container
// You define: CPU/memory requirements, ports, env vars
// AWS handles: scheduling, health checks, load balancing

// ── Serverless (AWS Lambda): write functions ──────────────────────
exports.handler = async (event) => {
  const { userId } = event.pathParameters;
  const user = await getUserById(userId);
  return { statusCode: 200, body: JSON.stringify(user) };
};
// You write function code. AWS handles: servers, scaling, OS, runtime.
// Pay per invocation. Auto-scales from 0 to millions.
// This is more PaaS / FaaS (Function as a Service)
```

> 📖 Reference: [IaaS vs PaaS vs SaaS — IBM](https://www.ibm.com/topics/iaas-paas-saas)

---

**Q52. What is object storage? How is it different from block storage and file storage?**

**Answer:**

| | Object Storage | Block Storage | File Storage |
|--|---------------|--------------|-------------|
| Structure | Flat namespace, key-value (buckets/objects) | Raw blocks (like a hard drive) | Hierarchical filesystem |
| Access | HTTP APIs (REST) | Mounted as disk via iSCSI/NVMe | NFS, SMB/CIFS mount |
| Metadata | Rich custom metadata per object | Minimal | Directory attributes |
| Use case | Images, videos, backups, static assets | DB data files, VM disks | Shared filesystems, NAS |
| Scalability | Effectively unlimited | Limited to single disk size | Limited |
| AWS | S3 | EBS | EFS |

```js
// ── Object Storage (S3) — HTTP access, immutable objects ─────────────
const { S3Client, PutObjectCommand, GetObjectCommand } = require('@aws-sdk/client-s3');
const s3 = new S3Client({ region: 'us-east-1' });

// Upload user avatar (object = key + data + metadata)
await s3.send(new PutObjectCommand({
  Bucket:      'my-app-avatars',
  Key:         `users/${userId}/avatar.jpg`,    // flat namespace (no real directory)
  Body:        imageBuffer,
  ContentType: 'image/jpeg',
  Metadata:    { userId: userId.toString(), uploadedAt: new Date().toISOString() },
  ACL:         'public-read'
}));

// Objects are immutable: to "update", you upload a new object with the same key
// Object URL: https://my-app-avatars.s3.amazonaws.com/users/42/avatar.jpg

// ── Block Storage (EBS) — mounted as disk, mutable in place ──────────
// EC2 instance has /dev/xvda (root volume, EBS block storage)
// Database (PostgreSQL) stores its data files on EBS
// Files can be read/written/modified at the byte level
// Fast random access → good for databases

// ── File Storage (EFS) — shared filesystem ───────────────────────────
// Multiple EC2 instances mount the same EFS filesystem at /efs
// Files are shared — all instances see the same files
// Good for: shared config files, CMS media, legacy apps expecting NFS
// Slower than EBS, but accessible from multiple instances
```

**When to use object storage in backend:**
- User-uploaded files (avatars, documents, images, videos).
- Static website assets served via CDN.
- Application logs, backups, data exports.
- Data lake raw storage.
- ML training datasets.

> 📖 Reference: [Storage Types — AWS](https://aws.amazon.com/what-is/object-storage/)

---

**Q53. What is a CDN (Content Delivery Network)? How does it improve backend performance?**

**Answer:**

A **CDN** is a globally distributed network of servers ("edge nodes") that cache and serve content from the location closest to the user — reducing latency by avoiding long-distance network hops to your origin server.

```
Without CDN:
User in Mumbai → requests image → travels to US-East server → response → Mumbai
Round trip: ~250ms

With CDN (Cloudflare/CloudFront):
User in Mumbai → requests image → Cloudflare Mumbai edge node → cached → response
Round trip: ~10ms (99% of the time, image was already cached locally)
Origin server never contacted for cached content ✅
```

**What CDNs cache and what they don't:**

```js
// ── Cache static assets at CDN (forever or long TTL) ─────────────────
// HTML: cache-busted by content hash in filename
// e.g., app.a3f2b9c1.js, style.7d8e9f0a.css
app.use('/static', express.static('public', {
  maxAge: '1y',              // browser caches for 1 year
  setHeaders: (res) => {
    res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
    // CDN caches for 1 year. Browser caches for 1 year.
    // Next deploy: new file hash → new URL → cache automatically busted
  }
}));

// ── Cache API responses at CDN (short TTL) ────────────────────────────
app.get('/api/products', async (req, res) => {
  const products = await getProducts();
  res.set('Cache-Control', 'public, max-age=60, s-maxage=300');
  // max-age=60:    browser caches 1 minute
  // s-maxage=300:  CDN (shared cache) caches 5 minutes
  res.json(products);
});

// ── Never cache at CDN ────────────────────────────────────────────────
app.get('/api/user/profile', requireAuth, async (req, res) => {
  const user = await getUser(req.user.id);
  res.set('Cache-Control', 'private, no-store');
  // private: CDN won't cache (user-specific data)
  // no-store: browser won't cache either
  res.json(user);
});

// ── CDN as DDoS protection ────────────────────────────────────────────
// Cloudflare absorbs DDoS at edge, only legitimate traffic reaches origin
// Bot detection, WAF rules applied at edge
// Origin IP is never exposed (attackers can't bypass CDN)
```

**CDN benefits for backend:**
- Offload 80-99% of static asset requests from origin servers.
- Reduce origin server bandwidth costs.
- DDoS mitigation at the edge.
- SSL termination at edge (faster TLS handshake).
- Global low-latency for a worldwide user base.

> 📖 Reference: [CDN — Cloudflare](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)

---

**Q54. What is auto-scaling? What is the difference between scheduled and dynamic scaling?**

**Answer:**

**Auto-scaling** automatically adjusts the number of compute instances (EC2, pods, containers) based on demand — scaling out when load increases, scaling in when it decreases.

| | Scheduled Scaling | Dynamic Scaling |
|--|------------------|----------------|
| Trigger | Time-based (cron schedule) | Metric-based (CPU, RPS, queue depth) |
| Predictability | Works for predictable traffic patterns | Works for unpredictable spikes |
| Reaction time | Pre-emptive (scales before load arrives) | Reactive (scales after load detected) |
| Use case | Business hours, weekly patterns | Flash sales, viral traffic |

```python
# ── AWS Auto Scaling: scheduled ────────────────────────────────────────
import boto3
autoscaling = boto3.client('autoscaling')

# Scale up Monday-Friday at 8 AM EST (before business hours)
autoscaling.put_scheduled_update_group_action(
    AutoScalingGroupName='api-asg',
    ScheduledActionName='scale-up-business-hours',
    Recurrence='0 13 * * MON-FRI',  # 8 AM EST = 13 UTC
    DesiredCapacity=20,
    MinSize=10
)

# Scale down at 8 PM EST (after business hours)
autoscaling.put_scheduled_update_group_action(
    AutoScalingGroupName='api-asg',
    ScheduledActionName='scale-down-off-hours',
    Recurrence='0 1 * * *',   # 1 AM UTC = 8 PM EST
    DesiredCapacity=3,
    MinSize=2
)
```

```yaml
# ── Kubernetes HPA: dynamic (CPU-based) ─────────────────────────────
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60  # scale when avg CPU > 60%
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60   # don't scale up more than once/min
      policies:
      - type: Percent
        value: 100              # can double pods in one step
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300  # wait 5 min before scaling down
      policies:
      - type: Pods
        value: 2                # remove at most 2 pods at a time
        periodSeconds: 60
```

**Combine both:**
```
Scheduled: pre-scale to 20 pods at 8 AM (known daily pattern)
Dynamic: if a viral spike hits at noon → auto-scales beyond the scheduled 20
Best of both worlds: predictable baseline + elastic spike handling
```

> 📖 Reference: [Auto Scaling — AWS](https://aws.amazon.com/autoscaling/)

---

**Q55. What is a VPC (Virtual Private Cloud)? Why do production backends run inside one?**

**Answer:**

A **VPC** is a logically isolated section of the cloud where you define your own virtual network — IP ranges, subnets, routing tables, and security groups. It's like having your own private data center within AWS.

```
Without VPC:
Your RDS database has a public IP → Anyone on the internet can try to connect!
Your Redis → exposed to internet → security nightmare

With VPC:
Internet → Internet Gateway → Public Subnet (load balancer only)
                                        ↓
                              Private Subnet (app servers)
                                        ↓
                              Private Subnet (databases, Redis)
                              ← No internet access! Only accessible within VPC ✅
```

```
VPC Architecture (production):
┌─────────────────────────────────────────────────────────────────┐
│  VPC: 10.0.0.0/16                                               │
│                                                                  │
│  ┌─────────────────────────────────────────────────────┐        │
│  │  Public Subnet 10.0.1.0/24 (us-east-1a)             │        │
│  │  Load Balancer (internet-facing)                     │        │
│  │  Bastion host (SSH jump server)                      │        │
│  └─────────────────────────────────────────────────────┘        │
│                         ↕                                        │
│  ┌─────────────────────────────────────────────────────┐        │
│  │  Private Subnet 10.0.2.0/24 (us-east-1a)            │        │
│  │  API servers (EC2 / ECS tasks)                       │        │
│  │  → Can reach internet via NAT Gateway (for npm install)│      │
│  │  → NOT reachable from internet directly              │        │
│  └─────────────────────────────────────────────────────┘        │
│                         ↕                                        │
│  ┌─────────────────────────────────────────────────────┐        │
│  │  Private Subnet 10.0.3.0/24 (us-east-1a)            │        │
│  │  RDS PostgreSQL (no public IP!)                      │        │
│  │  ElastiCache Redis (no public IP!)                   │        │
│  │  → Only accessible from Private Subnet above         │        │
│  └─────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

```hcl
# Terraform VPC setup
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  enable_dns_hostnames = true
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private_app" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.2.0/24"
}

resource "aws_subnet" "private_db" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.3.0/24"
}

# Security group: only allow DB access from app subnet
resource "aws_security_group" "rds" {
  vpc_id = aws_vpc.main.id
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]  # only from app servers ✅
  }
  # No ingress from internet!
}
```

> 📖 Reference: [VPC — AWS Docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

---

## 10. Engineering Practices & Team Collaboration

---

**Q56. What is a post-mortem / incident review? What makes a good blameless post-mortem?**

**Answer:**

A **post-mortem** (or incident review) is a structured analysis conducted after an incident (outage, data loss, security breach) to understand what happened, why it happened, and how to prevent it in future.

**Blameless** means: the goal is to improve systems and processes, NOT to find and punish the person who made a mistake.

```
Why blameless matters:
If people fear blame, they hide problems, don't report near-misses, don't take ownership.
Blameless culture: people share openly → better learning → fewer future incidents.
"The system allowed this mistake to happen. How do we fix the system?"

Google's SRE principle:
"Individuals don't cause incidents — system design and processes do."
```

**Good post-mortem structure:**

```markdown
# Incident Post-Mortem: Payment Service Outage
**Date:** 2024-06-01 | **Duration:** 47 minutes | **Severity:** P1
**Author:** [name] | **Reviewers:** [names]

## Impact
- 23,000 payment attempts failed
- $180,000 in lost revenue
- 3 enterprise customers affected

## Timeline (UTC)
- 14:23 — Deploy v2.4.1 to production
- 14:31 — Alert fires: payment error rate > 5%
- 14:35 — On-call engineer paged, begins investigation
- 14:42 — Root cause identified: connection pool exhaustion
- 15:03 — Rollback to v2.4.0 complete
- 15:10 — Error rate returns to normal

## Root Cause
v2.4.1 introduced a database query that held connections open
for the duration of a long Stripe API call (avg 3s).
With 20 max connections and ~10 req/sec, pool exhausted in ~2 min.
The connection was NOT released in the finally block (programmer oversight).

## Contributing Factors
- No connection timeout configured (allowed connections to be held indefinitely)
- Load testing didn't simulate Stripe API latency
- Code review missed the missing finally block

## What Went Well
- Alert fired within 8 minutes of deploy
- On-call engineer identified root cause in 7 minutes
- Rollback completed in 21 minutes

## Action Items
| Action | Owner | Due |
|--------|-------|-----|
| Add connection timeout to DB config (5s max) | @devA | June 3 |
| Add connection hold time to deployment dashboard | @devB | June 5 |
| Add Stripe latency simulation to load test | @devC | June 8 |
| Add "connection released in finally?" to code review checklist | @teamLead | June 3 |

## Lessons Learned
Connection pools need timeouts. Load tests must simulate realistic 3rd-party latencies.
Checklist items catch patterns that reviews miss.
```

> 📖 Reference: [Blameless Post-Mortems — Google SRE Book](https://sre.google/sre-book/postmortem-culture/)

---

**Q57. What is technical debt? How do you communicate it to non-technical stakeholders?**

**Answer:**

**Technical debt** is the implied cost of future rework caused by choosing an expedient (quick/easy) solution now instead of a better long-term approach.

```
Like financial debt:
Taking a shortcut = borrowing against the future
The "interest" = increasing maintenance cost, slower feature development, more bugs

Types:
Deliberate (intentional): "We'll do this properly after launch"
Inadvertent: "We didn't know better at the time"
Bit rot: Code that was fine but environment changed around it
```

**Communicating to non-technical stakeholders:**

```
❌ Technical framing (they don't care):
"Our authentication module uses deprecated OAuth 1.0 and the JWT library is 3 major versions
behind. The user service has cyclomatic complexity of 42 and zero unit test coverage."

✅ Business impact framing:
"Our login system has accumulated shortcuts that are now causing problems:
  - It takes 3x longer to add new sign-in methods (email, SSO, etc.) than it should
  - It's the #1 source of security bug reports this quarter
  - Our last security audit flagged it as high risk
  - Fixing it properly: 6 weeks of work
  - NOT fixing it: every new auth feature takes 3 weeks instead of 1, risk of breach"

Frame as:
→ Velocity impact: "Feature X takes 3 weeks instead of 1 because of Y"
→ Risk: "If left unaddressed, risk of [incident] increases by Z%"
→ Cost: "We'll spend $X in maintenance over 12 months vs $Y to fix now"
→ ROI: "Fixing this enables us to ship [business feature] 2 months earlier"
```

```js
// Tracking technical debt systematically
// Option 1: Dedicated tickets/epic in your issue tracker
// Label: "tech-debt" | Priority: Medium | Estimate: 3 weeks

// Option 2: TODO/FIXME comments with ticket references
// TODO(TECH-123): Replace with proper retry logic using exponential backoff
// FIXME(TECH-456): This O(n²) loop needs to be refactored before user count > 10K

// Option 3: Architecture Decision Records (ADR)
// "We chose X knowing it has these limitations.
//  When [condition] is met, we should revisit and migrate to Y."
```

> 📖 Reference: [Technical Debt — Martin Fowler](https://martinfowler.com/bliki/TechnicalDebt.html)

---

**Q58. What is an Architecture Decision Record (ADR)? Why should teams write them?**

**Answer:**

An **ADR** is a short document that captures an architectural decision — what was decided, why, and what alternatives were considered. It creates a durable record of why the system looks the way it does.

```
Without ADRs (6 months later):
"Why are we using RabbitMQ instead of Kafka?"
"Why is the auth service written in Go while everything else is Node.js?"
→ Nobody remembers. The original decision-maker left. No documentation.
→ Team debates the same question again. Wasted time.

With ADRs:
"Why RabbitMQ?" → Look up ADR-042: "At the time, team had RabbitMQ expertise and
the throughput was sufficient. Kafka was rejected because it required a ZooKeeper
cluster we weren't ready to operate. Revisit if message volume exceeds 50K/sec."
→ Instant clarity. Context preserved. Faster onboarding.
```

**ADR template:**

```markdown
# ADR-042: Use RabbitMQ for Task Queue

## Status
Accepted (2024-03-15)

## Context
We need a message broker to decouple background jobs (email sending, report generation)
from request processing. The system processes ~500 tasks/second at peak.

## Decision
We will use RabbitMQ (not Kafka, not Redis Streams) as our primary task queue.

## Rationale
- Team has 3 engineers with RabbitMQ experience
- Throughput requirement (500/s) is well within RabbitMQ's capabilities
- RabbitMQ has superior support for complex routing (exchanges, bindings)
  which we need for per-customer worker isolation
- Managed offering available on CloudAMQP

## Alternatives Considered

### Apache Kafka
- Rejected: overkill for our use case; requires ZooKeeper/KRaft cluster
  we don't have operational expertise for
- Rejected: log-based model unsuitable for tasks we want deleted after ACK
- Revisit if: throughput > 50K/sec OR we need event replay/stream processing

### Redis Streams
- Rejected: Redis is our cache; mixing caching and queuing creates risk
- Rejected: less mature consumer group management
- Rejected: limited routing capabilities

### AWS SQS
- Considered: simpler operationally, but limited routing and no local dev support
- May revisit if we migrate fully to AWS-native stack

## Consequences
- Team must maintain RabbitMQ cluster (or use CloudAMQP managed service)
- All new async work should go through RabbitMQ
- If throughput grows beyond ~10K/sec, revisit Kafka migration

## Review Date
2025-03-15 (annual review)
```

> 📖 Reference: [ADRs — Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)

---

**Q59. What is pair programming? What are its benefits and when is it not appropriate?**

**Answer:**

**Pair programming** is a practice where two developers work together at a single workstation — one writes code (driver), the other reviews and guides (navigator). Roles switch regularly.

```
Driver:    writes code, focuses on implementation details
Navigator: watches, thinks about design, catches bugs, suggests approaches, checks docs

Every 25-30 minutes: switch roles
```

**Benefits:**

```
1. Knowledge sharing: navigator learns how driver thinks; both learn the codebase together
   → New team member paired with senior → onboards 3x faster

2. Fewer bugs: "two sets of eyes" catches errors in real-time
   → Studies show 15-20% fewer defects than solo coding
   → Cheaper than code review catching bugs after the fact

3. Better design: discussing trade-offs out loud reveals assumptions
   → "Why did you use a HashMap here?" → "Oh, a Set would be better actually"

4. Code review built-in: navigator reviews as driver writes
   → Pair-programmed code often doesn't need a separate PR review

5. Bus factor: two people understand the same code
   → One person on vacation? The other knows the code
```

**When NOT to use it:**

```
❌ Exploratory/research work: need to read docs, experiment, prototype alone
   → Better: spike solo, then pair to implement

❌ Simple/repetitive tasks: writing boilerplate, copy-paste refactoring
   → Pairing adds overhead without proportional benefit

❌ Mismatched skill level with no intent to teach
   → If senior does all the work while junior watches → not real pairing

❌ Different timezones / async environments
   → Remote async pairing is harder; use async code review instead

❌ When one person is fatigued or has context-switching issues
   → 4-6 hours of deep pairing per day is the max for most people
```

> 📖 Reference: [Pair Programming — Martin Fowler](https://martinfowler.com/articles/on-pair-programming.html)

---

**Q60. What is the difference between a code review and a design review? What do you look for in each?**

**Answer:**

| | Code Review | Design Review |
|--|------------|--------------|
| When | After code is written (PR review) | Before coding begins |
| Focus | Implementation correctness, quality | Architecture, approach, trade-offs |
| Artifact | Code diff (PR) | Design doc, diagram, RFC |
| Scope | Module/feature level | System/cross-service level |
| Goal | Catch bugs, enforce standards, share knowledge | Align on approach, catch design flaws early |
| Cost to change | Low (change the code) | Low (change the document) |

**Code Review checklist:**

```
Correctness:
✅ Does it do what the PR description says?
✅ Are edge cases handled? (null, empty, max values, concurrent access)
✅ Are errors handled and returned properly?
✅ Is there no obvious N+1 query issue?

Security:
✅ Is user input validated and sanitized?
✅ Are SQL queries parameterized?
✅ Is authorization checked for every sensitive operation?
✅ Are secrets not hardcoded?

Quality:
✅ Are functions single-purpose and reasonably sized?
✅ Is the naming clear and consistent?
✅ Is there adequate test coverage for new/changed logic?
✅ Is there unnecessary code or dead code?
✅ Can I understand what it does without reading every line?

Operational:
✅ Are important events logged?
✅ Are metrics/traces emitted for new code paths?
✅ Is the migration backward-compatible?
```

**Design Review checklist:**

```
Correctness:
✅ Does the design achieve the stated goals?
✅ Are all edge cases and failure modes considered?
✅ Is the consistency model appropriate?

Architecture:
✅ Does it fit the existing architecture patterns?
✅ Are service responsibilities clearly bounded?
✅ Is there unnecessary coupling between components?
✅ Are there simpler alternatives that achieve the same goal?

Operations:
✅ How will this be monitored and debugged in production?
✅ How does it handle failure of dependent services?
✅ Is the data migration strategy safe (zero-downtime)?
✅ What is the rollback plan?

Performance:
✅ What are the expected throughput and latency characteristics?
✅ Does the design hold up at 10x current scale?
✅ Are there potential bottlenecks?

Trade-offs:
✅ What are the explicit trade-offs made? (consistency vs availability, etc.)
✅ Are there significant technical debts incurred? If so, are they tracked?
```

> 📖 Reference: [Code Review Best Practices — Google Engineering](https://google.github.io/eng-practices/review/)

---

## 🎯 Quick Revision Cheatsheet

| # | Topic | Key Point |
|---|-------|-----------|
| Q1 | Microservices vs Monolith | Micro=independently deployable, DB per service; Mono=simple, all-or-nothing |
| Q2 | Microservices trade-offs | Benefits: scale/deploy independently; Costs: network latency, distributed data, ops overhead |
| Q3 | Service discovery | Client-side=app queries registry; Server-side=LB queries registry (Kubernetes default) |
| Q4 | API Gateway | Single entry point; handles auth, rate limiting, routing, SSL termination, logging |
| Q5 | Inter-service communication | Sync=REST/gRPC (wait for response); Async=Kafka/queues (fire and forget) |
| Q6 | Circuit Breaker | CLOSED→OPEN after N failures; fail-fast when OPEN; HALF-OPEN to test recovery |
| Q7 | Saga pattern | Orchestration=central coordinator; Choreography=event-driven, each service reacts |
| Q8 | Outbox pattern | Write to DB + outbox in ONE transaction; relay publishes outbox to Kafka |
| Q9 | Distributed transactions | Impossible without 2PC (which blocks); use Sagas + eventual consistency instead |
| Q10 | Sidecar pattern | Helper container in same pod; handles logging, proxying, TLS without changing app |
| Q11 | Kafka vs RabbitMQ | Kafka=log-based, retain, replay, multiple independent consumers; RabbitMQ=message queue, delete after ACK |
| Q12 | Kafka partitions | Topic=N partitions; 1 consumer per partition in a group; more partitions=more parallelism |
| Q13 | Queue vs Pub/Sub | Queue=one consumer gets msg; Pub/Sub=ALL subscribers get msg |
| Q14 | Event-driven arch | Loose coupling, resilient, scalable; challenges: eventual consistency, debugging |
| Q15 | Event Sourcing | Store events not state; replay events to rebuild state; full audit trail |
| Q16 | CQRS | Separate write model (commands) from read model (queries); different data stores optimized per use case |
| Q17 | Kafka consumer lag | End offset - committed offset; fix by adding consumers (≤ partition count) or batching |
| Q18 | CI pipeline | Lint→Typecheck→Unit Tests→Integration Tests→Security Audit→Build→Docker Build |
| Q19 | CD vs CDP | Delivery=manual prod approval; Deployment=auto-deploy to prod if tests pass |
| Q20 | Blue-green | Two identical envs; instant traffic switch; instant rollback; 2x infra cost |
| Q21 | Canary | Gradual rollout (5%→25%→50%→100%); real user validation; small blast radius |
| Q22 | Feature flags | Deploy code with flag OFF; enable gradually; rollback instantly without redeploy |
| Q23 | IaC | Infrastructure defined as code; reproducible, versioned, automated; Terraform, CDK |
| Q24 | Rollback strategy | K8s: `kubectl rollout undo`; Blue-green: switch LB; DB: expand-contract migrations |
| Q25 | Kubernetes | Orchestrates containers across cluster; self-healing, autoscaling, service discovery |
| Q26 | Pod/Deployment/Service | Pod=container(s); Deployment=manages pods; Service=stable endpoint to pods |
| Q27 | ConfigMap vs Secret | ConfigMap=non-sensitive config; Secret=base64 credentials (use external secrets in prod) |
| Q28 | HPA | Auto-scales pod count based on CPU/memory/custom metrics |
| Q29 | Liveness vs Readiness | Liveness=is it alive? (restart if not); Readiness=is it ready for traffic? (remove from LB if not) |
| Q30 | Namespaces | Virtual cluster within K8s; isolate by env or team; resource quotas per namespace |
| Q31 | Sharding | Split rows across multiple DB servers by shard key; range/hash/directory strategies |
| Q32 | Replication | Primary handles writes, replicas serve reads; WAL streamed from primary to replicas |
| Q33 | WAL | Changes written to WAL before data files; ensures durability + enables replication |
| Q34 | Sync vs Async replication | Sync=zero data loss, higher latency; Async=faster writes, potential data loss on failure |
| Q35 | Time-series DB | Optimized for timestamp-indexed data; auto-downsampling, compression; InfluxDB, TimescaleDB |
| Q36 | Two Generals | Proves reliable consensus over unreliable network is impossible; design for idempotency |
| Q37 | Distributed lock | Redis SET NX EX; Lua for atomic release; use Redlock for multi-node |
| Q38 | Consistent hashing | Keys distributed on ring; adding/removing nodes remaps ~1/N keys (not all) |
| Q39 | Quorum | W + R > N ensures consistency; Cassandra ONE=fastest, QUORUM=balanced, ALL=strictest |
| Q40 | Async distributed system | Real networks are async; timeout≠failure; design idempotent retries |
| Q41 | Three pillars | Logs=what happened; Metrics=how many/fast; Traces=where in which service |
| Q42 | Distributed tracing | Trace=full journey; Span=one operation; propagated via headers (W3C traceparent) |
| Q43 | Prometheus | Pull-based metrics scraping; PromQL for queries; rate(), histogram_quantile() |
| Q44 | Grafana | Visualization layer over Prometheus; dashboards, alerts, multi-source correlation |
| Q45 | SLI/SLO/SLA | SLI=measured value; SLO=internal target; SLA=external contract with consequences |
| Q46 | Alert fatigue | Too many non-actionable alerts; fix: alert on symptoms, require runbook, sustained duration |
| Q47 | API backward compat | Expand-contract: add new + keep old → migrate → remove old; add Deprecation header |
| Q48 | Long polling/SSE/WS | Long polling=re-request loop; SSE=server push (HTTP); WebSocket=bidirectional TCP |
| Q49 | Contract testing | Consumer defines expectations; provider verifies; Pact Broker stores contracts |
| Q50 | OpenAPI/Swagger | Machine-readable API spec; auto-generates docs, SDKs, validation middleware |
| Q51 | IaaS/PaaS/SaaS | IaaS=manage OS+app; PaaS=just deploy app; SaaS=just configure |
| Q52 | Object/Block/File storage | Object=S3 HTTP flat; Block=EBS disk; File=EFS shared filesystem |
| Q53 | CDN | Cache at edge globally; reduce latency 10-25x; offload origin; DDoS protection |
| Q54 | Auto-scaling | Scheduled=time-based (predictable); Dynamic=metric-based (reactive to load) |
| Q55 | VPC | Private cloud network; public subnet for LB; private subnets for app+DB; no internet to DB |
| Q56 | Blameless post-mortem | Timeline + root cause + contributing factors + action items; blame system not people |
| Q57 | Technical debt | Frame as business impact: velocity/risk/cost; track as tickets; schedule paydown |
| Q58 | ADR | Decision + context + alternatives considered + consequences; audit trail for "why" |
| Q59 | Pair programming | Driver writes, navigator guides; benefits: knowledge sharing, fewer bugs; not for all tasks |
| Q60 | Code vs Design review | Code=implementation correctness; Design=architecture alignment before coding begins |

---

## 🤝 Contributing

Found a better explanation or a cleaner code example?
- Fork the repo, update the answer, open a PR.
- PR title format: `[Solution-Day-04] Improve answer for Q6`

---

> ⭐ Star this repo if it helped you prepare!
