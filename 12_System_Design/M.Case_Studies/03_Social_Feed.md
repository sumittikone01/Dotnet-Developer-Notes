# 📰 Case Study: Social Feed (e.g. Twitter/X timeline, Instagram feed)

## 📌 What is it?

A system that shows each user a personalized, chronologically (or algorithmically) ordered stream of posts from accounts they follow.

## 🤔 Requirements

### Functional

- Users can post content.
- Users can follow other users.
- Each user sees a feed aggregating posts from everyone they follow.

### Non-Functional

- **Read-heavy**: feeds are viewed far more often than posts are created.
- **Low latency** on feed load — users expect it instantly on app open.
- Handle **celebrity/hot users** with millions of followers (huge fan-out imbalance).
- Eventually consistent is acceptable (a few seconds delay before a post appears is fine).

## 🧠 Core Design Question: How do you generate each user's feed?

| Approach                             | How it works                                                                                       | Tradeoff                                                                                                             |
| ------------------------------------ | -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Pull (Fan-out on Read)**     | When a user opens their feed, query posts from everyone they follow, merge & sort in real time     | Simple, but slow feed reads for users following many people                                                          |
| **Push (Fan-out on Write)** ⭐ | When a user posts, immediately**push** that post into the precomputed feed of every follower | Feed reads are instant (just read precomputed list), but writes get expensive for accounts with huge follower counts |
| **Hybrid** ⭐⭐                | Push for normal users; Pull for celebrity accounts (merge at read time only for celebrities)       | Best of both — used by real systems like Twitter                                                                    |

### 🖼 Fan-out on Write (Push Model)

```
User X posts "Hello world"
        │
        ▼
  Fan-out Service
        │
   ┌────┼────┬────────┬────────┐
   ▼    ▼    ▼        ▼        ▼
Follower1 Follower2 Follower3 ... FollowerN
(each gets the post pre-inserted
 into their own precomputed feed)
```

**Problem**: if User X has 50 million followers, one post triggers 50 million writes — extremely expensive and slow.

### 🖼 Hybrid Solution

```
Normal user posts   → Fan-out on Write (push to followers' feeds immediately)
Celebrity posts     → NOT pushed; fetched via Fan-out on Read
                       (merged into follower's feed only when they open the app)
```

This avoids the "celebrity problem" — a huge write storm — while keeping regular users' feeds instantly available.

## ⚙️ High-Level Architecture

```
┌────────┐        ┌──────────────┐
│ User posts│─────▶│ Post Service  │───▶ persists to Post DB
└────────┘        └──────┬───────┘
                          │ triggers
                          ▼
                  ┌───────────────┐
                  │ Fan-out Service│
                  └───────┬───────┘
                          │ (skip for celebrity accounts)
                          ▼
                  ┌───────────────────┐
                  │  Feed Cache (Redis) │  ── precomputed per-user feed (list of post IDs)
                  └───────────────────┘
                          ▲
                          │ read
                  ┌───────────────┐
                  │  User opens app │
                  └───────────────┘
```

## 📊 Data Storage

| Data                           | Store                                       | Why                                                        |
| ------------------------------ | ------------------------------------------- | ---------------------------------------------------------- |
| **Posts**                | SQL/NoSQL (e.g. Cassandra)                  | Write-once, read-many; partition by`post_id`/`user_id` |
| **Follow graph**         | Graph-friendly store or SQL adjacency table | `follower_id`, `followee_id` pairs                     |
| **Precomputed feed**     | Redis (list of post IDs per user)           | Fast reads — feed is just a cached list                   |
| **Media (images/video)** | Object storage (S3) + CDN                   | Large binary blobs, best served via CDN (see`01_CDN.md`) |

## 🔀 Feed Read Flow

```
1. User opens app
2. Fetch precomputed feed (post IDs) from Redis for that user
3. If some followed accounts are celebrities → fetch their recent posts live, merge in by timestamp
4. Batch-fetch full post content (text/media) for those IDs
5. Return merged, sorted feed
```

## ⚡ Scaling Considerations

- **Fan-out is the core bottleneck** — the hybrid push/pull split exists specifically to handle the extreme skew between average users and celebrity accounts.
- **Caching precomputed feeds** in Redis avoids recomputing joins/sorts on every single app open.
- **Pagination**: feeds are infinite-scroll — fetch small pages (e.g. 20 posts) at a time, not the whole feed at once.
- **CDN** for images/videos in posts — feed metadata (post IDs, text) is lightweight, but media is the actual bandwidth-heavy part.

## 🚨 Common Mistakes in Interviews

- ❌ Only proposing pure Fan-out on Write, without addressing the celebrity/hot-key problem.
- ❌ Only proposing pure Fan-out on Read, without addressing feed-read latency at scale.
- ❌ Forgetting media storage needs a **separate strategy** (CDN + object storage) from post metadata.
- ❌ Not discussing pagination for infinite-scroll feeds.

## 💡 Key Takeaways

- **Hybrid fan-out** (push for normal users, pull for celebrities) is the real-world standard approach.
- Precompute feeds into a cache (Redis) so reads are cheap.
- Separate storage strategy for posts (structured data) vs. media (CDN + object storage).
- This is a classic example of trading write cost for read speed — and adapting that tradeoff per-account based on follower count.

## 🎤 Interview Questions

1. What's the difference between Fan-out on Write and Fan-out on Read, and what's each one's weakness?
2. How does the "celebrity problem" break a pure push-based feed design?
3. How would you design the hybrid approach to decide when to use push vs pull?
4. How would you paginate an infinite-scroll feed efficiently?

## 📝 30-second Revision Cheat Sheet

- Feed generation: Push (fan-out on write) vs Pull (fan-out on read) vs **Hybrid** (real-world standard).
- Hybrid: normal users get pushed feeds; celebrities are pulled/merged at read time.
- Precomputed feed cached in Redis for instant reads.
- Media served via CDN + object storage, separate from post metadata.
