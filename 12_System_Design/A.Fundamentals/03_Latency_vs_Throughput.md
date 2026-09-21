# 📌 Latency vs Throughput

## 📌 What is it?

- **Latency**: time taken to complete a *single* request (measured in ms).
- **Throughput**: number of requests a system can handle *per unit time* (measured in requests/sec, or QPS).

## 🤔 Why do we need it?

These two metrics often **trade off against each other**. Optimizing blindly for one can hurt the other — understanding both lets you make the right call for your use case (e.g., a payment API cares about latency; a batch analytics pipeline cares about throughput).

## 🧠 Intuition

Latency = how fast **one** car crosses a bridge.
Throughput = how many cars cross the bridge **per minute**.

## 🌍 Real-world analogy

A highway toll booth:

- **Latency** = time for one car to pass through a booth.
- **Throughput** = total cars passing through all booths per hour.
- Adding more booths (parallelism) increases throughput but doesn't necessarily reduce the time for one car — unless there's less queuing.

## 📊 Comparison table

| Aspect             | Latency                                 | Throughput                                  |
| ------------------ | --------------------------------------- | ------------------------------------------- |
| Measures           | Time per request                        | Requests per second                         |
| Unit               | ms, seconds                             | req/sec, QPS, TPS                           |
| Improved by        | Faster CPU, caching, fewer network hops | Parallelism, load balancing, batching       |
| User-facing impact | "Is the page fast?"                     | "Can the site handle Black Friday traffic?" |
| Example priority   | Video calls, gaming, trading            | Batch jobs, log processing, bulk uploads    |

## 🖼 Relationship diagram

```
Throughput ≈ Concurrency / Latency

If Latency ↓ and Concurrency stays same → Throughput ↑
If Concurrency ↑ (more parallel workers) → Throughput ↑ (even if Latency stays same)
```

## ⚡ Performance considerations

- **Little's Law**: `L = λ × W` (avg items in system = arrival rate × avg time in system). Used to reason about queue buildup.
- High throughput with poor latency = system is "fast on average" but individual users suffer (bad UX).
- Low latency with poor throughput = system feels snappy but collapses under real load.

## 🚨 Common mistakes

- Assuming reducing latency automatically improves throughput (not always true — depends on concurrency model).
- Benchmarking only average latency, ignoring **p95/p99 latency** (tail latency), which is what real users feel during spikes.
- Adding more servers (scaling throughput) without fixing a slow downstream dependency (won't help latency).

## 💡 Best practices

- Track **p50, p95, p99 latency** — not just averages.
- Use caching/CDNs to cut latency; use load balancing/horizontal scaling to boost throughput.
- Define which one matters more for your specific system before optimizing.

## 🎤 Interview questions

- What's the difference between latency and throughput, with an example?
- Can you increase throughput without changing latency? How?
- Why do engineers care about p99 latency instead of just the average?

## 📝 30-second revision cheat sheet

- **Latency** = speed of one request. **Throughput** = volume of requests handled.
- More parallelism → ↑ throughput, latency may stay same or worsen under contention.
- Always monitor p95/p99 latency, not just averages.
