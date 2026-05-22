Yes. There are absolutely **HLD/system-design patterns**, similar in spirit to GoF design patterns, but at a **distributed-systems / architecture level**.

In LLD, patterns solve object/class design problems:

```text
Factory, Strategy, Observer, Decorator, Singleton, Adapter
```

In HLD, patterns solve system-level problems:

```text
How do I scale reads?
How do I handle async work?
How do I avoid duplicate payment?
How do I prevent double booking?
How do I keep search index updated?
How do I handle large media files?
How do I make microservices communicate reliably?
How do I survive downstream failure?
```

If you learn these HLD patterns, you can answer most system design questions by composing them.

---

# 1. Mental model

Most HLD interviews are not about inventing a system from scratch. They are about recognizing which **architecture patterns** apply.

Example:

## Design YouTube

Use:

- Object Storage Pattern
- CDN Pattern
- Async Processing Pattern
- Metadata DB Pattern
- Search Index Pattern
- Feed/Recommendation Pattern
- Cache-Aside Pattern
- Event-Driven Pattern

## Design Uber

Use:

- Geospatial Index Pattern
- Real-Time Connection Pattern
- State Machine Pattern
- Distributed Lock / Atomic Assignment Pattern
- Event-Driven Workflow Pattern
- Idempotency Pattern
- Payment Saga Pattern

## Design Ticketmaster

Use:

- Inventory Reservation Pattern
- Pessimistic/Optimistic Locking
- TTL Hold Pattern
- Queue-Based Load Leveling
- Payment Saga
- Idempotency
- Strong Consistency Boundary

So yes — these are reusable HLD patterns.

---

# 2. The most important HLD patterns

Below is the practical list.

---

# A. Scaling and traffic patterns

## 1. Load Balancer Pattern

### Problem

One server cannot handle all traffic.

### Pattern

Put a load balancer in front of multiple stateless service instances.

```text
Client -> Load Balancer -> App Instance 1
                      -> App Instance 2
                      -> App Instance 3
```

### Use when

- You need horizontal scaling.
- You need high availability.
- You need traffic distribution.

### Examples

- API services
- web apps
- Spring Boot services

### Interview phrase

> “I’d keep the application layer stateless and put it behind a load balancer so we can scale horizontally.”

---

## 2. Reverse Proxy Pattern

### Problem

You need routing, SSL termination, compression, auth, or request filtering.

### Pattern

Place reverse proxy/API gateway before backend services.

```text
Client -> NGINX/API Gateway/ALB -> Services
```

### Use when

- You have multiple services.
- You need path-based routing.
- You need TLS termination.
- You need auth/rate limiting.

---

## 3. API Gateway Pattern

### Problem

Clients should not directly call dozens of backend services.

### Pattern

API Gateway handles cross-cutting concerns.

```text
Client -> API Gateway -> User Service
                    -> Order Service
                    -> Payment Service
```

### Handles

- authentication
- authorization
- throttling
- routing
- request validation
- API versioning
- logging

### Use when

- Public APIs
- microservices
- mobile/web clients

---

## 4. BFF — Backend for Frontend Pattern

### Problem

Different clients need different API shapes.

### Pattern

Create frontend-specific backend layer.

```text
React Web -> Web BFF -> Services
Mobile App -> Mobile BFF -> Services
```

### Use when

- Frontend would otherwise call many APIs.
- You need UI-specific aggregation.
- Mobile and web have different needs.

### Example

Gartner.com homepage:

```text
BFF aggregates:
- user profile
- recommended research
- saved content
- entitlement status
- recent activity
```

### Interview phrase

> “A BFF reduces frontend chattiness and hides backend service complexity.”

---

# B. Caching patterns

## 5. Cache-Aside Pattern

### Problem

Database reads are expensive or frequent.

### Pattern

Application checks cache first, then DB.

```text
App -> Redis
   miss
App -> DB
App -> Redis set
```

