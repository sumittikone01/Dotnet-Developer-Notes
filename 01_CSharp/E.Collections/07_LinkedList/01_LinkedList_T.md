# 🔗 LinkedList<T></t>

## 📌 What is it?

`LinkedList<T>` is a **doubly-linked list** — a sequence of nodes where each node holds a value plus references to both the **previous** and **next** node, rather than storing elements in one contiguous memory block like an array or `List<T>`.

```csharp
LinkedList<int> list = new LinkedList<int>();
list.AddLast(1);
list.AddLast(2);
list.AddFirst(0);

foreach (var item in list)
    Console.Write(item + " ");
// Output: 0 1 2
```

## 🤔 Why do we need it?

`List<T>` is backed by a contiguous array — inserting or removing an element **in the middle** requires shifting every subsequent element (O(n)). `LinkedList<T>` avoids this: since nodes just point to their neighbors, inserting or removing a node **once you already have a reference to it** is O(1) — no shifting required.

## 🌍 Real-world analogy

A **treasure hunt chain** 🧩 — each clue tells you where to find the next one (and the previous one, in a doubly-linked version). To insert a new clue in the middle, you just re-point two arrows — you don't need to physically move every clue after it, unlike renumbering pages in a bound book (which is what an array/`List<T>` insertion is like).

## 🖼 Structure — Doubly-Linked Nodes

```
null ← [Prev|1|Next] ⇄ [Prev|2|Next] ⇄ [Prev|3|Next] → null
         Head                              Tail
```

Each node (`LinkedListNode<T>`) stores:

- `.Value` — the actual data
- `.Previous` — reference to the prior node (or `null` if it's the head)
- `.Next` — reference to the following node (or `null` if it's the tail)

## 💻 Core Operations

```csharp
var list = new LinkedList<string>();

list.AddLast("B");
list.AddLast("C");
list.AddFirst("A");            // list is now: A, B, C

LinkedListNode<string> nodeB = list.Find("B");
list.AddBefore(nodeB, "A.5");   // insert before a specific node: A, A.5, B, C
list.AddAfter(nodeB, "B.5");    // insert after a specific node:  A, A.5, B, B.5, C

list.Remove("A.5");             // removes by value (first match)
list.RemoveFirst();             // removes head
list.RemoveLast();              // removes tail
```

## 📊 Time Complexity — The Key Tradeoff

| Operation                                     | `List<T>` (array-backed)             | `LinkedList<T>`                          |
| --------------------------------------------- | -------------------------------------- | ------------------------------------------ |
| Access by index                               | O(1)                                   | **O(n)** — must walk from head/tail |
| Insert/Remove at**known node** (middle) | O(n) — shifting required              | **O(1)** — just re-point pointers   |
| Insert/Remove at start/end                    | O(n) for start, amortized O(1) for end | **O(1)** for both                    |
| Search for a value                            | O(n)                                   | O(n)                                       |

⚠️ Important nuance: `LinkedList<T>`'s O(1) insert/remove only applies **once you already have a reference to the node** (e.g. via `Find()`). Finding that node in the first place is still O(n) — so a "search then insert" operation overall is still O(n), same as with a `List<T>`.

## 🚨 The Most Important Gotcha: No Indexer!

```csharp
var list = new LinkedList<int>();
list.AddLast(10);
list.AddLast(20);

// int x = list[0];  ❌ COMPILE ERROR — LinkedList<T> has NO indexer!

// Must traverse manually:
int x = list.First.Value;   // ✅ works — access via .First / .Last
```

This is why `LinkedList<T>` is used **far less often in practice** than `List<T>` — most everyday code needs indexed access, which `LinkedList<T>` simply doesn't provide.

## 💻 Real-World Use Cases

- **LRU (Least Recently Used) Cache**: combined with a `Dictionary` for O(1) lookup + O(1) reordering of recently-used items to the front/back.
- **Undo history with frequent mid-sequence edits**: when items are added/removed from arbitrary positions frequently, and indexed access isn't needed.
- **Implementing other data structures**: deques, certain graph representations.

```csharp
// Simplified LRU cache sketch — LinkedList + Dictionary combo
var cache = new Dictionary<int, LinkedListNode<int>>();
var order = new LinkedList<int>();

void Access(int key)
{
    if (cache.TryGetValue(key, out var node))
    {
        order.Remove(node);         // O(1) — we already have the node reference
        order.AddFirst(node);        // move to "most recently used" position
    }
    else
    {
        var newNode = order.AddFirst(key);
        cache[key] = newNode;
    }
}
```

## 🚨 Common Mistakes

- ❌ Defaulting to `LinkedList<T>` for general-purpose use — in practice, `List<T>` outperforms it for most real-world scenarios because indexed access is O(1) and CPU cache locality (contiguous memory) makes array-backed structures faster in practice, even for some "theoretical O(n) vs O(1)" middle-insert cases at typical sizes.
- ❌ Trying to use `list[i]` — doesn't compile; `LinkedList<T>` has no indexer.
- ❌ Assuming `LinkedList<T>`'s insert/remove is always O(1) — it's only O(1) once you already hold the target node reference; finding that node is still O(n).
- ❌ Forgetting `LinkedList<T>` uses more memory per element than `List<T>` — each node stores extra `Previous`/`Next` reference overhead beyond just the value.

## 💡 Best Practices

- Use `LinkedList<T>` specifically when you need **frequent insertions/removals at arbitrary, already-known positions** and don't need indexed access — e.g. LRU cache implementations.
- Default to `List<T>` for general-purpose sequential data — it's simpler, has better cache locality, and covers the vast majority of real-world needs.
- Combine `LinkedList<T>` with a `Dictionary<TKey, LinkedListNode<T>>` when you need both **fast lookup** and **fast reordering** (the classic LRU cache pattern).

## 🎤 Interview Questions

1. Why does `LinkedList<T>` provide O(1) insertion/removal in the middle, while `List<T>` needs O(n)?
2. Why doesn't `LinkedList<T>` support indexed access like `list[i]`?
3. In what real-world scenario would you specifically choose `LinkedList<T>` over `List<T>`?
4. Why is `List<T>` often faster in practice than `LinkedList<T>`, even for operations where linked lists have better theoretical complexity?

## 📝 30-second Revision Cheat Sheet

- `LinkedList<T>` = doubly-linked nodes, each with `.Previous`/`.Next` references.
- O(1) insert/remove **once you have the node reference**; O(n) to find that node.
- **No indexer** — can't do `list[i]`; access via `.First`/`.Last` or traversal.
- Real-world use: LRU cache (paired with a `Dictionary` for lookup).
- `List<T>` outperforms it for most everyday scenarios — use `LinkedList<T>` only for its specific strengths.
