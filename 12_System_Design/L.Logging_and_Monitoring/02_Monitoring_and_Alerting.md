# 📊 Monitoring & Alerting

## 📌 What is it?

**Monitoring** is continuously collecting metrics about a system's health and performance. **Alerting** is automatically notifying humans when those metrics indicate something is wrong — so problems are caught **before** users complain, not after.

## 🤔 Why do we need it?

Logs (see `01_Logging_Basics.md`) tell you what happened *after* you go looking. Monitoring tells you something is wrong **right now**, often before any user notices — and alerting makes sure a human actually finds out.

> Logging = the black box recorder. Monitoring = the dashboard of gauges in the cockpit, live, while flying.

## 📊 The Three Pillars of Observability

| Pillar            | What it answers                                           | Example                            |
| ----------------- | --------------------------------------------------------- | ---------------------------------- |
| **Metrics** | *How is the system performing, numerically, over time?* | CPU %, request latency, error rate |
| **Logs**    | *What exactly happened, in detail?*                     | "Order 123 failed: DB timeout"     |
| **Traces**  | *Where did time go across a multi-service request?*     | See`03_Distributed_Tracing.md`   |

Together, these three give a full picture: metrics tell you *something* is wrong, logs and traces tell you *why*.

## ⚙️ Key Metrics to Monitor (the "Golden Signals")

| Signal               | Meaning                                                   |
| -------------------- | --------------------------------------------------------- |
| **Latency**    | How long requests take                                    |
| **Traffic**    | How many requests the system is receiving                 |
| **Errors**     | Rate of failed requests                                   |
| **Saturation** | How "full" the system is (CPU, memory, disk, connections) |

## 🖼 Monitoring Flow

```
┌─────────────┐     metrics      ┌──────────────┐
│ Application  │ ───────────────▶│  Metrics DB   │
│  Servers     │                 │ (Prometheus)  │
└─────────────┘                 └──────┬───────┘
                                         │
                          ┌──────────────┼──────────────┐
                          ▼                              ▼
                  ┌───────────────┐             ┌─────────────────┐
                  │   Dashboard    │             │  Alert Manager   │
                  │   (Grafana)    │             │                  │
                  └───────────────┘             └────────┬────────┘
                                                            │ threshold breached
                                                            ▼
                                                  ┌───────────────────┐
                                                  │ Notify: Slack /    │
                                                  │ PagerDuty / Email  │
                                                  └───────────────────┘
```

## 📊 Alerting Best Practices — Thresholds

| Approach                        | Example                                            | Issue                                                        |
| ------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **Static threshold**      | "Alert if CPU > 90%"                               | Simple but can false-positive on normal spikes               |
| **Rate-of-change**        | "Alert if error rate doubled in 5 min"             | Catches sudden regressions                                   |
| **SLO-based (burn rate)** | "Alert if we're burning our error budget too fast" | ⭐ Modern best practice — ties alerts to actual user impact |

## 💻 Example: Prometheus Alert Rule

```yaml
groups:
  - name: api-alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Error rate above 5% for 10 minutes"
```

This fires only if the error rate stays elevated for **10 minutes** (`for: 10m`) — avoiding alerts on tiny, momentary blips.

## 🚨 Common Mistakes

- ❌ **Alert fatigue**: too many low-value alerts → engineers start ignoring all of them, including real ones.
- ❌ Alerting on causes instead of symptoms (e.g. alerting on "CPU high" instead of "users experiencing errors" — CPU can spike harmlessly).
- ❌ No `for` duration → alerts fire on momentary blips, causing noise.
- ❌ No on-call escalation path — an alert fires but nobody sees it in time.
- ❌ Dashboards nobody looks at until an incident (should be checked proactively, not just reactively).

## 💡 Best Practices

- Alert on **symptoms that affect users** (latency, error rate) rather than every internal cause.
- Use **SLOs (Service Level Objectives)** to decide what's actually worth alerting on.
- Every alert should be **actionable** — if there's nothing to do about it, it shouldn't page anyone.
- Build dashboards around the **Golden Signals** for fast triage during incidents.
- Route alerts by severity (critical → page immediately, warning → ticket/Slack).

## 🎤 Interview Questions

1. What are the "three pillars of observability" and how do they complement each other?
2. What is "alert fatigue" and how do you prevent it?
3. Why might alerting on CPU usage alone be misleading, and what's a better approach?
4. What are the "Golden Signals" and why are they useful for monitoring any service?

## 📝 30-second Revision Cheat Sheet

- Monitoring = continuous visibility into system health; Alerting = notifying humans when something's wrong.
- Three pillars: Metrics, Logs, Traces.
- Golden Signals: Latency, Traffic, Errors, Saturation.
- Alert on **user-impacting symptoms**, not every internal fluctuation.
- Avoid alert fatigue — every alert must be actionable.