### Use when

- Read-heavy system
- Data can be slightly stale
- You want simple cache control

### Examples

- product details
- user profile
- content metadata
- permissions

### Interview phrase

> “I’d start with cache-aside using Redis and TTL.”

---

## 6. Read-Through Cache Pattern

### Problem

You want cache itself to load data.

### Pattern

Application asks cache; cache loads DB on miss.

```text
App -> Cache -> DB
```

### Use when

- Cache provider supports it.
- You want simpler app code.

---

## 7. Write-Through Cache Pattern

### Problem

You want cache and DB updated together.

### Pattern

Write goes to cache and DB synchronously.

```text
App -> Cache -> DB
```

### Use when

- Fresh cache is important.
- Write latency can be slightly higher.

---

## 8. Write-Behind / Write-Back Pattern

### Problem

DB writes are too slow/heavy.

### Pattern

Write to cache/queue first, persist later.

```text
App -> Cache
Cache -> DB async
```

### Use when

- High write throughput
- Eventual persistence acceptable

### Avoid for

- payments
- bookings
- banking

---

## 9. Stale-While-Revalidate Pattern

### Problem

Cache expiry causes latency spikes or stampede.

### Pattern

Return stale cached data immediately and refresh in background.

```text
Request -> stale cache returned
        -> async refresh triggered
```

### Use when

- Slightly stale data is acceptable.
- You want low latency.

### Examples

- homepage modules
- recommendations
- content metadata

---

## 10. Cache Stampede Protection Pattern

### Problem

Many requests rebuild same expired cache key.

### Pattern

Use:

- per-key lock
- request coalescing
- TTL jitter
- background refresh

### Example

```text
Only one request rebuilds product:123.
Others wait or receive stale value.
```

---

# C. Data storage patterns

## 11. Database per Service Pattern

### Problem

Microservices sharing DB become tightly coupled.

### Pattern

Each service owns its own database.

```text
Order Service -> Order DB
Payment Service -> Payment DB
User Service -> User DB
```

### Use when

- Microservices need independent ownership.
- Strong service boundaries matter.

### Tradeoff

Cross-service joins become impossible. Need APIs/events.

---

## 12. Shared Database Pattern

### Problem

You need simplicity.

### Pattern

Multiple modules/services use same database.

### Use when

- Early stage
- modular monolith
- small team
- strong consistency needed

### Tradeoff

Tight coupling.

---

## 13. CQRS — Command Query Responsibility Segregation

### Problem

Read and write requirements are different.

### Pattern

Separate write model from read model.

```text
Command side -> normalized DB
Query side -> denormalized read model/cache/search index
```

### Use when

- Read-heavy system
- Complex queries
- Need denormalized views
- Write model differs from read model

### Examples

- dashboards
- feeds
- reports
- search systems

### Interview phrase

> “I’d keep the transactional write model normalized and build denormalized read models asynchronously.”

---

## 14. Materialized View Pattern

### Problem

Complex queries are slow.

### Pattern

Precompute and store query result.

```text
Raw data -> Aggregation job -> Materialized view
```

### Use when

- dashboards
- analytics
- expensive joins
- reports

---

## 15. Search Index Pattern

### Problem

Relational DB is not ideal for full-text search.

### Pattern

Use Elasticsearch/OpenSearch as derived search index.

```text
DB update -> Event/CDC -> Indexer -> OpenSearch
```

### Use when

- full-text search
- fuzzy search
- ranking
- autocomplete
- facets

### Interview phrase

> “DB remains the source of truth; search index is eventually consistent.”

---

## 16. Object Storage Pattern

### Problem

Large files should not live in DB or app servers.

### Pattern

Store metadata in DB, blob in object storage.

```text
Metadata -> PostgreSQL
File -> S3
Delivery -> CDN
```

### Use when

- images
- videos
- PDFs
- backups
- reports

---

# D. Asynchronous processing patterns

