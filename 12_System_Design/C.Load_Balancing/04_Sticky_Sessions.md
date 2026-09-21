# 📌 Sticky Sessions

## 📌 What is it?

**Sticky sessions** (a.k.a. session affinity) is a load balancer feature that routes **all requests from the same client** to the **same backend server** for the duration of their session.

## 🤔 Why do we need it?

Some applications store session data **in memory on a specific server** (e.g., shopping cart, login state, WebSocket connection). If a user's next request goes to a *different* server that doesn't have that data, the app breaks or the user gets logged out.

## 🌍 Real-world analogy

Always being served by the **same waiter** throughout your restaurant visit, so they remember your order and preferences — instead of a new waiter each time who has no idea what you ordered.

## ⚙️ Internal working

```
Request 1 (User A) ──▶ LB ──▶ Server 2  (LB sets cookie: "server=2")
Request 2 (User A) ──▶ LB ──▶ Server 2  (cookie says server=2 → routed there again)
Request 3 (User B) ──▶ LB ──▶ Server 1  (new user, no cookie yet → LB picks one)
```

Common mechanisms:

- **Cookie-based**: LB sets a cookie identifying which server handled the first request.
- **IP Hash**: client IP is hashed to consistently map to the same server (see 02_Algorithms).

## 📊 Sticky Sessions vs Stateless Design

| Aspect                | Sticky Sessions                                      | Stateless + Shared Store (Redis)           |
| --------------------- | ---------------------------------------------------- | ------------------------------------------ |
| Session data location | In-memory on one specific server                     | External store, any server can read it     |
| Server failure impact | User loses session if that server dies               | No impact — any server can serve the user |
| Load distribution     | Can become uneven (some servers get "hot" users)     | Always evenly distributed                  |
| Scaling complexity    | Simple to add sticky routing, but limits flexibility | Slightly more setup, but scales cleanly    |
| Recommended for       | Legacy apps, quick fixes                             | Modern, horizontally-scaled systems        |

## 🚨 Common mistakes

- Relying on sticky sessions as a permanent solution instead of moving to a shared session store — it silently reintroduces a single point of failure per user.
- Not handling the case where a "sticky" server goes down — the user's session data is lost entirely unless there's a fallback.
- Sticky sessions can cause **uneven load** if some users generate far more traffic than others (all routed to the same server).

## 💡 Best practices

- Prefer **stateless servers + a shared session store** (Redis/Memcached) over sticky sessions wherever possible — it's more resilient and scales better (ties into [[Horizontal_Scaling]]).
- If sticky sessions are unavoidable (e.g., WebSockets, legacy systems), combine them with session replication or a fast session-recovery mechanism.
- Monitor load distribution — sticky sessions can hide load imbalance until it's severe.

## 🎤 Interview questions

- What problem do sticky sessions solve, and what's the downside?
- How would you design session management to avoid needing sticky sessions at all?
- What happens to a user's session if the "sticky" server crashes?

## 📝 30-second revision cheat sheet

- Sticky sessions = same client always routed to the same server (via cookie or IP hash).
- Needed when session data lives in server memory — but creates a soft single point of failure per user.
- Better long-term fix: **stateless servers + shared session store** (Redis
