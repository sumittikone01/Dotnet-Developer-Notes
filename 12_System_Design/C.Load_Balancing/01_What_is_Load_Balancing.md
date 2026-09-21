# 📌 What is Load Balancing?

## 📌 What is it?

A **Load Balancer (LB)** is a component that sits between clients and backend servers and **distributes incoming requests** across multiple servers, so no single server gets overwhelmed.

## 🤔 Why do we need it?

- One server can only handle so many concurrent requests before it slows down or crashes.
- Without an LB, all traffic hits one machine → single point of failure, no scalability.
- With an LB, you can add/remove servers seamlessly as demand changes (works hand-in-hand with [[Horizontal_Scaling]]).

## 🌍 Real-world analogy

A supermarket with multiple checkout counters and a person directing new customers to the shortest queue. That "director" is the load balancer; each counter is a server.

## ⚙️ Internal working

```
                     ┌───────────────┐
   Clients ───────▶  │ Load Balancer │
                     └───────┬───────┘
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
         Server 1        Server 2        Server 3
        (healthy)        (healthy)       (unhealthy ✗ - skipped)
```

1. Client sends a request to the LB's public address (not directly to a server).
2. LB picks a healthy backend server (based on an algorithm — see next note).
3. LB forwards the request, gets the response, and relays it back to the client.
4. LB continuously runs **health checks** to skip dead servers.

## 📊 Types of Load Balancers

| Type                             | Works at                                   | Example use case                                       |
| -------------------------------- | ------------------------------------------ | ------------------------------------------------------ |
| **L4 (Transport layer)**   | TCP/UDP, IP + port                         | Fast, protocol-agnostic routing                        |
| **L7 (Application layer)** | HTTP/HTTPS content (headers, URL, cookies) | Route`/api/*` vs `/images/*` to different services |
| **Hardware LB**            | Dedicated physical appliance               | Large enterprises, very high throughput                |
| **Software LB**            | Runs as software (NGINX, HAProxy, AWS ALB) | Most modern cloud-native systems                       |
| **DNS-based LB**           | Resolves domain to different IPs           | Geographic/global traffic distribution                 |

## 🚨 Common mistakes

- Treating the load balancer itself as un-scalable — a single LB can become a new single point of failure (solved with LB redundancy / DNS round-robin between multiple LBs).
- Forgetting **sticky sessions** requirements for stateful apps (see [[Sticky_Sessions]]).
- Not configuring proper health checks — traffic keeps going to a dead server.

## 💡 Best practices

- Always run **at least two load balancers** in an active-passive or active-active setup.
- Use L7 load balancing when you need routing based on URL path, headers, or cookies.
- Keep backend servers stateless so any server can serve any request (see [[Horizontal_Scaling]]).

## 🎤 Interview questions

- What is the difference between L4 and L7 load balancing?
- How does a load balancer know a server is unhealthy?
- Why would a system need more than one load balancer?

## 📝 30-second revision cheat sheet

- LB = traffic cop distributing requests across servers.
- L4 = fast, IP/port based. L7 = smart, content-aware routing.
- Needs redundancy + health checks to avoid becoming its own single point of failur
