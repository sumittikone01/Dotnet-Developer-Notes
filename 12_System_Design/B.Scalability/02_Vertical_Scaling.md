# 📌 Vertical Scaling

## 📌 What is it?

**Vertical scaling (scale-up)** means increasing the power (CPU, RAM, disk, SSD speed) of a **single existing machine** to handle more load.

## 🤔 Why do we need it?

It's the **simplest** way to handle more traffic — no architecture changes, no distributed systems complexity. Great for early-stage systems or as a quick stop-gap.

## 🌍 Real-world analogy

Instead of hiring more cashiers (horizontal), you give your **one cashier a faster till and more hands** — upgrading the same worker rather than adding more. Related concept: [[Horizontal_Scaling]] does the opposite by adding more workers instead.

## 📊 Vertical vs Horizontal Scaling

| Aspect                  | Vertical Scaling                                   | Horizontal Scaling                              |
| ----------------------- | -------------------------------------------------- | ----------------------------------------------- |
| Method                  | Upgrade one machine                                | Add more machines                               |
| Complexity              | Low (no code changes)                              | High (needs load balancer, stateless design)    |
| Ceiling                 | Hardware limit (finite)                            | Practically unlimited                           |
| Downtime                | Often requires reboot/downtime                     | Can add servers with zero downtime              |
| Cost curve              | Expensive at high-end hardware                     | Cheaper commodity servers, cost scales linearly |
| Single point of failure | Yes — still one machine                           | No — redundancy built in                       |
| Good for                | Early-stage apps, relational DBs (harder to shard) | Large-scale, high-traffic systems               |

## 🖼 Diagram

```
Before:  [ 4 CPU, 8GB RAM ] 
After :  [ 32 CPU, 256GB RAM ]   ← same single machine, just bigger
```

## 🚨 Common mistakes

- Relying on vertical scaling indefinitely — you eventually hit a hardware/cost ceiling.
- Not planning a migration path to horizontal scaling before you desperately need it.
- Ignoring that vertical scaling is still a **single point of failure** — one crash = full outage.

## 💡 Best practices

- Use vertical scaling for **quick wins** and for components that are hard to distribute (e.g., a single-writer primary DB).
- Combine both: vertically scale the database primary while horizontally scaling stateless app servers.
- Monitor headroom (CPU/RAM usage trends) to know when you're approaching the ceiling.

## 🎤 Interview questions

- When would you choose vertical scaling over horizontal scaling?
- What are the limits of vertical scaling?
- Can a system use both vertical and horizontal scaling together? Give an example.

## 📝 30-second revision cheat sheet

- Vertical scaling = **bigger machine**. Simple, but has a hardware ceiling and remains a SPOF.
- Horizontal scaling = **more machines**. Complex, but near-unlimited growth.
- Real systems usually use **both**: scale-up the DB primary, scale-out the app tie
