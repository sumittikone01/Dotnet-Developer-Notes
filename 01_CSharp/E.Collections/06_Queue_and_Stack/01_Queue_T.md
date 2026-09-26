# 🚶 Queue<T></t>

## 📌 What is it?

`Queue<T>` is a collection that processes elements in **FIFO order** — **First In, First Out**. The first element added is always the first one removed.

```csharp
Queue<string> queue = new Queue<string>();
queue.Enqueue("Alice");   // joins the back of the line
queue.Enqueue("Bob");
queue.Enqueue("Charlie");

Console.WriteLine(queue.Dequeue());   // "Alice" — first in, first out
Console.WriteLine(queue.Dequeue());   // "Bob"
```

## 🤔 Why do we need it?

Many real-world processes are inherently **ordered by arrival time** — the first request/task/customer to arrive should be the first one handled. Trying to model this with a `List<T>` (removing from the front with `RemoveAt(0)`) is **O(n)** — every remaining element must shift left. A `Queue<T>` handles this specific access pattern in **O(1)**.

## 🌍 Real-world analogy

A **line at a coffee shop** ☕ — whoever joined the line first gets served first. New people join at the **back**; service happens from the **front**. That's exactly FIFO behavior.

## ⚙️ Core Operations

| Method                   | Purpose                                                             |
| ------------------------ | ------------------------------------------------------------------- |
| `Enqueue(item)`        | Add an item to the**back** of the queue                       |
| `Dequeue()`            | Remove and return the item at the**front** — throws if empty |
| `Peek()`               | View the front item**without removing it** — throws if empty |
| `TryDequeue(out item)` | Safe version — returns`bool`, no exception if empty              |
| `TryPeek(out item)`    | Safe peek — returns`bool`, no exception if empty                 |
| `Count`                | Number of items currently in the queue                              |

```csharp
var queue = new Queue<int>();
queue.Enqueue(1);
queue.Enqueue(2);

int front = queue.Peek();          // 1 — just looks, doesn't remove
int removed = queue.Dequeue();     // 1 — removes and returns

if (queue.TryDequeue(out int val))
    Console.WriteLine(val);         // safe — won't throw if queue is empty
```

## 🖼 Visualizing FIFO Behavior

```
Enqueue(A) → [A]
Enqueue(B) → [A, B]
Enqueue(C) → [A, B, C]

Dequeue()  → removes A → [B, C]      (A was first in, so first out)
Dequeue()  → removes B → [C]
```

## 💻 Real-World Use Cases

- **Task/job processing**: process background jobs in the order they were submitted.
- **Breadth-First Search (BFS)**: exploring a graph/tree level-by-level relies fundamentally on a queue.
- **Print spooling**: documents print in the order they were sent.
- **Message/request buffering**: handling incoming requests in arrival order.

```csharp
// Classic BFS using a Queue<T>
void BFS(Node start)
{
    var visited = new HashSet<Node>();
    var queue = new Queue<Node>();
    queue.Enqueue(start);
    visited.Add(start);

    while (queue.Count > 0)
    {
        var current = queue.Dequeue();
        Console.WriteLine(current.Value);

        foreach (var neighbor in current.Neighbors)
        {
            if (!visited.Contains(neighbor))
            {
                visited.Add(neighbor);
                queue.Enqueue(neighbor);
            }
        }
    }
}
```

## 📊 Time Complexity

| Operation   | Complexity     |
| ----------- | -------------- |
| `Enqueue` | O(1) amortized |
| `Dequeue` | O(1)           |
| `Peek`    | O(1)           |

Internally backed by a circular buffer (array), which is why both ends can be operated on efficiently without shifting elements.

## 🚨 Common Mistakes

- ❌ Calling `Dequeue()` or `Peek()` on an **empty queue** — throws `InvalidOperationException`. Always check `Count > 0` first, or use `TryDequeue()`/`TryPeek()`.
- ❌ Using a `List<T>` with `RemoveAt(0)` to simulate FIFO behavior — works, but is O(n) per removal instead of `Queue<T>`'s O(1).
- ❌ Confusing `Queue<T>` (FIFO) with `Stack<T>` (LIFO — see `02_Stack_T.md`) — easy to mix up which end processes first.
- ❌ Assuming a `Queue<T>` supports indexed access (`queue[0]`) — it does **not**; you can only access the front via `Peek()`/`Dequeue()`.

## 💡 Best Practices

- Use `Queue<T>` whenever processing order must match **arrival order** — task queues, request buffering, BFS traversal.
- Use `TryDequeue()`/`TryPeek()` in code paths where the queue might legitimately be empty, to avoid exception-driven control flow.
- Don't use `Queue<T>` when you need indexed access or need to search/insert in the middle — it's purpose-built for front/back operations only.

## 🎤 Interview Questions

1. What does FIFO mean, and how does `Queue<T>` implement it?
2. Why is `Queue<T>` more efficient than using `List<T>.RemoveAt(0)` to simulate the same behavior?
3. What is the classic algorithmic use case that fundamentally relies on a queue?
4. What happens if you call `Dequeue()` on an empty queue, and how would you avoid that exception?

## 📝 30-second Revision Cheat Sheet

- `Queue<T>` = FIFO — First In, First Out.
- `Enqueue()` adds to the back; `Dequeue()` removes from the front.
- O(1) for `Enqueue`/`Dequeue`/`Peek` — much better than `List<T>.RemoveAt(0)`'s O(n).
- Classic use case: **Breadth-First Search (BFS)**, task processing, print spooling.
- Use `TryDequeue()`/`TryPeek()` to avoid exceptions on an empty queue.
