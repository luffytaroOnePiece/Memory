# 🗄️ Caching System Design — Interview Preparation Guide

> **SDE → Senior → Staff → Principal**  
> System Design · Distributed Systems · Performance Engineering

---

## 📊 Level Expectations at a Glance

| Topic | SDE (L3–L4) | Senior SDE (L5) | Staff (L6) | Principal (L7+) |
|---|---|---|---|---|
| Cache Basics | Define & use | Choose patterns | Design strategy | Org-wide vision |
| Eviction Policies | LRU / LFU | Compare tradeoffs | Custom policies | Research-level |
| Consistency | Eventual basics | Coherence models | Distributed CAP | Novel guarantees |
| Scale & Capacity | Basic sizing | Working sets | Fleet planning | Cost modeling |
| Failure Handling | Retry logic | Fallback patterns | Chaos resilience | Platform SLOs |
| Observability | Hit/miss rate | P99 latency | Anomaly detection | Infra-wide metrics |
| Security | Awareness | Data sanitization | Threat modeling | Policy framework |

---

## 🟦 SDE (L3–L4) — Foundational Caching Knowledge

> **What interviewers expect:** You can explain what a cache is, correctly use common caching patterns in code, and reason about basic tradeoffs.

### 1. Core Concepts
- What is a cache and why do we use it? (latency reduction, DB offloading)
- Cache hit vs cache miss — what happens in each case
- Hot data vs cold data — intuition for what to cache
- TTL (Time-To-Live) — setting expiration and why it matters

> 💡 Be able to sketch a read path: `Request → Cache → (miss) → DB → Cache write`

### 2. Eviction Policies
- **LRU** (Least Recently Used) — how it works, when to use
- **LFU** (Least Frequently Used) — difference from LRU
- **FIFO** — simple but often suboptimal
- **Random eviction** — sometimes surprisingly competitive

> 💡 Know how to implement LRU using a HashMap + Doubly Linked List

### 3. Caching Patterns

| Pattern | How it works | Best for |
|---|---|---|
| Cache-Aside | App checks cache, falls back to DB, populates cache | General purpose reads |
| Read-Through | Cache auto-loads from DB on miss | Transparent caching |
| Write-Through | Write to cache AND DB simultaneously | Strong consistency |
| Write-Back | Write to cache, async flush to DB | High write throughput |

> 💡 Cache-aside is the most commonly used pattern. Know its stale data risk.

### 4. Where Caches Live
- **In-process / in-memory** (e.g., a HashMap in your service)
- **Sidecar / local Redis / Memcached**
- **Distributed cache** (shared across service instances)
- **CDN / edge caching** for static assets

### 5. Basic Consistency
- Eventual consistency — cache may be stale for TTL duration
- Cache invalidation on write — how and when to evict or update
- Stale reads — product-level tolerance for stale data

### 6. Interview Questions at This Level
- Design a key-value store with a cache layer
- How would you add caching to reduce DB load for a read-heavy API?
- What is cache invalidation and why is it hard?
- Implement LRU cache in code

---

## 🟩 Senior SDE (L5) — Architectural Thinking

> **What interviewers expect:** You can architect caching for a production system, reason about tradeoffs, and discuss failure modes independently.

### 1. Distributed Caching
- **Redis Cluster vs Memcached** — consistency, data structures, persistence
- **Consistent hashing** — how cache keys map to nodes, virtual nodes
- **Replication in Redis** — primary/replica, sentinel, cluster modes
- **Partitioning strategies** — hash-based, range-based, directory-based

> 💡 Redis supports rich data types (sorted sets, streams); Memcached is simpler/faster for plain KV

### 2. Cache Stampede / Thundering Herd
- **Problem:** many concurrent misses hitting the DB simultaneously
- **Solutions:**
  - Mutex/lock-based refresh
  - Probabilistic early expiration (XFetch algorithm)
  - Request coalescing — deduplicate concurrent misses into one DB fetch
  - Background refresh — proactive TTL renewal before expiry

