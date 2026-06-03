# 🚀 Backend Interview Questions — Day 3: 2 Years Experience (2 YOE)

> **Series:** Backend Interview Prep · Fresher → 10 Years of Experience  
> **Level:** Mid-Junior Developer — 2 Years of Experience  
> **Total Questions:** 60  
> **Answers:** Will be added soon — contributions welcome!

---

## 📌 Series Navigation

| Day | Experience Level | File |
|-----|-----------------|------|
| Day 1 | Fresher (0 YOE) | ✅ [Available](./day-01-fresher.md) |
| Day 2 | 1 Year | ✅ [Available](./day-02-1yoe.md) |
| Day 3 | 2 Years | ✅ You are here |
| Day 4 | 3 Years | 🔜 Coming Soon |
| ... | ... | ... |

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

**Q1. What is caching? What problem does it solve in backend systems?**
> Answer coming soon.  
> 📖 Reference: [Caching Overview — AWS](https://aws.amazon.com/caching/)

---

**Q2. What is the difference between in-memory cache, distributed cache, and CDN cache?**
> Answer coming soon.  
> 📖 Reference: [Types of Caching — Cloudflare](https://www.cloudflare.com/learning/cdn/what-is-caching/)

---

**Q3. Explain the Cache-Aside (Lazy Loading) pattern. What are its pros and cons?**
> Answer coming soon.  
> 📖 Reference: [Cache-Aside Pattern — Microsoft](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside)

---

**Q4. What is the difference between Write-Through and Write-Behind (Write-Back) caching?**
> Answer coming soon.  
> 📖 Reference: [Caching Strategies — Hazelcast](https://hazelcast.com/blog/a-hitchhikers-guide-to-caching-patterns/)

---

**Q5. What is a cache eviction policy? Explain LRU, LFU, and FIFO.**
> Answer coming soon.  
> 📖 Reference: [Cache Eviction Policies — Redis Docs](https://redis.io/docs/reference/eviction/)

---

**Q6. What is a cache stampede (thundering herd problem)? How do you prevent it?**
> Answer coming soon.  
> 📖 Reference: [Cache Stampede — Wikipedia](https://en.wikipedia.org/wiki/Cache_stampede)

---

**Q7. What is cache invalidation? Why is it considered one of the hardest problems in CS?**
> Answer coming soon.  
> 📖 Reference: [Cache Invalidation — Martin Fowler](https://martinfowler.com/bliki/TwoHardThings.html)

---

**Q8. What is Redis? What are its common use cases beyond caching?**
> Answer coming soon.  
> 📖 Reference: [Redis Use Cases — Redis Docs](https://redis.io/docs/about/)

---

**Q9. What is a TTL (Time To Live) in caching? How do you decide what TTL to set?**
> Answer coming soon.  
> 📖 Reference: [TTL in Caching — Cloudflare](https://www.cloudflare.com/learning/cdn/glossary/time-to-live-ttl/)

---

## 2. JWT & OAuth 2.0 Deep Dive

**Q10. What are the security risks of storing a JWT in localStorage vs an HttpOnly cookie?**
> Answer coming soon.  
> 📖 Reference: [JWT Storage — OWASP](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#local-storage)

---

**Q11. What is JWT token expiry? How do you handle token refresh without logging the user out?**
> Answer coming soon.  
> 📖 Reference: [Silent Refresh — Auth0](https://auth0.com/docs/tokens/refresh-tokens/use-refresh-tokens)

---

**Q12. What is token revocation? Why is it hard with JWTs and how do you work around it?**
> Answer coming soon.  
> 📖 Reference: [JWT Revocation — Pragmatic Web Security](https://pragmaticwebsecurity.com/articles/oauthoidc/jwt-token-revocation.html)

---

**Q13. Explain the OAuth 2.0 Authorization Code Flow step by step.**
> Answer coming soon.  
> 📖 Reference: [Authorization Code Flow — Auth0](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow)

---

**Q14. What is PKCE (Proof Key for Code Exchange)? Why was it introduced?**
> Answer coming soon.  
> 📖 Reference: [PKCE — Auth0](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce)

---

**Q15. What is OpenID Connect (OIDC)? How does it differ from OAuth 2.0?**
> Answer coming soon.  
> 📖 Reference: [OIDC vs OAuth — Okta](https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc)

---

**Q16. What is role-based access control (RBAC)? How would you implement it in a REST API?**
> Answer coming soon.  
> 📖 Reference: [RBAC — Auth0](https://auth0.com/docs/manage-users/access-control/rbac)

---

## 3. Database Indexing & Query Optimization

**Q17. What is a composite index? When should you use one vs a single-column index?**
> Answer coming soon.  
> 📖 Reference: [Composite Indexes — Use The Index Luke](https://use-the-index-luke.com/sql/where-clause/the-equals-operator/concatenated-keys)

---

**Q18. What is a covering index? How can it eliminate table lookups?**
> Answer coming soon.  
> 📖 Reference: [Covering Index — Use The Index Luke](https://use-the-index-luke.com/sql/clustering/index-only-scan-covering-index)

---

**Q19. What is a full-text search index? How is it different from a LIKE query?**
> Answer coming soon.  
> 📖 Reference: [Full-Text Search — PostgreSQL Docs](https://www.postgresql.org/docs/current/textsearch.html)

---

**Q20. What is query caching at the database level? How is it different from application-level caching?**
> Answer coming soon.  
> 📖 Reference: [DB Query Cache — MySQL Docs](https://dev.mysql.com/doc/refman/8.0/en/query-cache.html)

---

**Q21. What is a database deadlock at the query level? How do you detect and resolve it?**
> Answer coming soon.  
> 📖 Reference: [Deadlocks — PostgreSQL Docs](https://www.postgresql.org/docs/current/explicit-locking.html#LOCKING-DEADLOCKS)

---

**Q22. What is the difference between a clustered index and a non-clustered index?**
> Answer coming soon.  
> 📖 Reference: [Clustered vs Non-Clustered Index — GeeksForGeeks](https://www.geeksforgeeks.org/difference-between-clustered-and-non-clustered-index/)

---

**Q23. How do you identify slow queries in production? What tools or techniques do you use?**
> Answer coming soon.  
> 📖 Reference: [Slow Query Log — MySQL Docs](https://dev.mysql.com/doc/refman/8.0/en/slow-query-log.html)

---

**Q24. What is database partitioning? What is the difference between horizontal and vertical partitioning?**
> Answer coming soon.  
> 📖 Reference: [DB Partitioning — AWS](https://aws.amazon.com/what-is/database-sharding/)

---

## 4. Docker & Containerization

**Q25. What is a Docker container? How is it different from a virtual machine?**
> Answer coming soon.  
> 📖 Reference: [Containers vs VMs — Docker Docs](https://www.docker.com/resources/what-container/)

---

**Q26. What is a Dockerfile? Explain common instructions: `FROM`, `RUN`, `COPY`, `CMD`, `EXPOSE`, `ENV`.**
> Answer coming soon.  
> 📖 Reference: [Dockerfile Reference — Docker Docs](https://docs.docker.com/engine/reference/builder/)

---

**Q27. What is a Docker image vs a Docker container? What is a Docker registry?**
> Answer coming soon.  
> 📖 Reference: [Docker Concepts — Docker Docs](https://docs.docker.com/get-started/overview/)

---

**Q28. What is Docker Compose? When would you use it?**
> Answer coming soon.  
> 📖 Reference: [Docker Compose — Docker Docs](https://docs.docker.com/compose/)

---

**Q29. What is a multi-stage Docker build? Why is it used to reduce image size?**
> Answer coming soon.  
> 📖 Reference: [Multi-stage Builds — Docker Docs](https://docs.docker.com/build/building/multi-stage/)

---

**Q30. What are Docker volumes? How do they differ from bind mounts?**
> Answer coming soon.  
> 📖 Reference: [Docker Volumes — Docker Docs](https://docs.docker.com/storage/volumes/)

---

**Q31. What is a Docker network? How do containers communicate with each other?**
> Answer coming soon.  
> 📖 Reference: [Docker Networking — Docker Docs](https://docs.docker.com/network/)

---

## 5. Background Jobs & Task Queues

**Q32. What is a background job? Give 3 real-world examples of when you'd use one.**
> Answer coming soon.  
> 📖 Reference: [Background Jobs — Microsoft Azure Docs](https://learn.microsoft.com/en-us/azure/architecture/best-practices/background-jobs)

---

**Q33. What is a message queue? How is it different from a task queue?**
> Answer coming soon.  
> 📖 Reference: [Message Queue vs Task Queue — CloudAMQP](https://www.cloudamqp.com/blog/message-queue-vs-task-queue.html)

---

**Q34. What is at-least-once vs at-most-once vs exactly-once delivery in message queues?**
> Answer coming soon.  
> 📖 Reference: [Message Delivery Guarantees — Cloudflare](https://developers.cloudflare.com/queues/reference/delivery-guarantees/)

---

**Q35. What is a dead letter queue (DLQ)? Why is it important?**
> Answer coming soon.  
> 📖 Reference: [Dead Letter Queue — AWS Docs](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)

---

**Q36. What is job idempotency in background processing? How do you ensure a job is safe to retry?**
> Answer coming soon.  
> 📖 Reference: [Idempotent Workers — Sidekiq Best Practices](https://github.com/sidekiq/sidekiq/wiki/Best-Practices#2-make-your-job-idempotent-and-transactional)

---

**Q37. What is the difference between a cron job and an event-driven background job?**
> Answer coming soon.  
> 📖 Reference: [Cron vs Event-Driven — AWS](https://aws.amazon.com/event-driven-architecture/)

---

## 6. API Design Patterns

**Q38. What is GraphQL? How does it compare to REST in terms of over-fetching and under-fetching?**
> Answer coming soon.  
> 📖 Reference: [GraphQL vs REST — Apollo](https://www.apollographql.com/blog/graphql-vs-rest)

---

**Q39. What is gRPC? What makes it faster than REST for internal service communication?**
> Answer coming soon.  
> 📖 Reference: [gRPC Introduction — gRPC Docs](https://grpc.io/docs/what-is-grpc/introduction/)

---

**Q40. What is the BFF (Backend for Frontend) pattern? When is it useful?**
> Answer coming soon.  
> 📖 Reference: [BFF Pattern — Sam Newman](https://samnewman.io/patterns/architectural/bff/)

---

**Q41. What is the difference between a public API and an internal API? How do you design each differently?**
> Answer coming soon.  
> 📖 Reference: [API Design — Zalando Guidelines](https://opensource.zalando.com/restful-api-guidelines/)

---

**Q42. What is API gateway throttling? How does it protect your backend services?**
> Answer coming soon.  
> 📖 Reference: [API Throttling — AWS API Gateway Docs](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html)

---

## 7. NoSQL Databases

**Q43. What are the different types of NoSQL databases? (Document, Key-Value, Column-Family, Graph)**
> Answer coming soon.  
> 📖 Reference: [NoSQL Types — MongoDB](https://www.mongodb.com/nosql-explained)

---

**Q44. What is MongoDB? How does it store data differently from a relational database?**
> Answer coming soon.  
> 📖 Reference: [MongoDB Introduction — MongoDB Docs](https://www.mongodb.com/docs/manual/introduction/)

---

**Q45. When would you choose a NoSQL database over a relational database?**
> Answer coming soon.  
> 📖 Reference: [SQL vs NoSQL — Datastax](https://www.datastax.com/blog/sql-vs-nosql)

---

**Q46. What is eventual consistency? How does it differ from strong consistency?**
> Answer coming soon.  
> 📖 Reference: [Eventual Consistency — Werner Vogels / AWS](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html)

---

**Q47. What is the CAP theorem? Explain CP vs AP systems with examples.**
> Answer coming soon.  
> 📖 Reference: [CAP Theorem — IBM](https://www.ibm.com/topics/cap-theorem)

---

## 8. Web Security — Intermediate

**Q48. What is the OWASP Top 10? Name at least 5 items and explain them briefly.**
> Answer coming soon.  
> 📖 Reference: [OWASP Top 10 — OWASP](https://owasp.org/www-project-top-ten/)

---

**Q49. What is a security header? Name 4 important HTTP security headers and their purpose.**
> Answer coming soon.  
> 📖 Reference: [Security Headers — OWASP](https://owasp.org/www-project-secure-headers/)

---

**Q50. What is the principle of least privilege? How does it apply to backend systems?**
> Answer coming soon.  
> 📖 Reference: [Least Privilege — OWASP](https://owasp.org/www-community/vulnerabilities/Least_Privilege_Violation)

---

**Q51. What is a secret manager? Why should you never store secrets in your codebase?**
> Answer coming soon.  
> 📖 Reference: [Secrets Management — AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)

---

**Q52. What is dependency confusion / supply chain attack? How do you protect against it?**
> Answer coming soon.  
> 📖 Reference: [Dependency Confusion — Alex Birsan](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610)

---

## 9. Performance & Scalability Basics

**Q53. What is latency vs throughput? How do you measure them?**
> Answer coming soon.  
> 📖 Reference: [Latency vs Throughput — AWS](https://aws.amazon.com/compare/the-difference-between-throughput-and-latency/)

---

**Q54. What is a load balancer? What is the difference between round-robin and least-connections algorithms?**
> Answer coming soon.  
> 📖 Reference: [Load Balancing — NGINX](https://www.nginx.com/resources/glossary/load-balancing/)

---

**Q55. What is connection keep-alive? How does it improve HTTP performance?**
> Answer coming soon.  
> 📖 Reference: [HTTP Keep-Alive — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Keep-Alive)

---

**Q56. What is database read replica? When and why would you use one?**
> Answer coming soon.  
> 📖 Reference: [Read Replicas — AWS RDS Docs](https://aws.amazon.com/rds/features/read-replicas/)

---

**Q57. What is a flame graph? How is it used to profile backend performance?**
> Answer coming soon.  
> 📖 Reference: [Flame Graphs — Brendan Gregg](https://www.brendangregg.com/flamegraphs.html)

---

## 10. Software Architecture Patterns

**Q58. What is the Layered (N-Tier) Architecture? What are the typical layers in a backend app?**
> Answer coming soon.  
> 📖 Reference: [Layered Architecture — Oreilly](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html)

---

**Q59. What is Domain-Driven Design (DDD)? What is a bounded context?**
> Answer coming soon.  
> 📖 Reference: [DDD — Martin Fowler](https://martinfowler.com/bliki/DomainDrivenDesign.html)

---

**Q60. What is the Strangler Fig pattern? How is it used to migrate a monolith to microservices?**
> Answer coming soon.  
> 📖 Reference: [Strangler Fig Pattern — Martin Fowler](https://martinfowler.com/bliki/StranglerFigApplication.html)

---

## 🤝 Contributing

Have a better question or want to add answers?
- Fork this repo, add your changes, and open a Pull Request!
- Follow the format: Question → Answer → Reference link
- PR title format: `[Day-03] Add answer for Q22`

---

## 📅 Up Next

- **Day 4:** 3 Years Experience — Microservices, message brokers, CI/CD pipelines, advanced system design

---

> ⭐ Star this repo if it helped you! Share with a friend preparing for backend interviews.