## 17. Queue-Based Load Leveling Pattern

### Problem

Traffic spikes overwhelm backend workers.

### Pattern

Put queue between request and processing.

```text
API -> Queue -> Workers
```

### Use when

- tasks can be async
- workload is bursty
- processing is slow

### Examples

- email sending
- video transcoding
- report generation
- code migration jobs

---

## 18. Event-Driven Architecture Pattern

### Problem

Services should be decoupled.

### Pattern

Services publish events, consumers react.

```text
OrderCreated -> Inventory Service
             -> Notification Service
             -> Analytics Service
```

### Use when

- multiple systems react to same business event
- async workflows
- extensibility

---

## 19. Pub/Sub Fanout Pattern

### Problem

One event must go to many consumers.

### Pattern

Publish event to topic. Multiple subscribers consume.

```text
ContentPublished -> Search Indexer
                 -> Notification Service
                 -> Recommendation Pipeline
```

### Use when

- independent consumers
- event fanout
- analytics

---

## 20. Competing Consumers Pattern

### Problem

Need to scale task processing.

### Pattern

Multiple workers consume from same queue.

```text
Queue -> Worker 1
      -> Worker 2
      -> Worker 3
```

### Use when

- background processing
- image resize
- email delivery
- migration jobs

---

## 21. Dead Letter Queue Pattern

### Problem

Some messages repeatedly fail.

### Pattern

After retry limit, move to DLQ.

```text
Queue -> Consumer fails -> Retry -> DLQ
```

### Use when

- poison messages
- debugging
- replay after fix

---

## 22. Retry with Exponential Backoff Pattern

### Problem

Transient failures happen.

### Pattern

Retry after increasing delay plus jitter.

```text
retry after 1s, 2s, 4s, 8s + jitter
```

### Use when

- temporary downstream failure
- network issue
- rate limit

### Must combine with

- idempotency
- max retry count
- DLQ

---

# E. Consistency and transaction patterns

## 23. Idempotency Key Pattern

### Problem

Retries may duplicate side effects.

### Pattern

Client sends unique idempotency key. Server stores request/result.

```text
POST /payments
Idempotency-Key: abc-123
```

### Use when

- payment
- order creation
- booking
- ride creation
- external API calls

### Interview phrase

> “For side-effect APIs, retries are inevitable, so I’d require idempotency keys.”

---

## 24. Optimistic Locking Pattern

### Problem

Concurrent updates may overwrite each other.

### Pattern

Use version column.

```sql
UPDATE inventory
SET quantity = quantity - 1,
    version = version + 1
WHERE item_id = 123
  AND version = 5;
```

### Use when

- conflicts are rare
- high concurrency desired

---

## 25. Pessimistic Locking Pattern

### Problem

Conflicts are likely and correctness is critical.

### Pattern

Lock row before updating.

```sql
SELECT * FROM seats
WHERE seat_id = 123
FOR UPDATE;
```

### Use when

- seat booking
- wallet update
- inventory reservation
- high contention

---

## 26. Reservation / TTL Hold Pattern

### Problem

Need to temporarily reserve scarce resource.

### Pattern

Resource moves:

```text
AVAILABLE -> HELD -> BOOKED
          -> EXPIRED
```

### Use when

- ticket booking
- hotel booking
- parking reservation
- inventory checkout

### Example

Seat held for 5 minutes while payment completes.

---

## 27. Saga Pattern

### Problem

Distributed transaction across services is hard.

### Pattern

Break workflow into local transactions plus compensating actions.

```text
Create Order
Reserve Inventory
Charge Payment
Ship Order
```

If payment fails:

```text
Release Inventory
Cancel Order
```

### Use when

- microservices
- long-running workflows
- distributed business transactions

### Types

- orchestration
- choreography

---

## 28. Transactional Outbox Pattern

### Problem

DB update succeeds but event publish fails.

### Pattern

Write business data and outbox event in same DB transaction.