### 3. Hot Key Problem
- A single key receiving extreme request volume (e.g., celebrity post)
- **Solution 1:** Local in-process cache layer (L1) in front of Redis (L2)
- **Solution 2:** Key sharding — fan out reads to N copies (`key:0` to `key:N-1`)
- **Solution 3:** Read replicas for hot Redis slots

### 4. Cache Sizing and Capacity Planning
- Working set analysis — what % of data is actually "hot"
- Cache hit rate as a function of cache size (Zipf distribution)
- Memory estimation: `key count × avg entry size + overhead`
- Eviction rate monitoring to detect undersized caches

> 💡 Rule of thumb: 20% of data often accounts for 80% of reads. Size accordingly.

### 5. Write Strategies Deep Dive
- **Write-through tradeoff:** safe but double-write latency for every write
- **Write-behind:** high throughput but risk of data loss on crash
- Choose write strategy based on read:write ratio and durability requirements

### 6. Negative Caching
- Cache "miss" results (e.g., user not found = null) to prevent DB hammering
- Use a short TTL for negative entries to avoid stale negatives
- Differentiate between "not found" and "not cached yet"

### 7. CDN Caching
- **Cache-Control headers:** `max-age`, `s-maxage`, `no-cache`, `no-store`, `must-revalidate`
- **ETags and conditional requests** (`If-None-Match`) for efficient validation
- **CDN purging strategies** — full purge vs tag-based vs URL-based
- **Origin shield** — collapsing CDN misses before hitting origin

### 8. Interview Questions at This Level
- Design a distributed rate limiter using Redis
- How do you cache a Twitter/Instagram timeline? Fan-out on write vs read?
- Design a cache for a product catalog with 10M items
- How do you handle cache invalidation across microservices?

---

## 🟨 Staff Engineer (L6) — System Design & Cross-Team Impact

> **What interviewers expect:** You drive caching strategy across services, make principled tradeoffs, reason about correctness under failure, and lead technical direction.

### 1. Multi-Tier Caching Architecture

```
Request
   │
   ▼
[L1] In-process cache (Caffeine/Guava) — sub-ms, no network
   │ miss
   ▼
[L2] Local Redis / sidecar — shared within host
   │ miss
   ▼
[L3] Distributed Redis / Memcached — shared across fleet
   │ miss
   ▼
[L4] CDN — geographic distribution, edge caching
   │ miss
   ▼
[Origin DB]
```

- Design promotion/demotion policies between tiers
- Define coherence protocol between tiers
- Articulate when each tier is appropriate

### 2. Consistency Models and CAP Tradeoffs

| Model | Description | Cost |
|---|---|---|
| Strong consistency | Synchronous invalidation across all nodes | High latency |
| Eventual consistency | Async propagation, bounded staleness | May serve stale data |
| Read-your-writes | Guarantee within session or user context | Session affinity needed |
| Monotonic reads | Never see older data after seeing newer | Complexity |

> 💡 Define SLAs explicitly: "data may be stale up to X seconds per entity type"

### 3. Cache Coherence in Distributed Systems
- **Broadcast invalidation** — publish invalidation events via pub/sub
- **Tag-based invalidation** — group related entries, invalidate by tag
- **Version vectors** — track cache version per entity, reject stale writes
- **Two-phase invalidation** — mark-then-delete to prevent race conditions

### 4. Cache Warming Strategies
- **Cold start problem:** empty cache causes DB overload on deployment
- **Pre-warming:** populate cache from DB dump/snapshot before traffic
- **Gradual traffic shifting:** bring up new instances with warming enabled
- **Cache seeding from access logs:** replay historical requests
- **Warm transfer:** between cache instances during rolling deploys

### 5. Failure Modes and Resilience

| Failure Mode | Impact | Mitigation |
|---|---|---|
| Cache node failure | Increased DB load | Circuit breaker + fallback |
| Cache poisoning | Serving incorrect data | Validate before caching |
| Network partition | Stale data served | Stale-while-revalidate |
| Redis failover | Availability gap | Sentinel/Cluster auto-failover |
| Miss storm | DB overload cascade | Request coalescing + bulkheads |

> 💡 Walk through failure scenarios end-to-end and describe mitigation for each

