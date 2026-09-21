# 📌 System Design Basics

## 📌 What is it?

System Design is the process of defining the **architecture, components, modules, interfaces, and data** of a system to satisfy specified requirements — both functional (what it does) and non-functional (how well it does it: speed, scale, reliability).

## 🤔 Why do we need it?

- Real-world systems (Instagram, Uber, Netflix) serve **millions of users** — a single-server, single-database app collapses under that load.
- Good design decisions made early save you from painful, expensive rewrites later.
- It's the core skill tested in **senior engineering interviews** — because it proves you can think beyond "does the code work?" to "does the system survive traffic, failure, and growth?"

## 🧠 Intuition

Writing code = building a room.
System Design = designing the **entire building** — plumbing (data flow), electricity (communication), elevators (load balancers), fire exits (fault tolerance), and how many people it can safely hold (scalability).

## 🌍 Real-world analogy

A restaurant kitchen:

- **Functional requirement**: cook and serve food.
- **Non-functional requirements**: serve within 10 minutes (latency), handle 200 orders/night (throughput), keep working if one stove breaks (reliability), and expand to more tables without redesigning the whole kitchen (scalability).

## ⚙️ Internal working — The Design Process

```
1. Clarify requirements  →  2. Estimate scale  →  3. Define API/data model
        ↓
4. High-level design (boxes & arrows)  →  5. Deep-dive on 2-3 components
        ↓
6. Identify bottlenecks  →  7. Discuss trade-offs
```

## 📊 Functional vs Non-Functional Requirements

| Aspect         | Functional                      | Non-Functional                                            |
| -------------- | ------------------------------- | --------------------------------------------------------- |
| Meaning        | What the system does            | How well the system does it                               |
| Examples       | "User can post a tweet"         | "Post visible in < 1s to followers"                       |
| Failure impact | Feature doesn't work            | System becomes slow/unreliable/insecure                   |
| Common ones    | CRUD operations, business logic | Scalability, availability, latency, consistency, security |

## 🚨 Common mistakes

- Jumping straight to a database schema before clarifying requirements.
- Ignoring scale — designing as if there will only ever be 100 users.
- Over-engineering a simple problem with microservices, Kafka, etc. when a monolith would do.

## 💡 Best practices

- Always start with **clarifying questions**: How many users? Read-heavy or write-heavy? Real-time or eventual consistency OK?
- Do **back-of-envelope estimation** (traffic, storage, bandwidth) before designing.
- Design in layers: start high-level, then zoom into critical components.

## 🎤 Interview questions

- What's the difference between functional and non-functional requirements?
- How do you approach an unfamiliar system design problem?
- Why does system design matter even if the code is "correct"?

## 📝 30-second revision cheat sheet

- System Design = architecture + trade-offs to meet functional + non-functional needs.
- Non-functional requirements (scale, latency, availability) usually drive the hardest decisions.
- Always clarify → estimate → design → deep-dive → discuss trade-offs.
