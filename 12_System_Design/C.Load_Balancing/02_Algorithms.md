# 📌 Load Balancing Algorithms

## 📌 What is it?

The **rule** a load balancer uses to decide *which* backend server should receive the next incoming request.

## 🤔 Why do we need it?

Different traffic patterns and server capacities need different strategies — the wrong algorithm can overload some servers while others sit idle.

## 📊 Comparison table

| Algorithm                      | How it works                                                              | Best for                                 | Weakness                                    |
| ------------------------------ | ------------------------------------------------------------------------- | ---------------------------------------- | ------------------------------------------- |
| **Round Robin**          | Requests go to servers in fixed rotating order                            | Servers with equal capacity              | Ignores real server load                    |
| **Weighted Round Robin** | Like round robin, but powerful servers get more requests                  | Servers with different capacities        | Weights must be manually tuned              |
| **Least Connections**    | Sends request to the server with fewest active connections                | Long-lived or variable-duration requests | Slightly more overhead to track connections |
| **Least Response Time**  | Sends to the server with fewest connections**and** fastest response | Latency-sensitive systems                | More complex to compute                     |
| **IP Hash**              | Client IP is hashed to consistently map to the same server                | Needed when session affinity matters     | Uneven distribution if IPs cluster          |
| **Random**               | Picks a server at random                                                  | Simple, stateless workloads              | No load awareness                           |

## 🖼 Round Robin vs Least Connections

```
Round Robin (fixed order, ignores load):
Req1 → S1   Req2 → S2   Req3 → S3   Req4 → S1 ...

Least Connections (load-aware):
S1: 2 active   S2: 5 active   S3: 1 active
        New request ──────────────▶ S3 (fewest connections)
```

## 🌍 Real-world analogy

- **Round Robin** = a teacher calling on students in seating order, regardless of who already answered a hard question.
- **Least Connections** = calling on whichever student currently has the fewest tasks on their desk.
- **IP Hash** = always assigning the same customer to the same support agent for continuity.

## 🚨 Common mistakes

- Using plain Round Robin when servers have very different hardware specs (overloads weaker servers).
- Using IP Hash without realizing it can create **hot spots** if many users share one IP (e.g., behind a corporate NAT).
- Not revisiting the algorithm choice as traffic patterns evolve.

## 💡 Best practices

- Use **Weighted Round Robin** or **Least Connections** for heterogeneous server fleets.
- Use **IP Hash** or cookie-based routing only when session affinity is truly required — prefer stateless design instead when possible (store session in Redis).
- Combine algorithm choice with proper health checks so unhealthy servers are excluded regardless of the algorithm.

## 🎤 Interview questions

- When would you choose Least Connections over Round Robin?
- What problem does IP Hash solve, and what's its downside?
- How would you load balance across servers with different hardware specs?

## 📝 30-second revision cheat sheet

- **Round Robin** = simple rotation. **Weighted RR** = rotation + capacity awareness.
- **Least Connections** = load-aware, good for variable request durations.
- **IP Hash** = same client → same server (session affinity), but risks hot spots.