```text
Transaction:
  update order
  insert outbox_event

Worker:
  publish event
  mark published
```

### Use when

- reliable event publishing
- avoiding dual-write problem

---

## 29. Inbox Pattern

### Problem

Same message may be consumed multiple times.

### Pattern

Consumer stores processed message IDs.

```text
if event_id already processed:
    skip
else:
    process and store event_id
```

### Use when

- at-least-once message delivery
- idempotent consumers

---

# F. Reliability patterns

## 30. Circuit Breaker Pattern

### Problem

Failing dependency causes cascading failure.

### Pattern

Stop calling unhealthy service temporarily.

States:

```text
CLOSED -> OPEN -> HALF_OPEN
```

### Use when

- external APIs
- payment gateway
- partner systems

---

## 31. Bulkhead Pattern

### Problem

One slow dependency exhausts all resources.

### Pattern

Isolate resources.

Examples:

- separate thread pools
- separate queues
- separate DB pools

### Use when

- service has multiple downstream dependencies
- one dependency should not bring down all

---

## 32. Graceful Degradation Pattern

### Problem

Optional dependency fails.

### Pattern

Return reduced functionality.

Example:

```text
Recommendation service down -> show popular content
```

### Use when

- recommendations
- personalization
- analytics
- non-critical widgets

---

## 33. Fallback Pattern

### Problem

Primary path fails.

### Pattern

Use fallback data/path.

Examples:

- cached last-known-good response
- default recommendation
- backup provider

---

## 34. Timeout Pattern

### Problem

Service waits forever.

### Pattern

Set strict timeout for downstream calls.

```text
Search service timeout: 300ms
Recommendation timeout: 200ms
```

### Use always.

---

# G. Real-time communication patterns

## 35. WebSocket Gateway Pattern

### Problem

Need real-time bidirectional communication.

### Pattern

Separate connection servers from business services.

```text
Client -> WebSocket Gateway -> Message/Event Service
```

### Use when

- chat
- live tracking
- collaborative editing
- trading dashboard

---

## 36. Presence Pattern

### Problem

Need to know who is online.

### Pattern

Store heartbeat/status in Redis with TTL.

```text
presence:user:123 = online, TTL 30s
```

### Use when

- chat
- collaboration
- driver availability
- gaming

---

## 37. Polling / Long Polling / SSE Pattern

### Use polling when

- simplicity matters
- updates are infrequent

### Use long polling when

- need near-real-time without WebSocket

### Use SSE when

- server-to-client one-way stream

### Use WebSocket when

- bidirectional real-time

---

# H. Feed and social patterns

## 38. Fanout-on-Write Pattern

### Problem

Need fast feed reads.

### Pattern

When user posts, push post to followers’ feeds.

```text
PostCreated -> write postId to follower feed caches
```

### Use when

- normal users
- read-heavy feeds

---

## 39. Fanout-on-Read Pattern

### Problem

Some users have too many followers.

### Pattern

Generate feed at read time.

```text
Read feed -> fetch posts from followed users
```

### Use when

- celebrity accounts
- avoiding write explosion

---

## 40. Hybrid Feed Pattern

### Pattern

- fanout-on-write for normal users
- fanout-on-read for celebrities
- merge/rank at read time

### Interview phrase

> “For feed systems, I’d use a hybrid model to avoid celebrity write amplification.”

---

# I. Geospatial patterns

## 41. Geohash / Grid Partitioning Pattern

### Problem

Need nearby search.

### Pattern

Divide world into grid cells/geohashes.

```text
location -> geohash cell -> nearby cell lookup
```

### Use when

- nearby drivers
- restaurants near me
- nearby friends

---

## 42. Redis GEO Pattern

### Problem

Need fast live proximity lookup.

### Pattern

Use Redis GEO for current active locations.

```text
GEOADD drivers:delhi lng lat driverId
GEOSEARCH drivers:delhi BYRADIUS 3 km
```

