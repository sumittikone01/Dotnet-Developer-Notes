# 04_Consensus_Algorithms

> **Consensus Algorithm** = a protocol that lets a group of distributed nodes **agree on a single value or decision**, even if some nodes are slow, crash, or messages get lost — and even though `01_CAP_Theorem.md` says perfect agreement during a partition is impossible.

> This is the missing piece from earlier chapters: `01_Replication.md` said "a replica can be promoted to primary" — but **who decides which replica gets promoted, and how do all the other nodes agree on that decision?** Consensus algorithms answer exactly that.

## 📌 What is it?

In a distributed system with multiple nodes, you often need everyone to agree on one thing:

- "Who is the new primary/leader?" (after `01_Replication.md`'s failover)
- "Is this the value that was actually written?"
- "In what order did these operations happen?"

A consensus algorithm is the **rulebook** that lets nodes reach agreement reliably, even with node failures or network delays — as long as a **majority** of nodes are up and can talk to each other.

## 🤔 Why do we need it?

Without consensus, distributed failover is dangerous:

```
Primary DB crashes.

Replica A thinks: "I haven't heard from the primary — I'll become the new primary!"
Replica B thinks: "I ALSO haven't heard from the primary — I'll become the new primary too!"

Now there are TWO primaries accepting writes independently — this is called "SPLIT BRAIN"
and it silently corrupts your data (two conflicting versions of the truth).
```

Consensus algorithms exist specifically to **prevent split-brain** and guarantee that the whole cluster agrees on exactly one outcome.

## 🌍 Real-world analogy

A **jury reaching a verdict**. Twelve jurors (nodes) must agree — not just have an opinion each. The process (structured discussion, voting rounds) ensures that even if a couple of jurors are stubborn or absent, the group still reaches **one** binding decision that everyone accepts, rather than several jurors each declaring a different verdict.

## 📊 Core idea: Majority (Quorum)

Almost all consensus algorithms rely on getting agreement from a **majority** of nodes (a "quorum"), not all of them:

| Cluster size | Majority needed | Nodes that can fail and consensus still works |
| ------------ | --------------- | --------------------------------------------- |
| 3 nodes      | 2               | 1                                             |
| 5 nodes      | 3               | 2                                             |
| 7 nodes      | 4               | 3                                             |

> This is exactly why production clusters (Zookeeper, etcd, Raft-based systems) are typically deployed with **odd numbers** of nodes (3, 5, 7) — it maximizes fault tolerance per node added and avoids ties.

## ⚙️ Internal working — Raft (the most widely used, easiest-to-understand algorithm)

Raft breaks consensus into two everyday problems: **electing a leader**, and **replicating a log** through that leader.

```
1. LEADER ELECTION

  All nodes start as "Follower"
       │
       ▼
  A follower's election timeout expires (hasn't heard from a leader)
       │
       ▼
  It becomes a "Candidate" and requests votes from other nodes
       │
       ▼
  If it gets votes from a MAJORITY → becomes the "Leader"
  (If two candidates split the vote, timeouts are randomized and a re-election happens)
```

```
2. LOG REPLICATION (once a Leader exists)

  Client sends write → Leader
                          │
                          ▼
              Leader appends to its own log
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
        Follower A   Follower B   Follower C   ← Leader sends the entry to all
             │            │            │
             └──── majority ACKs ──────┘
                          │
                          ▼
              Leader commits the entry
              (now safe — tells followers to commit too)
                          │
                          ▼
              Leader responds "success" to the client
```

**Key insight:** a write is only considered "committed" (safe, durable) once a **majority** of nodes have it — this is exactly the mechanism that prevents split-brain: only one node can ever get majority votes for leadership at a given term, so there can never be two legitimate leaders at once.

## 📊 Raft vs Paxos (the two most famous consensus algorithms)

| Aspect             | **Paxos**                                                                       | **Raft**                                         |
| ------------------ | ------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| Age / origin       | Older (1989), foundational but notoriously hard to understand and implement correctly | Newer (2014), designed explicitly to be understandable |
| Structure          | No explicit leader concept in the base protocol; more abstract                        | Explicit leader election, easier mental model          |
| Real-world usage   | Google Chubby, early Zookeeper internals                                              | etcd, Consul, CockroachDB, many modern systems         |
| Practical takeaway | Historically important, rarely implemented from scratch today                         | The go-to choice for new systems needing consensus     |

## 🖼 Where consensus fits with what you already know

| Earlier concept                                                           | How consensus connects                                                                                                                                 |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Replication + Failover (`01_Replication.md`, `03_Fault_Tolerance.md`) | Consensus is HOW the cluster safely agrees on which replica becomes the new primary, without split-brain                                               |
| CAP Theorem (`01_CAP_Theorem.md`)                                       | Consensus algorithms are fundamentally**CP** — they intentionally pause/block (lose some availability) rather than risk two conflicting leaders |
| Zookeeper / etcd                                                          | These are ready-made "consensus as a service" tools — most engineers use one of these rather than implementing Raft/Paxos themselves                  |

## 💻 Code examples

Consensus algorithms are rarely hand-rolled in application code — instead, you use a battle-tested coordination service. Here's what that looks like in practice.

### Basic — using a distributed lock (backed by consensus) to prevent split-brain in your own app

```csharp
// Conceptual: acquiring a distributed lock via a coordination service (e.g., backed by etcd/Zookeeper/Redis+Redlock)
// This ensures only ONE instance of your app performs a critical action at a time,
// even if multiple instances are running behind a load balancer.

public async Task<bool> TryBecomeLeaderAsync(string lockKey, TimeSpan leaseTime)
{
    // Under the hood, this call goes through a consensus-backed coordination service
    // that guarantees only one caller across the entire cluster can hold this lock at once.
    bool acquired = await _coordinationClient.TryAcquireLockAsync(lockKey, leaseTime);
    return acquired;
}
```

```csharp
public async Task RunScheduledJobSafely()
{
    // Only the instance that wins the leader lock actually runs the job —
    // prevents the classic "5 servers all running the same nightly batch job" bug
    if (await TryBecomeLeaderAsync("nightly-report-job", TimeSpan.FromMinutes(10)))
    {
        await RunNightlyReportAsync();
    }
    // Other instances simply skip — they didn't win the lock
}
```

### Intermediate — conceptual quorum-based read (illustrating the majority principle)

```csharp
public bool TryGetConsistentValue(List<INode> nodes, string key, out string value)
{
    int majorityNeeded = (nodes.Count / 2) + 1;
    var responses = new List<string>();

    foreach (var node in nodes)
    {
        if (node.TryRead(key, out string nodeValue))
            responses.Add(nodeValue);
    }

    // Only trust the value if a MAJORITY of nodes agree on it
    var mostCommon = responses.GroupBy(v => v)
                               .OrderByDescending(g => g.Count())
                               .FirstOrDefault();

    if (mostCommon != null && mostCommon.Count() >= majorityNeeded)
    {
        value = mostCommon.Key;
        return true;
    }

    value = null!;
    return false; // no majority agreement — can't safely trust any single value
}
```

## ⚡ Performance considerations

- Consensus adds **latency** — every write typically needs a round trip to a majority of nodes before being considered safe, unlike a single-node write.
- This is a deliberate trade-off: consensus systems favor **correctness over raw speed** (a CP choice, per `01_CAP_Theorem.md`).
- Don't use a full consensus protocol for every piece of application data — reserve it for genuinely critical coordination decisions (leader election, distributed locks, configuration that must never conflict).

## 🚨 Common mistakes

- ❌ Building your own leader-election logic with simple timeouts/heartbeats and no true majority-quorum guarantee — this is exactly how split-brain bugs get introduced.
- ❌ Assuming "we have replicas" automatically means "we're safe from split-brain" — replication alone doesn't guarantee agreement on who's the leader; you need consensus for that.
- ❌ Using a heavyweight consensus-backed store for extremely high-frequency, low-stakes data — the latency cost isn't worth it there.

## 💡 Best practices

- ✅ Don't reimplement Raft/Paxos yourself — use a proven coordination service (Zookeeper, etcd, Consul) for leader election and distributed locking.
- ✅ Deploy consensus clusters with an **odd number** of nodes (3 or 5 is typical) to maximize fault tolerance per node.
- ✅ Reserve consensus-backed coordination for genuinely critical decisions (leader election, distributed locks, cluster configuration) — not general application data.

## 🎤 Interview Quick-Fire Q&A

| Question                                               | Answer                                                                                                                                      |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| What problem do consensus algorithms solve?            | Getting distributed nodes to agree on a single value/decision reliably, even with failures — critically, preventing "split-brain"          |
| What is "split-brain"?                                 | A situation where two nodes both believe they are the leader/primary and accept conflicting writes independently                            |
| Why do consensus clusters use odd numbers of nodes?    | It maximizes fault tolerance per added node and avoids tie votes when computing a majority                                                  |
| Name the two most well-known consensus algorithms.     | Paxos and Raft                                                                                                                              |
| Is a consensus system more CP or AP under CAP Theorem? | CP — it intentionally sacrifices some availability (pausing when it can't get a majority) to guarantee consistency and prevent split-brain |

## 📝 30-second Revision Cheat Sheet

- Consensus = getting distributed nodes to agree on one decision, preventing "split-brain" (two conflicting leaders).
- Relies on a **majority (quorum)** — clusters typically use odd node counts (3, 5, 7).
- Raft = leader election + log replication, designed to be understandable; Paxos = older, harder, foundational.
- A write is only "committed" once a majority of nodes acknowledge it.
- Consensus is fundamentally a **CP** choice (per CAP Theorem) — correctness over raw availability.
- In practice: use Zookeeper/etcd/Consul rather than implementing Raft/Paxos yourself.

---

✅ **F.Distributed_Systems chapter complete** (01–04). Next up: **G.Communication**.
