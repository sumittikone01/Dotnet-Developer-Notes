# 🔗 Case Study: URL Shortener (e.g. bit.ly, TinyURL)

## 📌 What is it?

A service that converts a long URL into a short, unique alias (e.g. `https://short.ly/aZ9kLp`) that redirects to the original URL when visited.

## 🤔 Requirements

### Functional

- Given a long URL, generate a unique short URL.
- Visiting the short URL redirects to the original long URL.
- (Optional) Custom aliases, expiration dates, analytics (click counts).

### Non-Functional

- **High availability** — redirects must basically never fail (broken links = broken product).
- **Low latency** — redirect should feel instant.
- **Read-heavy**: reads (redirects) vastly outnumber writes (URL creation) — often 100:1 or more.
- Short URLs shouldn't be guessable in bulk (some security consideration), and should not collide.

## 🧠 Core Design Question: How to Generate the Short Code?

| Approach                                          | How it works                                                                                         | Tradeoff                                                        |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| **Hashing (MD5/SHA + truncate)**            | Hash the long URL, take first 7 chars                                                                | Collisions possible, need collision handling                    |
| **Base62 encoding of auto-increment ID** ⭐ | DB assigns sequential ID → convert to Base62 (`a-z`, `A-Z`, `0-9`)                            | Simple, no collisions, but IDs are guessable/sequential         |
| **Pre-generated key pool**                  | Background job pre-generates random Base62 keys, stores unused ones in a pool; each request pops one | No collision risk, avoids predictability, adds infra complexity |

### 🖼 Base62 Approach — Why 62 characters?

```
Base62 alphabet: a-z (26) + A-Z (26) + 0-9 (10) = 62 characters

7 characters of Base62 → 62^7 ≈ 3.5 trillion unique combinations
→ more than enough for years of usage at massive scale
```

## ⚙️ High-Level Architecture

```
                     ┌─────────────┐
   Client ─────────▶ │ Load Balancer│
                     └──────┬──────┘
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
          ┌──────────────┐    ┌──────────────┐
          │  App Server 1 │    │  App Server 2 │
          └──────┬───────┘    └──────┬───────┘
                 │                    │
                 ▼                    ▼
          ┌────────────────────────────────┐
          │     Cache (Redis) — hot URLs    │  ◀── most reads served here
          └────────────────┬───────────────┘
                            │ cache miss
                            ▼
          ┌────────────────────────────────┐
          │   Database (URL mappings)       │
          │   short_code → long_url          │
          └────────────────────────────────┘
```

## 📊 Database Schema (simplified)

| Column          | Type       | Notes                               |
| --------------- | ---------- | ----------------------------------- |
| `short_code`  | VARCHAR(7) | Primary key                         |
| `long_url`    | TEXT       | Original URL                        |
| `created_at`  | TIMESTAMP  |                                     |
| `expires_at`  | TIMESTAMP  | Nullable                            |
| `click_count` | BIGINT     | For analytics (often updated async) |

A **key-value store** (like DynamoDB or Cassandra) fits well here since access is purely by `short_code` — no complex joins needed.

## 🔀 Read Path (Redirect) — the critical, high-traffic path

```
1. User visits short.ly/aZ9kLp
2. App checks Cache (Redis) for "aZ9kLp"
   → Cache HIT (very common, since popular links get hit repeatedly) → return long URL instantly
   → Cache MISS → query DB, populate cache, return long URL
3. Respond with HTTP 301/302 redirect to the long URL
```

**301 (Permanent) vs 302 (Temporary) redirect:**

- `301` → browsers cache the redirect, reducing future server hits, but you lose click analytics on repeat visits.
- `302` → every click hits your server, giving accurate analytics, at the cost of more load.
- Most URL shorteners use **302** specifically to preserve click-tracking data.

## ⚡ Scaling Considerations

- **Caching** (Redis) is critical — with a 100:1 read:write ratio, most redirects should never touch the DB.
- **Database sharding**: since access is always by `short_code`, this is a natural sharding key (see `02_Sharding.md`).
- **CDN/Edge caching** for popular redirects can push latency even lower (see `01_CDN.md`).
- Use a **separate async pipeline** for updating `click_count` (don't block the redirect on a write).

## 🚨 Common Mistakes in Interviews

- ❌ Forgetting to discuss the read-heavy nature and caching strategy.
- ❌ Using auto-increment IDs directly as short codes without considering they're guessable/enumerable.
- ❌ Ignoring what happens on **hash collisions** if using a hash-based approach.
- ❌ Not discussing 301 vs 302 tradeoff for analytics.

## 💡 Key Takeaways

- Read-heavy system → **cache aggressively** (see `D.Caching` chapter).
- Base62 encoding of a unique ID is the simplest robust approach.
- Key-value DB fits the access pattern well; shard by `short_code`.
- Use 302 redirects if click analytics matter.

## 🎤 Interview Questions

1. How would you generate unique short codes at scale without collisions?
2. Why is caching especially critical for this specific system?
3. What's the difference between using a 301 vs 302 redirect here, and why does it matter?
4. How would you shard the database as this system grows to billions of URLs?

## 📝 30-second Revision Cheat Sheet

- Read-heavy system (100:1+) → cache is the star of the design.
- Base62 encode an auto-increment ID (or pre-generated key pool) for short codes.
- Key-value store, sharded by `short_code`.
- Use 302 redirects to preserve click analytics.
