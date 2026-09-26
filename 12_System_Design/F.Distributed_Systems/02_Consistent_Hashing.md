# 02_Consistent_Hashing

> **Consistent Hashing** = a hashing technique that maps both data and servers onto the same circular space (a "hash ring"), so that adding or removing a server only requires reshuffling a **small fraction** of the data — not almost all of it.

> Directly solves the resharding pain flagged in `02_Sharding.md` ("hash-based sharding requires re-hashing and moving huge amounts of data"). This chapter is that fix.

## 📌 What is it?

With plain hash-based sharding (`hash(key) % N`), changing `N` (the number of servers) changes the result of the modulo for **almost every key** — meaning almost all data has to move. Consistent Hashing arranges servers and keys on a **ring** so that only the keys immediately "owned" by a changed server need to move.

## 🤔 Why do we need it?

Recall from `02_Sharding.md`:

> *"Resharding is expensive and risky — hash-based sharding requires re-hashing and moving huge amounts of data."*

```
Plain hash-based sharding: hash(key) % N

N = 4 servers:  hash(key) % 4
N = 5 servers:  hash(key) % 5   ← almost EVERY key's result changes!

Example: hash("product:101") = 37
   37 % 4 = 1   → was on Server 1
   37 % 5 = 2   → now on Server 2   (moved, even though nothing about this key changed)
```

Nearly every key gets remapped to a different server just because you added ONE new server. At scale, that means moving terabytes of data for a single capacity change. Consistent Hashing fixes this.

## 🌍 Real-world analogy

A **circular parking lot with numbered spots**, where each car (key) walks clockwise from its assigned spot on the circle until it finds the nearest parking attendant (server) stationed somewhere on that circle. If one attendant leaves, only the cars that were walking toward *that specific attendant* need to redirect to the next one — every other car's routing is completely unaffected.

## ⚙️ Internal working — The Hash Ring

```
Step 1: Hash both SERVERS and KEYS onto the same circular range (e.g., 0 to 2^32 - 1)

                    0/360°
                      │
        Server D ─────┼───── Server A
       (hash=300)     │      (hash=20)
                      │
   270° ──────────────┼────────────── 90°
                      │
        Server C ─────┼───── Server B
       (hash=200)     │      (hash=100)
                      │
                    180°

Step 2: To find which server owns a KEY, hash the key, then walk
        CLOCKWISE from that point until you hit the first server.

   key "product:101" → hash = 50
   → walk clockwise from 50 → first server found = Server B (hash=100)
   → Server B owns this key
```

## 🖼 Why removing/adding a server only affects a small slice

```
BEFORE — Server B is removed:

  ... Server A (20) ── [keys 21-100] ── Server B (100) ── [keys 101-200] ── Server C (200) ...

AFTER — Server B removed:

  ... Server A (20) ── [keys 21-200, now ALL walk clockwise to Server C] ── Server C (200) ...

ONLY the keys that used to belong to Server B (21-100) get reassigned — to Server C (their new
clockwise neighbor). Server A's keys, Server C's original keys, Server D's keys — ALL UNCHANGED.
```

Compare this to `% N` hashing, where changing the server count reshuffled **almost everything**. Consistent Hashing only reshuffles the slice that belonged to the removed/added server.

## 📊 Plain Hash (`% N`) vs Consistent Hashing

| Aspect                                       | Plain Hash (`hash % N`)          | Consistent Hashing                                                   |
| -------------------------------------------- | ---------------------------------- | -------------------------------------------------------------------- |
| Keys remapped when a server is added/removed | Almost ALL keys                    | Only keys near the changed server (~1/N of total)                    |
| Implementation complexity                    | Very simple                        | Moderate (needs a sorted ring structure)                             |
| Used in                                      | Simple, fixed-size sharding setups | Redis Cluster, DynamoDB, Cassandra, CDN routing, load balancers      |
| Risk of uneven distribution                  | Even, if N is fixed                | Can be uneven with few servers — fixed with "virtual nodes" (below) |

## 🖼 Virtual Nodes — fixing uneven distribution

With only a handful of real servers, their positions on the ring can land unevenly (e.g., all clustered together), giving one server way more keys than another.

**Fix:** Each physical server is hashed onto the ring **multiple times** under different virtual identities (e.g., `ServerA#1`, `ServerA#2`, `ServerA#3`...), spreading its "presence" more evenly around the ring.

```
Without virtual nodes:              With virtual nodes (3 each):

  A ─────────── B                    A1 ── B2 ── A2 ── B1 ── A3 ── B3
  (huge gap = uneven load)           (evenly interleaved = balanced load)
```