### Use when

- live driver locations
- ephemeral location data

---

# J. Media/file processing patterns

## 43. Direct Upload to Object Storage Pattern

### Problem

App servers should not handle large file uploads.

### Pattern

Backend issues presigned URL. Client uploads to S3 directly.

```text
Client -> Backend: get signed URL
Client -> S3: upload file
Backend -> DB: save metadata
```

### Use when

- images
- videos
- documents
- PDFs

---

## 44. Async Media Processing Pattern

### Problem

Video/image processing is slow.

### Pattern

Upload event triggers worker.

```text
MediaUploaded -> Queue -> Transcoder/Thumbnail Worker
```

### Use when

- video transcoding
- image resizing
- PDF processing

---

## 45. CDN Delivery Pattern

### Problem

Users need low-latency global content access.

### Pattern

Serve static/media content via CDN.

```text
User -> CDN edge -> S3/origin
```

### Use when

- videos
- images
- PDFs
- static assets

---

# K. Security patterns

## 46. Token-Based Authentication Pattern

### Pattern

Use OAuth2/OIDC/JWT.

```text
Client -> login -> token
Client -> API with token
```

### Use when

- stateless APIs
- distributed services

---

## 47. RBAC Pattern

### Problem

Permissions based on role.

```text
ADMIN
EDITOR
USER
```

### Use when

- enterprise apps
- admin panels

---

## 48. ABAC Pattern

### Problem

Permissions depend on attributes.

```text
user.organizationId == document.organizationId
subscription.active == true
```

### Use when

- entitlement systems
- document access
- multi-tenant systems

---

## 49. Signed URL Pattern

### Problem

Private file access must be secure.

### Pattern

Backend verifies permission and generates short-lived URL.

```text
User -> Backend authorization
Backend -> signed S3/CloudFront URL
User -> download
```

---

# L. Deployment and operations patterns

## 50. Blue-Green Deployment Pattern

### Pattern

Maintain two environments.

```text
Blue = current
Green = new
Switch traffic after validation
```

### Use when

- fast rollback needed

---

## 51. Canary Deployment Pattern

### Pattern

Gradually shift traffic.

```text
1% -> 5% -> 25% -> 100%
```

### Use when

- risky changes
- user-facing systems

---

## 52. Feature Flag Pattern

### Pattern

Deploy code separately from feature release.

```text
if feature_enabled(user):
    use new flow
else:
    use old flow
```

### Use when

- gradual rollout
- A/B testing
- kill switch

---

## 53. Observability Pattern

### Pattern

Instrument:

- logs
- metrics
- traces
- correlation IDs

### Use always.

Golden signals:

```text
latency
traffic
errors
saturation
```

---

# M. Partitioning and scaling data patterns

## 54. Horizontal Sharding Pattern

### Problem

One DB cannot store/serve all data.

### Pattern

Split data across shards.

```text
user_id % N -> shard
```

### Use when

- large data volume
- high write throughput

---

## 55. Read Replica Pattern

### Problem

Primary DB overloaded with reads.

### Pattern

Send writes to primary, reads to replicas.

```text
Write -> Primary
Read -> Replica
```

### Use when

- read-heavy relational systems

---

## 56. Hot Key Mitigation Pattern

### Problem

One key/partition gets too much traffic.

### Pattern

Use:

- caching
- salting
- replication
- special-case handling

Example:

```text
celebrity_feed:user123:shard1
celebrity_feed:user123:shard2
```

---

# 3. Mapping HLD patterns to common interview questions

## Design URL Shortener

Use:

- API Gateway
- Unique ID Generation
- Cache-Aside
- Read Replica
- Analytics Event Pipeline
- Rate Limiting

---

## Design Instagram

Use:

- Object Storage
- CDN
- Direct Upload
- Feed Fanout
- Redis Cache
- Search Index
- Notification Pub/Sub
- Async Media Processing

---

## Design Uber

Use:

