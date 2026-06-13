# ✅ Solutions — Day 1: Freshers (0 YOE)

> **Series:** Backend Interview Prep · Fresher → 10 Years of Experience  
> **Level:** Fresher / Entry-Level  
> **Questions File:** [day-01-fresher.md](./day-01-fresher.md)  
> **Total Answers:** 60  

> 💡 **Tip:** Try answering each question yourself before reading the solution. That's how you actually learn.

---

## 📚 Table of Contents

1. [Internet & Web Fundamentals](#1-internet--web-fundamentals)
2. [HTTP & REST APIs](#2-http--rest-apis)
3. [Databases & SQL](#3-databases--sql)
4. [Data Structures & Algorithms](#4-data-structures--algorithms-backend-context)
5. [Operating System Basics](#5-operating-system-basics)
6. [Version Control (Git)](#6-version-control-git)
7. [Basic Programming Concepts](#7-basic-programming-concepts)
8. [Authentication & Security Basics](#8-authentication--security-basics)
9. [System Design Basics](#9-system-design-basics)
10. [Miscellaneous / HR-Tech Round](#10-miscellaneous--hr-tech-round)

---

## 1. Internet & Web Fundamentals

---

**Q1. What happens when you type a URL in the browser and press Enter?**

**Answer:**
1. **DNS Resolution** — The browser checks its cache, then queries a DNS resolver to convert the domain name (e.g., `google.com`) into an IP address.
2. **TCP Connection** — A TCP handshake (SYN → SYN-ACK → ACK) is established between the browser and the server.
3. **TLS Handshake** — If HTTPS, a TLS handshake negotiates encryption keys.
4. **HTTP Request** — The browser sends an HTTP GET request to the server.
5. **Server Response** — The server processes the request and returns an HTTP response (HTML, JSON, etc.).
6. **Rendering** — The browser parses and renders the response.

> 📖 Reference: [What happens when you type a URL (GitHub)](https://github.com/alex/what-happens-when)

---

**Q2. What is the difference between HTTP and HTTPS?**

**Answer:**
HTTP (HyperText Transfer Protocol) transfers data in **plain text** — anyone intercepting the traffic can read it. HTTPS is HTTP with **TLS (Transport Layer Security)** on top, which encrypts the data in transit, verifies the server's identity via a certificate, and prevents man-in-the-middle attacks. All modern APIs and websites should use HTTPS.

> 📖 Reference: [HTTP vs HTTPS — Cloudflare](https://www.cloudflare.com/learning/ssl/why-is-http-not-secure/)

---

**Q3. What is DNS and how does it work?**

**Answer:**
DNS (Domain Name System) is the internet's phone book — it maps human-readable domain names (e.g., `google.com`) to IP addresses (e.g., `142.250.80.46`). When you visit a domain:
1. Browser checks local cache → OS cache → router cache.
2. If not cached, a **recursive resolver** (usually your ISP) queries the **root nameservers**, then **TLD nameservers** (`.com`), then the **authoritative nameserver** for the domain.
3. The IP is returned and cached with a TTL (Time to Live).

> 📖 Reference: [How DNS Works — Cloudflare](https://www.cloudflare.com/learning/dns/what-is-dns/)

---

**Q4. What is an IP address? What is the difference between IPv4 and IPv6?**

**Answer:**
An IP address is a unique numerical label assigned to every device on a network, used to identify and communicate with it.

| | IPv4 | IPv6 |
|--|------|------|
| Format | `192.168.1.1` (32-bit) | `2001:0db8::1` (128-bit) |
| Addresses | ~4.3 billion | ~340 undecillion |
| Status | Exhausted | Gradually replacing IPv4 |

IPv6 was introduced because IPv4 addresses ran out. Most modern systems support both (dual-stack).

> 📖 Reference: [IPv4 vs IPv6 — Cloudflare](https://www.cloudflare.com/learning/network-layer/ipv4-vs-ipv6/)

---

**Q5. What is the OSI model? Name the 7 layers.**

**Answer:**
The OSI (Open Systems Interconnection) model is a conceptual framework that standardizes how different network systems communicate. From bottom to top:

| Layer | Name | Example |
|-------|------|---------|
| 7 | Application | HTTP, FTP, DNS |
| 6 | Presentation | TLS/SSL, JPEG encoding |
| 5 | Session | Session management |
| 4 | Transport | TCP, UDP |
| 3 | Network | IP, routing |
| 2 | Data Link | Ethernet, MAC addresses |
| 1 | Physical | Cables, radio waves |

Memory trick: **"All People Seem To Need Data Processing"** (top to bottom).

> 📖 Reference: [OSI Model — Cloudflare](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)

---

**Q6. What is the difference between TCP and UDP?**

**Answer:**

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery, ordering, retransmission | No guarantee |
| Speed | Slower due to overhead | Faster |
| Use cases | HTTP, databases, email | Video streaming, gaming, DNS, VoIP |

Use TCP when data integrity matters. Use UDP when speed matters more than perfect delivery.

> 📖 Reference: [TCP vs UDP — GeeksForGeeks](https://www.geeksforgeeks.org/differences-between-tcp-and-udp/)

---

**Q7. What is a port number? Name a few well-known ports.**

**Answer:**
A port is a 16-bit number (0–65535) that identifies a specific process or service on a host. IP addresses route traffic to a machine; ports route it to the correct application.

| Port | Service |
|------|---------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 27017 | MongoDB |

> 📖 Reference: [Port Numbers — Wikipedia](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)

---

**Q8. What is a cookie? How is it different from a session?**

**Answer:**
A **cookie** is a small piece of data stored in the **browser** by the server. It is sent with every subsequent request to that domain. Cookies are used for authentication tokens, preferences, and tracking.

A **session** stores data on the **server** side. The browser only holds a session ID (usually in a cookie). The actual data lives in server memory or a database.

| | Cookie | Session |
|--|--------|---------|
| Storage | Client (browser) | Server |
| Security | Accessible to client (unless HttpOnly) | More secure |
| Size limit | ~4KB | Practically unlimited |

> 📖 Reference: [Cookies vs Sessions — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

---

## 2. HTTP & REST APIs

---

**Q9. What are HTTP methods? Explain GET, POST, PUT, PATCH, and DELETE.**

**Answer:**

| Method | Purpose | Idempotent? | Body? |
|--------|---------|-------------|-------|
| GET | Retrieve a resource | ✅ Yes | No |
| POST | Create a new resource | ❌ No | Yes |
| PUT | Replace an entire resource | ✅ Yes | Yes |
| PATCH | Partially update a resource | ✅ Yes | Yes |
| DELETE | Delete a resource | ✅ Yes | Optional |

**Example:**
- `GET /users/1` → fetch user with ID 1
- `POST /users` → create a new user
- `PATCH /users/1` → update only the email field of user 1
- `DELETE /users/1` → delete user 1

> 📖 Reference: [HTTP Methods — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

---

**Q10. What are HTTP status codes? Explain the major categories (1xx–5xx) with examples.**

**Answer:**

| Range | Category | Examples |
|-------|----------|---------|
| 1xx | Informational | `100 Continue` |
| 2xx | Success | `200 OK`, `201 Created`, `204 No Content` |
| 3xx | Redirection | `301 Moved Permanently`, `304 Not Modified` |
| 4xx | Client Error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests` |
| 5xx | Server Error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

> 📖 Reference: [HTTP Status Codes — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

---

**Q11. What is a REST API? What are its key principles (constraints)?**

**Answer:**
REST (Representational State Transfer) is an architectural style for designing networked APIs. Its 6 key constraints are:
1. **Client-Server** — UI and backend are separated.
2. **Stateless** — Each request contains all information needed; server stores no client state.
3. **Cacheable** — Responses must define whether they can be cached.
4. **Uniform Interface** — Consistent resource naming and HTTP methods.
5. **Layered System** — Client doesn't know if it talks to the actual server or a proxy.
6. **Code on Demand** *(optional)* — Server can send executable code to the client.

> 📖 Reference: [REST API — RedHat](https://www.redhat.com/en/topics/api/what-is-a-rest-api)

---

**Q12. What is the difference between REST and SOAP?**

**Answer:**

| | REST | SOAP |
|--|------|------|
| Protocol | Architectural style (HTTP) | Strict protocol |
| Format | JSON, XML, plain text | XML only |
| Speed | Faster, lightweight | Slower, more overhead |
| Standards | Flexible | WS-Security, WSDL, strict contracts |
| Use case | Web/mobile APIs | Enterprise, banking, legacy systems |

REST is the modern default for web APIs. SOAP is still used in industries that require strict contracts and built-in security (banking, healthcare).

> 📖 Reference: [REST vs SOAP — AWS](https://aws.amazon.com/compare/the-difference-between-soap-rest/)

---

**Q13. What is JSON? Why is it used in APIs?**

**Answer:**
JSON (JavaScript Object Notation) is a lightweight, human-readable data format for storing and exchanging data. It uses key-value pairs and arrays.

```json
{
  "id": 1,
  "name": "Alice",
  "roles": ["admin", "user"]
}
```

It is the standard for REST APIs because it is language-agnostic, easy to parse, compact compared to XML, and natively supported in JavaScript (and all major languages).

> 📖 Reference: [JSON Introduction — MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON)

---

**Q14. What is a request header and response header? Give examples.**

**Answer:**
Headers are key-value pairs sent alongside HTTP requests and responses to provide metadata.

**Common Request Headers:**
- `Content-Type: application/json` — tells the server the body format
- `Authorization: Bearer <token>` — authentication
- `Accept: application/json` — tells the server what format the client expects

**Common Response Headers:**
- `Content-Type: application/json` — format of the response body
- `Cache-Control: max-age=3600` — caching instructions
- `X-RateLimit-Remaining: 99` — rate limit info

> 📖 Reference: [HTTP Headers — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

---

**Q15. What is CORS and why does it exist?**

**Answer:**
CORS (Cross-Origin Resource Sharing) is a browser security mechanism that restricts web pages from making requests to a different domain than the one that served the page.

For example, a page at `frontend.com` cannot call `api.backend.com` unless the backend explicitly allows it via CORS headers:

```
Access-Control-Allow-Origin: https://frontend.com
Access-Control-Allow-Methods: GET, POST
```

It exists to prevent **malicious websites** from making unauthorized API calls on behalf of a logged-in user (CSRF-like attacks). Note: CORS is **browser-enforced only** — server-to-server calls are not restricted.

> 📖 Reference: [CORS Explained — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

---

**Q16. What is the difference between synchronous and asynchronous API calls?**

**Answer:**
- **Synchronous:** The caller waits (blocks) until the operation completes before moving on. Simple but inefficient for slow operations.
- **Asynchronous:** The caller initiates the request and continues working. It gets notified (via callback, Promise, event) when the result is ready.

**Backend example:**
```js
// Synchronous (blocking)
const data = fs.readFileSync('file.txt');

// Asynchronous (non-blocking)
fs.readFile('file.txt', (err, data) => { /* handle */ });
```

Async is critical in backend systems to handle thousands of concurrent requests without blocking threads.

> 📖 Reference: [Sync vs Async — freeCodeCamp](https://www.freecodecamp.org/news/synchronous-vs-asynchronous-in-javascript/)

---

**Q17. What is an API rate limit? Why is it important?**

**Answer:**
Rate limiting restricts how many API requests a client can make within a given time window (e.g., 100 requests per minute per user).

**Why it matters:**
- **Prevents abuse** — protects against brute-force attacks, scrapers, and DoS.
- **Ensures fairness** — one client can't starve others.
- **Controls costs** — protects downstream services and databases from overload.

When exceeded, the server typically returns `429 Too Many Requests`. The response usually includes `Retry-After` or `X-RateLimit-Reset` headers.

> 📖 Reference: [API Rate Limiting — Cloudflare](https://www.cloudflare.com/learning/bots/what-is-rate-limiting/)

---

## 3. Databases & SQL

---

**Q18. What is a database? What is the difference between RDBMS and NoSQL?**

**Answer:**
A **database** is an organized collection of data with mechanisms to store, retrieve, and manage it efficiently.

| | RDBMS | NoSQL |
|--|-------|-------|
| Structure | Tables with rows and columns | Documents, key-value, graph, column |
| Schema | Fixed schema | Flexible / schema-less |
| Query | SQL | Varies (MongoDB query, Redis commands) |
| ACID | Strong ACID support | Often eventual consistency |
| Examples | PostgreSQL, MySQL, SQLite | MongoDB, Redis, Cassandra, DynamoDB |

Use RDBMS for structured, relational data. Use NoSQL for unstructured, high-scale, or flexible-schema needs.

> 📖 Reference: [SQL vs NoSQL — MongoDB](https://www.mongodb.com/nosql-explained/nosql-vs-sql)

---

**Q19. What is a primary key and a foreign key?**

**Answer:**
- **Primary Key:** A column (or set of columns) that uniquely identifies each row in a table. It cannot be NULL and must be unique. Example: `user_id` in a `users` table.
- **Foreign Key:** A column in one table that references the primary key of another table, establishing a relationship between the two. Example: `order.user_id` references `users.user_id`.

```sql
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));
CREATE TABLE orders (id INT PRIMARY KEY, user_id INT REFERENCES users(id));
```

> 📖 Reference: [Keys in DBMS — GeeksForGeeks](https://www.geeksforgeeks.org/types-of-keys-in-relational-model-candidate-super-primary-alternate-and-foreign/)

---

**Q20. What are the different types of SQL JOINs? Explain with examples.**

**Answer:**

| JOIN Type | Returns |
|-----------|---------|
| INNER JOIN | Rows matching in BOTH tables |
| LEFT JOIN | All rows from left + matching from right (NULL if no match) |
| RIGHT JOIN | All rows from right + matching from left |
| FULL OUTER JOIN | All rows from both, with NULLs where no match |
| CROSS JOIN | Every combination of rows from both tables |

```sql
-- INNER JOIN: only users who have orders
SELECT u.name, o.id FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: all users, even those with no orders
SELECT u.name, o.id FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

> 📖 Reference: [SQL JOINs — W3Schools](https://www.w3schools.com/sql/sql_join.asp)

---

**Q21. What is normalization? Explain 1NF, 2NF, and 3NF.**

**Answer:**
Normalization organizes a database to reduce data redundancy and improve integrity.

- **1NF (First Normal Form):** Each column holds atomic (indivisible) values. No repeating groups. Each row is unique.
- **2NF (Second Normal Form):** Meets 1NF + every non-key column is fully dependent on the **entire** primary key (no partial dependencies). Applies to composite keys.
- **3NF (Third Normal Form):** Meets 2NF + no transitive dependencies (non-key columns don't depend on other non-key columns).

**Example of a 3NF violation:** A `students` table with `zip_code` and `city` columns — `city` depends on `zip_code`, not on the student's primary key. Fix: move `zip_code → city` to a separate `zip_codes` table.

> 📖 Reference: [Database Normalization — GeeksForGeeks](https://www.geeksforgeeks.org/normal-forms-in-dbms/)

---

**Q22. What is an index in a database? Why is it used?**

**Answer:**
An index is a data structure (typically a B-Tree) that allows the database to find rows much faster without scanning every row in the table — like an index in a book.

```sql
-- Without index: full table scan O(n)
SELECT * FROM users WHERE email = 'alice@example.com';

-- With index: O(log n) lookup
CREATE INDEX idx_users_email ON users(email);
```

**Trade-offs:** Indexes speed up reads but **slow down writes** (INSERT/UPDATE/DELETE) because the index must be updated too. Don't over-index.

> 📖 Reference: [Database Indexes — Use The Index, Luke](https://use-the-index-luke.com/sql/anatomy)

---

**Q23. What are ACID properties in a database transaction?**

**Answer:**

| Property | Meaning |
|----------|---------|
| **Atomicity** | All operations in a transaction succeed or all are rolled back. No partial updates. |
| **Consistency** | A transaction brings the database from one valid state to another, respecting all constraints. |
| **Isolation** | Concurrent transactions don't interfere with each other. Each sees a consistent view. |
| **Durability** | Once committed, data persists even after a system crash (written to disk). |

**Example:** In a bank transfer (debit A, credit B), ACID ensures you can't debit A without crediting B, even if the server crashes mid-transaction.

> 📖 Reference: [ACID Properties — GeeksForGeeks](https://www.geeksforgeeks.org/acid-properties-in-dbms/)

---

**Q24. What is the difference between `WHERE` and `HAVING` in SQL?**

**Answer:**
- **WHERE** filters rows **before** grouping. Works on individual rows.
- **HAVING** filters groups **after** `GROUP BY`. Works on aggregated results.

```sql
-- WHERE: filter rows before aggregation
SELECT department, COUNT(*) FROM employees
WHERE salary > 50000
GROUP BY department;

-- HAVING: filter after aggregation
SELECT department, COUNT(*) FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
```

> 📖 Reference: [WHERE vs HAVING — W3Schools](https://www.w3schools.com/sql/sql_having.asp)

---

**Q25. What is a stored procedure? How is it different from a function?**

**Answer:**
A **stored procedure** is a precompiled block of SQL code stored in the database that can be called by name. It can perform DML operations (INSERT, UPDATE, DELETE) and doesn't need to return a value.

A **function** must return a value and is typically used in SELECT expressions. It cannot perform certain side effects (like transactions in most databases).

| | Stored Procedure | Function |
|--|-----------------|---------|
| Return value | Optional | Mandatory |
| Usage | Called with `EXEC` / `CALL` | Used in SELECT, WHERE |
| Side effects | Allowed | Usually not allowed |

> 📖 Reference: [Stored Procedures — GeeksForGeeks](https://www.geeksforgeeks.org/what-is-stored-procedures-in-sql/)

---

**Q26. What is a transaction in a database? How do you rollback a transaction?**

**Answer:**
A transaction is a sequence of SQL operations treated as a single unit of work. It either fully succeeds (COMMIT) or fully fails (ROLLBACK).

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 500 WHERE id = 1;
  UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;

-- If something goes wrong:
ROLLBACK;
```

ROLLBACK undoes all changes made since the `BEGIN`, restoring the database to its previous state.

> 📖 Reference: [Database Transactions — IBM](https://www.ibm.com/topics/database-transaction)

---

**Q27. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?**

**Answer:**

| Command | What it does | Rollback? | Removes structure? |
|---------|-------------|-----------|-------------------|
| DELETE | Removes specific rows (can use WHERE) | ✅ Yes | No |
| TRUNCATE | Removes ALL rows instantly, resets auto-increment | ❌ Usually no | No |
| DROP | Removes the entire table (structure + data) | ❌ No | Yes |

```sql
DELETE FROM users WHERE id = 5;    -- remove one user
TRUNCATE TABLE logs;                -- clear all logs fast
DROP TABLE temp_data;              -- delete the table entirely
```

> 📖 Reference: [DELETE vs TRUNCATE vs DROP — GeeksForGeeks](https://www.geeksforgeeks.org/difference-between-delete-drop-and-truncate/)

---

## 4. Data Structures & Algorithms (Backend Context)

---

**Q28. What is the difference between an Array and a Linked List?**

**Answer:**

| | Array | Linked List |
|--|-------|-------------|
| Memory | Contiguous | Non-contiguous (nodes with pointers) |
| Access | O(1) random access by index | O(n) — must traverse from head |
| Insert/Delete | O(n) — shifting required | O(1) at head/tail if pointer known |
| Cache friendly | ✅ Yes | ❌ No |

**Backend use case:** Arrays are used for fixed-size data, buffers, and lookup tables. Linked lists underpin queues, LRU caches, and adjacency lists.

> 📖 Reference: [Array vs Linked List — GeeksForGeeks](https://www.geeksforgeeks.org/linked-list-vs-array/)

---

**Q29. What is a Stack and a Queue? Name real-world backend use cases.**

**Answer:**
- **Stack:** LIFO (Last In, First Out). Push/pop from the same end.
- **Queue:** FIFO (First In, First Out). Enqueue at back, dequeue from front.

**Backend use cases:**

| Structure | Use Case |
|-----------|---------|
| Stack | Call stack, undo history, DFS traversal, expression evaluation |
| Queue | Message queues (RabbitMQ, SQS), task queues, BFS traversal, request buffering |

> 📖 Reference: [Stack and Queue — GeeksForGeeks](https://www.geeksforgeeks.org/difference-between-stack-and-queue-data-structures/)

---

**Q30. What is a Hash Map / Hash Table? How does it handle collisions?**

**Answer:**
A Hash Map stores key-value pairs. A **hash function** converts the key into an index in an underlying array, giving O(1) average-case read/write.

**Collision** happens when two keys hash to the same index. Strategies:
- **Chaining:** Each bucket holds a linked list of entries.
- **Open Addressing:** On collision, probe the next available bucket (linear probing, quadratic probing).

**Backend use case:** Caching (Redis), session stores, deduplication, frequency counts.

> 📖 Reference: [Hash Tables — CS50](https://cs50.harvard.edu/x/2024/notes/5/)

---

**Q31. What is Big-O notation? What is O(1), O(n), O(log n), and O(n²)?**

**Answer:**
Big-O describes how an algorithm's runtime or space grows as input size `n` increases.

| Notation | Name | Example |
|----------|------|---------|
| O(1) | Constant | Hash map lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Loop through array |
| O(n log n) | Linearithmic | Merge sort |
| O(n²) | Quadratic | Nested loops |

Always aim for O(1) or O(log n) for hot paths in backend code. An O(n²) loop on a 1M-row dataset is a disaster.

> 📖 Reference: [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)

---

**Q32. What is the difference between BFS and DFS? When would you use each?**

**Answer:**

| | BFS (Breadth-First Search) | DFS (Depth-First Search) |
|--|---------------------------|--------------------------|
| Traversal | Level by level | Goes deep down one path first |
| Data structure | Queue | Stack (or recursion) |
| Shortest path | ✅ Yes (unweighted) | ❌ No |
| Memory | More (holds entire level) | Less |

**Backend use cases:**
- BFS: Finding shortest path in a graph (e.g., social network degrees of separation), level-order tree traversal.
- DFS: Detecting cycles, topological sort, tree serialization.

> 📖 Reference: [BFS vs DFS — GeeksForGeeks](https://www.geeksforgeeks.org/difference-between-bfs-and-dfs/)

---

**Q33. What is a binary search? What is its time complexity?**

**Answer:**
Binary search finds a target value in a **sorted array** by repeatedly halving the search space.

```
Array: [1, 3, 5, 7, 9, 11]  Target: 7
→ Mid = 5 → too small → search right half [7, 9, 11]
→ Mid = 9 → too big  → search left half [7]
→ Mid = 7 → found! ✅
```

**Time complexity:** O(log n) — each step halves the problem. Much faster than linear search O(n) on large datasets. **Requirement:** The array must be sorted.

> 📖 Reference: [Binary Search — Khan Academy](https://www.khanacademy.org/computing/computer-science/algorithms/binary-search/a/binary-search)

---

## 5. Operating System Basics

---

**Q34. What is the difference between a process and a thread?**

**Answer:**

| | Process | Thread |
|--|---------|--------|
| Definition | An independent program in execution | A unit of execution within a process |
| Memory | Own memory space | Shares memory with other threads in the same process |
| Communication | Expensive (IPC: pipes, sockets) | Easy (shared memory) |
| Crash impact | Crash is isolated | One thread crash can kill the whole process |
| Overhead | Heavy to create | Lightweight |

**Backend example:** Node.js runs as a single process with a single thread (event loop). Python/Java web servers typically use multiple threads or processes per request.

> 📖 Reference: [Process vs Thread — GeeksForGeeks](https://www.geeksforgeeks.org/difference-between-process-and-thread/)

---

**Q35. What is a deadlock? How can it be prevented?**

**Answer:**
A deadlock occurs when two or more threads are waiting for each other to release a resource, so none of them can proceed.

**Example:** Thread A holds Lock 1 and waits for Lock 2. Thread B holds Lock 2 and waits for Lock 1. Both wait forever.

**Four conditions for deadlock (Coffman conditions):** Mutual exclusion, Hold and wait, No preemption, Circular wait.

**Prevention strategies:**
- **Lock ordering:** Always acquire locks in the same order.
- **Timeouts:** Give up after waiting too long and retry.
- **Deadlock detection:** Database engines detect and kill one of the transactions.

> 📖 Reference: [Deadlock — GeeksForGeeks](https://www.geeksforgeeks.org/introduction-of-deadlock-in-operating-system/)

---

**Q36. What is virtual memory?**

**Answer:**
Virtual memory is an OS technique that gives each process the illusion of having its own large, contiguous memory space, regardless of physical RAM available. The OS maps virtual addresses to physical RAM or disk (swap space).

**Benefits:**
- Processes are isolated from each other.
- Programs can use more memory than physically available (using swap).
- Simplifies memory management for programs.

**Backend relevance:** Memory-hungry services (e.g., Java heaps, large in-memory caches) interact directly with OS virtual memory management.

> 📖 Reference: [Virtual Memory — CS61C Berkeley](https://cs61c.eecs.berkeley.edu/)

---

**Q37. What is context switching?**

**Answer:**
Context switching is the OS process of saving the state of a currently running process/thread (registers, program counter, stack) and loading the saved state of another one, so the CPU can switch between them.

It enables multitasking but has a cost: saving/restoring state takes CPU cycles. Too many context switches degrade performance. This is why thread pools limit the number of threads and why event-loop architectures (Node.js) avoid it.

> 📖 Reference: [Context Switching — GeeksForGeeks](https://www.geeksforgeeks.org/context-switch-in-operating-system/)

---

**Q38. What is the difference between concurrency and parallelism?**

**Answer:**
- **Concurrency:** Multiple tasks are **in progress** at the same time, but not necessarily running simultaneously. They take turns (interleaved execution). One CPU core can be concurrent.
- **Parallelism:** Multiple tasks run **simultaneously** on multiple CPU cores at the exact same instant.

Rob Pike's analogy: "Concurrency is about *dealing with* lots of things at once. Parallelism is about *doing* lots of things at once."

A Node.js server handles many concurrent requests on one thread using async I/O. A Spark job processes data in parallel across many cores.

> 📖 Reference: [Concurrency vs Parallelism — Rob Pike](https://go.dev/blog/waza-talk)

---

## 6. Version Control (Git)

---

**Q39. What is Git? What is the difference between Git and GitHub?**

**Answer:**
**Git** is a distributed version control system that tracks changes to files over time. It runs locally and doesn't require internet access.

**GitHub** is a cloud-hosted platform built on top of Git that adds collaboration features: pull requests, issues, CI/CD, access control, and remote repository hosting.

| | Git | GitHub |
|--|-----|--------|
| Type | CLI tool | Web platform |
| Runs | Locally | In the cloud |
| Alternatives | — | GitLab, Bitbucket |

Git is the engine. GitHub is the garage.

> 📖 Reference: [Git Handbook — GitHub](https://guides.github.com/introduction/git-handbook/)

---

**Q40. What is the difference between `git merge` and `git rebase`?**

**Answer:**
Both integrate changes from one branch into another, but differently:

- **Merge:** Creates a new merge commit that combines two branch histories. Preserves full history. Safe for shared branches.
- **Rebase:** Rewrites commits from the source branch on top of the target, creating a linear history. Cleaner log, but rewrites history — dangerous on shared branches.

```bash
# Merge: preserves history
git checkout main && git merge feature-branch

# Rebase: linear history
git checkout feature-branch && git rebase main
```

**Rule of thumb:** Rebase local/personal branches. Merge shared/public branches.

> 📖 Reference: [Merge vs Rebase — Atlassian](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

---

**Q41. What is a pull request (PR)? What is a code review?**

**Answer:**
A **Pull Request** is a proposal to merge a branch into another (usually `main`). It notifies team members, shows the diff, and provides a space for discussion before merging.

A **code review** is the process where peers examine the proposed changes for correctness, readability, security issues, test coverage, and alignment with team standards before approval.

Good code reviews catch bugs early, spread knowledge, and maintain consistency. They are one of the highest-leverage activities on a software team.

> 📖 Reference: [About Pull Requests — GitHub Docs](https://docs.github.com/en/pull-requests)

---

**Q42. What is `.gitignore` and why is it important?**

**Answer:**
`.gitignore` is a file that tells Git which files or directories to exclude from version control. Git will not track or commit them.

**Common entries:**
```
node_modules/    # dependencies (huge, reproducible)
.env             # secrets and API keys
dist/            # build artifacts
*.log            # log files
.DS_Store        # macOS system files
```

It is critical because committing `node_modules` bloats the repo, and committing `.env` exposes secrets. Every project should have a `.gitignore` from day one.

> 📖 Reference: [.gitignore — Git Documentation](https://git-scm.com/docs/gitignore)

---

**Q43. Explain the Git workflow: `clone`, `add`, `commit`, `push`, `pull`.**

**Answer:**

```bash
git clone <url>       # Copy a remote repo to your local machine
git add file.js       # Stage changes (mark for the next commit)
git commit -m "msg"   # Save staged changes to local history with a message
git push origin main  # Upload local commits to the remote repo
git pull              # Download + merge latest changes from the remote
```

**Typical flow:**
1. `clone` the repo once.
2. Make changes → `add` → `commit` locally.
3. `pull` to get teammates' changes.
4. `push` your commits to remote.

> 📖 Reference: [Git Basics — Pro Git Book](https://git-scm.com/book/en/v2)

---

## 7. Basic Programming Concepts

---

**Q44. What is OOP? Explain the four pillars: Encapsulation, Abstraction, Inheritance, Polymorphism.**

**Answer:**
OOP (Object-Oriented Programming) models software around objects that bundle data (fields) and behavior (methods).

| Pillar | Meaning | Example |
|--------|---------|---------|
| **Encapsulation** | Hide internal state; expose only through methods | `private balance`, `getBalance()` |
| **Abstraction** | Expose only what's necessary, hide complexity | `payment.charge()` hides gateway logic |
| **Inheritance** | A class inherits properties/methods of a parent class | `Dog extends Animal` |
| **Polymorphism** | Same interface, different behaviors | `shape.area()` works for Circle and Square |

> 📖 Reference: [OOP Concepts — GeeksForGeeks](https://www.geeksforgeeks.org/object-oriented-programming-oops-concept-in-java/)

---

**Q45. What is the difference between a compiled language and an interpreted language?**

**Answer:**

| | Compiled | Interpreted |
|--|----------|------------|
| How | Source code → machine code before execution | Source code executed line-by-line at runtime |
| Speed | Faster at runtime | Slower (overhead at runtime) |
| Examples | C, C++, Go, Rust | Python, Ruby, JavaScript (traditionally) |
| Error detection | Compile-time errors caught early | Errors found only at runtime |

Modern languages blur this line — Java compiles to bytecode run on JVM, JS uses JIT compilation. The distinction is less sharp today but the concept still matters.

> 📖 Reference: [Compiled vs Interpreted — freeCodeCamp](https://www.freecodecamp.org/news/compiled-versus-interpreted-languages/)

---

**Q46. What is recursion? What is a base case and why is it important?**

**Answer:**
Recursion is when a function calls itself to solve a smaller version of the same problem.

```js
function factorial(n) {
  if (n === 0) return 1;        // base case
  return n * factorial(n - 1); // recursive case
}
```

The **base case** is the condition where the function stops calling itself. Without it, you get infinite recursion and a **stack overflow** (the call stack runs out of memory).

Every recursive solution can be converted to an iterative one. Use recursion when it makes the solution clearer (tree traversal, divide-and-conquer).

> 📖 Reference: [Recursion — Khan Academy](https://www.khanacademy.org/computing/computer-science/algorithms/recursive-algorithms/a/recursion)

---

**Q47. What is the difference between pass by value and pass by reference?**

**Answer:**
- **Pass by value:** A **copy** of the variable is passed. Changing it inside the function does not affect the original.
- **Pass by reference:** A **reference (pointer)** to the original variable is passed. Changes inside the function affect the original.

```python
# Pass by value (primitives in most languages)
def increment(x):
    x += 1
n = 5
increment(n)
print(n)  # Still 5

# Pass by reference (objects/lists)
def append_item(lst):
    lst.append(99)
my_list = [1, 2]
append_item(my_list)
print(my_list)  # [1, 2, 99] — modified!
```

> 📖 Reference: [Pass by Value vs Reference — GeeksForGeeks](https://www.geeksforgeeks.org/difference-between-call-by-value-and-call-by-reference/)

---

## 8. Authentication & Security Basics

---

**Q48. What is the difference between Authentication and Authorization?**

**Answer:**
- **Authentication (AuthN):** Verifying **who you are**. (e.g., logging in with username + password, verifying a JWT token).
- **Authorization (AuthZ):** Verifying **what you're allowed to do**. (e.g., checking if a user has the `admin` role before accessing `/admin`).

**Flow:** Authentication happens first. If it passes, authorization is checked.

**Example:** A user logs in (AuthN). The system checks if they have permission to delete a post (AuthZ). Two entirely separate concerns — never confuse them.

> 📖 Reference: [AuthN vs AuthZ — Auth0](https://auth0.com/docs/get-started/identity-fundamentals/authentication-and-authorization)

---

**Q49. What is hashing? How is it different from encryption?**

**Answer:**

| | Hashing | Encryption |
|--|---------|-----------|
| Direction | One-way (irreversible) | Two-way (reversible with a key) |
| Purpose | Verify integrity / store passwords | Protect data in transit or at rest |
| Output | Fixed-length digest | Variable-length ciphertext |
| Examples | bcrypt, SHA-256, MD5 | AES, RSA, TLS |

Passwords should be **hashed** (never encrypted), because you should never need to recover the original. On login, hash the input and compare digests.

**Never use MD5 or SHA-1 for passwords** — use bcrypt, scrypt, or Argon2 which include salting and are deliberately slow.

> 📖 Reference: [Hashing vs Encryption — Okta](https://www.okta.com/identity-101/hashing-vs-encryption/)

---

**Q50. What is SQL Injection? How do you prevent it?**

**Answer:**
SQL Injection is when an attacker inserts malicious SQL into an input field that gets executed by the database.

**Vulnerable code:**
```js
// User inputs: ' OR '1'='1
const query = `SELECT * FROM users WHERE name = '${userInput}'`;
// Becomes: SELECT * FROM users WHERE name = '' OR '1'='1'
// Returns ALL users!
```

**Prevention:**
1. **Parameterized queries / prepared statements** (primary defense):
```js
db.query('SELECT * FROM users WHERE name = ?', [userInput]);
```
2. **Input validation** — reject unexpected characters.
3. **ORM usage** — most ORMs handle escaping automatically.
4. **Least privilege** — DB user should only have permissions it needs.

> 📖 Reference: [SQL Injection — OWASP](https://owasp.org/www-community/attacks/SQL_Injection)

---

## 9. System Design Basics

---

**Q51. What is a client-server architecture?**

**Answer:**
Client-server is a model where **clients** (browsers, mobile apps) request resources or services, and **servers** (backend applications) process and return responses.

```
[Client (Browser)] --HTTP Request--> [Server (API)] --Query--> [Database]
[Client]           <--HTTP Response- [Server]       <--Data--- [Database]
```

Key properties:
- **Separation of concerns** — frontend and backend are decoupled.
- **Centralized logic** — business rules live on the server.
- **Scalability** — servers can be scaled independently.

Most web applications, mobile apps, and APIs follow this model.

> 📖 Reference: [Client-Server Model — GeeksForGeeks](https://www.geeksforgeeks.org/client-server-model/)

---

**Q52. What is a monolithic application? What are its advantages and disadvantages?**

**Answer:**
A monolith is a single, unified application where all components (auth, payments, notifications, etc.) are deployed together as one unit.

**Advantages:**
- Simple to develop, test, and deploy initially.
- Easy to debug (single process, single log stream).
- No network overhead between components.

**Disadvantages:**
- Scales as one unit — can't scale individual components.
- A bug in one module can crash the whole app.
- Large codebases become hard to maintain.
- Deployment is all-or-nothing — risky.

Most systems start as a monolith (and should) before evolving to microservices when the pain is real.

> 📖 Reference: [Monolithic Architecture — Martin Fowler](https://martinfowler.com/bliki/MonolithFirst.html)

---

**Q53. What is caching? Why is it used in backend systems?**

**Answer:**
Caching stores copies of frequently accessed data in fast storage (memory) so future requests can be served faster without hitting the original source (database, external API).

**Why use it:**
- **Speed** — memory access (~100ns) vs database (~10ms) is orders of magnitude faster.
- **Reduce load** — protects databases from being overwhelmed.
- **Cost** — fewer database queries = lower infrastructure costs.

**Common caching layers:**
- **Application cache** — in-memory (Node.js Map, Python dict)
- **Distributed cache** — Redis, Memcached
- **CDN cache** — for static assets and API responses at edge

> 📖 Reference: [Caching Overview — AWS](https://aws.amazon.com/caching/)

---

## 10. Miscellaneous / HR-Tech Round

---

**Q54. What is the difference between a library and a framework?**

**Answer:**
- **Library:** A collection of reusable functions you call from your code. **You are in control** of the flow. Example: `lodash`, `axios`, `moment.js`.
- **Framework:** A structure that **calls your code** at specific points. It controls the flow — you fill in the blanks. Example: Express, Django, Spring Boot.

The key difference is **Inversion of Control (IoC)**: with a library, your code calls it. With a framework, it calls you.

> **Analogy:** A library is a toolbox. A framework is a house — it gives you the structure, and you fill in the rooms.

> 📖 Reference: [Library vs Framework — freeCodeCamp](https://www.freecodecamp.org/news/the-difference-between-a-framework-and-a-library-bd133054023f/)

---

**Q55. What is an environment variable? Why should secrets not be hardcoded?**

**Answer:**
An environment variable is a dynamic named value set outside the application code, accessible at runtime via `process.env` (Node.js), `os.environ` (Python), etc.

```bash
# .env file
DATABASE_URL=postgres://user:pass@host/db
JWT_SECRET=supersecretkey123
```

**Why never hardcode secrets:**
- Source code is shared with teams and committed to Git — secrets become visible.
- Git history is permanent — even deleted secrets remain in history.
- Different environments (dev/staging/prod) need different values.
- Leaked credentials in public repos are exploited within minutes by bots.

> 📖 Reference: [Environment Variables — 12 Factor App](https://12factor.net/config)

---

**Q56. Explain the concept of MVC (Model-View-Controller) architecture.**

**Answer:**
MVC separates an application into three components:

| Layer | Responsibility | Example |
|-------|---------------|---------|
| **Model** | Data + business logic | Database queries, validation rules |
| **View** | Presentation layer | HTML templates, JSON responses |
| **Controller** | Handles input, coordinates M and V | Route handlers, request parsing |

**Request flow:**
```
Request → Controller → Model (fetch data) → Controller → View (format response) → Response
```

MVC keeps code organized and testable. Most backend frameworks (Express, Django, Rails, Spring) follow MVC or a variant of it.

> 📖 Reference: [MVC Pattern — MDN](https://developer.mozilla.org/en-US/docs/Glossary/MVC)

---

**Q57. What is Docker at a high level? Why do developers use it?**

**Answer:**
Docker is a platform that packages applications and their dependencies into **containers** — lightweight, portable, isolated units that run consistently on any machine.

**Without Docker:** "It works on my machine but not in production" — different OS versions, missing packages, conflicting dependencies.

**With Docker:** The container ships everything needed. Same behavior everywhere.

```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]
```

**Key benefits:** Consistency across environments, faster onboarding, easy horizontal scaling, isolation between services.

> 📖 Reference: [Docker Overview — Docker Docs](https://docs.docker.com/get-started/overview/)

---

**Q58. What is an ORM? Name one ORM you know (e.g., Sequelize, Hibernate, SQLAlchemy).**

**Answer:**
An ORM (Object-Relational Mapper) is a library that maps database tables to classes/objects in code, allowing you to query and manipulate the database using your programming language instead of raw SQL.

```js
// Without ORM (raw SQL)
db.query('SELECT * FROM users WHERE id = ?', [1]);

// With ORM (Sequelize / Prisma)
const user = await User.findById(1);
```

**Popular ORMs:**
- JavaScript: Sequelize, Prisma, TypeORM
- Python: SQLAlchemy, Django ORM
- Java: Hibernate
- Ruby: ActiveRecord

ORMs speed up development and prevent SQL injection by default, but can generate inefficient queries — raw SQL is sometimes necessary for performance-critical paths.

> 📖 Reference: [What is an ORM — freeCodeCamp](https://www.freecodecamp.org/news/what-is-an-orm-the-meaning-of-object-relational-mapping-database-tools/)

---

**Q59. What is middleware in the context of a web framework?**

**Answer:**
Middleware is a function that sits between the incoming request and the final route handler. It has access to the request, response, and the `next()` function to pass control to the next middleware.

```js
// Express.js middleware example
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);  // logging
  next(); // pass to next middleware
});

app.use(authMiddleware);    // authentication
app.use(rateLimitMiddleware); // rate limiting
app.get('/users', getUsers); // final handler
```

Middleware is used for: logging, authentication, rate limiting, request validation, CORS headers, compression, error handling. It enables clean, reusable cross-cutting concerns.

> 📖 Reference: [Middleware Explained — Express Docs](https://expressjs.com/en/guide/using-middleware.html)

---

**Q60. What is the difference between horizontal scaling and vertical scaling?**

**Answer:**

| | Vertical Scaling (Scale Up) | Horizontal Scaling (Scale Out) |
|--|----------------------------|-------------------------------|
| How | Add more CPU/RAM to existing server | Add more servers |
| Limit | Physical hardware ceiling | Theoretically unlimited |
| Cost | Exponentially expensive at high end | More linear cost |
| Downtime | Usually requires restart | Can be done without downtime |
| Complexity | Simple | Requires load balancer, state management |

**Example:** Going from 8GB to 64GB RAM is vertical. Adding 10 more server instances behind a load balancer is horizontal.

Modern cloud-native backends are designed for horizontal scaling. Stateless services scale horizontally easily; stateful ones (databases) require more care (sharding, replication).

> 📖 Reference: [Horizontal vs Vertical Scaling — Cloudflare](https://www.cloudflare.com/learning/performance/horizontal-scaling-vs-vertical-scaling/)

---

## 🎯 Quick Revision Cheatsheet

| # | Question | One-Line Answer |
|---|---------|----------------|
| Q1 | URL in browser | DNS → TCP → TLS → HTTP Request → Response → Render |
| Q2 | HTTP vs HTTPS | HTTPS = HTTP + TLS encryption |
| Q5 | OSI layers | Physical, Data Link, Network, Transport, Session, Presentation, Application |
| Q6 | TCP vs UDP | TCP = reliable + ordered; UDP = fast + no guarantee |
| Q9 | HTTP methods | GET=read, POST=create, PUT=replace, PATCH=update, DELETE=remove |
| Q10 | Status codes | 2xx=success, 3xx=redirect, 4xx=client error, 5xx=server error |
| Q15 | CORS | Browser policy restricting cross-origin requests; solved with headers |
| Q23 | ACID | Atomicity, Consistency, Isolation, Durability |
| Q27 | DELETE vs TRUNCATE vs DROP | Row-level vs all rows vs whole table |
| Q31 | Big-O | O(1) constant, O(log n) binary search, O(n) linear, O(n²) nested loops |
| Q34 | Process vs Thread | Process = isolated; Thread = shared memory within a process |
| Q44 | OOP pillars | Encapsulation, Abstraction, Inheritance, Polymorphism |
| Q48 | AuthN vs AuthZ | Who are you vs what can you do |
| Q49 | Hash vs Encrypt | Hash = one-way; Encrypt = two-way with key |
| Q50 | SQL Injection | Malicious SQL via inputs; prevent with parameterized queries |
| Q54 | Library vs Framework | You call library; framework calls you (IoC) |
| Q60 | H-scale vs V-scale | More servers vs bigger server |

---

## 🤝 Contributing

Found a better explanation? Want to add a code snippet?
- Fork the repo, improve the answer, open a PR.
- PR title format: `[Solution-Day-01] Improve answer for Q23`

---

> ⭐ Star this repo if it helped you! One day of prep at a time.
