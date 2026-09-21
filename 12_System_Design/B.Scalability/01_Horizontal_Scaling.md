# 📌 Horizontal Scaling

## 📌 What is it?

**Horizontal scaling (scale-out)** means adding **more machines/servers** to handle increased load, instead of making one machine more powerful.

## 🤔 Why do we need it?

A single server has a hardware ceiling (CPU, RAM). Once you hit it, the only way to keep growing is to spread the load across many machines — this is how companies like Google and Netflix serve billions of requests.

## 🌍 Real-world analogy

A single cashier (one server) can only serve so many customers per hour. Instead of making that one cashier superhuman, you **open more checkout counters** — that's horizontal scaling.

## ⚙️ Internal working

```
                 ┌───────────┐
 Client ───────▶ │ Load      │
                 │ Balancer  │
                 └─────┬─────┘
           ┌───────────┼───────────┐
           ▼           ▼           ▼
      Server 1     Server 2     Server 3
```

- A **load balancer** distributes incoming requests across many identical servers.
- Servers should be **stateless** (no user session stored locally) so any server can handle any request.

## 📊 Requirements for effective horizontal scaling

| Requirement                              | Why                                               |
| ---------------------------------------- | ------------------------------------------------- |
| Stateless app servers                    | Any request can go to any server                  |
| Load balancer                            | Distributes traffic evenly                        |
| Shared session/cache store (e.g., Redis) | Sessions survive even if a specific server dies   |
| Distributed-friendly database            | DB must also scale (see Database Scaling chapter) |

## 🚨 Common mistakes

- Storing session data **in-memory on one server** — breaks when the load balancer routes a user to a different server.
- Assuming horizontal scaling is "free" — it adds complexity: distributed logging, distributed transactions, network latency between services.
- Scaling app servers but forgetting the database becomes the new bottleneck.

## 💡 Best practices

- Keep application servers **stateless**; externalize session/state to Redis or a DB.
- Use **auto-scaling groups** to add/remove servers automatically based on load.
- Design for failure — any single server should be disposable/replaceable.

## 🎤 Interview questions

- What does it mean for a server to be "stateless," and why does it matter for horizontal scaling?
- What are the challenges of horizontal scaling that vertical scaling doesn't have?
- How would you handle user sessions in a horizontally scaled system?

## 📝 30-second revision cheat sheet

- Horizontal scaling = **more machines**, distributed via a load balancer.
- Requires **stateless servers** + shared session/cache store.
- Nearly unlimited growth potential, but adds distributed-systems complexity.
