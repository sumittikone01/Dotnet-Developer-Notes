# 🏆 PriorityQueue<TElement, TPriority>

## 📌 What is it?

`PriorityQueue<TElement, TPriority>` (introduced in .NET 6) is a queue where elements are dequeued **based on priority**, not arrival order — the element with the **lowest priority value** always comes out first, regardless of when it was added.

```csharp
var pq = new PriorityQueue<string, int>();
pq.Enqueue("Low priority task", 5);
pq.Enqueue("Urgent task", 1);
pq.Enqueue("Medium task", 3);

Console.WriteLine(pq.Dequeue());   // "Urgent task" — priority 1 is lowest, comes out first
Console.WriteLine(pq.Dequeue());   // "Medium task" — priority 3
```

## 🤔 Why do we need it?

A regular `Queue<T>` (see `01_Queue_T.md`) only respects **arrival order** (FIFO). But many real-world problems need to process the **most important/urgent** item next, regardless of when it arrived — e.g. hospital triage, task scheduling, or pathfinding algorithms that always expand the "cheapest" next option.

## 🌍 Real-world analogy

An **airport security fast-track lane** ✈️ — a first-class passenger who just arrived can go ahead of an economy passenger who's been waiting longer. Order of arrival doesn't matter here — **priority** does.

## ⚙️ Internal Working — Binary Heap

`PriorityQueue<TElement, TPriority>` is implemented internally as a **binary min-heap** — a tree structure where each parent node's priority is always **less than or equal to** its children's, guaranteeing the smallest-priority element is always at the root, ready to be dequeued in O(log n).

```
                [1] Urgent
               /         \
          [3] Medium    [5] Low

Dequeue() → removes root (priority 1) → heap re-balances:

          [3] Medium
               \
              [5] Low
```

## 💻 Basic Operations

```csharp
var pq = new PriorityQueue<string, int>();

pq.Enqueue("Task A", 10);
pq.Enqueue("Task B", 2);
pq.Enqueue("Task C", 5);

string next = pq.Peek();          // "Task B" — lowest priority (2), without removing
string dequeued = pq.Dequeue();    // "Task B" — removes and returns

// Safe versions — no exception if empty
if (pq.TryDequeue(out string element, out int priority))
{
    Console.WriteLine($"{element} (priority {priority})");
}
```

## 📊 Time Complexity

| Operation   | Complexity                   |
| ----------- | ---------------------------- |
| `Enqueue` | O(log n)                     |
| `Dequeue` | O(log n)                     |
| `Peek`    | O(1) — root is always known |

This is the key tradeoff vs `Queue<T>`: you get **priority-based ordering** at the cost of O(log n) instead of O(1) for enqueue/dequeue.

## 📊 Lower Value = Higher Priority (Important Gotcha!)

By default, `PriorityQueue<TElement,TPriority>` treats a **lower priority number as "more urgent"** (min-heap behavior) — this trips up many people expecting the opposite ("higher number = more important").

```csharp
pq.Enqueue("Item", 1);   // this comes out FIRST (lowest value = highest priority)
pq.Enqueue("Item", 10);  // this comes out LAST
```

**To reverse this** (so a higher number means higher priority), supply a custom comparer:

```csharp
var maxPQ = new PriorityQueue<string, int>(
    Comparer<int>.Create((a, b) => b.CompareTo(a))   // reversed comparison
);
maxPQ.Enqueue("Item", 10);   // now comes out FIRST
maxPQ.Enqueue("Item", 1);    // now comes out LAST
```

## 💻 Real-World Use Cases

- **Task scheduling**: run higher-priority jobs before lower-priority ones.
- **Dijkstra's Algorithm / A\* pathfinding**: always expand the node with the lowest known cost/distance next.
- **Event simulation**: process events in the order of their scheduled time, not the order they were created.
- **Load balancing**: route requests to the least-loaded server first.

```csharp
// Simplified Dijkstra's-style usage
var pq = new PriorityQueue<Node, int>();
pq.Enqueue(startNode, 0);

while (pq.Count > 0)
{
    var current = pq.Dequeue();   // always the closest unvisited node
    // ... process current, enqueue its neighbors with updated distances
}
```

## 🚨 Common Mistakes

- ❌ Assuming higher priority values are dequeued first — the **default is a min-heap** (lowest value first); this is the single most common gotcha with this type.
- ❌ Calling `Dequeue()`/`Peek()` on an empty priority queue — throws `InvalidOperationException`; use `TryDequeue()`/`TryPeek()` instead.
- ❌ Assuming elements with **equal priority** come out in a specific order (like FIFO) — `PriorityQueue<T>` makes **no stability guarantee** for equal priorities; if you need tie-breaking by arrival order, you must add a secondary tiebreaker yourself (e.g. combine priority with an incrementing sequence number).
- ❌ Trying to update an element's priority after enqueuing — `PriorityQueue<T>` doesn't directly support this; typical workaround is to enqueue an updated entry and treat outdated ones as stale/ignorable when dequeued.

## 💡 Best Practices

- Remember: **lower value = dequeued first** by default; supply a custom `IComparer<TPriority>` to reverse this if needed.
- Use `TryDequeue()`/`TryPeek()` to avoid exceptions when the queue might be empty.
- If tie-breaking order matters for equal priorities, add a secondary key (like insertion sequence) to your priority comparison.
- This is the natural, idiomatic choice for implementing **Dijkstra's algorithm**, **A\***, and any "always process the cheapest/most urgent item next" scheduling logic.

## 🎤 Interview Questions

1. What data structure is `PriorityQueue<TElement,TPriority>` built on internally, and why does that give O(log n) enqueue/dequeue?
2. Does a lower or higher priority value get dequeued first by default, and how would you reverse that behavior?
3. Name a classic algorithm that relies fundamentally on a priority queue.
4. Does `PriorityQueue<T>` guarantee FIFO order among elements with equal priority? How would you handle that if you needed it?

## 📝 30-second Revision Cheat Sheet

- `PriorityQueue<TElement,TPriority>` = dequeue by priority, not arrival order.
- Backed by a **binary min-heap** — O(log n) enqueue/dequeue, O(1) peek.
- **Default: lower priority value = dequeued first** — common gotcha; reverse with a custom comparer.
- No stability guarantee for equal priorities — add a tiebreaker if needed.
- Classic use case: **Dijkstra's/A\*** pathfinding, task scheduling.
