
# 📌 Availability and Reliability

## 📌 What is it?

- **Availability**: the percentage of time a system is **operational and accessible** (e.g., 99.9% uptime).
- **Reliability**: the probability a system performs **correctly** without failure over a given period — i.e., it does the right thing, not just "is reachable."

## 🤔 Why do we need it?

Users expect systems to be up 24/7 and to behave correctly every time. Downtime or incorrect behavior (e.g., a payment charged twice) directly costs money, trust, and reputation.

## 🧠 Intuition

- Availability answers: **"Is the system up right now?"**
- Reliability answers: **"When it responds, can I trust the result?"**
  A system can be available (responds instantly) but unreliable (returns wrong/corrupted data).

## 📊 The "Nines" table

| Availability %         | Downtime / year | Downtime / month |
| ---------------------- | --------------- | ---------------- |
| 99% ("two nines")      | ~3.65 days      | ~7.3 hours       |
| 99.9% ("three nines")  | ~8.76 hours     | ~43.8 minutes    |
| 99.99% ("four nines")  | ~52.6 minutes   | ~4.4 minutes     |
| 99.999% ("five nines") | ~5.26 minutes   | ~26 seconds      |

## 🌍 Real-world analogy

An ATM:

- **Available** = the ATM is switched on and responds when you insert your card.
- **Reliable** = when it says "₹5,000 dispensed," it actually gives you ₹5,000 — every single time, not just most of the time.

## ⚙️ Internal working — how systems achieve both

| Technique                             | Improves                                    |
| ------------------------------------- | ------------------------------------------- |
| Redundancy (multiple servers/regions) | Availability                                |
| Failover / health checks              | Availability                                |
| Load balancing                        | Availability + reliability                  |
| Retries with idempotency              | Reliability                                 |
| Data replication                      | Both                                        |
| Monitoring + alerting                 | Both (faster detection)                     |
| Graceful degradation                  | Availability (partial service > no service) |

## 🖼 Failover flow

```
Client → Load Balancer → [Server A (healthy)] → Response
                       ↘ [Server B (down)]  ✗ skipped via health check
```

## 🚨 Common mistakes

- Confusing "high availability" with "no bugs" — a system can be up but returning wrong data.
- Single point of failure (SPOF): one database, one server, one region — a single crash takes everything down.
- No monitoring — you find out about downtime from angry users instead of alerts.

## 💡 Best practices

- Eliminate SPOFs: redundant servers, multi-AZ/multi-region databases.
- Design for **graceful degradation** (e.g., show cached data if the live service is down) instead of full outage.
- Make operations **idempotent** so retries after failures don't cause duplicate side effects.
- Set an SLA (Service Level Agreement) target and monitor against it.

## 🎤 Interview questions

- What's the difference between availability and reliability?
- How would you design a system to achieve 99.99% availability?
- What is a single point of failure, and how do you eliminate one?

## 📝 30-second revision cheat sheet

- **Availability** = system is reachable. **Reliability** = system behaves correctly.
- "Nines" measure allowed downtime — more nines = exponentially less downtime.
- Achieve both via redundancy, failover, replication, and eliminating SPOFs.
