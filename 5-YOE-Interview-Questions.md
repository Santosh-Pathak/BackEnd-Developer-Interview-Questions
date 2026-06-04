# 🚀 Backend Interview Questions — Day 6: 5 Years Experience (5 YOE)

> **Series:** Backend Interview Prep · Fresher → 10 Years of Experience  
> **Level:** Senior Developer — 5 Years of Experience  
> **Total Questions:** 60  
> **Answers:** Will be added soon — contributions welcome!

---

## 📌 Series Navigation

| Day | Experience Level | File |
|-----|-----------------|------|
| Day 1 | Fresher (0 YOE) | ✅ [Available](./day-01-fresher.md) |
| Day 2 | 1 Year | ✅ [Available](./day-02-1yoe.md) |
| Day 3 | 2 Years | ✅ [Available](./day-03-2yoe.md) |
| Day 4 | 3 Years | ✅ [Available](./day-04-3yoe.md) |
| Day 5 | 4 Years | ✅ [Available](./day-05-4yoe.md) |
| Day 6 | 5 Years | ✅ You are here |
| Day 7 | 6 Years | 🔜 Coming Soon |
| ... | ... | ... |

---

## 📚 Table of Contents

1. [Distributed Systems — Deep Dive](#1-distributed-systems--deep-dive)
2. [Database Internals](#2-database-internals)
3. [Large-Scale System Design](#3-large-scale-system-design)
4. [Platform Engineering & Developer Experience](#4-platform-engineering--developer-experience)
5. [Advanced Kafka & Streaming](#5-advanced-kafka--streaming)
6. [Networking — Deep Dive](#6-networking--deep-dive)
7. [Storage Systems](#7-storage-systems)
8. [Multi-Tenancy & SaaS Architecture](#8-multi-tenancy--saas-architecture)
9. [Cost Engineering & Efficiency](#9-cost-engineering--efficiency)
10. [Senior Engineering Practices](#10-senior-engineering-practices)

---

## 1. Distributed Systems — Deep Dive

**Q1. What is the FLP impossibility theorem? What does it mean for distributed consensus?**
> Answer coming soon.  
> 📖 Reference: [FLP Impossibility — The Paper Trail](https://www.the-paper-trail.org/post/2008-08-13-a-brief-tour-of-flp-impossibility/)

---

**Q2. What is the Raft consensus algorithm? How does leader election work?**
> Answer coming soon.  
> 📖 Reference: [Raft Consensus — Raft Visualization](https://raft.github.io/)

---

**Q3. What is the Paxos algorithm? How does it differ from Raft?**
> Answer coming soon.  
> 📖 Reference: [Paxos Made Simple — Leslie Lamport](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)

---

**Q4. What are vector clocks? How do they help track causality in distributed systems?**
> Answer coming soon.  
> 📖 Reference: [Vector Clocks — Amazon Dynamo Paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)

---

**Q5. What is the difference between linearizability and serializability?**
> Answer coming soon.  
> 📖 Reference: [Linearizability vs Serializability — Peter Bailis](http://www.bailis.org/blog/linearizability-versus-serializability/)

---

**Q6. What is a split-brain problem? How do distributed systems handle it?**
> Answer coming soon.  
> 📖 Reference: [Split Brain — Martin Kleppmann](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)

---

**Q7. What is idempotency key? How do you design an idempotent payment API?**
> Answer coming soon.  
> 📖 Reference: [Idempotency Keys — Stripe Docs](https://stripe.com/docs/api/idempotent_requests)

---

**Q8. What is the gossip protocol? How is it used in distributed systems like Cassandra?**
> Answer coming soon.  
> 📖 Reference: [Gossip Protocol — Cassandra Docs](https://cassandra.apache.org/doc/latest/cassandra/operating/gossip.html)

---

**Q9. What is a Merkle tree? How is it used for data verification in distributed systems?**
> Answer coming soon.  
> 📖 Reference: [Merkle Trees — Cloudflare](https://www.cloudflare.com/learning/security/what-is-a-merkle-tree/)

---

**Q10. What is the difference between strong consistency, weak consistency, and eventual consistency? Give real system examples of each.**
> Answer coming soon.  
> 📖 Reference: [Consistency Models — Jepsen](https://jepsen.io/consistency)

---

## 2. Database Internals

**Q11. How does a B-Tree index work internally? Why is it preferred over a hash index for range queries?**
> Answer coming soon.  
> 📖 Reference: [B-Tree Indexes — Use The Index Luke](https://use-the-index-luke.com/sql/anatomy/the-tree)

---

**Q12. What is MVCC (Multi-Version Concurrency Control)? How does PostgreSQL implement it?**
> Answer coming soon.  
> 📖 Reference: [MVCC — PostgreSQL Docs](https://www.postgresql.org/docs/current/mvcc-intro.html)

---

**Q13. What is a transaction isolation level? Compare READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, and SERIALIZABLE.**
> Answer coming soon.  
> 📖 Reference: [Isolation Levels — PostgreSQL Docs](https://www.postgresql.org/docs/current/transaction-iso.html)

---

**Q14. What is a phantom read? What isolation level prevents it?**
> Answer coming soon.  
> 📖 Reference: [Phantom Reads — Martin Kleppmann](https://martin.kleppmann.com/2015/09/26/transactions-at-odd-isolation-levels.html)

---

**Q15. What is vacuum in PostgreSQL? Why is it necessary and what happens if it is neglected?**
> Answer coming soon.  
> 📖 Reference: [VACUUM — PostgreSQL Docs](https://www.postgresql.org/docs/current/routine-vacuuming.html)

---

**Q16. What is a LSM tree (Log-Structured Merge Tree)? How does it differ from a B-Tree? Which databases use it?**
> Answer coming soon.  
> 📖 Reference: [LSM Trees — Ben Stopford](https://www.benstopford.com/2015/02/14/log-structured-merge-trees/)

---

**Q17. What is write amplification in databases? Why does it matter for SSD storage?**
> Answer coming soon.  
> 📖 Reference: [Write Amplification — RocksDB Docs](https://github.com/facebook/rocksdb/wiki/Write-Amplification)

---

**Q18. How does Cassandra handle writes and reads differently from PostgreSQL?**
> Answer coming soon.  
> 📖 Reference: [Cassandra Architecture — Datastax](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/architecture/archIntro.html)

---

## 3. Large-Scale System Design

**Q19. How would you design a system like Twitter/X — focusing on the feed generation problem?**
> Answer coming soon.  
> 📖 Reference: [Twitter Architecture — High Scalability](http://highscalability.com/blog/2013/7/8/the-architecture-twitter-uses-to-deal-with-150m-active-users.html)

---

**Q20. How would you design a ride-sharing system like Uber? Focus on location tracking and matching.**
> Answer coming soon.  
> 📖 Reference: [Uber Architecture — Uber Engineering Blog](https://www.uber.com/en-IN/blog/microservice-architecture/)

---

**Q21. How would you design a distributed cache like Memcached or Redis Cluster?**
> Answer coming soon.  
> 📖 Reference: [Distributed Cache Design — ByteByteGo](https://bytebytego.com/courses/system-design-interview/design-a-cache-system)

---

**Q22. How would you design a global content delivery system? What challenges arise with geo-distribution?**
> Answer coming soon.  
> 📖 Reference: [CDN Design — Cloudflare Blog](https://blog.cloudflare.com/the-internet-in-2020/)

---

**Q23. How would you design a real-time collaborative document editor (like Google Docs)?**
> Answer coming soon.  
> 📖 Reference: [Operational Transformation — Google Research](https://drive.googleblog.com/2010/09/whats-different-about-new-google-docs.html)

---

**Q24. How would you design a distributed ID generation system? Compare UUID, Snowflake, and ULID.**
> Answer coming soon.  
> 📖 Reference: [Snowflake ID — Twitter Engineering](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake)

---

## 4. Platform Engineering & Developer Experience

**Q25. What is platform engineering? How does it differ from traditional DevOps?**
> Answer coming soon.  
> 📖 Reference: [Platform Engineering — Humanitec](https://humanitec.com/blog/what-is-platform-engineering)

---

**Q26. What is an Internal Developer Platform (IDP)? What problems does it solve?**
> Answer coming soon.  
> 📖 Reference: [Internal Developer Platform — internaldeveloperplatform.org](https://internaldeveloperplatform.org/what-is-an-internal-developer-platform/)

---

**Q27. What is GitOps? How does it differ from traditional CI/CD?**
> Answer coming soon.  
> 📖 Reference: [GitOps — Weaveworks](https://www.weave.works/technologies/gitops/)

---

**Q28. What is Helm in the context of Kubernetes? What problem does it solve?**
> Answer coming soon.  
> 📖 Reference: [Helm — Helm Docs](https://helm.sh/docs/intro/using_helm/)

---

**Q29. What is a service catalog? Why is it important in organizations with many microservices?**
> Answer coming soon.  
> 📖 Reference: [Service Catalog — Backstage.io](https://backstage.io/docs/features/software-catalog/)

---

**Q30. What is observability-as-code? How do you manage dashboards and alerts in version control?**
> Answer coming soon.  
> 📖 Reference: [Observability as Code — Grafana](https://grafana.com/blog/2022/12/06/the-complete-guide-to-managing-grafana-as-code/)

---

## 5. Advanced Kafka & Streaming

**Q31. What is Kafka's storage architecture? How do segments, offsets, and retention work?**
> Answer coming soon.  
> 📖 Reference: [Kafka Storage — Confluent](https://developer.confluent.io/courses/architecture/get-started/)

---

**Q32. What is a Kafka compacted topic? When would you use one?**
> Answer coming soon.  
> 📖 Reference: [Log Compaction — Kafka Docs](https://kafka.apache.org/documentation/#compaction)

---

**Q33. What is exactly-once semantics in Kafka? How does it work internally?**
> Answer coming soon.  
> 📖 Reference: [Exactly Once — Confluent](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)

---

**Q34. What is Kafka Streams? How does it compare to using a separate stream processor like Flink?**
> Answer coming soon.  
> 📖 Reference: [Kafka Streams — Confluent](https://developer.confluent.io/courses/kafka-streams/get-started/)

---

**Q35. What is schema registry? Why is Avro or Protobuf preferred over JSON in Kafka?**
> Answer coming soon.  
> 📖 Reference: [Schema Registry — Confluent](https://docs.confluent.io/platform/current/schema-registry/index.html)

---

## 6. Networking — Deep Dive

**Q36. What is HTTP/2? What improvements does it bring over HTTP/1.1?**
> Answer coming soon.  
> 📖 Reference: [HTTP/2 — Cloudflare](https://www.cloudflare.com/learning/performance/http2-vs-http1.1/)

---

**Q37. What is HTTP/3 and QUIC? Why was UDP chosen as the transport?**
> Answer coming soon.  
> 📖 Reference: [HTTP/3 — Cloudflare](https://www.cloudflare.com/learning/performance/what-is-http3/)

---

**Q38. What is TLS 1.3? What security and performance improvements does it bring?**
> Answer coming soon.  
> 📖 Reference: [TLS 1.3 — Cloudflare](https://www.cloudflare.com/learning/ssl/why-use-tls-1.3/)

---

**Q39. What is a WebSocket? How does the upgrade handshake from HTTP work?**
> Answer coming soon.  
> 📖 Reference: [WebSockets — MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)

---

**Q40. What is connection draining? Why is it needed during rolling deployments?**
> Answer coming soon.  
> 📖 Reference: [Connection Draining — AWS Docs](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/config-conn-drain.html)

---

## 7. Storage Systems

**Q41. What is the difference between row-oriented and column-oriented databases? When do you use each?**
> Answer coming soon.  
> 📖 Reference: [Row vs Column Storage — Kleppmann DDIA](https://dataintensive.net/)

---

**Q42. What is Apache Parquet? Why is it preferred for analytical workloads?**
> Answer coming soon.  
> 📖 Reference: [Apache Parquet — Apache Docs](https://parquet.apache.org/docs/)

---

**Q43. What is a write-optimized vs read-optimized storage layout? Give database examples.**
> Answer coming soon.  
> 📖 Reference: [Storage Layouts — CMU Database Group](https://15445.courses.cs.cmu.edu/fall2022/slides/04-storage2.pdf)

---

**Q44. What is tiered storage in Kafka or object stores? How does it reduce cost?**
> Answer coming soon.  
> 📖 Reference: [Tiered Storage — Confluent](https://docs.confluent.io/platform/current/kafka/tiered-storage.html)

---

**Q45. What is the difference between strong and eventual consistency in DynamoDB? How do you control it?**
> Answer coming soon.  
> 📖 Reference: [DynamoDB Consistency — AWS Docs](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)

---

## 8. Multi-Tenancy & SaaS Architecture

**Q46. What are the three models of multi-tenancy: silo, pool, and bridge? What are the trade-offs?**
> Answer coming soon.  
> 📖 Reference: [SaaS Multi-Tenancy — AWS](https://docs.aws.amazon.com/whitepapers/latest/saas-architecture-fundamentals/multi-tenancy-in-a-saas-environment.html)

---

**Q47. How do you implement tenant isolation at the database level in a multi-tenant system?**
> Answer coming soon.  
> 📖 Reference: [Tenant Isolation — AWS SaaS Factory](https://aws.amazon.com/partners/programs/saas-factory/)

---

**Q48. What is tenant-aware rate limiting? How is it different from global rate limiting?**
> Answer coming soon.  
> 📖 Reference: [Per-Tenant Rate Limiting — AWS API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html)

---

**Q49. What is a noisy neighbor problem in multi-tenant systems? How do you prevent it?**
> Answer coming soon.  
> 📖 Reference: [Noisy Neighbor — Microsoft Azure](https://learn.microsoft.com/en-us/azure/architecture/antipatterns/noisy-neighbor/noisy-neighbor)

---

## 9. Cost Engineering & Efficiency

**Q50. What is FinOps? Why is cost awareness important for senior backend engineers?**
> Answer coming soon.  
> 📖 Reference: [FinOps — FinOps Foundation](https://www.finops.org/introduction/what-is-finops/)

---

**Q51. What are the most common sources of cloud cost waste in a backend system?**
> Answer coming soon.  
> 📖 Reference: [Cloud Cost Optimization — AWS](https://aws.amazon.com/aws-cost-management/cost-optimization/)

---

**Q52. What is spot/preemptible instance? When is it safe to use one for backend workloads?**
> Answer coming soon.  
> 📖 Reference: [Spot Instances — AWS](https://aws.amazon.com/ec2/spot/)

---

**Q53. How do you right-size compute resources for a backend service? What metrics do you use?**
> Answer coming soon.  
> 📖 Reference: [Right Sizing — AWS Compute Optimizer](https://aws.amazon.com/compute-optimizer/)

---

## 10. Senior Engineering Practices

**Q54. What is Conway's Law? How does it influence microservices team structure?**
> Answer coming soon.  
> 📖 Reference: [Conway's Law — Martin Fowler](https://martinfowler.com/bliki/ConwaysLaw.html)

---

**Q55. What is the Inverse Conway Maneuver? How do you use org structure to drive architecture?**
> Answer coming soon.  
> 📖 Reference: [Inverse Conway — ThoughtWorks](https://www.thoughtworks.com/radar/techniques/inverse-conway-maneuver)

---

**Q56. How do you evaluate the operational readiness of a new service before it goes to production?**
> Answer coming soon.  
> 📖 Reference: [Production Readiness — Google SRE Book](https://sre.google/sre-book/production-readiness-review/)

---

**Q57. What is a design doc (technical design document)? What sections should it always include?**
> Answer coming soon.  
> 📖 Reference: [Design Docs — Google Engineering](https://www.industrialempathy.com/posts/design-docs-at-google/)

---

**Q58. How do you mentor junior developers effectively without becoming a bottleneck?**
> Answer coming soon.  
> 📖 Reference: [Mentoring — Will Larson](https://lethain.com/mentoring/)

---

**Q59. What is an on-call rotation? What makes a good on-call experience?**
> Answer coming soon.  
> 📖 Reference: [On-Call — PagerDuty](https://response.pagerduty.com/oncall/being_oncall/)

---

**Q60. How do you decide whether to build vs buy a component for your backend system?**
> Answer coming soon.  
> 📖 Reference: [Build vs Buy — Martin Fowler](https://martinfowler.com/bliki/BuyOrBuild.html)

---

## 🤝 Contributing

Have a better question or want to add answers?
- Fork this repo, add your changes, and open a Pull Request!
- Follow the format: Question → Answer → Reference link
- PR title format: `[Day-06] Add answer for Q11`

---

## 📅 Up Next

- **Day 7:** 6 Years Experience — Advanced distributed systems, Kafka internals, observability engineering, staff-level thinking

---

> ⭐ Star this repo if it helped you! Share with a friend preparing for backend interviews.
