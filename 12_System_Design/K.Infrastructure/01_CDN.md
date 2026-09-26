# 🌐 CDN (Content Delivery Network)

## 📌 What is it?

A **CDN** is a geographically distributed network of servers (called **edge servers** or **PoPs — Points of Presence**) that cache and deliver content **closer to the end user**, instead of every request traveling all the way to the origin server.

## 🤔 Why do we need it?

- **Latency**: Physics — data travel time increases with distance. A user in Mumbai fetching an asset from a server in Virginia adds real, unavoidable round-trip delay.
- **Origin server load**: Offloads static content requests from your main servers.
- **Reliability**: If one edge location fails, traffic reroutes to another.
- **DDoS protection**: CDNs absorb massive traffic spikes across their distributed network.

## 🌍 Real-world analogy

Instead of every customer in the world ordering directly from **one factory in China**, a company builds **regional warehouses** (in the US, Europe, Asia) stocked with popular products. Customers get their order from the nearest warehouse — much faster than shipping from the single factory every time.

## ⚙️ Internal Working

```
                     ┌──────────────────┐
                     │   Origin Server   │
                     │   (e.g. Mumbai)   │
                     └─────────▲────────┘
                               │ (cache miss only)
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
       ┌────────────┐   ┌────────────┐   ┌────────────┐
       │ Edge Node  │   │ Edge Node  │   │ Edge Node  │
       │  (US)      │   │ (Europe)   │   │ (Asia)     │
       └─────▲──────┘   └─────▲──────┘   └─────▲──────┘
             │                │                │
          US Users       Europe Users      Asia Users
```

### Request Flow

1. User requests `image.jpg` from `cdn.example.com`.
2. DNS routes the user to the **nearest edge server** (via Anycast/GeoDNS).
3. **Cache hit** → edge server returns the file immediately (fast ⚡).
4. **Cache miss** → edge server fetches it from the origin, caches it, then returns it. Subsequent requests from that region hit the cache.

## 📊 What CDNs typically cache

| Content Type                           | Cacheable?  | Notes                           |
| -------------------------------------- | ----------- | ------------------------------- |
| Images, CSS, JS (static assets)        | ✅ Yes      | Ideal use case                  |
| Videos                                 | ✅ Yes      | Streaming CDNs specialize here  |
| HTML pages (static)                    | ✅ Yes      | For static sites/SSG            |
| API responses (dynamic, personalized)  | ⚠️ Rarely | Only if identical for all users |
| User-specific data (e.g. bank balance) | ❌ No       | Must always hit origin          |

## 💻 Example: Setting Cache Headers (controls CDN caching behavior)

```http
Cache-Control: public, max-age=86400   # cache for 24 hours
```

```http
Cache-Control: no-store   # never cache (sensitive data)
```

```http
Cache-Control: private, max-age=0, must-revalidate   # user-specific, always re-check
```

## ⚡ Performance Considerations

- **Cache hit ratio** is the key CDN metric — higher = less load on origin, faster responses.
- Use **cache invalidation/purging** carefully when content updates (see `05_Cache_Invalidation.md`).
- Combine with **versioned file names** (e.g. `app.v2.js`) to avoid stale-cache issues entirely — new version = new URL = automatic fresh fetch.

## 🚨 Common Mistakes

- ❌ Caching personalized/sensitive content publicly.
- ❌ Not setting proper `Cache-Control` headers — CDN falls back to conservative (or no) caching.
- ❌ Forgetting to invalidate cached content after a deploy, causing users to see stale assets.
- ❌ Assuming CDN solves *all* latency — it doesn't help dynamic, non-cacheable, per-user API calls.

## 💡 Best Practices

- Use **cache-busting via filename versioning/hashing** for static assets.
- Set explicit `Cache-Control` headers — don't rely on defaults.
- Use CDN for static assets first — it's the highest-value, lowest-risk win.
- Consider CDN-level **edge compute** (e.g. Cloudflare Workers) for lightweight logic close to users.

## 🎤 Interview Questions

1. How does a CDN decide which edge server serves a given user?
2. What's the difference between a cache hit and cache miss in a CDN context, and how do you measure/improve hit ratio?
3. Why shouldn't personalized API responses be cached at the CDN layer?
4. How would you handle cache invalidation when you deploy a new version of a static asset?

## 📝 30-second Revision Cheat Sheet

- CDN = distributed edge servers caching content closer to users.
- Solves: latency, origin load, DDoS resilience.
- Best for: static assets (images, CSS, JS, video).
- Controlled via `Cache-Control` headers.
- Pair with filename versioning to avoid stale-cache problems.