- Redis GEO
- Geohash
- WebSocket Gateway
- State Machine
- Distributed Locking
- Idempotency
- Saga for Payment
- Event-Driven Architecture

---

## Design Ticketmaster

Use:

- Reservation/TTL Hold
- Pessimistic Locking
- Queue-Based Load Leveling
- Idempotency
- Payment Saga
- Outbox
- Read Replicas

---

## Design WhatsApp

Use:

- WebSocket Gateway
- Presence
- Message Queue
- Offline Storage
- Pub/Sub
- Push Notifications
- At-least-once Delivery
- Idempotent Consumers

---

## Design YouTube

Use:

- Direct Upload
- Object Storage
- Async Transcoding
- CDN
- Metadata DB
- Search Index
- Recommendation Pipeline
- Cache-Aside

---

## Design Payment System

Use:

- Idempotency
- Ledger
- Saga
- Outbox
- Reconciliation
- Retry with Backoff
- Audit Logs
- Strong Consistency Boundary

---

## Design Metrics Monitoring

Use:

- Agent-based Collection
- Queue-Based Ingestion
- Time-Series DB
- Downsampling
- Retention Policies
- Alerting Pipeline

---

# 4. The ultimate HLD pattern cheat sheet

Remember this mapping:

```text
Need to scale reads?
-> Cache, CDN, read replicas, denormalized read model, materialized views.

Need to scale writes?
-> Queue, sharding, batching, append-only storage, partitioning.

Need full-text search?
-> OpenSearch/Elasticsearch as derived index.

Need large file handling?
-> Object storage + CDN + presigned URLs.

Need async work?
-> Queue + workers + DLQ + retries.

Need multiple consumers for same event?
-> Pub/Sub.

Need reliable event after DB update?
-> Transactional outbox.

Need avoid duplicate side effects?
-> Idempotency key.

Need prevent double booking?
-> Locking + reservation state machine + unique constraints.

Need distributed transaction?
-> Prefer Saga over 2PC.

Need real-time updates?
-> WebSocket/SSE + connection manager.

Need nearby search?
-> Geohash/Redis GEO/PostGIS.

Need high availability?
-> Load balancer + stateless services + multi-AZ + replication.

Need safe deployment?
-> Feature flags + canary/blue-green.

Need production readiness?
-> Logs + metrics + traces + alerts + dashboards.
```

---

# 5. How to use these patterns in an interview

When given any HLD question, think:

## Step 1: What type of system is this?

```text
Content/media?
Feed/social?
Booking/payment?
Real-time/chat?
Search?
Location?
Analytics/metrics?
Workflow/job processing?
```

## Step 2: Pick the core patterns.

Example:

> “Design Gartner.com research portal.”

Patterns:

```text
BFF
Content metadata DB
Object storage for PDFs
CDN
Search index
Entitlement service
Cache-aside
Event-driven publishing
Notification pub/sub
Observability
```

## Step 3: Say the tradeoff.

Example:

> “I’d use OpenSearch for full-text search, but PostgreSQL remains the source of truth. The tradeoff is search becomes eventually consistent, so we need indexing lag monitoring and retry/DLQ for failed index events.”

That is the kind of answer interviewers love.

---

# Final answer

Yes — HLD has patterns just like GoF, but they are called **architecture patterns**, **distributed system patterns**, **cloud patterns**, or **system design patterns**.

The most useful ones to master are:

```text
Cache-aside
CDN delivery
API Gateway
BFF
CQRS
Search index
Object storage
Queue-based load leveling
Pub/Sub
Saga
Outbox
Idempotency
Distributed locking
Reservation/TTL hold
Circuit breaker
Bulkhead
Retry/backoff
WebSocket gateway
Fanout-on-write/read
Geohash/Redis GEO
Read replicas
Sharding
Feature flags
Canary deployment
Observability
```

If you can recognize and combine these patterns, you can handle almost any HLD interview question.
