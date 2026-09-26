# 🔀 Reverse Proxy

## 📌 What is it?

A **Reverse Proxy** is a server that sits **in front of backend servers** and forwards client requests to them, returning the backend's response back to the client — the client never talks to the backend directly.

> Not to be confused with a **Forward Proxy**, which sits in front of *clients* (hides the client from the server). A Reverse Proxy hides the *server* from the client.

## 🌍 Real-world analogy

A company's **receptionist**. Visitors never walk directly into employees' offices — they talk to the receptionist, who routes them to the right person, and can also handle simple requests (like giving directions) without bothering anyone inside.

## 🖼 Forward Proxy vs Reverse Proxy

```
Forward Proxy (hides the CLIENT):
Client ──▶ Forward Proxy ──▶ Internet ──▶ Server
  (Server doesn't know which client sent it)

Reverse Proxy (hides the SERVER):
Client ──▶ Reverse Proxy ──▶ Backend Servers
  (Client doesn't know which backend server responded)
```

## 🤔 Why do we need it?

| Purpose                       | Description                                                       |
| ----------------------------- | ----------------------------------------------------------------- |
| **Load balancing**      | Distributes requests across multiple backend instances            |
| **SSL/TLS termination** | Handles HTTPS decryption once, backend gets plain HTTP internally |
| **Security**            | Hides internal server IPs/topology from the public internet       |
| **Caching**             | Can cache static responses before hitting backend                 |
| **Compression**         | Gzip/Brotli compression handled centrally                         |
| **Single entry point**  | One domain/IP routes to many internal services                    |

## ⚙️ Internal Working — Example Flow

```
                          ┌──────────────────────┐
 Client ── HTTPS ────────▶│    Reverse Proxy      │
                          │  (Nginx / HAProxy)    │
                          │  - TLS termination     │
                          │  - Load balancing      │
                          └─────────┬─────────────┘
                                     │  HTTP (internal)
                 ┌───────────────────┼───────────────────┐
                 ▼                   ▼                   ▼
          ┌────────────┐      ┌────────────┐      ┌────────────┐
          │ App Server │      │ App Server │      │ App Server │
          │     1      │      │     2      │      │     3      │
          └────────────┘      └────────────┘      └────────────┘
```

## 💻 Example (Nginx config as a Reverse Proxy)

```nginx
server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate     /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    location / {
        proxy_pass http://backend_pool;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

upstream backend_pool {
    server 10.0.0.1:5000;
    server 10.0.0.2:5000;
    server 10.0.0.3:5000;
}
```

## 📊 Reverse Proxy vs API Gateway vs Load Balancer

| Tool                    | Primary Role                        | Extra Capabilities                                                                         |
| ----------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------ |
| **Reverse Proxy** | Forward requests to backend         | TLS termination, caching, basic routing                                                    |
| **Load Balancer** | Distribute traffic across instances | Health checks (see`03_Health_Checks.md`)                                                 |
| **API Gateway**   | Entry point for microservices       | Auth, rate limiting, request transformation, routing by service (see`03_API_Gateway.md`) |

*In practice these overlap heavily — Nginx can do all three; an API Gateway is essentially a reverse proxy with more application-aware features.*

## 🚨 Common Mistakes

- ❌ Forgetting to forward the real client IP (`X-Forwarded-For` header) — backend logs show the proxy's IP for every request instead of the actual user.
- ❌ Not handling WebSocket upgrade headers when proxying real-time connections.
- ❌ Single reverse proxy instance = single point of failure — needs its own redundancy.

## 💡 Best Practices

- Always terminate TLS at the proxy, keep internal traffic on a private network.
- Forward `X-Forwarded-For` / `X-Real-IP` headers for accurate client identification.
- Run reverse proxies in **redundant pairs** (e.g. active-passive with a floating IP, or behind a cloud load balancer).
- Use it to centralize cross-cutting concerns (compression, security headers) instead of duplicating logic in every backend service.

## 🎤 Interview Questions

1. What's the difference between a forward proxy and a reverse proxy?
2. Why would you terminate SSL at the reverse proxy instead of at each backend server?
3. How does a reverse proxy preserve the original client IP for backend logging?
4. How is a Reverse Proxy different from an API Gateway?

## 📝 30-second Revision Cheat Sheet

- Reverse Proxy = sits in front of servers, hides them from clients.
- Forward Proxy hides the *client*; Reverse Proxy hides the *server*.
- Common uses: load balancing, TLS termination, caching, security.
- Nginx/HAProxy are common implementations.
- Always forward `X-Forwarded-For` for real client IP tracking.
