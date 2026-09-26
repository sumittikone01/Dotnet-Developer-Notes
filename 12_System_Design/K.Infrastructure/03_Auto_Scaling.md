# 📈 Auto Scaling

## 📌 What is it?

**Auto Scaling** automatically adjusts the number of running server instances (or resources) **based on real-time demand** — adding capacity when traffic increases, and removing it when traffic drops.

## 🤔 Why do we need it?

- Manually provisioning for **peak traffic** wastes money most of the time (servers sit idle).
- Manually provisioning for **average traffic** causes outages during spikes.
- Traffic is rarely constant — auto scaling matches capacity to actual demand automatically, in real time, without human intervention.

## 🌍 Real-world analogy

A **restaurant** that calls in extra staff only during Friday dinner rush, and sends them home once the rush ends — instead of paying for a full night-shift crew every single day regardless of how busy it actually is.

## 📊 Two Types of Scaling (recap — see `01_Horizontal_Scaling.md` / `02_Vertical_Scaling.md`)

| Type                              | What Auto Scaling does                                                                                |
| --------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Horizontal Auto Scaling** | Adds/removes server instances ⭐ (most common)                                                        |
| **Vertical Auto Scaling**   | Increases/decreases resources (CPU/RAM) of existing instances (less common, usually requires restart) |

## ⚙️ Internal Working

```
                    ┌─────────────────────┐
                    │   Monitoring System  │
                    │ (CPU%, RPS, Queue)   │
                    └──────────┬──────────┘
                               │ metrics
                               ▼
                    ┌─────────────────────┐
                    │  Auto Scaling Policy │
                    │ "If CPU > 70% for 5m,│
                    │   add 2 instances"   │
                    └──────────┬──────────┘
                               │ triggers
                               ▼
                    ┌─────────────────────┐
                    │   Instance Pool      │
                    └─────────────────────┘

Low traffic:  [■■]              (2 instances)
Traffic spike: [■■■■■■]         (auto-scaled to 6)
Traffic drops: [■■]             (scaled back down)
```

### Common Trigger Metrics

- CPU utilization (%)
- Memory utilization (%)
- Requests per second
- Queue length (for worker/consumer systems)
- Custom application metrics (e.g. active user sessions)

## 📊 Scaling Strategies

| Strategy                     | Description                                                   | Example                          |
| ---------------------------- | ------------------------------------------------------------- | -------------------------------- |
| **Reactive scaling**   | Scale based on current metrics crossing a threshold           | CPU > 70% → add instance        |
| **Scheduled scaling**  | Scale at known times                                          | Scale up every weekday at 9 AM   |
| **Predictive scaling** | Uses ML/historical patterns to scale*before* the spike hits | Anticipates Black Friday traffic |

## 💻 Example (AWS Auto Scaling Group policy — conceptual)

```yaml
AutoScalingGroup:
  MinSize: 2
  MaxSize: 10
  DesiredCapacity: 2
  TargetTrackingPolicy:
    Metric: CPUUtilization
    TargetValue: 60
```

This says: *keep average CPU around 60% by adding/removing instances automatically, staying between 2 and 10 instances.*

## ⚡ Performance Considerations

- **Cold start delay**: New instances take time to boot, warm up, and pass health checks (see `03_Health_Checks.md`) — scaling isn't instantaneous.
- **Cooldown periods**: Prevent rapid scale up/down oscillation ("flapping") by waiting before the next scaling action.
- Must work together with a **Load Balancer** to distribute traffic to new instances as they come online.

## 🚨 Common Mistakes

- ❌ No cooldown period → constant scale up/down thrashing on fluctuating metrics.
- ❌ Scaling triggers too slow to react → users experience errors before new capacity comes online.
- ❌ Not testing whether the app can actually handle **fast horizontal scale-out** (e.g. stateful apps that store session in local memory break when new instances spin up — see stateless design principle).
- ❌ No `MaxSize` cap → a bug causing runaway scaling can cause a massive unexpected bill.

## 💡 Best Practices

- Keep application **stateless** so any instance can serve any request (store session/state externally — e.g. Redis).
- Set sensible `Min`, `Max`, and cooldown values.
- Use **predictive scaling** for known traffic patterns (e.g. sales events) to avoid cold-start lag.
- Monitor and alert on scaling events themselves, not just app metrics.

## 🎤 Interview Questions

1. What's the difference between reactive and predictive auto scaling?
2. Why does auto scaling require the application to be stateless?
3. What is a "cooldown period" in auto scaling and why is it needed?
4. What challenges does auto scaling introduce that a fixed-size deployment doesn't have?

## 📝 30-second Revision Cheat Sheet

- Auto Scaling = adjusts instance count automatically based on demand.
- Mostly horizontal scaling in practice.
- Triggered by metrics: CPU, RPS, queue length, etc.
- Needs: stateless apps, cooldown periods, and pairing with a load balancer.
- Watch out for cold-start delay and scaling thrash.
