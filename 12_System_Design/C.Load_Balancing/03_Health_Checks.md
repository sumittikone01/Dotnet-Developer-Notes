# 📌 Scalability Patterns

## 📌 What is it?

Common, reusable **architectural strategies** used to help a system handle growing load beyond simple vertical/horizontal scaling.

## 🤔 Why do we need it?

Scaling isn't just "add more servers" — different bottlenecks (compute, data, traffic bursts) need different patterns. Knowing the right pattern for the right problem is what separates a working prototype from a production-grade system.

## 📊 Key patterns overview

| Pattern                                    | What it does                                           | Solves                                     |
| ------------------------------------------ | ------------------------------------------------------ | ------------------------------------------ |
| **Load Balancing**                   | Distributes traffic across servers                     | Overloaded single server                   |
| **Caching**                          | Stores frequently accessed data closer to the client   | Repeated expensive reads                   |
| **Database Replication**             | Copies data to read replicas                           | Read-heavy bottlenecks                     |
| **Sharding/Partitioning**            | Splits data across multiple DBs                        | Write-heavy / large dataset                |
| **Asynchronous Processing (Queues)** | Defers slow work to background workers                 | Slow, non-urgent tasks blocking requests   |
| **CDN**                              | Serves static content from edge locations              | High latency for global users              |
| **Microservices**                    | Splits a monolith into independently scalable services | One component needs more scale than others |
| **Auto-scaling**                     | Adds/removes servers based on real-time load           | Unpredictable traffic spikes               |

## 🖼 Where each pattern fits

```
Client → CDN (static assets)
       → Load Balancer
              → App Servers (auto-scaled)
                     → Cache (Redis) → DB Read Replicas
                     → Message Queue → Background Workers
                     → DB Primary (sharded)
```

## 🌍 Real-world analogy

Scaling a restaurant chain:

- **Load balancing** = a host seating customers evenly across sections.
- **Caching** = pre-cooked popular items ready to serve instantly.
- **Sharding** = separate kitchens per cuisine type, each handling its own orders.
- **Queues** = takeout orders processed in the background without blocking dine-in service.
- **CDN** = opening branches near customers instead of one central kitchen shipping everywhere.

## 🚨 Common mistakes

- Applying every pattern at once ("resume-driven design") when the system doesn't need that scale yet.
- Adding caching without a cache-invalidation strategy (stale data bugs).
- Sharding too early, before understanding real query/access patterns — hard to undo.

## 💡 Best practices

- Scale **one bottleneck at a time** — identify what's actually failing under load (compute? DB reads? DB writes?) before applying a pattern.
- Start simple: load balancer + cache solves most early-stage scaling problems.
- Reach for sharding, microservices, and message queues only when a single-node solution genuinely can't keep up.

## 🎤 Interview questions

- What scalability patterns would you apply to a system with a read-heavy workload vs write-heavy workload?
- Why might over-applying scalability patterns early be a bad idea?
- How does asynchronous processing help scalability?

## 📝 30-second revision cheat sheet

- Common patterns: **Load Balancing, Caching, Replication, Sharding, Queues, CDN, Microservices, Auto-scaling.**
- Match the pattern to the actual bottleneck — don't over-engineer.
- Most systems start with just load balancer + cache and grow from ther
