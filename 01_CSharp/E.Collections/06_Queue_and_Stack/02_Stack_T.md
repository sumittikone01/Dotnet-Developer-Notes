# 🥞 Stack<T></t>

## 📌 What is it?

`Stack<T>` is a collection that processes elements in **LIFO order** — **Last In, First Out**. The most recently added element is always the first one removed.

```csharp
Stack<string> stack = new Stack<string>();
stack.Push("Alice");    // goes on top
stack.Push("Bob");
stack.Push("Charlie");

Console.WriteLine(stack.Pop());   // "Charlie" — last in, first out
Console.WriteLine(stack.Pop());   // "Bob"
```

## 🤔 Why do we need it?

Many problems naturally require processing things in **reverse order of arrival** — undoing the most recent action first, backtracking through the most recent step, or evaluating nested expressions from the innermost outward. `Stack<T>` models exactly this "most recent first" access pattern in O(1).

## 🌍 Real-world analogy

A **stack of plates** 🍽️ — you always take the **top** plate off (the one placed most recently), and you always add new plates to the **top** too. You never grab a plate from the middle or bottom without first removing everything above it.

## ⚙️ Core Operations

| Method                | Purpose                                                           |
| --------------------- | ----------------------------------------------------------------- |
| `Push(item)`        | Add an item to the**top** of the stack                      |
| `Pop()`             | Remove and return the item at the**top** — throws if empty |
| `Peek()`            | View the top item**without removing it** — throws if empty |
| `TryPop(out item)`  | Safe version — returns`bool`, no exception if empty            |
| `TryPeek(out item)` | Safe peek — returns`bool`, no exception if empty               |
| `Count`             | Number of items currently in the stack                            |

```csharp
var stack = new Stack<int>();
stack.Push(1);
stack.Push(2);

int top = stack.Peek();       // 2 — just looks, doesn't remove
int removed = stack.Pop();    // 2 — removes and returns

if (stack.TryPop(out int val))
    Console.WriteLine(val);    // safe — won't throw if stack is empty
```

## 🖼 Visualizing LIFO Behavior

```
Push(A) → [A]
Push(B) → [A, B]
Push(C) → [A, B, C]        ← C is on TOP

Pop()   → removes C → [A, B]   (C was last in, so first out)
Pop()   → removes B → [A]
```

## 💻 Real-World Use Cases

- **Undo/Redo functionality**: the most recent action is the first one undone.
- **Function call stack**: how programming languages themselves track nested method calls — the most recently called method returns first.
- **Depth-First Search (DFS)**: exploring a graph/tree by going as deep as possible before backtracking — relies fundamentally on a stack (explicit or via recursion, which uses the call stack implicitly).
- **Balanced parentheses/expression validation**: matching brackets by pushing openers and popping on closers.
- **Browser back button**: navigating "back" pops the most recently visited page.

```csharp
// Classic use: validating balanced parentheses
bool IsBalanced(string expression)
{
    var stack = new Stack<char>();
  
    foreach (char c in expression)
    {
        if (c == '(') stack.Push(c);
        else if (c == ')')
        {
            if (stack.Count == 0) return false;   // closing without a matching opener
            stack.Pop();
        }
    }
  
    return stack.Count == 0;   // balanced only if every opener was matched
}

IsBalanced("(a(b)c)");   // true
IsBalanced("(a(b)c");    // false — unmatched opener
```

## 📊 Time Complexity

| Operation | Complexity     |
| --------- | -------------- |
| `Push`  | O(1) amortized |
| `Pop`   | O(1)           |
| `Peek`  | O(1)           |

Internally backed by an array (similar growth strategy to `List<T>` — doubling on resize).

## 📊 `Stack<T>` (LIFO) vs `Queue<T>` (FIFO) — Quick Comparison

| Aspect           | `Stack<T>`               | `Queue<T>`               |
| ---------------- | -------------------------- | -------------------------- |
| Order            | Last In, First Out         | First In, First Out        |
| Add              | `Push()`                 | `Enqueue()`              |
| Remove           | `Pop()` (from top)       | `Dequeue()` (from front) |
| Classic use case | DFS, undo/redo, call stack | BFS, task processing       |

## 🚨 Common Mistakes

- ❌ Calling `Pop()` or `Peek()` on an **empty stack** — throws `InvalidOperationException`. Check `Count > 0` first, or use `TryPop()`/`TryPeek()`.
- ❌ Confusing `Stack<T>` (LIFO) with `Queue<T>` (FIFO) — a very common mix-up; remember: stack = plates (top only), queue = line (front and back).
- ❌ Assuming `Stack<T>` supports indexed access — it does **not**; only the top is directly accessible.
- ❌ Using recursion for very deep problems without realizing recursion itself uses an implicit call stack — deep enough recursion can cause a `StackOverflowException`; an explicit `Stack<T>`-based iterative approach avoids that limit.

## 💡 Best Practices

- Use `Stack<T>` whenever the most **recently added** item needs to be processed first — undo operations, DFS, expression parsing/validation.
- Prefer an **explicit** `Stack<T>` over deep recursion when processing very large/deep structures, to avoid stack overflow risk from the call stack itself.
- Use `TryPop()`/`TryPeek()` in code paths where the stack might legitimately be empty.

## 🎤 Interview Questions

1. What does LIFO mean, and how does `Stack<T>` implement it?
2. What's a classic algorithm that fundamentally relies on a stack (explicitly or via recursion)?
3. How would you validate balanced parentheses using a stack?
4. Why might you prefer an explicit `Stack<T>` over recursion for very deep traversal problems?

## 📝 30-second Revision Cheat Sheet

- `Stack<T>` = LIFO — Last In, First Out.
- `Push()` adds to the top; `Pop()` removes from the top.
- O(1) for `Push`/`Pop`/`Peek`.
- Classic use cases: **DFS**, undo/redo, call stack, balanced parentheses validation.
- Explicit stacks avoid `StackOverflowException` risk that deep recursion can hit.
