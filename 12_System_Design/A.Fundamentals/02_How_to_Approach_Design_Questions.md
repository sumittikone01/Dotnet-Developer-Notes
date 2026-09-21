# 📌 How to Approach System Design Questions

## 📌 What is it?

A repeatable **framework** for tackling any system design problem (interview or real project) so you never freeze staring at a blank whiteboard.

## 🤔 Why do we need it?

Without structure, people either dive into low-level details too early (e.g., picking a specific DB index type before knowing the scale) or stay too abstract forever. A framework keeps the conversation focused and demonstrates senior-level thinking.

## ⚙️ Internal working — The 5-Step Framework

```
Step 1: Requirements Clarification
   ├─ Functional: core features (MVP scope)
   └─ Non-functional: scale, latency, consistency, availability

Step 2: Back-of-Envelope Estimation
   ├─ Users (DAU/MAU), QPS (queries/sec)
   ├─ Storage (data size × growth over years)
   └─ Bandwidth (read/write ratio)

Step 3: High-Level Design
   └─ Draw boxes: Client → Load Balancer → App Servers → Cache → DB

Step 4: Deep Dive
   └─ Pick 1–2 critical components (e.g., "how does the feed get generated?")

Step 5: Identify Bottlenecks & Trade-offs
   └─ Single points of failure, hot partitions, consistency vs availability
```

## 📊 What to clarify first (checklist)

| Question                              | Why it matters                             |
| ------------------------------------- | ------------------------------------------ |
| How many users / requests per second? | Decides if you need caching, sharding, CDN |
| Read-heavy or write-heavy?            | Decides caching strategy, DB choice        |
| Strong or eventual consistency OK?    | Decides SQL vs NoSQL, replication strategy |
| Real-time requirement?                | Decides polling vs WebSockets vs pub-sub   |
| Global or single-region users?        | Decides CDN, multi-region replication      |

## 🌍 Real-world analogy

Think of it like a doctor's diagnosis: you don't prescribe medicine (pick a database) before asking about symptoms (requirements) and running basic tests (estimation). Jumping to a solution too early is like prescribing blindly.

## 🚨 Common mistakes

- Spending 20 minutes on requirements and leaving no time for the actual design.
- Forgetting to estimate scale — "millions of users" changes almost every decision.
- Never circling back to discuss trade-offs — interviewers want to see you *reason*, not just draw boxes.

## 💡 Best practices

- Timebox each step (~5 min requirements, ~5 min estimation, rest on design + deep dive).
- Always say your assumptions out loud ("I'll assume 10M DAU unless told otherwise").
- End with: "Given more time, I'd also consider X" — shows awareness of what's left out.

## 🎤 Interview questions

- Walk me through how you'd approach designing a URL shortener.
- Why is back-of-envelope estimation important before designing?
- How do you decide which component to deep-dive into?

## 📝 30-second revision cheat sheet

**Clarify → Estimate → High-level design → Deep dive → Trade-offs.**
Always state assumptions. Timebox each phase. Show reasoning, not just diagrams.