## 💻 Code examples

### Basic — a minimal consistent hash ring in C#

```csharp
public class ConsistentHashRing
{
    private readonly SortedDictionary<int, string> _ring = new();
    private readonly int _virtualNodesPerServer;

    public ConsistentHashRing(int virtualNodesPerServer = 100)
    {
        _virtualNodesPerServer = virtualNodesPerServer;
    }

    public void AddServer(string serverName)
    {
        for (int i = 0; i < _virtualNodesPerServer; i++)
        {
            int hash = ComputeHash($"{serverName}#{i}");
            _ring[hash] = serverName;
        }
    }

    public void RemoveServer(string serverName)
    {
        for (int i = 0; i < _virtualNodesPerServer; i++)
        {
            int hash = ComputeHash($"{serverName}#{i}");
            _ring.Remove(hash);
        }
    }

    public string GetServerForKey(string key)
    {
        int hash = ComputeHash(key);

        // Find the first server clockwise from this hash (SortedDictionary keeps keys ordered)
        foreach (var kvp in _ring)
        {
            if (kvp.Key >= hash)
                return kvp.Value;
        }

        // Wrap around the ring back to the first server
        return _ring.First().Value;
    }

    private int ComputeHash(string input)
    {
        using var md5 = System.Security.Cryptography.MD5.Create();
        byte[] bytes = md5.ComputeHash(System.Text.Encoding.UTF8.GetBytes(input));
        return BitConverter.ToInt32(bytes, 0);
    }
}
```

### Intermediate — usage in a shard router (replacing the `% N` approach from `02_Sharding.md`)

```csharp
public class ShardRouter
{
    private readonly ConsistentHashRing _ring = new();

    public ShardRouter(IEnumerable<string> shardConnectionStrings)
    {
        foreach (var connStr in shardConnectionStrings)
            _ring.AddServer(connStr);
    }

    public string GetConnectionStringForUser(int userId)
    {
        // No more "hash(userId) % N" — the ring handles minimal-reshuffle routing
        return _ring.GetServerForKey(userId.ToString());
    }

    // Adding a shard is now cheap — only a fraction of keys need to move
    public void AddShard(string newConnStr) => _ring.AddServer(newConnStr);
}
```

## ⚡ Performance considerations

- Lookup is `O(log N)` using a sorted structure (e.g., `SortedDictionary`/binary search) — very fast even with many virtual nodes.
- More virtual nodes = better load balance, but more memory/lookup overhead — 100–200 virtual nodes per physical server is a common practical range.
- This is exactly the mechanism behind **Redis Cluster**, **DynamoDB**, and **Cassandra**'s partitioning — not just an academic exercise.

## 🚨 Common mistakes

- ❌ Using plain `hash % N` sharding in a system expected to scale up/down frequently — guarantees expensive full reshuffles.
- ❌ Skipping virtual nodes — with too few real servers, the ring can be very unevenly distributed, causing hotspots.
- ❌ Assuming consistent hashing eliminates data movement entirely — it minimizes it, but the keys that *do* need to move still must be physically migrated (a real operational task).

## 💡 Best practices

- ✅ Use consistent hashing (with virtual nodes) instead of plain modulo hashing for any system expected to scale its server count over time.
- ✅ Tune the virtual-node count based on measured load distribution, not a guess — start around 100–150 per node and adjust.
- ✅ Pair with monitoring for per-server load, so you catch any residual imbalance early.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                           | Answer                                                                                                                              |
| ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| What problem does Consistent Hashing solve?                                        | Minimizing data movement when servers are added/removed, compared to plain`hash % N` sharding which remaps almost everything      |
| How does the "ring" work at a high level?                                          | Both servers and keys are hashed onto a circular space; a key belongs to the first server found walking clockwise from its position |
| What fraction of keys move when a server is added/removed with consistent hashing? | Roughly`1/N` of the keys (only those near the changed server), not all of them                                                    |
| What problem do "virtual nodes" solve?                                             | Uneven distribution of keys when there are only a few physical servers on the ring                                                  |
| Name two real systems that use consistent hashing.                                 | Redis Cluster and DynamoDB (also Cassandra, many CDNs and load balancers)                                                           |

## 📝 30-second Revision Cheat Sheet

- Consistent Hashing = map servers + keys onto a ring; key belongs to the next server clockwise.
- Fixes the "% N" problem: adding/removing a server only reshuffles ~1/N of keys, not nearly all.
- Virtual nodes = each physical server appears multiple times on the ring → even load distribution.
- Lookup is O(log N) with a sorted structure.
- Used in real systems: Redis Cluster, DynamoDB, Cassandra.
