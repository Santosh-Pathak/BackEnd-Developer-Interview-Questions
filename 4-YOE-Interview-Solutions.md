# ✅ Solutions — Day 5: 4 Years Experience (4 YOE)

> **Series:** Backend Interview Prep · Fresher → 10 Years of Experience  
> **Level:** Mid-Level Developer — 4 Years of Experience  
> **Questions File:** [day-05-4yoe.md](./day-05-4yoe.md)  
> **Total Answers:** 60  

> 💡 **Tip:** At 4 YOE, interviews shift from "do you know this?" to "can you design this?" and "what trade-offs did you make?" Every answer here explains the reasoning behind decisions.

---

## 📚 Table of Contents

1. [System Design — Core Patterns](#1-system-design--core-patterns)
2. [Redis — Deep Dive](#2-redis--deep-dive)
3. [Advanced Testing Strategies](#3-advanced-testing-strategies)
4. [Performance Engineering](#4-performance-engineering)
5. [Advanced Microservices Patterns](#5-advanced-microservices-patterns)
6. [Search & Indexing Systems](#6-search--indexing-systems)
7. [Data Pipelines & Streaming](#7-data-pipelines--streaming)
8. [Advanced Security](#8-advanced-security)
9. [Reliability & Fault Tolerance](#9-reliability--fault-tolerance)
10. [Engineering Leadership Basics](#10-engineering-leadership-basics)

---

## 1. System Design — Core Patterns

---

**Q1. How would you design a URL shortener (like bit.ly)? Walk through the key components.**

**Answer:**

A URL shortener maps a long URL to a short code (e.g., `bit.ly/a3Xk9`) and redirects users when they visit the short link. This is a classic system design question because it touches encoding, storage, caching, and scale.

**Requirements (clarify first):**
- 100M URLs created/day, 10B redirects/day (read-heavy: 100:1 ratio)
- Short codes: 7 characters (62^7 = 3.5 trillion unique codes — enough)
- Links never expire (unless specified)
- Analytics (click count, geo) — optional

**Core Architecture:**

```
Write path:  Client → API → ID Generator → DB → return short code
Read path:   Client → API → Cache (Redis) → DB (if miss) → 301/302 redirect

Components:
1. ID Generator    — unique numeric ID for each URL
2. Base62 Encoder  — converts numeric ID → short code
3. Storage (DB)    — stores short_code → original_url mapping
4. Cache (Redis)   — caches hot short_code lookups
5. API Service     — handles create + redirect
6. Analytics       — click tracking (async, Kafka)
```

**Short code generation:**

```js
// Base62 encoding: digits + lowercase + uppercase = 62 chars
const CHARS = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';

function encodeBase62(num) {
  let result = '';
  while (num > 0) {
    result = CHARS[num % 62] + result;
    num = Math.floor(num / 62);
  }
  return result.padStart(7, '0'); // always 7 chars
}

function decodeBase62(str) {
  return str.split('').reduce((acc, char) => acc * 62 + CHARS.indexOf(char), 0);
}

// encodeBase62(1)         → '0000001'
// encodeBase62(1000000)   → '04c92'
// encodeBase62(3521614606207) → 'zzzzzzz' (max 7-char base62)
```

**ID generation at scale:**

```js
// Option 1: DB auto-increment (simple, sequential)
// Problem: single DB = bottleneck at scale

// Option 2: Snowflake IDs (distributed, sortable)
// 64-bit: 41-bit timestamp + 10-bit machine ID + 12-bit sequence
// Each server generates unique IDs without coordination

// Option 3: Redis INCR (fast, centralized)
const nextId = await redis.incr('url:counter'); // atomic increment
const shortCode = encodeBase62(nextId);

// Option 4: Pre-generated ID ranges
// ID service pre-assigns ranges to each app server
// Server 1 uses IDs 1-1000, Server 2 uses 1001-2000, etc.
// Servers request new range when current is exhausted
```

**Database schema:**

```sql
CREATE TABLE urls (
  id          BIGSERIAL PRIMARY KEY,
  short_code  VARCHAR(10)  NOT NULL UNIQUE,
  long_url    TEXT         NOT NULL,
  user_id     BIGINT,
  created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
  expires_at  TIMESTAMPTZ,
  click_count BIGINT       NOT NULL DEFAULT 0
);

CREATE INDEX idx_urls_short_code ON urls(short_code);
-- short_code is the hot lookup path — must be indexed
```

**API implementation:**

```js
const express = require('express');
const app = express();

// CREATE: POST /shorten
app.post('/shorten', async (req, res) => {
  const { longUrl, customCode, expiresIn } = req.body;

  // Validate URL
  if (!isValidUrl(longUrl)) {
    return res.status(400).json({ error: 'Invalid URL' });
  }

  // Check if URL already shortened (dedup)
  const existing = await db.query(
    'SELECT short_code FROM urls WHERE long_url = $1 AND user_id = $2',
    [longUrl, req.user?.id]
  );
  if (existing.rows[0]) {
    return res.json({ shortUrl: `https://sho.rt/${existing.rows[0].short_code}` });
  }

  // Generate short code
  let shortCode = customCode;
  if (!shortCode) {
    const id = await redis.incr('url:id:counter');
    shortCode = encodeBase62(id);
  }

  // Store
  await db.query(
    'INSERT INTO urls (short_code, long_url, user_id, expires_at) VALUES ($1, $2, $3, $4)',
    [shortCode, longUrl, req.user?.id, expiresIn ? new Date(Date.now() + expiresIn) : null]
  );

  // Cache immediately (avoid cold start on first visit)
  await redis.setex(`url:${shortCode}`, 86400, longUrl);

  res.json({ shortUrl: `https://sho.rt/${shortCode}` });
});

// REDIRECT: GET /:code
app.get('/:code', async (req, res) => {
  const { code } = req.params;

  // 1. Check cache first
  const cached = await redis.get(`url:${code}`);
  if (cached) {
    // Track click asynchronously
    kafka.produce('url-clicks', { code, timestamp: Date.now(), ip: req.ip });
    return res.redirect(301, cached); // 301 = permanent (browser caches)
  }

  // 2. Cache miss → hit DB
  const result = await db.query(
    'SELECT long_url, expires_at FROM urls WHERE short_code = $1',
    [code]
  );

  if (!result.rows[0]) {
    return res.status(404).json({ error: 'Short URL not found' });
  }

  const { long_url, expires_at } = result.rows[0];

  // Check expiry
  if (expires_at && new Date() > new Date(expires_at)) {
    return res.status(410).json({ error: 'This link has expired' });
  }

  // Populate cache
  await redis.setex(`url:${code}`, 86400, long_url);

  // Track click
  kafka.produce('url-clicks', { code, timestamp: Date.now(), ip: req.ip });

  res.redirect(301, long_url);
});
```

**Scale considerations:**

```
Read throughput: 10B redirects/day = ~115K req/sec
Cache hit rate: 80%+ (most redirects are for popular links)
Only 23K req/sec hit the DB → easily handled by read replicas

Storage: 100M URLs/day × 365 days × 500 bytes = ~18 TB/year
→ Use tiered storage: hot data in PostgreSQL, cold in S3

301 vs 302:
301 Permanent: browser caches → future visits don't hit your server → saves bandwidth
302 Temporary: browser always hits your server → better for analytics accuracy
→ Use 302 if click analytics are critical, 301 otherwise
```

> 📖 Reference: [URL Shortener Design — Educative](https://www.educative.io/courses/grokking-the-system-design-interview/m2ygV4E81AR)

---

**Q2. How would you design a rate limiter? What algorithms can be used?**

**Answer:**

A rate limiter controls how many requests a client (by IP, user ID, or API key) can make within a time window. It protects backend services from abuse, DoS, and overload.

**5 main algorithms:**

```
1. Fixed Window Counter   — simplest, but has burst problem at window boundary
2. Sliding Window Log     — accurate, but high memory (stores every request timestamp)
3. Sliding Window Counter — good balance of accuracy and memory
4. Token Bucket           — most common; allows bursts up to bucket size
5. Leaky Bucket           — smooth output rate; queues excess requests
```

**Fixed Window Counter:**

```js
// Count requests per fixed time window (e.g., per minute)
async function fixedWindowRateLimit(userId, limit = 100, windowSeconds = 60) {
  const window = Math.floor(Date.now() / (windowSeconds * 1000));
  const key = `ratelimit:fixed:${userId}:${window}`;

  const count = await redis.incr(key);
  if (count === 1) await redis.expire(key, windowSeconds);

  return {
    allowed: count <= limit,
    remaining: Math.max(0, limit - count),
    resetAt: (window + 1) * windowSeconds * 1000
  };
}

// Problem: boundary burst
// Limit: 100/minute. Window resets at 12:00:00
// User sends 100 at 11:59:50 → allowed (window 1)
// Window resets at 12:00:00
// User sends 100 at 12:00:01 → allowed (window 2)
// → 200 requests in 11 seconds! Not what we wanted.
```

**Sliding Window Counter (best balance):**

```js
// Weighted combination of current + previous window
async function slidingWindowRateLimit(userId, limit = 100, windowSeconds = 60) {
  const now = Date.now();
  const currentWindow = Math.floor(now / (windowSeconds * 1000));
  const previousWindow = currentWindow - 1;

  // Get counts from both windows
  const [currentCount, prevCount] = await redis.mget(
    `ratelimit:${userId}:${currentWindow}`,
    `ratelimit:${userId}:${previousWindow}`
  );

  // How far into current window are we? (0.0 to 1.0)
  const elapsed = (now % (windowSeconds * 1000)) / (windowSeconds * 1000);

  // Weighted count: previous window weighted by remaining time
  const weightedCount =
    parseInt(currentCount || 0) +
    parseInt(prevCount || 0) * (1 - elapsed);

  if (weightedCount >= limit) {
    return { allowed: false, remaining: 0 };
  }

  // Increment current window
  await redis.incr(`ratelimit:${userId}:${currentWindow}`);
  await redis.expire(`ratelimit:${userId}:${currentWindow}`, windowSeconds * 2);

  return {
    allowed: true,
    remaining: Math.floor(limit - weightedCount - 1)
  };
}
```

**Token Bucket (most flexible, allows controlled bursts):**

```js
// Tokens refill at a constant rate. Each request consumes 1 token.
// If no tokens → request rejected
async function tokenBucketRateLimit(userId, {
  capacity = 100,    // max bucket size (max burst)
  refillRate = 10,   // tokens added per second
} = {}) {
  const key = `tokenbucket:${userId}`;
  const now = Date.now() / 1000; // seconds

  // Lua script for atomic read-modify-write
  const luaScript = `
    local key = KEYS[1]
    local capacity = tonumber(ARGV[1])
    local refillRate = tonumber(ARGV[2])
    local now = tonumber(ARGV[3])

    local data = redis.call('HMGET', key, 'tokens', 'lastRefill')
    local tokens = tonumber(data[1]) or capacity
    local lastRefill = tonumber(data[2]) or now

    -- Refill tokens based on elapsed time
    local elapsed = now - lastRefill
    tokens = math.min(capacity, tokens + elapsed * refillRate)

    if tokens >= 1 then
      tokens = tokens - 1
      redis.call('HMSET', key, 'tokens', tokens, 'lastRefill', now)
      redis.call('EXPIRE', key, 3600)
      return {1, math.floor(tokens)} -- allowed, remaining
    else
      redis.call('HMSET', key, 'tokens', tokens, 'lastRefill', now)
      redis.call('EXPIRE', key, 3600)
      return {0, 0} -- rejected
    end
  `;

  const [allowed, remaining] = await redis.eval(
    luaScript, 1, key, capacity, refillRate, now
  );

  return { allowed: allowed === 1, remaining };
}

// Express middleware
const rateLimitMiddleware = (options) => async (req, res, next) => {
  const identifier = req.user?.id || req.ip;
  const result = await tokenBucketRateLimit(identifier, options);

  res.setHeader('X-RateLimit-Limit', options.capacity);
  res.setHeader('X-RateLimit-Remaining', result.remaining);

  if (!result.allowed) {
    return res.status(429).json({
      error: 'Too Many Requests',
      retryAfter: Math.ceil(1 / options.refillRate) // seconds until 1 token refills
    });
  }
  next();
};

app.use('/api/', rateLimitMiddleware({ capacity: 100, refillRate: 10 }));
app.use('/api/auth/login', rateLimitMiddleware({ capacity: 5, refillRate: 0.1 }));
```

> 📖 Reference: [Rate Limiting Algorithms — Cloudflare Blog](https://blog.cloudflare.com/counting-things-a-lot-of-different-things/)

---

**Q3. What is the token bucket algorithm? How does it differ from the leaky bucket algorithm?**

**Answer:**

Both model request flow as a "bucket" but with opposite perspectives:

**Token Bucket:** Tokens accumulate in the bucket at a fixed rate. Each request consumes a token. Requests can burst as long as tokens are available.

```
Token Bucket:
    [🪙🪙🪙🪙🪙] ← tokens added at rate R
         ↑
    request arrives → consume 1 token → allowed
    no tokens → reject request

Behavior: BURSTY output allowed (up to bucket size)
```

**Leaky Bucket:** Requests enter the bucket and leak out at a fixed rate. Excess requests overflow (dropped) or wait in the queue.

```
Leaky Bucket:
    Requests IN → [🟦🟦🟦🟦] → leak out at FIXED rate R
                       ↑
               overflow → reject

Behavior: SMOOTH output at exactly rate R
```

**Code comparison:**

```js
// ── Token Bucket: allows bursting ─────────────────────────────────────
class TokenBucket {
  constructor(capacity, refillRatePerSec) {
    this.capacity = capacity;
    this.tokens = capacity;   // start full
    this.refillRate = refillRatePerSec;
    this.lastRefill = Date.now();
  }

  consume(tokens = 1) {
    // Refill based on elapsed time
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.capacity, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;

    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true;  // allowed
    }
    return false;   // rejected
  }
}

const bucket = new TokenBucket(10, 2); // 10 capacity, 2 tokens/sec
bucket.consume(); // true  (9 tokens left)
bucket.consume(); // true  (8 tokens left)
// ... 10 rapid requests → all pass (burst of 10 allowed!)
// 11th request → false (no tokens)
// Wait 500ms → 1 token refilled → allowed again

// ── Leaky Bucket: smooth output ──────────────────────────────────────
class LeakyBucket {
  constructor(capacity, leakRatePerSec) {
    this.capacity = capacity;
    this.queue = [];
    this.leakRate = leakRatePerSec;

    // Leak at fixed rate
    setInterval(() => {
      if (this.queue.length > 0) {
        const request = this.queue.shift();
        request.resolve(); // process one request per interval
      }
    }, 1000 / leakRatePerSec); // e.g., every 100ms for 10 req/sec
  }

  async add(request) {
    if (this.queue.length >= this.capacity) {
      throw new Error('Bucket overflow — request rejected');
    }
    return new Promise((resolve, reject) => {
      this.queue.push({ request, resolve, reject });
    });
  }
}
// Output: ALWAYS at most leakRate requests/sec, regardless of input burst
// No bursting — output is perfectly smooth
```

**Key difference:**

| | Token Bucket | Leaky Bucket |
|--|-------------|-------------|
| Output pattern | Bursty (up to capacity) | Smooth (exactly rate R) |
| Request handling | Accept or reject immediately | Accept or queue (then process at fixed rate) |
| Use case | APIs where burst is OK | Protecting downstream at precise rate |
| Examples | AWS API Gateway, most rate limiters | Video streaming bitrate control, network shapers |

> 📖 Reference: [Token Bucket vs Leaky Bucket — GeeksForGeeks](https://www.geeksforgeeks.org/difference-between-token-bucket-and-leaky-bucket-algorithm/)

---

**Q4. How would you design a notification system that supports email, SMS, and push notifications?**

**Answer:**

A notification system decouples your application from delivery channels. It must be reliable (no lost notifications), scalable (millions/day), and extensible (add new channels easily).

**Architecture:**

```
Services → Notification API → Message Queue (Kafka) → Channel Workers → Providers
                                                     → Email Worker  → SendGrid/SES
                                                     → SMS Worker    → Twilio
                                                     → Push Worker   → FCM/APNs
```

**Database schema:**

```sql
-- User notification preferences
CREATE TABLE notification_preferences (
  user_id         BIGINT PRIMARY KEY,
  email_enabled   BOOLEAN DEFAULT TRUE,
  sms_enabled     BOOLEAN DEFAULT FALSE,
  push_enabled    BOOLEAN DEFAULT TRUE,
  email_address   VARCHAR(255),
  phone_number    VARCHAR(20),
  push_tokens     JSONB DEFAULT '[]'  -- array of device tokens
);

-- Notification log (audit + dedup)
CREATE TABLE notifications (
  id              BIGSERIAL PRIMARY KEY,
  user_id         BIGINT NOT NULL,
  type            VARCHAR(100) NOT NULL,  -- 'ORDER_SHIPPED', 'PAYMENT_FAILED'
  channel         VARCHAR(50)  NOT NULL,  -- 'email', 'sms', 'push'
  status          VARCHAR(50)  NOT NULL DEFAULT 'pending',
  payload         JSONB        NOT NULL,
  idempotency_key VARCHAR(255) UNIQUE,    -- prevent duplicate sends
  sent_at         TIMESTAMPTZ,
  failed_at       TIMESTAMPTZ,
  error_message   TEXT,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_notifications_user_type ON notifications(user_id, type, created_at);
CREATE INDEX idx_notifications_status ON notifications(status) WHERE status = 'pending';
```

**Notification API service:**

```js
class NotificationService {
  async send({ userId, type, data, channels = null }) {
    // 1. Load user preferences
    const prefs = await this.getUserPreferences(userId);

    // 2. Determine which channels to use
    const activeChannels = channels || this.getChannelsFromPreferences(prefs, type);

    // 3. Render template for each channel
    const template = await this.templateEngine.render(type, data);

    // 4. Enqueue for each channel (async, reliable)
    const jobs = activeChannels.map(channel => ({
      userId,
      channel,
      type,
      idempotencyKey: `${userId}:${type}:${data.eventId}:${channel}`,
      payload: {
        to:      this.getDestination(prefs, channel),
        subject: template[channel].subject,
        body:    template[channel].body,
        data:    data
      }
    }));

    // 5. Publish to Kafka (each channel has its own topic/partition)
    await Promise.all(jobs.map(job =>
      this.kafka.produce(`notifications.${job.channel}`, {
        key:   userId.toString(),
        value: JSON.stringify(job)
      })
    ));

    // 6. Log in DB
    await this.db.query(`
      INSERT INTO notifications (user_id, type, channel, payload, idempotency_key)
      VALUES ${jobs.map((_, i) => `($${i*5+1},$${i*5+2},$${i*5+3},$${i*5+4},$${i*5+5})`).join(',')}
      ON CONFLICT (idempotency_key) DO NOTHING
    `, jobs.flatMap(j => [j.userId, j.type, j.channel, JSON.stringify(j.payload), j.idempotencyKey]));
  }
}

// Channel workers (separate processes, independent scaling)
class EmailWorker {
  async processMessage(job) {
    const { userId, idempotencyKey, payload } = job;

    // Idempotency check: already sent?
    const existing = await db.query(
      'SELECT id FROM notifications WHERE idempotency_key = $1 AND status = $2',
      [idempotencyKey, 'sent']
    );
    if (existing.rows[0]) return; // already delivered

    try {
      await sendgrid.send({
        to:      payload.to,
        subject: payload.subject,
        html:    payload.body
      });

      await db.query(
        "UPDATE notifications SET status='sent', sent_at=NOW() WHERE idempotency_key=$1",
        [idempotencyKey]
      );
    } catch (err) {
      await db.query(
        "UPDATE notifications SET status='failed', error_message=$1, failed_at=NOW() WHERE idempotency_key=$2",
        [err.message, idempotencyKey]
      );
      throw err; // re-throw for Kafka retry
    }
  }
}
```

**Priority queues (critical vs marketing):**

```js
// Use separate Kafka topics for different priorities
// ORDER_SHIPPED → high-priority topic → processed immediately
// WEEKLY_DIGEST → low-priority topic → processed during off-peak hours

const TOPICS = {
  HIGH: 'notifications.high',     // transactional: OTP, payment confirmation
  MEDIUM: 'notifications.medium', // order updates, account alerts
  LOW: 'notifications.low'        // marketing, digests
};

// Rate limit by provider (Twilio: 1 SMS/sec per number)
const limiter = new RateLimiter({ tokensPerInterval: 1, interval: 'second' });
```

> 📖 Reference: [Notification System Design — ByteByteGo](https://bytebytego.com/courses/system-design-interview/design-a-notification-system)

---

**Q5. What is a fan-out pattern? When is fan-out-on-write vs fan-out-on-read better?**

**Answer:**

**Fan-out** is the process of delivering one event or piece of content to many recipients. The two strategies differ in WHEN the distribution happens.

**Fan-out-on-Write:** When a user posts, immediately write to all followers' feeds.

```
User A (10M followers) posts:
→ Write to feed of follower 1
→ Write to feed of follower 2
→ Write to feed of 10 million followers
Total: 10M writes per post

Read: User opens feed → read their pre-computed feed (1 fast read) ✅
Write: 10M writes per celebrity post 😱
```

**Fan-out-on-Read:** When a user opens their feed, compute it on the fly by merging followed users' posts.

```
User B (following 500 users) opens feed:
→ Query posts from user 1, user 2, ..., user 500
→ Merge and sort by timestamp
Total: 1 read triggers 500 queries (or N+1 → use JOIN/UNION)

Write: 1 write per post ✅
Read: Expensive computation per feed load 😱
```

**Hybrid approach (Twitter/X strategy):**

```js
class FeedService {
  async publishPost(authorId, post) {
    // Store the post itself
    await db.query('INSERT INTO posts VALUES ($1, $2, $3, $4)',
      [post.id, authorId, post.content, post.createdAt]);

    const author = await User.findById(authorId);

    if (author.followerCount < 10_000) {
      // Small following: fan-out-on-write to all followers' feeds
      await this.fanOutWrite(authorId, post);
    } else {
      // Celebrity (>10K followers): skip fan-out, use fan-out-on-read
      // Just store the post — followers will pull it on read
      await redis.setex(`post:${post.id}`, 86400, JSON.stringify(post));
    }
  }

  async fanOutWrite(authorId, post) {
    // Get all followers
    const followerIds = await db.query(
      'SELECT follower_id FROM follows WHERE followee_id = $1',
      [authorId]
    );

    // Write to each follower's feed cache (Redis sorted set by timestamp)
    const pipeline = redis.pipeline();
    for (const { follower_id } of followerIds.rows) {
      pipeline.zadd(
        `feed:${follower_id}`,
        post.createdAt.getTime(), // score = timestamp (for ordering)
        post.id.toString()
      );
      pipeline.zremrangebyrank(`feed:${follower_id}`, 0, -501); // keep top 500
      pipeline.expire(`feed:${follower_id}`, 86400 * 7); // 7 day TTL
    }
    await pipeline.exec();
  }

  async getFeed(userId, page = 1, limit = 20) {
    // 1. Get pre-computed feed posts (from fan-out-on-write users)
    const cachedPostIds = await redis.zrevrange(
      `feed:${userId}`, (page-1)*limit, page*limit - 1
    );

    // 2. Get followed celebrities (fan-out-on-read)
    const celebrities = await this.getFollowedCelebrities(userId);
    const celebPosts = await Promise.all(
      celebrities.map(id =>
        db.query('SELECT * FROM posts WHERE author_id=$1 ORDER BY created_at DESC LIMIT 5', [id])
      )
    );

    // 3. Merge + sort all posts
    const allPosts = await Promise.all([
      ...cachedPostIds.map(id => db.query('SELECT * FROM posts WHERE id=$1', [id])),
      ...celebPosts.flat().map(p => Promise.resolve({ rows: [p] }))
    ]);

    return allPosts
      .flatMap(r => r.rows)
      .sort((a, b) => b.created_at - a.created_at)
      .slice(0, limit);
  }
}
```

| | Fan-out-on-Write | Fan-out-on-Read |
|--|-----------------|----------------|
| Read speed | Fast (pre-computed) | Slow (computed on read) |
| Write speed | Slow (N writes per post) | Fast (1 write) |
| Best for | Users with small follower counts | Celebrities/influencers |
| Storage | High (N copies of content) | Low (1 copy) |

> 📖 Reference: [Fan-out — High Scalability](http://highscalability.com/blog/2012/4/4/)

---

**Q6. How would you design a distributed job scheduler?**

**Answer:**

A distributed job scheduler runs tasks at specified times across multiple workers, with reliability, deduplication, and fault tolerance.

**Requirements:**
- Schedule jobs by cron expression or exact datetime
- At-least-once execution (don't miss jobs)
- Exactly-once where possible (use idempotency)
- Fault tolerance (worker crashes don't lose jobs)
- Visibility (see job status, history)

**Architecture:**

```
Scheduler Service → Job Store (DB) → Queue (Redis/Kafka) → Workers
                          ↑
                    Distributed Lock
                   (only 1 scheduler runs at a time)
```

**Implementation:**

```js
// Job definition schema
// jobs table: id, name, cron_expression, handler, payload, status, next_run_at, last_run_at

class DistributedJobScheduler {
  constructor(db, redis, workers) {
    this.db = db;
    this.redis = redis;
    this.workers = workers; // { 'send-report': ReportWorker, 'cleanup': CleanupWorker }
  }

  // Register a job
  async schedule(name, cronExpression, handler, payload = {}) {
    const nextRun = cronParser.parse(cronExpression).next().toDate();

    await this.db.query(`
      INSERT INTO jobs (name, cron_expression, handler, payload, next_run_at, status)
      VALUES ($1, $2, $3, $4, $5, 'scheduled')
      ON CONFLICT (name) DO UPDATE
      SET cron_expression=$2, handler=$3, payload=$4, next_run_at=$5
    `, [name, cronExpression, handler, JSON.stringify(payload), nextRun]);
  }

  // Main scheduler loop (runs on ONE node at a time via distributed lock)
  async start() {
    while (true) {
      const lock = await this.acquireLock('scheduler:master', 30000);
      if (!lock) {
        await sleep(5000); // not the leader — wait
        continue;
      }

      try {
        await this.tick();
      } finally {
        await this.releaseLock('scheduler:master', lock);
      }

      await sleep(1000); // poll every second
    }
  }

  async tick() {
    // Find jobs due to run NOW
    const dueJobs = await this.db.query(`
      SELECT * FROM jobs
      WHERE next_run_at <= NOW()
      AND status = 'scheduled'
      FOR UPDATE SKIP LOCKED   -- atomic: claim a job, skip already-claimed ones
      LIMIT 100
    `);

    for (const job of dueJobs.rows) {
      // Mark as running (prevents double-execution)
      await this.db.query(
        "UPDATE jobs SET status='running', last_run_at=NOW() WHERE id=$1",
        [job.id]
      );

      // Dispatch to worker queue
      await this.redis.lpush('job:queue', JSON.stringify({
        jobId: job.id,
        handler: job.handler,
        payload: job.payload,
        runId: crypto.randomUUID()  // unique per execution
      }));

      // Schedule next run
      const nextRun = cronParser.parse(job.cron_expression).next().toDate();
      await this.db.query(
        "UPDATE jobs SET status='scheduled', next_run_at=$1 WHERE id=$2",
        [nextRun, job.id]
      );
    }
  }
}

// Worker process (multiple instances)
class JobWorker {
  async start() {
    while (true) {
      const [_, jobJson] = await redis.brpop('job:queue', 0); // blocking pop
      const job = JSON.parse(jobJson);

      // Idempotency: check if this runId was already executed
      const alreadyRan = await redis.setnx(`ran:${job.runId}`, '1');
      if (!alreadyRan) {
        console.log('Job already executed, skipping:', job.runId);
        continue;
      }
      await redis.expire(`ran:${job.runId}`, 86400);

      try {
        const handler = this.handlers[job.handler];
        await handler(job.payload);

        await db.query(
          "INSERT INTO job_runs (job_id, run_id, status, completed_at) VALUES ($1,$2,'success',NOW())",
          [job.jobId, job.runId]
        );
      } catch (err) {
        await db.query(
          "INSERT INTO job_runs (job_id, run_id, status, error, completed_at) VALUES ($1,$2,'failed',$3,NOW())",
          [job.jobId, job.runId, err.message]
        );

        // Retry logic: put back in queue with delay
        if (job.retries < 3) {
          await sleep(Math.pow(2, job.retries) * 1000); // exponential backoff
          await redis.lpush('job:queue', JSON.stringify({ ...job, retries: (job.retries || 0) + 1 }));
        }
      }
    }
  }
}
```

> 📖 Reference: [Job Scheduler Design — ByteByteGo](https://bytebytego.com/courses/system-design-interview/distributed-job-scheduler)

---

**Q7. What is back-pressure in a streaming or queue system? How do you handle it?**

**Answer:**

**Back-pressure** occurs when a consumer processes data slower than a producer generates it. Without back-pressure handling, the queue grows unboundedly, eventually causing out-of-memory crashes or massive lag.

```
Normal:
Producer (100 msg/sec) → Queue → Consumer (100 msg/sec) ✅ balanced

Back-pressure:
Producer (1000 msg/sec) → Queue → Consumer (100 msg/sec)
Queue grows by 900 msg/sec → eventually: crash or massive lag 💥
```

**Strategies to handle back-pressure:**

```js
// ── Strategy 1: Bounded queue (drop or block on full) ─────────────────
const queue = new BoundedQueue({ maxSize: 10000 });

producer.on('message', (msg) => {
  if (queue.isFull()) {
    // Option A: Drop (fire-and-forget, acceptable data loss)
    metrics.increment('messages.dropped');
    return;

    // Option B: Block producer (creates back-pressure upstream)
    await queue.put(msg); // blocks until space available

    // Option C: Shed load (reject at API level)
    // Return 503 to caller — they retry later
  }
  queue.push(msg);
});

// ── Strategy 2: Consumer signals back-pressure to producer ────────────
// Reactive Streams / Project Reactor pattern

// Producer checks demand before emitting
class BackPressureProducer extends EventEmitter {
  constructor() {
    super();
    this.demand = 0; // requested by consumer
  }

  request(n) {
    this.demand += n;
    this.emit('demand', n);
  }

  produceWhenDemanded() {
    if (this.demand > 0) {
      const item = this.generateItem();
      this.demand--;
      this.emit('data', item);
    }
    // If no demand: stop producing (pause)
  }
}

// Consumer requests items as it can handle them
producer.request(10); // I can handle 10 items
producer.on('data', async (item) => {
  await processItem(item); // slow processing
  producer.request(1); // I'm ready for 1 more
});

// ── Strategy 3: Kafka consumer back-pressure ───────────────────────────
// Don't auto-commit offsets — only commit after processing
const consumer = kafka.consumer({ groupId: 'my-group' });

await consumer.run({
  autoCommit: false,  // manual commit = natural back-pressure
  eachMessage: async ({ message, resolveOffset, heartbeat }) => {
    await processSlowly(message); // take as long as needed

    // Only advance offset after processing — Kafka won't send more
    // than `max.poll.records` until we commit
    await resolveOffset(message.offset);
    await heartbeat(); // tell broker we're still alive
  }
});

// ── Strategy 4: Adaptive rate limiting ────────────────────────────────
// Slow down producer when queue depth grows
class AdaptiveProducer {
  async produce(item) {
    const queueDepth = await redis.llen('processing:queue');

    // Calculate delay based on queue depth
    if (queueDepth > 5000) {
      await sleep(100); // slow down significantly
    } else if (queueDepth > 1000) {
      await sleep(10);  // slow down moderately
    }
    // else: no delay, produce at full speed

    await redis.lpush('processing:queue', JSON.stringify(item));
  }
}
```

> 📖 Reference: [Backpressure — Reactive Manifesto](https://www.reactivemanifesto.org/glossary#Back-Pressure)

---

**Q8. What is the difference between push and pull models in system design?**

**Answer:**

| | Push Model | Pull Model |
|--|-----------|-----------|
| Who initiates | Producer sends to consumer | Consumer requests from producer |
| Latency | Low (immediate delivery) | Higher (must wait for poll interval) |
| Consumer control | Little (producer decides rate) | Full (consumer decides when to fetch) |
| Back-pressure | Hard (producer may overwhelm consumer) | Natural (consumer only asks for what it can handle) |
| Examples | WebSockets, SSE, Kafka push, email | REST polling, Kafka pull, RSS, GraphQL subscriptions |

```js
// ── PUSH: server streams updates to client ────────────────────────────
// WebSocket: real-time chat
wss.on('connection', (ws) => {
  const listener = (event) => ws.send(JSON.stringify(event));
  eventEmitter.on('new-message', listener);

  ws.on('close', () => eventEmitter.off('new-message', listener));
});
// Problem: if client is slow, server buffers and memory grows

// ── PULL: client asks for updates when ready ──────────────────────────
// Kafka consumer (pull-based broker)
await consumer.run({
  eachMessage: async ({ message }) => {
    await processMessage(message); // take as long as needed
    // Kafka only sends next batch after consumer polls again
  }
});
// Natural back-pressure: consumer controls pace ✅

// ── Hybrid: push with flow control ────────────────────────────────────
// gRPC server streaming with flow control
async function* generateStream(request) {
  for await (const item of dataSource) {
    yield item; // gRPC applies flow control — pauses if client is slow
  }
}

// ── Decision guide ────────────────────────────────────────────────────
/*
Use PUSH when:
- Real-time is critical (chat, live scores, notifications)
- Consumer is always ready and fast
- Low-latency matters more than throughput control

Use PULL when:
- Consumer processes at variable speed
- Back-pressure is important (can't drop messages)
- Batch processing (fetch 100 items, process, fetch next 100)
- Consumer needs to replay or reprocess (Kafka)
*/
```

> 📖 Reference: [Push vs Pull — Martin Fowler](https://martinfowler.com/articles/201701-event-driven.html)

---

**Q9. How would you design an API for a leaderboard that handles millions of users?**

**Answer:**

A leaderboard ranks users by score. The key operations are: update score, get global rank, get top-N users, get nearby users (users around your rank).

**Redis Sorted Set is the perfect data structure:**

```
Redis Sorted Set: members with scores, automatically sorted
ZADD  → add/update member with score
ZINCRBY → increment score atomically
ZRANK  → get rank (0-indexed, ascending)
ZREVRANK → get rank (0-indexed, descending — rank 0 = highest score)
ZREVRANGE → get top-N members
ZRANGEBYSCORE → get members in score range
```

**Implementation:**

```js
const redis = require('ioredis');
const client = new redis();

const LEADERBOARD_KEY = 'game:leaderboard:global';

class LeaderboardService {
  // Add or update a user's score
  async updateScore(userId, score) {
    // ZADD: if user exists, replaces score; if not, adds them
    await client.zadd(LEADERBOARD_KEY, score, userId.toString());

    // Also update in DB for persistence
    await db.query(`
      INSERT INTO leaderboard_scores (user_id, score, updated_at)
      VALUES ($1, $2, NOW())
      ON CONFLICT (user_id) DO UPDATE SET score=$2, updated_at=NOW()
    `, [userId, score]);
  }

  // Increment score (for game events: +10 points for a kill)
  async incrementScore(userId, points) {
    const newScore = await client.zincrby(LEADERBOARD_KEY, points, userId.toString());
    await db.query(
      'UPDATE leaderboard_scores SET score=score+$1 WHERE user_id=$2',
      [points, userId]
    );
    return parseFloat(newScore);
  }

  // Get user's rank (1-indexed for display)
  async getUserRank(userId) {
    const rank = await client.zrevrank(LEADERBOARD_KEY, userId.toString());
    if (rank === null) return null; // user not on leaderboard
    return rank + 1; // convert 0-indexed to 1-indexed
  }

  // Get top N players with their scores and names
  async getTopN(n = 100) {
    // ZREVRANGE with WITHSCORES: returns [member, score, member, score, ...]
    const results = await client.zrevrange(LEADERBOARD_KEY, 0, n - 1, 'WITHSCORES');

    // Parse pairs
    const entries = [];
    for (let i = 0; i < results.length; i += 2) {
      entries.push({ userId: results[i], score: parseFloat(results[i + 1]) });
    }

    // Enrich with user details (names, avatars) from cache
    const userIds = entries.map(e => e.userId);
    const users = await this.getUserDetails(userIds);

    return entries.map((entry, idx) => ({
      rank:     idx + 1,
      userId:   entry.userId,
      score:    entry.score,
      username: users[entry.userId]?.username,
      avatar:   users[entry.userId]?.avatar
    }));
  }

  // Get N users around a specific user (±N/2 above and below)
  async getNearbyUsers(userId, radius = 5) {
    const rank = await client.zrevrank(LEADERBOARD_KEY, userId.toString());
    if (rank === null) return [];

    const start = Math.max(0, rank - radius);
    const end = rank + radius;

    const results = await client.zrevrange(LEADERBOARD_KEY, start, end, 'WITHSCORES');

    const entries = [];
    for (let i = 0; i < results.length; i += 2) {
      entries.push({
        rank:   start + Math.floor(i / 2) + 1,
        userId: results[i],
        score:  parseFloat(results[i + 1]),
        isMe:   results[i] === userId.toString()
      });
    }
    return entries;
  }

  // Weekly leaderboard: use time-windowed keys
  async getWeeklyKey() {
    const weekNumber = getWeekNumber(new Date());
    return `game:leaderboard:week:${weekNumber}`;
  }

  async updateWeeklyScore(userId, points) {
    const key = await this.getWeeklyKey();
    await client.zincrby(key, points, userId.toString());
    await client.expire(key, 86400 * 14); // keep for 2 weeks
  }
}

// API endpoints
app.get('/leaderboard/top', async (req, res) => {
  const top = await leaderboardService.getTopN(100);
  res.json(top);
  // Cache this response: changes only when scores change
});

app.get('/leaderboard/me', requireAuth, async (req, res) => {
  const [rank, nearby] = await Promise.all([
    leaderboardService.getUserRank(req.user.id),
    leaderboardService.getNearbyUsers(req.user.id, 10)
  ]);
  res.json({ myRank: rank, nearby });
});

// Scale: 10M users in sorted set
// ZREVRANK: O(log N) = log(10M) ≈ 23 operations → microseconds
// ZREVRANGE top 100: O(log N + 100) → microseconds
// Redis handles millions of operations/second ✅
```

> 📖 Reference: [Leaderboard Design — Redis Docs](https://redis.com/solutions/use-cases/leaderboards/)

---

**Q10. What is data denormalization at scale? How does Twitter/X handle the home timeline problem?**

**Answer:**

**Denormalization at scale** means deliberately duplicating data to make reads faster — trading write complexity for read performance.

**Twitter's home timeline problem:**

```
Naive approach (normalized):
Timeline request: SELECT tweets FROM tweets 
                  WHERE author_id IN (SELECT followee_id FROM follows WHERE follower_id=?)
                  ORDER BY created_at DESC LIMIT 20

For a user following 1000 people → 1 query with JOIN on potentially millions of rows
At 300M DAU × multiple timeline loads → billions of expensive queries/day → DB dies
```

**Twitter's actual solution (hybrid fan-out):**

```js
class TwitterTimelineService {
  async publishTweet(authorId, content) {
    // 1. Store the tweet in tweets table
    const tweet = await db.query(
      'INSERT INTO tweets (author_id, content, created_at) VALUES ($1,$2,NOW()) RETURNING *',
      [authorId, content]
    );

    const author = await User.findById(authorId);

    if (author.followerCount <= 5_000) {
      // SMALL ACCOUNT: fan-out-on-write to all followers
      await this.fanOutToFollowers(authorId, tweet.rows[0]);
    }
    // CELEBRITY (>5K followers): NO fan-out on write
    // Followers pull celebrity tweets on read
    // Why? Beyoncé posting = writing to 30M timelines = 30M writes instantly → disaster
  }

  async fanOutToFollowers(authorId, tweet) {
    // Chunk followers to avoid blocking (process 1000 at a time)
    let offset = 0;
    const CHUNK = 1000;

    while (true) {
      const followers = await db.query(
        'SELECT follower_id FROM follows WHERE followee_id=$1 LIMIT $2 OFFSET $3',
        [authorId, CHUNK, offset]
      );

      if (followers.rows.length === 0) break;

      // Write to each follower's Redis timeline cache
      const pipeline = redis.pipeline();
      for (const { follower_id } of followers.rows) {
        // Timeline is a sorted set: score=timestamp, member=tweetId
        pipeline.zadd(`timeline:${follower_id}`, tweet.created_at.getTime(), tweet.id.toString());
        pipeline.zremrangebyrank(`timeline:${follower_id}`, 0, -801); // keep max 800
        pipeline.expire(`timeline:${follower_id}`, 86400 * 7);
      }
      await pipeline.exec();

      offset += CHUNK;
      await sleep(10); // rate limit the fan-out (avoid Redis overload)
    }
  }

  async getHomeTimeline(userId, count = 20) {
    // 1. Get pre-computed timeline from Redis (regular users they follow)
    const tweetIds = await redis.zrevrange(`timeline:${userId}`, 0, count - 1);

    // 2. Get celebrities this user follows (fan-out-on-read for them)
    const celebrities = await db.query(`
      SELECT f.followee_id FROM follows f
      JOIN users u ON u.id = f.followee_id
      WHERE f.follower_id=$1 AND u.follower_count > 5000
    `, [userId]);

    // 3. Fetch recent celebrity tweets directly
    const celebTweets = await Promise.all(
      celebrities.rows.map(({ followee_id }) =>
        redis.zrevrange(`user:tweets:${followee_id}`, 0, 5) // celebrity tweet cache
      )
    );

    // 4. Merge: pre-computed + celebrity tweets, sort, trim
    const allIds = [...new Set([...tweetIds, ...celebTweets.flat()])];

    // 5. Fetch tweet details (from cache or DB)
    const tweets = await this.getTweetDetails(allIds);
    return tweets.sort((a, b) => b.createdAt - a.createdAt).slice(0, count);
  }
}
```

**Key insight:** Twitter stores each tweet once but distributes timeline membership across 300M Redis sorted sets. At peak: 400K writes/sec for fan-out of a viral tweet from a regular user. For celebrities: pull on read, cached per celebrity.

> 📖 Reference: [Twitter Timeline — High Scalability](http://highscalability.com/blog/2013/7/8/the-architecture-twitter-uses-to-deal-with-150m-active-users.html)


---

## 2. Redis — Deep Dive

---

**Q11. What are Redis data structures? Explain String, List, Set, Sorted Set, Hash, and their use cases.**

**Answer:**

```js
const redis = require('ioredis');
const client = new redis();

// ── 1. STRING — plain value, counter, cached object ──────────────────
await client.set('user:42:name', 'Alice');
await client.get('user:42:name');          // 'Alice'
await client.setex('session:abc', 3600, JSON.stringify({ userId: 42 })); // with TTL
await client.incr('page:views:home');      // atomic counter → 1, 2, 3...
await client.incrby('user:42:credits', 10); // add 10 credits atomically

// Use cases: caching, session store, counters, idempotency keys, feature flags

// ── 2. LIST — ordered by insertion, double-ended queue ───────────────
await client.lpush('queue:emails', JSON.stringify({ to: 'alice@x.com' })); // push left
await client.rpush('queue:emails', JSON.stringify({ to: 'bob@x.com' }));   // push right
await client.lpop('queue:emails');  // pop from left (FIFO queue)
await client.rpop('queue:emails');  // pop from right (LIFO stack)
await client.brpop('queue:emails', 0); // BLOCKING pop (waits for item)
await client.lrange('queue:emails', 0, -1); // get all items

// Use cases: task queues, recent activity feed (lpush + ltrim), undo history

// ── 3. SET — unordered, unique members ──────────────────────────────
await client.sadd('online:users', '42', '43', '44');
await client.sismember('online:users', '42'); // 1 (exists)
await client.smembers('online:users');         // ['42', '43', '44']
await client.srem('online:users', '42');       // remove
await client.scard('online:users');            // count
// Set operations:
await client.sinter('users:premium', 'users:active'); // intersection
await client.sunion('users:US', 'users:CA');           // union
await client.sdiff('users:all', 'users:banned');       // difference

// Use cases: unique visitors, tags, online users, mutual friends, permissions

// ── 4. SORTED SET — like Set but each member has a numeric score ─────
// Members are always sorted by score (ascending). Ties broken alphabetically.
await client.zadd('leaderboard', 1500, 'alice');
await client.zadd('leaderboard', 2200, 'bob');
await client.zadd('leaderboard', 800, 'charlie');
await client.zincrby('leaderboard', 100, 'alice'); // alice now 1600

await client.zrevrange('leaderboard', 0, -1, 'WITHSCORES');
// ['bob','2200','alice','1600','charlie','800']

await client.zrevrank('leaderboard', 'alice'); // 1 (0-indexed, bob is 0)
await client.zrangebyscore('leaderboard', 1000, 2000); // users with score 1000-2000

// Use cases: leaderboards, priority queues, rate limiting windows, time-series events

// ── 5. HASH — field-value pairs (like a flat object/dictionary) ──────
await client.hset('user:42', 'name', 'Alice', 'email', 'alice@x.com', 'age', '30');
await client.hget('user:42', 'name');           // 'Alice'
await client.hmget('user:42', 'name', 'email'); // ['Alice', 'alice@x.com']
await client.hgetall('user:42');                // { name:'Alice', email:'alice@x.com', age:'30' }
await client.hincrby('user:42', 'loginCount', 1); // atomic increment of a field
await client.hdel('user:42', 'age');

// Use cases: user profiles, session data, counters per entity, feature flags per user
// More memory-efficient than storing JSON string (Redis can compress small hashes)

// ── 6. STREAM — append-only log with consumer groups (Redis 5+) ──────
await client.xadd('events', '*', 'type', 'ORDER_PLACED', 'orderId', '99');
// '*' = auto-generate ID (timestamp-based)

// Consumer group (like Kafka consumer groups)
await client.xgroup('CREATE', 'events', 'order-processor', '$', 'MKSTREAM');
const messages = await client.xreadgroup(
  'GROUP', 'order-processor', 'consumer-1',
  'COUNT', 10, 'BLOCK', 2000, 'STREAMS', 'events', '>'
);
// '>' = only undelivered messages

// Use cases: event logging, activity feeds, reliable message queuing

// ── 7. HyperLogLog — probabilistic unique count (very memory efficient) ──
await client.pfadd('unique:visitors:today', 'user:42', 'user:43', 'user:44');
await client.pfcount('unique:visitors:today'); // approximate count (±0.81% error)
// Memory: 12KB per HyperLogLog regardless of number of unique elements!
// vs Set: ~50 bytes per member

// Use cases: unique visitor counts, unique search queries, event deduplication at scale
```

> 📖 Reference: [Redis Data Types — Redis Docs](https://redis.io/docs/data-types/)

---

**Q12. How does Redis achieve persistence? What is the difference between RDB and AOF?**

**Answer:**

By default Redis is in-memory only — a restart loses all data. Persistence options let Redis survive restarts.

| | RDB (Redis Database) | AOF (Append Only File) |
|--|---------------------|----------------------|
| Mechanism | Point-in-time snapshot | Log of every write command |
| Data loss on crash | Up to last snapshot interval | Up to last fsync (configurable) |
| Restart speed | Fast (load snapshot directly) | Slow (replay all commands) |
| File size | Compact (binary snapshot) | Larger (text commands, grows over time) |
| I/O overhead | Low (periodic fork + dump) | Higher (write to log on every command) |
| Use case | Backup/DR, acceptable data loss | Durability, minimal data loss |

```bash
# ── RDB configuration (redis.conf) ──────────────────────────────────
save 900 1      # snapshot if ≥1 change in 900 seconds (15 min)
save 300 10     # snapshot if ≥10 changes in 300 seconds (5 min)
save 60 10000   # snapshot if ≥10000 changes in 60 seconds

rdbcompression yes   # compress RDB file (smaller)
dbfilename dump.rdb
dir /var/lib/redis/

# How RDB works:
# 1. Redis forks a child process (copy-on-write, no blocking)
# 2. Child writes snapshot to disk
# 3. Parent continues serving requests
# 4. Child replaces old RDB file when done
# Downside: if crash between snapshots → lose up to N minutes of data

# ── AOF configuration ────────────────────────────────────────────────
appendonly yes
appendfilename "appendonly.aof"

# fsync policy:
appendfsync always    # fsync after every command → 0 data loss, slowest
appendfsync everysec  # fsync every second → at most 1 second of data loss ← RECOMMENDED
appendfsync no        # let OS decide → fastest, up to OS buffer loss

# AOF rewrite (compaction): over time AOF grows huge
# "SET x 1" then "SET x 2" → AOF has 2 entries, but only final value matters
# Rewrite: compact AOF to minimal commands to reproduce current state
auto-aof-rewrite-percentage 100  # rewrite when AOF doubles in size
auto-aof-rewrite-min-size 64mb   # but only if > 64MB

# How AOF works:
# Every write command appended to file
# On restart: replay all commands to rebuild state
# Slower restart for large datasets, but near-zero data loss with everysec
```

**Hybrid (recommended for production):**

```bash
# Use both RDB + AOF
aof-use-rdb-preamble yes  # AOF file starts with RDB snapshot, then appends changes
# Benefits: fast restart (RDB) + durability (AOF)
# This is the default in Redis 4+
```

**Practical advice:**

```
Development: disable both (pure in-memory, fast, lose data on restart = fine)
Staging: RDB only (periodic backups, some data loss ok)
Production cache: RDB with reasonable interval (data can be rebuilt from DB)
Production primary store: AOF everysec + RDB (minimal data loss + fast restart)
```

> 📖 Reference: [Redis Persistence — Redis Docs](https://redis.io/docs/management/persistence/)

---

**Q13. What is Redis Sentinel? How does it provide high availability?**

**Answer:**

**Redis Sentinel** is a system for automatic failover. It monitors Redis instances and automatically promotes a replica to primary if the primary goes down — no manual intervention needed.

```
Redis Sentinel Architecture:

Sentinel 1 ──┐
Sentinel 2 ──┼──► Monitor Primary Redis ──► Replica 1
Sentinel 3 ──┘                           ──► Replica 2

Normal: clients write to Primary
Primary fails:
1. Sentinels detect failure (quorum agreement needed)
2. Sentinels elect a leader Sentinel
3. Leader promotes a Replica → new Primary
4. Clients reconnect (Sentinel tells them new Primary address)
```

```js
// Redis Sentinel client configuration (ioredis)
const redis = new Redis({
  sentinels: [
    { host: 'sentinel-1.internal', port: 26379 },
    { host: 'sentinel-2.internal', port: 26379 },
    { host: 'sentinel-3.internal', port: 26379 }
  ],
  name: 'mymaster',          // sentinel master name
  sentinelRetryStrategy: (times) => Math.min(times * 100, 3000)
});

// ioredis automatically:
// - Queries sentinels to find current primary
// - Reconnects to new primary on failover
// Your app code doesn't change:
await redis.set('key', 'value');
await redis.get('key');
```

```bash
# sentinel.conf
sentinel monitor mymaster 10.0.1.1 6379 2
# "2" = quorum: 2 out of 3 sentinels must agree primary is down before failover

sentinel down-after-milliseconds mymaster 5000  # 5s timeout to declare primary down
sentinel failover-timeout mymaster 60000        # failover must complete in 60s
sentinel parallel-syncs mymaster 1             # 1 replica syncs at a time during failover
```

**Sentinel vs Cluster:**

| | Sentinel | Cluster |
|--|----------|--------|
| Purpose | High availability (HA) for single shard | HA + horizontal scaling (sharding) |
| Sharding | No | Yes (16384 hash slots across nodes) |
| Data capacity | Limited to single node's RAM | Scales horizontally |
| Complexity | Lower | Higher |
| Use case | Up to ~100GB single dataset | Multi-TB datasets, very high write throughput |

> 📖 Reference: [Redis Sentinel — Redis Docs](https://redis.io/docs/management/sentinel/)

---

**Q14. What is Redis Cluster? How does it shard data across nodes?**

**Answer:**

**Redis Cluster** shards data across multiple Redis nodes using hash slots. It provides both sharding (horizontal scalability) and replication (high availability) without a single coordinator.

```
Redis Cluster: 6 nodes (3 primaries + 3 replicas)

Primary 1 (slots 0-5460)    ←→ Replica 1
Primary 2 (slots 5461-10922) ←→ Replica 2
Primary 3 (slots 10923-16383) ←→ Replica 3

Total: 16384 hash slots distributed evenly
```

**How sharding works:**

```js
// Key → hash slot → node assignment
// slot = CRC16(key) % 16384

// Examples:
// CRC16("user:42") % 16384 = 3821 → Primary 1 handles this key
// CRC16("order:99") % 16384 = 9001 → Primary 2 handles this key

// Hash tags: force related keys to same slot
// {user:42}:profile and {user:42}:orders → both hash on "user:42" → same slot → same node
// Allows MULTI/EXEC transactions on these keys (must be on same node)
await client.set('{user:42}:profile', JSON.stringify(profile));
await client.set('{user:42}:orders', JSON.stringify(orders));
// Both go to same node → can pipeline/transaction together ✅
```

```js
// Cluster client automatically routes to the right node
const { Cluster } = require('ioredis');

const cluster = new Cluster([
  { host: 'redis-node-1', port: 6379 },
  { host: 'redis-node-2', port: 6379 },
  { host: 'redis-node-3', port: 6379 }
], {
  redisOptions: { password: process.env.REDIS_PASSWORD },
  scaleReads: 'slave'  // read from replicas to offload primaries
});

// Transparent routing — same API as single Redis
await cluster.set('user:42:name', 'Alice'); // routed to correct node
await cluster.get('user:42:name');           // routed to same node

// Cross-slot operations NOT supported:
// cluster.mget('{user:42}:name', '{order:99}:total') → ❌ different slots
// Use pipeline within same slot, or fetch individually
```

**Adding a node (resharding):**

```bash
# Add new node to cluster
redis-cli --cluster add-node new-node:6379 existing-node:6379

# Move hash slots to new node (redistributes data)
redis-cli --cluster reshard existing-node:6379
# Specify how many slots to move and from which nodes
# Data migrated live — no downtime ✅
```

> 📖 Reference: [Redis Cluster — Redis Docs](https://redis.io/docs/management/scaling/)

---

**Q15. How would you implement a distributed rate limiter using Redis?**

**Answer:**

A distributed rate limiter uses Redis so all application servers share the same counter state — one server can't be bypassed by routing requests to a different node.

```js
// ── Sliding Window Counter (most accurate, production-ready) ──────────
class DistributedRateLimiter {
  constructor(redis) {
    this.redis = redis;
  }

  // Lua script: atomic check + increment
  async isAllowed(identifier, { limit, windowMs }) {
    const now = Date.now();
    const windowStart = now - windowMs;
    const key = `ratelimit:${identifier}`;

    const luaScript = `
      local key = KEYS[1]
      local now = tonumber(ARGV[1])
      local windowStart = tonumber(ARGV[2])
      local limit = tonumber(ARGV[3])
      local windowMs = tonumber(ARGV[4])

      -- Remove expired entries
      redis.call('ZREMRANGEBYSCORE', key, '-inf', windowStart)

      -- Count current requests in window
      local count = redis.call('ZCARD', key)

      if count < limit then
        -- Add this request with timestamp as both score and member
        redis.call('ZADD', key, now, now .. '-' .. math.random(1000000))
        redis.call('PEXPIRE', key, windowMs)
        return {1, limit - count - 1}  -- allowed, remaining
      else
        -- Get oldest entry to calculate retry-after
        local oldest = redis.call('ZRANGE', key, 0, 0, 'WITHSCORES')
        local retryAfter = 0
        if #oldest > 0 then
          retryAfter = tonumber(oldest[2]) + windowMs - now
        end
        return {0, 0, retryAfter}  -- rejected, remaining=0, retry-after ms
      end
    `;

    const result = await this.redis.eval(
      luaScript, 1, key,
      now.toString(), windowStart.toString(),
      limit.toString(), windowMs.toString()
    );

    return {
      allowed:    result[0] === 1,
      remaining:  result[1],
      retryAfter: result[2] ? Math.ceil(result[2] / 1000) : 0 // seconds
    };
  }
}

// Express middleware factory
const createRateLimit = (redis, options) => {
  const limiter = new DistributedRateLimiter(redis);

  return async (req, res, next) => {
    // Identify by: user ID (authenticated) or IP (anonymous)
    const identifier = req.user?.id
      ? `user:${req.user.id}`
      : `ip:${req.ip}`;

    const result = await limiter.isAllowed(identifier, options);

    // Standard rate limit headers
    res.setHeader('X-RateLimit-Limit', options.limit);
    res.setHeader('X-RateLimit-Remaining', result.remaining);
    res.setHeader('X-RateLimit-Reset', Date.now() + options.windowMs);

    if (!result.allowed) {
      res.setHeader('Retry-After', result.retryAfter);
      return res.status(429).json({
        error: 'Too Many Requests',
        message: `Rate limit exceeded. Try again in ${result.retryAfter} seconds.`,
        retryAfter: result.retryAfter
      });
    }

    next();
  };
};

// Usage: different limits for different endpoints
app.use('/api/', createRateLimit(redis, { limit: 1000, windowMs: 60_000 }));       // 1000/min
app.use('/api/auth/login', createRateLimit(redis, { limit: 5, windowMs: 300_000 })); // 5/5min
app.use('/api/payments', createRateLimit(redis, { limit: 10, windowMs: 60_000 }));  // 10/min
app.use('/api/search', createRateLimit(redis, { limit: 60, windowMs: 60_000 }));    // 60/min
```

> 📖 Reference: [Rate Limiting with Redis — Redis Docs](https://redis.io/learn/howtos/ratelimiting)

---

**Q16. What is a Redis pipeline? How does it improve throughput?**

**Answer:**

A **Redis pipeline** batches multiple commands into a single network round trip. Without pipelining, each command requires a full network round trip (send command → receive response). With pipelining, all commands are sent at once and responses received all at once.

```
Without pipeline:
Client → SET a 1 → Redis → OK → Client  (round trip 1)
Client → SET b 2 → Redis → OK → Client  (round trip 2)
Client → SET c 3 → Redis → OK → Client  (round trip 3)
Total: 3 × RTT (e.g., 3 × 1ms = 3ms)

With pipeline:
Client → SET a 1 ──┐
         SET b 2    ├──► Redis → OK ──┐
         SET c 3 ──┘                  ├──► Client
                         OK           │
                         OK ──────────┘
Total: 1 × RTT + processing (e.g., 1ms + tiny processing)
Speed improvement: ~3x for 3 commands, ~100x for 100 commands
```

```js
// ── Without pipeline: N round trips ──────────────────────────────────
async function updateUsersBad(users) {
  for (const user of users) { // 1000 users = 1000 round trips!
    await client.set(`user:${user.id}:name`, user.name);
    await client.set(`user:${user.id}:email`, user.email);
    await client.expire(`user:${user.id}:name`, 3600);
  }
}

// ── With pipeline: 1 round trip ──────────────────────────────────────
async function updateUsersGood(users) {
  const pipeline = client.pipeline();

  for (const user of users) {
    pipeline.set(`user:${user.id}:name`, user.name);
    pipeline.set(`user:${user.id}:email`, user.email);
    pipeline.expire(`user:${user.id}:name`, 3600);
  }

  const results = await pipeline.exec();
  // results: [[null, 'OK'], [null, 'OK'], [null, 1], ...]
  // Each result: [error, value]
}

// ── Real-world example: fan-out to followers ──────────────────────────
async function fanOutTweet(authorId, tweet, followerIds) {
  const pipeline = client.pipeline();
  const score = tweet.createdAt.getTime();

  for (const followerId of followerIds) { // could be 10,000 followers
    pipeline.zadd(`timeline:${followerId}`, score, tweet.id.toString());
    pipeline.zremrangebyrank(`timeline:${followerId}`, 0, -801); // keep max 800
    pipeline.expire(`timeline:${followerId}`, 86400 * 7);
  }

  await pipeline.exec(); // all 30,000 commands in ONE round trip ✅
}

// ── Pipeline vs MULTI/EXEC (transaction) ─────────────────────────────
// Pipeline: commands sent in batch, but NOT atomic — other clients can interleave
// MULTI/EXEC: all commands atomic — no other client can interleave

// Pipeline (batching only):
const results = await client.pipeline().set('a', 1).set('b', 2).exec();

// MULTI/EXEC (atomic transaction):
const results = await client.multi().set('a', 1).set('b', 2).exec();
// If another client sets 'a' between set('a') and set('b'): pipeline allows it, MULTI doesn't
```

> 📖 Reference: [Redis Pipelining — Redis Docs](https://redis.io/docs/manual/pipelining/)

---

**Q17. What is a Redis Lua script? When would you use it over a regular command?**

**Answer:**

A **Redis Lua script** executes a block of Lua code atomically on the Redis server. Unlike pipelining (which batches commands but isn't atomic), Lua scripts guarantee atomicity — no other client can execute between script steps.

**When to use Lua scripts:**
1. Read-modify-write operations that must be atomic (check-then-set)
2. Complex logic that requires multiple Redis commands as one unit
3. Avoiding race conditions in distributed systems

```js
// ── Problem: non-atomic check-and-set ────────────────────────────────
// Race condition without Lua:
const tokens = parseInt(await client.get('credits:user:42'));
if (tokens >= 10) {
  // Another request might decrement here before we do!
  await client.decrby('credits:user:42', 10);
  return true; // might go negative!
}

// ── Solution: atomic Lua script ──────────────────────────────────────
const deductCreditsScript = `
  local key = KEYS[1]
  local amount = tonumber(ARGV[1])

  local current = tonumber(redis.call('GET', key)) or 0

  if current >= amount then
    redis.call('DECRBY', key, amount)
    return 1  -- success
  else
    return 0  -- insufficient credits
  end
`;

// Load script (cached by SHA for efficiency)
const sha = await client.script('LOAD', deductCreditsScript);

// Execute: atomic, no race condition possible
const success = await client.evalsha(sha, 1, 'credits:user:42', '10');
// success: 1 = deducted, 0 = insufficient

// ── More complex example: rate limiter with Lua ──────────────────────
const rateLimitScript = `
  local key = KEYS[1]
  local limit = tonumber(ARGV[1])
  local ttl = tonumber(ARGV[2])

  local current = redis.call('INCR', key)  -- atomic increment

  if current == 1 then
    redis.call('EXPIRE', key, ttl)  -- set TTL only on first call
  end

  if current > limit then
    return {0, current}  -- rejected
  end

  return {1, current}  -- allowed
`;

// ── Lua vs MULTI/EXEC comparison ─────────────────────────────────────
/*
MULTI/EXEC:
  - Atomic
  - No conditional logic (can't branch based on values read mid-transaction)
  - Uses WATCH for optimistic locking

Lua Script:
  - Atomic
  - Full conditional logic (if/else, loops)
  - Can read values and branch → much more powerful
  - Runs server-side (no extra network trips for conditionals)

Use Lua when:
✅ Complex conditional read-modify-write
✅ Need multiple reads + conditional writes atomically
✅ Implementing custom data structures on Redis
✅ Rate limiters, leaderboard operations, inventory management

Use MULTI/EXEC when:
✅ Simple batch of writes with no conditionals
✅ More readable for team (Lua can be hard to debug)
*/
```

> 📖 Reference: [Redis Scripting — Redis Docs](https://redis.io/docs/manual/programmability/eval-intro/)

---

**Q18. What is Redis Pub/Sub? What are its limitations compared to Kafka?**

**Answer:**

**Redis Pub/Sub** allows publishers to send messages to channels and subscribers to receive them — real-time messaging without polling.

```js
// Publisher
const publisher = new Redis();
await publisher.publish('notifications:user:42', JSON.stringify({
  type: 'ORDER_SHIPPED',
  orderId: 99,
  trackingNumber: 'UPS123'
}));

// Subscriber (separate connection required)
const subscriber = new Redis();
await subscriber.subscribe('notifications:user:42', 'notifications:global');

subscriber.on('message', (channel, message) => {
  console.log(`Received on ${channel}:`, JSON.parse(message));
});

// Pattern subscribe (wildcard)
await subscriber.psubscribe('notifications:user:*');
subscriber.on('pmessage', (pattern, channel, message) => {
  // Receives all user notification channels
});
```

**Redis Pub/Sub limitations vs Kafka:**

| | Redis Pub/Sub | Apache Kafka |
|--|--------------|-------------|
| Message persistence | ❌ No — fire and forget | ✅ Yes — configurable retention |
| Replay | ❌ No | ✅ Yes — replay from any offset |
| Delivery if offline | ❌ Lost — subscriber must be connected | ✅ Stored — delivered when consumer reconnects |
| Consumer groups | ❌ No | ✅ Yes — load-balance across consumers |
| Ordering | ❌ No per-channel ordering | ✅ Per-partition ordering |
| Scale | Limited by single Redis node | Distributed, very high throughput |
| Backpressure | ❌ No | ✅ Yes (pull-based) |
| Use case | Real-time ephemeral events | Reliable event streaming, audit logs |

**When Redis Pub/Sub is appropriate:**

```js
// ✅ Real-time presence updates (user went online/offline)
// If subscriber is offline → they don't care about old presence events
publisher.publish('presence', JSON.stringify({ userId: 42, status: 'online' }));

// ✅ Cache invalidation across multiple app servers
// When data changes: notify all app servers to evict their local cache
publisher.publish('cache:invalidate', JSON.stringify({ key: 'product:catalog' }));
app.locals.subscribe('cache:invalidate', (channel, msg) => {
  const { key } = JSON.parse(msg);
  localCache.del(key);
});

// ✅ Real-time collaborative editing presence (who is viewing which document)
// ❌ NOT appropriate for:
// - Order events (must not lose; use Kafka)
// - Payment notifications (must deliver even if subscriber offline; use Kafka)
// - Audit logs (must persist; use Kafka)
```

> 📖 Reference: [Redis Pub/Sub — Redis Docs](https://redis.io/docs/manual/pubsub/)

---

## 3. Advanced Testing Strategies

---

**Q19. What is property-based testing? How is it different from example-based testing?**

**Answer:**

**Example-based testing:** You write specific inputs and expected outputs.
**Property-based testing:** You define properties that must ALWAYS hold true, and the framework generates hundreds of random inputs to try to find a counterexample.

```js
// ── Example-based testing (traditional) ──────────────────────────────
describe('calculateDiscount', () => {
  it('gives 10% off for orders over $100', () => {
    expect(calculateDiscount(150, 'VIP')).toBe(15);
  });
  it('gives no discount under $100', () => {
    expect(calculateDiscount(50, 'VIP')).toBe(0);
  });
  // Only tests 2 specific cases — you might miss edge cases like $100 exactly
});

// ── Property-based testing (fast-check library) ──────────────────────
const fc = require('fast-check');

describe('calculateDiscount — properties', () => {
  it('discount is always between 0 and total', () => {
    fc.assert(fc.property(
      fc.float({ min: 0, max: 10000 }),  // random total amount
      fc.constantFrom('VIP', 'REGULAR', 'NEW'), // random tier
      (total, tier) => {
        const discount = calculateDiscount(total, tier);
        // Property: discount must always be within [0, total]
        return discount >= 0 && discount <= total;
      }
    ), { numRuns: 1000 }); // runs 1000 random inputs
  });

  it('higher tier always gets equal or more discount', () => {
    fc.assert(fc.property(
      fc.float({ min: 0, max: 10000 }),
      (total) => {
        const vipDiscount = calculateDiscount(total, 'VIP');
        const regularDiscount = calculateDiscount(total, 'REGULAR');
        // Property: VIP always gets same or more discount
        return vipDiscount >= regularDiscount;
      }
    ));
  });

  it('encode-decode roundtrip always returns original', () => {
    fc.assert(fc.property(
      fc.string(),  // any string, including unicode, empty, very long
      (str) => {
        return decode(encode(str)) === str; // must always be true
      }
    ));
    // fast-check will try: '', 'a', '🎉', '\n\t', 'a'.repeat(10000), etc.
  });
});

// ── Real backend example: pagination ─────────────────────────────────
it('paginating all items returns each item exactly once', () => {
  fc.assert(fc.property(
    fc.array(fc.integer(), { minLength: 0, maxLength: 1000 }),
    fc.integer({ min: 1, max: 100 }), // page size
    (items, pageSize) => {
      const allPages = [];
      let page = 0;

      while (true) {
        const pageItems = paginate(items, page, pageSize);
        if (pageItems.length === 0) break;
        allPages.push(...pageItems);
        page++;
      }

      // Property: paginating everything returns exact same items
      return JSON.stringify(allPages.sort()) === JSON.stringify([...items].sort());
    }
  ));
  // fast-check finds: what if items is empty? What if pageSize > items.length?
  // Automatically shrinks counterexamples to smallest failing case
});
```

**When to use property-based testing:**
- Parsers, encoders, decoders (roundtrip properties)
- Mathematical functions (commutativity, associativity)
- Sorting, pagination, filtering (order/completeness properties)
- Any function with clear invariants

> 📖 Reference: [Property-Based Testing — HypothesisWorks](https://hypothesis.works/articles/what-is-property-based-testing/)

---

**Q20. What is mutation testing? How does it measure the quality of your test suite?**

**Answer:**

**Mutation testing** modifies (mutates) your source code in small ways (kill a condition, change `>` to `>=`, delete a line) and checks if your tests catch the change. If your tests still pass after a mutation → the mutation "survived" → your tests missed this behavior.

```
Process:
1. Take source code
2. Introduce a small mutation (e.g., change `if (count > 0)` to `if (count >= 0)`)
3. Run test suite against mutated code
4. If tests FAIL → mutation "killed" → tests are good ✅
5. If tests PASS → mutation "survived" → tests have a gap ⚠️

Mutation Score = Killed Mutations / Total Mutations × 100
High score (>80%) = strong test suite
```

```js
// Original code:
function calculateShipping(weight, distance) {
  if (weight > 50) {
    return distance * 0.5 + 20; // heavy package surcharge
  }
  return distance * 0.3;
}

// Mutant 1: change '>' to '>='
function calculateShipping(weight, distance) {
  if (weight >= 50) { // ← mutation
    return distance * 0.5 + 20;
  }
  return distance * 0.3;
}
// If your tests don't test weight === 50 specifically → this mutation SURVIVES
// You have a gap: behavior at boundary weight=50 is untested

// Mutant 2: change 0.5 to 0.6
// Mutant 3: remove the +20
// Mutant 4: invert condition to weight < 50
// Each surviving mutant = a test you should write

// ── Stryker (JavaScript mutation testing tool) ────────────────────────
// stryker.conf.json
{
  "mutate": ["src/**/*.js"],
  "testRunner": "jest",
  "reporters": ["html", "progress"],
  "thresholds": { "high": 80, "low": 60, "break": 50 }
}

// Run: npx stryker run
// Output:
// Mutation score: 73.33%
// Survived mutants (need more tests):
//   src/pricing.js:15 - ConditionalExpression: weight > 50 => weight >= 50
//   src/pricing.js:16 - ArithmeticOperator: distance * 0.5 => distance / 0.5
//   src/pricing.js:17 - RemoveFromArray: return distance * 0.5 + 20 => return distance * 0.5
```

**Mutation testing vs code coverage:**

```
Code coverage 100%: every LINE is executed by tests
Mutation testing: tests actually ASSERT the correct behavior on those lines

You can have 100% line coverage with useless tests:
it('runs without error', () => {
  calculateShipping(30, 100); // executed! but no expect() → no assertion
  // Coverage: 100% ✅, Mutation score: 0% ❌ (all mutants survive)
});
```

> 📖 Reference: [Mutation Testing — Stryker](https://stryker-mutator.io/docs/mutation-testing-elements/what-is-mutation-testing/)

---

**Q21. What is a test double? Explain the difference between a mock, stub, spy, and fake.**

**Answer:**

**Test doubles** are objects that stand in for real dependencies in tests. Martin Fowler defined 5 types:

| Type | Returns | Verifies calls | Has behavior |
|------|---------|---------------|-------------|
| **Dummy** | Nothing useful | No | No |
| **Stub** | Predefined response | No | Minimal |
| **Spy** | Real or configured | Yes (after fact) | Real |
| **Mock** | Configured | Yes (expectations set before) | Minimal |
| **Fake** | Real-ish | No | Full working implementation |

```js
// ── DUMMY: placeholder — passed but never used ────────────────────────
function createUser(name, logger) {  // logger required but not always used
  return { id: Math.random(), name };
}
const dummyLogger = {}; // doesn't need methods — never called in this test
const user = createUser('Alice', dummyLogger);

// ── STUB: returns predetermined values ───────────────────────────────
// "When this method is called, return THIS"
const userRepositoryStub = {
  findById: async (id) => ({ id, name: 'Alice', email: 'alice@x.com' }), // hardcoded
  save:     async () => {},
};

const service = new UserService(userRepositoryStub);
const user = await service.getUser(42);
expect(user.name).toBe('Alice');
// We don't verify HOW findById was called — just that it returned the right thing

// ── SPY: wraps real implementation, records calls ─────────────────────
const emailService = new RealEmailService();
const sendEmailSpy = jest.spyOn(emailService, 'sendEmail');

await userService.registerUser({ email: 'alice@x.com' }, emailService);

// Verify it was called — after the fact
expect(sendEmailSpy).toHaveBeenCalledWith('alice@x.com', 'Welcome!');
expect(sendEmailSpy).toHaveBeenCalledTimes(1);
// emailService.sendEmail still RUNS the real implementation

// ── MOCK: pre-programmed expectations (strict verification) ──────────
// Set up expectations BEFORE the test, verify AFTER
const mockEmailService = {
  sendEmail: jest.fn()
};

// If sendEmail is NOT called → test FAILS (mock verifies expectations)
await userService.registerUser({ email: 'alice@x.com' }, mockEmailService);
expect(mockEmailService.sendEmail).toHaveBeenCalledWith('alice@x.com', expect.any(String));

// ── FAKE: working implementation, simplified for testing ──────────────
// In-memory database instead of real PostgreSQL
class FakeUserRepository {
  constructor() { this.users = new Map(); }

  async findById(id)   { return this.users.get(id) || null; }
  async save(user)     { this.users.set(user.id, user); return user; }
  async delete(id)     { this.users.delete(id); }
  async findAll()      { return [...this.users.values()]; }
}

// Full working implementation — tests can create, save, delete, find
const repo = new FakeUserRepository();
const service = new UserService(repo);

await service.createUser({ name: 'Alice', email: 'alice@x.com' });
const users = await service.getAllUsers();
expect(users).toHaveLength(1);
expect(users[0].name).toBe('Alice');
// No real DB needed — tests are fast and isolated ✅

// When to use which:
// Dummy:  required parameter you don't need
// Stub:   control what a dependency returns (for different scenarios)
// Spy:    verify a method was called while keeping real behavior
// Mock:   verify method interactions are correct (behavior testing)
// Fake:   replace slow/external dependency with fast working version
```

> 📖 Reference: [Test Doubles — Martin Fowler](https://martinfowler.com/bliki/TestDouble.html)

---

**Q22. What is chaos engineering? What is the principle behind Netflix's Chaos Monkey?**

**Answer:**

**Chaos engineering** is the practice of deliberately introducing failures into a production system to discover weaknesses BEFORE they cause customer-facing incidents. "If it's going to break, let it break in a controlled way."

**Netflix's Chaos Monkey:**
Netflix created Chaos Monkey to randomly terminate EC2 instances in production. The principle: if any server can die at any time, engineers MUST build resilient, stateless services. This forces the org to practice reliability, not just plan for it.

```
"The best way to avoid failure is to fail constantly."
— Netflix engineering blog
```

**Chaos experiment structure:**

```js
// Every chaos experiment follows this structure:
const experiment = {
  // 1. Define steady state: what does "normal" look like?
  steadyState: {
    condition: async () => {
      const errorRate = await prometheus.query(
        'rate(http_requests_total{status=~"5.."}[1m]) / rate(http_requests_total[1m])'
      );
      return errorRate < 0.01; // less than 1% errors = healthy
    }
  },

  // 2. Hypothesis: system remains in steady state despite chaos
  hypothesis: 'Terminating one API pod should not affect error rate',

  // 3. The chaos action
  action: async () => {
    const pods = await k8s.listPods('default', { labelSelector: 'app=api' });
    const randomPod = pods.items[Math.floor(Math.random() * pods.items.length)];
    await k8s.deletePod('default', randomPod.metadata.name);
    console.log(`💥 Killed pod: ${randomPod.metadata.name}`);
  },

  // 4. Rollback if things go wrong
  rollback: async () => {
    await k8s.scaleDeployment('api', 5); // restore to desired replicas
  }
};

// Chaos toolkit execution
class ChaosRunner {
  async run(experiment) {
    // 1. Verify steady state BEFORE
    const beforeState = await experiment.steadyState.condition();
    if (!beforeState) {
      console.log('System not in steady state — aborting experiment');
      return;
    }

    console.log('✅ Steady state verified — starting chaos');

    try {
      // 2. Apply chaos
      await experiment.action();
      await sleep(60000); // observe for 60 seconds

      // 3. Verify steady state AFTER
      const afterState = await experiment.steadyState.condition();

      if (afterState) {
        console.log('✅ Hypothesis CONFIRMED — system resilient to this failure');
      } else {
        console.log('❌ Hypothesis REJECTED — this failure causes user impact!');
        // File an incident, create a ticket to fix the weakness
        await createTicket({
          title: experiment.hypothesis,
          priority: 'HIGH',
          description: 'Chaos experiment revealed a reliability gap'
        });
      }
    } catch (err) {
      console.error('Chaos experiment failed unexpectedly:', err);
    } finally {
      await experiment.rollback();
    }
  }
}
```

**Types of chaos experiments:**

```bash
# Network chaos (latency, packet loss, DNS failure)
tc qdisc add dev eth0 root netem delay 200ms 50ms  # 200ms ± 50ms latency
tc qdisc add dev eth0 root netem loss 10%          # 10% packet loss

# Tools: Chaos Mesh (Kubernetes), Gremlin, AWS Fault Injection Simulator
# Netflix: ChAP (Chaos Automation Platform), FIT (Failure Injection Testing)

# Common experiments:
# - Kill a pod → service should auto-recover
# - Saturate CPU → circuit breakers should trip
# - Fill disk → graceful degradation or alert fires
# - Block external API → fallbacks activate
# - Introduce high DB latency → timeouts trigger, users see degraded mode
```

> 📖 Reference: [Chaos Engineering — Principles of Chaos](https://principlesofchaos.org/)

---

**Q23. What is load testing vs stress testing vs soak testing? When do you run each?**

**Answer:**

| Test Type | Goal | Load Level | Duration | When to run |
|-----------|------|-----------|----------|-------------|
| **Load test** | Verify system performs at expected load | Normal + peak load | 30-60 min | Before every major release |
| **Stress test** | Find breaking point | Beyond peak, until failure | Until system breaks | Before launches, capacity planning |
| **Soak test** | Find memory leaks, gradual degradation | Normal load | Hours or days | Monthly, after major changes |
| **Spike test** | Handle sudden traffic burst | Zero → 10x in seconds | Short | Before marketing campaigns |

```js
// ── k6 load test scripts ──────────────────────────────────────────────
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

const errorRate = new Rate('errors');

// Load Test: normal + peak load profile
export let options = {
  stages: [
    { duration: '5m', target: 100 },   // ramp up to 100 users over 5 min
    { duration: '10m', target: 100 },  // stay at 100 for 10 min (normal load)
    { duration: '5m', target: 300 },   // ramp to 300 (peak load)
    { duration: '10m', target: 300 },  // stay at peak for 10 min
    { duration: '5m', target: 0 },     // ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests under 500ms
    errors:            ['rate<0.01'],  // error rate under 1%
  }
};

// Stress Test: keep increasing until system breaks
export let stressOptions = {
  stages: [
    { duration: '5m',  target: 100 },
    { duration: '5m',  target: 500 },
    { duration: '5m',  target: 1000 },
    { duration: '5m',  target: 2000 },   // double every 5 min
    { duration: '5m',  target: 5000 },   // until latency spikes / errors occur
    { duration: '10m', target: 5000 },   // observe at breaking point
    { duration: '5m',  target: 0 },
  ]
};

// Soak Test: run at normal load for 24 hours
export let soakOptions = {
  stages: [
    { duration: '5m',  target: 100 },
    { duration: '24h', target: 100 },  // 24 hours at normal load
    { duration: '5m',  target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    // Memory should not grow over 24h — if it does: memory leak!
  }
};

export default function() {
  const res = http.get('https://api.myapp.com/products');
  errorRate.add(res.status !== 200);
  check(res, {
    'status 200':       r => r.status === 200,
    'latency < 500ms':  r => r.timings.duration < 500,
  });
  sleep(1);
}
```

**What each test reveals:**

```
Load test reveals:
- Normal operation performance (does p95 meet SLO?)
- Whether auto-scaling triggers correctly
- Database connection pool sizing

Stress test reveals:
- Maximum capacity (breaking point)
- Which component fails first (bottleneck)
- How gracefully the system degrades

Soak test reveals:
- Memory leaks (heap grows continuously over 24h)
- Connection leaks (DB connections never released)
- Log rotation issues (disk fills up)
- Gradual performance degradation (GC pressure)
```

> 📖 Reference: [Types of Performance Testing — k6 Docs](https://k6.io/docs/test-types/)

---

**Q24. What is a flaky test? Why are flaky tests dangerous in a CI pipeline?**

**Answer:**

A **flaky test** is a test that produces inconsistent results — passing sometimes and failing other times — without any code changes. Flakiness is caused by timing issues, external dependencies, shared state, or randomness.

```
Flaky test characteristics:
- Run same test 10 times → passes 7, fails 3 → no code change
- Fails in CI but passes locally (or vice versa)
- "Just re-run the pipeline" becomes standard practice
```

**Why flaky tests are dangerous:**

```
1. Erosion of trust: developers stop trusting CI
   "Oh, that failure? It's probably just flaky — just re-run"
   → Real bugs start getting re-run past instead of fixed

2. Delayed deployments: re-running pipelines adds 30-60 minutes per deployment
   Team doing 10 deploys/day × 1 re-run each = 5-10 hours wasted per day

3. False safety: "tests passed" doesn't actually mean the code works
   → Deploy broken code with confidence because CI showed green (eventually)
```

**Common causes and fixes:**

```js
// ── Cause 1: Timing/async issues ────────────────────────────────────
// ❌ Flaky: depends on timing
test('notification appears', async () => {
  clickButton();
  await sleep(1000); // 🚨 might not be enough on slow CI
  expect(screen.getByText('Saved!')).toBeInTheDocument();
});

// ✅ Fixed: wait for element explicitly
test('notification appears', async () => {
  clickButton();
  await screen.findByText('Saved!', { timeout: 5000 }); // wait up to 5s
  expect(screen.getByText('Saved!')).toBeInTheDocument();
});

// ── Cause 2: Shared database state ──────────────────────────────────
// ❌ Flaky: test A creates user, test B assumes no users exist
test('B: user count is 0', async () => {
  const count = await db.count('users');
  expect(count).toBe(0); // FAILS if test A ran first and didn't clean up
});

// ✅ Fixed: each test cleans up or uses transactions
beforeEach(async () => {
  await db.query('TRUNCATE users CASCADE');
  // OR: wrap each test in a transaction and ROLLBACK after
});

// ── Cause 3: Port/resource conflicts ─────────────────────────────────
// ❌ Flaky: hardcoded port might be in use
const server = app.listen(3000);

// ✅ Fixed: use random available port
const server = app.listen(0); // OS assigns available port
const port = server.address().port;

// ── Cause 4: Test order dependency ──────────────────────────────────
// ❌ Flaky: test B depends on state created by test A
// Tests must be independent and runnable in any order

// ✅ Fixed: each test sets up its own state
beforeEach(async () => { await createTestFixtures(); });
afterEach(async () => { await cleanupTestFixtures(); });

// ── Cause 5: Time-dependent logic ────────────────────────────────────
// ❌ Flaky: test fails at midnight when date changes
const today = new Date().toDateString();
expect(getFormattedDate(new Date())).toBe(today);

// ✅ Fixed: inject time
test('formats date correctly', () => {
  const fixedDate = new Date('2024-06-01T12:00:00Z');
  expect(getFormattedDate(fixedDate)).toBe('June 1, 2024');
});

// ── Detecting and tracking flaky tests ──────────────────────────────
// Run each test N times to detect flakiness:
// pytest --count=10 test_payment.py
// jest --testNamePattern="payment" --passWithNoTests

// In CI: quarantine flaky tests
// Mark as @flaky → run separately → don't block deployment
// Track flaky test rate as a team metric → fix when rate exceeds threshold
```

> 📖 Reference: [Flaky Tests — Google Testing Blog](https://testing.googleblog.com/2020/12/test-flakiness-one-of-main-challenges.html)
