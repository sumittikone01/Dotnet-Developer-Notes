# 06_Cache_Eviction_Policies

> **Eviction Policy** = the rule a cache uses to decide **which entries to remove when it's full** and new data needs space.

> Different problem from `05_Cache_Invalidation.md`: invalidation removes data because it's *stale/wrong*; eviction removes data because the cache is *out of room* — the data might still be perfectly valid.

## 📌 What is it?

Caches have limited memory. When the cache is full and a new item needs to be stored, the eviction policy decides **which existing item gets kicked out** to make room.

## 🤔 Why do we need it?

- RAM is finite — you can't cache everything forever.
- A good eviction policy keeps the **most valuable** (likely to be reused) data in cache and discards the **least valuable**, maximizing your hit ratio for a given memory budget.
- The wrong policy for your access pattern can quietly tank performance (e.g., evicting hot data while cold data lingers).

## 🌍 Real-world analogy

Your **phone's recent apps list** with limited slots. When you open a new app and the list is full, the phone has to close one — usually the one you haven't touched in the longest time (LRU-like behavior).

## 📊 Common Eviction Policies

| Policy              | Full name             | Rule                                                | Best for                                   |
| ------------------- | --------------------- | --------------------------------------------------- | ------------------------------------------ |
| **LRU**       | Least Recently Used   | Evict the item not accessed for the longest time    | General-purpose, most common default       |
| **LFU**       | Least Frequently Used | Evict the item accessed the fewest total times      | Data with a stable "popularity" pattern    |
| **FIFO**      | First In, First Out   | Evict the oldest-inserted item, regardless of usage | Simple queues, when recency doesn't matter |
| **TTL-based** | Time-To-Live expiry   | Evict items once their expiry time passes           | Naturally time-sensitive data              |
| **Random**    | Random Replacement    | Evict a random item                                 | Very simple, low overhead, unpredictable   |
| **MRU**       | Most Recently Used    | Evict the item accessed*most* recently            | Rare — used for scan-resistant workloads  |

## 🖼 LRU in action (ASCII diagram)

```
Cache capacity = 3

Access order: A, B, C, D

Step 1: cache [A]
Step 2: cache [A, B]
Step 3: cache [A, B, C]        (full)
Step 4: access D → evict LEAST recently used (A) → cache [B, C, D]

Now access B again:
Step 5: cache reorders → [C, D, B]   (B is now "most recent")

Access E → cache full → evict LEAST recently used (C) → [D, B, E]
```

LRU is typically implemented with a **Doubly Linked List + HashMap**:

- HashMap → O(1) lookup of any key
- Doubly Linked List → O(1) move-to-front on access, O(1) remove-from-tail on eviction

## ⚙️ Internal working — why LRU needs a linked list + hashmap

```
HashMap:  key → Node pointer   (O(1) find)

Doubly Linked List (ordered by recency):
   [MOST RECENT] ⇄ Node ⇄ Node ⇄ Node ⇄ [LEAST RECENT]
        ↑                                    ↑
   move here on                        evict from here
   every access                        when cache is full
```

## 💻 Code examples

### Basic — .NET `IMemoryCache` size-limited with LRU-like eviction

```csharp
// Program.cs
builder.Services.AddMemoryCache(options =>
{
    options.SizeLimit = 1000; // total "size units" the cache can hold
});

// When setting entries, assign a size and priority
var options = new MemoryCacheEntryOptions
{
    Size = 1,                                   // counts toward SizeLimit
    Priority = CacheItemPriority.Normal,        // Low / Normal / High / NeverRemove
    SlidingExpiration = TimeSpan.FromMinutes(10)
};

_cache.Set("product:5", product, options);
```

> ASP.NET Core's `MemoryCache` evicts based on **size pressure + priority + expiration**, roughly approximating LRU behavior — it's not a strict textbook LRU, but conceptually similar.

### Intermediate — a minimal LRU cache from scratch (interview-style)

```csharp
public class LruCache<TKey, TValue>
{
    private readonly int _capacity;
    private readonly Dictionary<TKey, LinkedListNode<(TKey key, TValue value)>> _map = new();
    private readonly LinkedList<(TKey key, TValue value)> _order = new();

    public LruCache(int capacity) => _capacity = capacity;

    public bool TryGet(TKey key, out TValue value)
    {
        if (_map.TryGetValue(key, out var node))
        {
            _order.Remove(node);
            _order.AddFirst(node); // mark as most recently used
            value = node.Value.value;
            return true;
        }
        value = default!;
        return false;
    }

    public void Put(TKey key, TValue value)
    {
        if (_map.TryGetValue(key, out var existing))
        {
            _order.Remove(existing);
        }
        else if (_map.Count >= _capacity)
        {
            // Evict least recently used = last node in the list
            var lru = _order.Last!;
            _map.Remove(lru.Value.key);
            _order.RemoveLast();
        }

        var node = new LinkedListNode<(TKey, TValue)>((key, value));
        _order.AddFirst(node);
        _map[key] = node;
    }
}
```

> This exact pattern (**"Design an LRU Cache"**) is one of the most frequently asked coding interview questions — know it by heart.

## ⚡ Performance considerations

- LRU: O(1) get and put with the linked-list + hashmap design — no reason to use a slower approach.
- LFU is more "accurate" for stable popularity patterns but costs more to maintain (needs frequency counters, more bookkeeping).
- FIFO/Random are cheapest to implement but can evict frequently-used data by bad luck — rarely used in serious production caches.

## 🚨 Common mistakes

- ❌ Picking LFU for bursty/trending data — an item that was extremely popular yesterday but is dead today stays cached ("pollution") since its historical count is high.
- ❌ Assuming `IMemoryCache` does textbook strict LRU — it's priority + size + expiration based, not a pure LRU implementation.
- ❌ Not setting any capacity/size limit at all — an "eviction policy" is meaningless if the cache can grow unbounded (memory leak risk, ties back to `01_Cache_Basics.md`).

## 💡 Best practices

- ✅ Default to **LRU** unless you have a specific, measured reason to use something else — it's the best general-purpose choice.
- ✅ Combine eviction policy with TTL expiration — they solve different problems (space pressure vs staleness) and work well together.
- ✅ For distributed caches (Redis), configure `maxmemory-policy` (e.g., `allkeys-lru`) — this is the Redis-native way to set eviction behavior.

## 🎤 Interview Quick-Fire Q&A

| Question                                                 | Answer                                                                                                                        |
| -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| What's the difference between eviction and invalidation? | Eviction removes data because the cache is full (data may still be valid); invalidation removes data because it's stale/wrong |
| What data structures implement LRU in O(1)?              | A HashMap (O(1) lookup) + a Doubly Linked List (O(1) reorder/remove)                                                          |
| What does LFU stand for and how does it differ from LRU? | Least Frequently Used — evicts based on total access count, not recency                                                      |
| What's a downside of LFU?                                | Old-but-once-popular items can "pollute" the cache since their historical count stays high even if unused now                 |
| What's Redis's setting for eviction policy called?       | `maxmemory-policy` (e.g., `allkeys-lru`, `allkeys-lfu`)                                                                 |

## 📝 30-second Revision Cheat Sheet

- Eviction = removing data because cache is full (not because it's wrong — that's invalidation).
- Common policies: **LRU** (most common default), LFU, FIFO, TTL-based, Random.
- LRU implementation = HashMap + Doubly Linked List → O(1) get/put.
- "Design an LRU Cache" is a classic interview coding question — practice it.
- Redis config: `maxmemory-policy` controls eviction behavior in production.