### 6. Observability and Monitoring
- **Key metrics:** hit rate, miss rate, eviction rate, latency (p50/p95/p99), memory utilization
- **Per-key observability** for hot key detection
- **Alerting:** hit rate drop > threshold (indicates eviction/invalidation storm)
- **Cache efficiency dashboards:** cost per request served, bytes per hit
- **Distributed tracing:** correlate cache interactions (Jaeger, Zipkin, X-Ray)

### 7. Caching for Specific Scenarios

| Scenario | Key Caching Decision |
|---|---|
| Social media feed | Fan-out on write vs read; precomputed vs on-demand |
| Rate limiter | Redis INCR/DECR + sliding window with atomic ops |
| Search autocomplete | Redis sorted sets (score = frequency) or in-memory trie |
| E-commerce inventory | Short TTL on stock counts, write-through for accuracy |
| API gateway | Varnish/Nginx proxy cache, response caching with Vary headers |
| Session management | Redis with TTL per session, sliding expiry |

### 8. Caching and Database Interaction
- **Dual-write atomicity problem:** write cache + DB, one fails
- **CDC (Change Data Capture):** drive cache invalidation from DB events
- **Read replicas as cache:** when is this better than a dedicated cache?
- **CQRS + cache:** separate read models pre-materialized for query patterns

### 9. Interview Questions at This Level
- Design a caching platform/service that 50 microservices can use
- How do you migrate a live system from no-cache to a caching layer with zero downtime?
- You have a cache hit rate of 60% in production. How do you diagnose and fix it?
- Design the caching strategy for a payments system where stale data = real money risk

---

## 🟥 Principal Engineer (L7+) — Platform Strategy & Industry Impact

> **What interviewers expect:** You shape caching strategy at org or industry scale, make research-informed decisions, evaluate build vs buy, and define standards adopted company-wide.

### 1. Caching as a Platform
- **Multi-tenancy:** namespace isolation, quota enforcement, noisy neighbor prevention
- **API design:** eviction policies as config, TTL defaults, invalidation contracts
- **Versioned cache schemas:** deploy schema changes without breaking consumers
- **SLA definition:** latency p99, availability, durability, consistency guarantees

### 2. Advanced Eviction and Admission Policies

| Policy | Description | Use Case |
|---|---|---|
| LRU | Evict least recently used | General purpose |
| LFU | Evict least frequently used | Frequency-skewed workloads |
| SLRU | Segmented LRU (probationary + protected) | Scan-resistant |
| TinyLFU | Probabilistic frequency counting | Caffeine cache |
| W-TinyLFU | Windowed TinyLFU (Caffeine default) | Best general performance |
| Admission filter | Bloom filter to prevent one-hit wonders | High-churn workloads |

> 💡 Know the academic literature: ARC, LIRS, Cliffhanger, W-TinyLFU papers

### 3. Geo-Distributed Caching
- **Multi-region consistency:** cross-region replication lag and conflict resolution
- **Cache routing:** reads → nearest region; writes → primary with async replication
- **Cache federation:** independent regional caches vs globally consistent cache
- **Data residency constraints:** GDPR, CCPA requirements affecting cache placement
- **Edge caching at PoPs:** consistency vs performance tradeoff by geography

### 4. Cost Modeling and Efficiency
- **Total cost:** memory cost + network cost + engineering overhead + operational burden
- **ROI analysis:** cache cost vs compute cost to regenerate vs DB query cost
- **Cache efficiency ratio:** `(data served from cache × compute saved) / infrastructure cost`
- **Rightsizing:** working set modeling + actual hit curves to size fleets
- **Autoscaling:** predictive scaling vs reactive scaling based on miss rate

### 5. Consistency at Extreme Scale
- **Lease-based consistency:** clients hold leases; invalidations wait for lease expiry
- **Memcache at Facebook:** leases solve stale sets, thundering herd, stale read window
- **Bounded staleness:** define max tolerable staleness per entity type in SLA
- **Version-stamped caches:** every write includes monotonically increasing version
- **CRDTs:** Conflict-free Replicated Data Types for eventually consistent aggregates

