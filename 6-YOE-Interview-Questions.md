# 🚀 Backend Interview Questions — Day 7: 6 Years Experience (6 YOE)

> **Series:** Backend Interview Prep · Fresher → 10 Years of Experience  
> **Level:** Senior Developer — 6 Years of Experience  
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
| Day 6 | 5 Years | ✅ [Available](./day-06-5yoe.md) |
| Day 7 | 6 Years | ✅ You are here |
| Day 8 | 7 Years | 🔜 Coming Soon |
| ... | ... | ... |

---

## 📚 Table of Contents

1. [Advanced Distributed Systems](#1-advanced-distributed-systems)
2. [Observability Engineering](#2-observability-engineering)
3. [Advanced Kafka Internals](#3-advanced-kafka-internals)
4. [Database — Expert Level](#4-database--expert-level)
5. [Large-Scale API & Gateway Design](#5-large-scale-api--gateway-design)
6. [Advanced Kubernetes & Orchestration](#6-advanced-kubernetes--orchestration)
7. [Incident Management & SRE](#7-incident-management--sre)
8. [Data Consistency Patterns](#8-data-consistency-patterns)
9. [Advanced Security Engineering](#9-advanced-security-engineering)
10. [Staff-Level Thinking & Communication](#10-staff-level-thinking--communication)

---

## 1. Advanced Distributed Systems

**Q1. What is the PACELC theorem? How does it extend the CAP theorem?**
> Answer coming soon.  
> 📖 Reference: [PACELC — Daniel Abadi](http://dbmsmusings.blogspot.com/2010/04/problems-with-cap-and-yahoos-little.html)

---

**Q2. What is a conflict-free replicated data type (CRDT)? Give a practical use case.**
> Answer coming soon.  
> 📖 Reference: [CRDTs — Martin Kleppmann](https://crdt.tech/)

---

**Q3. What is a logical clock vs a physical clock in distributed systems? What is Hybrid Logical Clock (HLC)?**
> Answer coming soon.  
> 📖 Reference: [Hybrid Logical Clocks — Cockroach Labs](https://www.cockroachlabs.com/blog/living-without-atomic-clocks/)

---

**Q4. What is Google Spanner's TrueTime API? How does it achieve external consistency?**
> Answer coming soon.  
> 📖 Reference: [Google Spanner — Google Research](https://research.google/pubs/pub39966/)

---

**Q5. What is a distributed snapshot? Explain the Chandy-Lamport algorithm.**
> Answer coming soon.  
> 📖 Reference: [Chandy-Lamport — The Paper Trail](https://www.the-paper-trail.org/post/2012-12-12-distributed-snapshots-consistent-cuts-of-distributed-systems/)

---

**Q6. What is tail latency amplification in distributed systems? Why does it get worse as you add more services?**
> Answer coming soon.  
> 📖 Reference: [Tail Latency — Jeff Dean, Google](https://research.google/pubs/pub40801/)

---

**Q7. What is the difference between pessimistic and optimistic concurrency control at the distributed system level?**
> Answer coming soon.  
> 📖 Reference: [Concurrency Control — CMU Database Group](https://15445.courses.cs.cmu.edu/fall2022/slides/17-twophaselocking.pdf)

---

**Q8. What is a leaderless replication model? How does DynamoDB or Cassandra achieve it?**
> Answer coming soon.  
> 📖 Reference: [Leaderless Replication — DDIA Kleppmann](https://dataintensive.net/)

---

**Q9. What is anti-entropy in distributed systems? How does it repair diverged replicas?**
> Answer coming soon.  
> 📖 Reference: [Anti-Entropy — Cassandra Docs](https://cassandra.apache.org/doc/latest/cassandra/operating/repair.html)

---

**Q10. What is a fencing token? How does it prevent a rogue node from causing data corruption after a network partition?**
> Answer coming soon.  
> 📖 Reference: [Fencing Tokens — Martin Kleppmann](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)

---

## 2. Observability Engineering

**Q11. What is OpenTelemetry? How does it unify logs, metrics, and traces across different vendors?**
> Answer coming soon.  
> 📖 Reference: [OpenTelemetry — CNCF](https://opentelemetry.io/docs/what-is-opentelemetry/)

---

**Q12. What is exemplar in Prometheus? How does it bridge metrics and traces?**
> Answer coming soon.  
> 📖 Reference: [Exemplars — Prometheus Docs](https://prometheus.io/docs/prometheus/latest/feature_flags/#exemplars-storage)

---

**Q13. What is cardinality in metrics? Why does high cardinality kill a time-series database?**
> Answer coming soon.  
> 📖 Reference: [Cardinality — Grafana Blog](https://grafana.com/blog/2022/02/15/what-are-cardinality-spikes-and-why-do-they-matter/)

---

**Q14. What is a USE method vs RED method for monitoring? When do you apply each?**
> Answer coming soon.  
> 📖 Reference: [USE and RED — Brendan Gregg](https://www.brendangregg.com/usemethod.html)

---

**Q15. What is continuous profiling? How does it differ from on-demand profiling?**
> Answer coming soon.  
> 📖 Reference: [Continuous Profiling — Pyroscope](https://pyroscope.io/blog/what-is-continuous-profiling/)

---

**Q16. What is synthetic monitoring? How does it complement real-user monitoring (RUM)?**
> Answer coming soon.  
> 📖 Reference: [Synthetic vs RUM — Datadog](https://www.datadoghq.com/blog/synthetic-monitoring-vs-rum/)

---

**Q17. How do you design an alerting system that minimizes false positives and alert fatigue?**
> Answer coming soon.  
> 📖 Reference: [Alerting Best Practices — Google SRE Book](https://sre.google/sre-book/monitoring-distributed-systems/)

---

## 3. Advanced Kafka Internals

**Q18. How does Kafka achieve fault tolerance through replication? What is ISR (In-Sync Replica)?**
> Answer coming soon.  
> 📖 Reference: [Kafka Replication — Confluent](https://developer.confluent.io/courses/architecture/replication/)

---

**Q19. What is the role of ZooKeeper in Kafka? How does KRaft (Kafka Raft) replace it?**
> Answer coming soon.  
> 📖 Reference: [KRaft — Confluent](https://developer.confluent.io/learn/kraft/)

---

**Q20. What is Kafka's min.insync.replicas setting? How does it relate to durability guarantees?**
> Answer coming soon.  
> 📖 Reference: [min.insync.replicas — Kafka Docs](https://kafka.apache.org/documentation/#brokerconfigs_min.insync.replicas)

---

**Q21. What is a Kafka consumer rebalance? What are the different partition assignment strategies?**
> Answer coming soon.  
> 📖 Reference: [Consumer Rebalancing — Confluent](https://www.confluent.io/blog/incremental-cooperative-rebalancing-in-kafka/)

---

**Q22. What is idempotent producer in Kafka? How does it prevent duplicate messages?**
> Answer coming soon.  
> 📖 Reference: [Idempotent Producer — Confluent](https://developer.confluent.io/courses/architecture/producer-config/)

---

## 4. Database — Expert Level

**Q23. What is a two-phase commit (2PC)? What are its failure modes?**
> Answer coming soon.  
> 📖 Reference: [2PC — Martin Kleppmann DDIA](https://dataintensive.net/)

---

**Q24. What is three-phase commit (3PC)? How does it attempt to solve 2PC's blocking problem?**
> Answer coming soon.  
> 📖 Reference: [3PC — The Paper Trail](https://www.the-paper-trail.org/post/2008-11-27-consensus-protocols-three-phase-commit/)

---

**Q25. What is a distributed SQL database? How does CockroachDB or YugabyteDB achieve global ACID transactions?**
> Answer coming soon.  
> 📖 Reference: [CockroachDB Architecture — CockroachDB Docs](https://www.cockroachlabs.com/docs/stable/architecture/overview.html)

---

**Q26. What is connection multiplexing at the database proxy level? How does PgBouncer's transaction mode work?**
> Answer coming soon.  
> 📖 Reference: [PgBouncer Transaction Mode — PgBouncer Docs](https://www.pgbouncer.org/config.html#pool_mode)

---

**Q27. What is the difference between a materialized view and a regular view? What are refresh strategies?**
> Answer coming soon.  
> 📖 Reference: [Materialized Views — PostgreSQL Docs](https://www.postgresql.org/docs/current/rules-materializedviews.html)

---

**Q28. What is a partial index? When does it outperform a full index?**
> Answer coming soon.  
> 📖 Reference: [Partial Indexes — PostgreSQL Docs](https://www.postgresql.org/docs/current/indexes-partial.html)

---

**Q29. What is database bloat in PostgreSQL? How does it affect performance and how do you fix it?**
> Answer coming soon.  
> 📖 Reference: [Table Bloat — pgExperts](https://www.pgexperts.com/document.html?id=60)

---

## 5. Large-Scale API & Gateway Design

**Q30. How would you design an API gateway from scratch? What are the core responsibilities it must handle?**
> Answer coming soon.  
> 📖 Reference: [API Gateway — Microsoft Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway)

---

**Q31. What is a GraphQL N+1 problem? How do you solve it with DataLoader?**
> Answer coming soon.  
> 📖 Reference: [DataLoader — GraphQL Foundation](https://github.com/graphql/dataloader)

---

**Q32. What is GraphQL federation? How does it allow multiple teams to own parts of a single schema?**
> Answer coming soon.  
> 📖 Reference: [GraphQL Federation — Apollo](https://www.apollographql.com/docs/federation/)

---

**Q33. What is API monetization? What technical components are needed to meter and bill API usage?**
> Answer coming soon.  
> 📖 Reference: [API Monetization — Kong](https://konghq.com/blog/api-monetization)

---

**Q34. What is a shadow deployment for APIs? How do you use it to test new versions safely?**
> Answer coming soon.  
> 📖 Reference: [Shadow Traffic — Martin Fowler](https://martinfowler.com/bliki/DarkLaunching.html)

---

## 6. Advanced Kubernetes & Orchestration

**Q35. What is a Kubernetes operator? How does it extend the Kubernetes API with custom controllers?**
> Answer coming soon.  
> 📖 Reference: [Kubernetes Operators — Red Hat](https://www.redhat.com/en/topics/containers/what-is-a-kubernetes-operator)

---

**Q36. What is pod disruption budget (PDB)? How does it protect availability during node drains?**
> Answer coming soon.  
> 📖 Reference: [Pod Disruption Budget — Kubernetes Docs](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)

---

**Q37. What is vertical pod autoscaling (VPA)? How does it differ from HPA?**
> Answer coming soon.  
> 📖 Reference: [VPA — Kubernetes Docs](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)

---

**Q38. What is KEDA (Kubernetes Event-Driven Autoscaling)? How does it autoscale based on Kafka lag?**
> Answer coming soon.  
> 📖 Reference: [KEDA — KEDA Docs](https://keda.sh/docs/latest/concepts/)

---

**Q39. What is a Kubernetes admission controller? What is a mutating vs validating webhook?**
> Answer coming soon.  
> 📖 Reference: [Admission Controllers — Kubernetes Docs](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

---

**Q40. What is eBPF? How is it used in Kubernetes networking and observability tools like Cilium?**
> Answer coming soon.  
> 📖 Reference: [eBPF — ebpf.io](https://ebpf.io/what-is-ebpf/)

---

## 7. Incident Management & SRE

**Q41. What is the MTTR, MTBF, and MTTD? How do you optimize each?**
> Answer coming soon.  
> 📖 Reference: [MTTR vs MTBF — PagerDuty](https://www.pagerduty.com/resources/learn/mttr-mttd-mttf-mtbf/)

---

**Q42. What is an incident commander role? How do you run a structured incident response?**
> Answer coming soon.  
> 📖 Reference: [Incident Command — PagerDuty](https://response.pagerduty.com/before/different_roles/)

---

**Q43. What is toil in SRE? How do you measure and reduce it?**
> Answer coming soon.  
> 📖 Reference: [Toil — Google SRE Book](https://sre.google/sre-book/eliminating-toil/)

---

**Q44. What is a game day exercise? How do you simulate failure scenarios safely in production?**
> Answer coming soon.  
> 📖 Reference: [Game Days — AWS](https://wa.aws.amazon.com/wat.concept.gameday.en.html)

---

**Q45. What is a capacity planning process? What inputs and models does it use?**
> Answer coming soon.  
> 📖 Reference: [Capacity Planning — Google SRE Book](https://sre.google/sre-book/software-engineering-in-sre/)

---

## 8. Data Consistency Patterns

**Q46. What is the Saga pattern's compensation transaction? How do you design one for a payment system?**
> Answer coming soon.  
> 📖 Reference: [Compensation Transactions — Microsoft](https://learn.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction)

---

**Q47. What is read-your-writes consistency? How do you implement it across replicated databases?**
> Answer coming soon.  
> 📖 Reference: [Read-Your-Writes — DDIA Kleppmann](https://dataintensive.net/)

---

**Q48. What is monotonic read consistency? How do session tokens help enforce it?**
> Answer coming soon.  
> 📖 Reference: [Monotonic Reads — DDIA Kleppmann](https://dataintensive.net/)

---

**Q49. What is the dual-write problem? What patterns exist to solve it reliably?**
> Answer coming soon.  
> 📖 Reference: [Dual Write Problem — Thorben Janssen](https://thorben-janssen.com/dual-writes/)

---

**Q50. What is the difference between optimistic UI updates and server-confirmed updates? How do you handle rollback on the backend?**
> Answer coming soon.  
> 📖 Reference: [Optimistic Updates — Apollo GraphQL](https://www.apollographql.com/docs/react/performance/optimistic-ui/)

---

## 9. Advanced Security Engineering

**Q51. What is a confused deputy attack? Give an example in the context of cloud IAM roles.**
> Answer coming soon.  
> 📖 Reference: [Confused Deputy — AWS Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)

---

**Q52. What is SSRF (Server-Side Request Forgery)? How is it exploited and mitigated in cloud environments?**
> Answer coming soon.  
> 📖 Reference: [SSRF — OWASP](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)

---

**Q53. What is secrets rotation? How do you rotate secrets without downtime?**
> Answer coming soon.  
> 📖 Reference: [Secrets Rotation — AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html)

---

**Q54. What is a software bill of materials (SBOM)? Why is it gaining importance in supply chain security?**
> Answer coming soon.  
> 📖 Reference: [SBOM — CISA](https://www.cisa.gov/sbom)

---

**Q55. What is container image scanning? How do tools like Trivy or Snyk fit into CI pipelines?**
> Answer coming soon.  
> 📖 Reference: [Trivy — Aqua Security](https://trivy.dev/)

---

## 10. Staff-Level Thinking & Communication

**Q56. How do you communicate a major architectural change to both engineering and non-engineering stakeholders?**
> Answer coming soon.  
> 📖 Reference: [Communicating Architecture — Will Larson](https://lethain.com/presenting-technical-vision/)

---

**Q57. What is a Request for Comments (RFC) process in engineering? How does it enable async decision-making?**
> Answer coming soon.  
> 📖 Reference: [RFC Process — Rust Lang](https://github.com/rust-lang/rfcs)

---

**Q58. How do you identify and manage technical risk in a large project?**
> Answer coming soon.  
> 📖 Reference: [Technical Risk — Will Larson](https://lethain.com/estimate-technical-risk/)

---

**Q59. What is the difference between a platform team and a product team? How do they interact?**
> Answer coming soon.  
> 📖 Reference: [Team Topologies — Team Topologies Book](https://teamtopologies.com/key-concepts)

---

**Q60. How do you balance shipping velocity with long-term architectural health in a fast-growing team?**
> Answer coming soon.  
> 📖 Reference: [Velocity vs Quality — Martin Fowler](https://martinfowler.com/articles/is-quality-worth-cost.html)

---

## 🤝 Contributing

Have a better question or want to add answers?
- Fork this repo, add your changes, and open a Pull Request!
- Follow the format: Question → Answer → Reference link
- PR title format: `[Day-07] Add answer for Q13`

---

## 📅 Up Next

- **Day 8:** 7 Years Experience — Cloud-native architecture, serverless patterns, advanced SRE, cross-org technical influence

---

> ⭐ Star this repo if it helped you! Share with a friend preparing for backend interviews.