### 6. Security and Compliance
- **Cache poisoning attack vectors:** input validation, signed cache tokens
- **Sensitive data in cache:** PII handling, encryption at rest/in-transit, field redaction
- **Audit logging:** who read what, when (compliance and forensics)
- **Data classification:** which classes may be cached, for how long, in which tiers
- **Access control:** ACL per namespace, service identity-based authorization

### 7. Research and Emerging Patterns
- **Predictive prefetching:** ML models predicting what to cache before requested
- **Semantic caching:** cache based on query semantics (especially relevant for LLM query caching)
- **Persistent memory (Intel Optane):** byte-addressable, DRAM-speed, disk-capacity
- **Disaggregated caching:** network-attached memory pools (CXL-based architectures)
- **Cache-oblivious algorithms:** optimal performance without explicit cache knowledge

### 8. Build vs Buy Decisions

| Option | Best For | Tradeoff |
|---|---|---|
| Redis OSS | General purpose, rich data structures | Ops burden |
| Redis Enterprise | Managed, active-active geo | Cost |
| ElastiCache / Memorystore | AWS/GCP native, managed | Vendor lock-in |
| Varnish / Nginx | HTTP caching, API gateway | HTTP only |
| In-house (Mcrouter, Pelikan) | Extreme scale, custom needs | Build cost |

### 9. Interview Questions at This Level
- How would you design the caching strategy for a company moving from monolith to microservices?
- A major streaming platform has 500M users. Design their caching infrastructure end-to-end.
- The cache hit rate for one geography is 30% lower than others. How do you investigate?
- How do you define and enforce caching standards across 200 engineering teams?
- Describe a caching innovation you'd contribute to the industry — what problem does it solve?

---

## ⚡ Quick Reference: Key Tradeoffs

| Topic | SDE | Senior | Staff | Principal |
|---|---|---|---|---|
| Cache-Aside vs Read-Through | Know both | Choose per use case | Define team standard | Platform default policy |
| Write-Through vs Write-Behind | Know tradeoffs | Apply per durability need | Risk modeling | Compliance + perf balance |
| LRU vs LFU | Implement LRU | Choose by workload | Custom policies | W-TinyLFU / research |
| Redis vs Memcached | Basic difference | Choose & configure | Fleet standardization | Build vs buy + TCO |
| Strong vs Eventual Consistency | Define both | Apply CAP reasoning | Per-entity SLA | Platform consistency model |
| TTL vs Event-Driven Invalidation | Use TTL | Event-driven for freshness | Hybrid strategy | Contract-based invalidation |
| Single vs Multi-Tier | Single tier | Two tiers | Design multi-tier | Fleet-wide tier strategy |

---

## 🎯 Top 5 System Design Scenarios

| Scenario | SDE | Senior | Staff | Principal |
|---|---|---|---|---|
| Twitter Feed | Simple DB + cache | Fan-out strategies | Celeb handling + tiers | Global consistency + cost |
| Rate Limiter | Redis INCR + TTL | Sliding window | Distributed + quota service | Multi-region + fairness |
| Autocomplete | Sorted set | Trie + Redis | Personalization + scale | ML-driven predictions |
| Product Catalog | TTL cache | Invalidation on update | CDN + app + DB tiers | Real-time inventory accuracy |
| Session Store | Redis KV + TTL | Sticky vs distributed | Multi-region sessions | Privacy + residency + scale |

---

## 📚 Resources to Study

- **Papers:** [Memcache at Facebook](https://research.facebook.com/publications/scaling-memcache-at-facebook/) · [W-TinyLFU](https://arxiv.org/abs/1512.00727) · [Redis internals](https://redis.io/docs/latest/)
- **Books:** Designing Data-Intensive Applications (Kleppmann) — Ch. 5 Replication, Ch. 6 Partitioning
- **Practice:** [system-design-primer](https://github.com/donnemartin/system-design-primer) · [Grokking System Design](https://www.educative.io/courses/grokking-modern-system-design-interview)

---

*Good luck with your interview preparation! 🚀*
