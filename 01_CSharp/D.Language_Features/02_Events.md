# 📢 Events

## 📌 What is it?

> An **event** is a special kind of delegate that implements the **Observer/Publish-Subscribe pattern** — it lets a class notify OTHER classes when something happens, without knowing who's listening.

---

## 🌍 Real-world analogy

Think of a **YouTube channel**. The channel (publisher) uploads a video and NOTIFIES all subscribers automatically. The channel doesn't need to know WHO subscribed or HOW MANY — it just broadcasts, and subscribers react.

---

## 🧠 Intuition — Events vs Plain Delegates

```
Delegate: anyone with a reference to it can INVOKE it directly (risky — external code could trigger it)
Event:    only the DECLARING class can invoke/raise it — others can only subscribe/unsubscribe
```

---

## 💻 Code Examples

**Basic:**

```csharp
public class OrderProcessor
{
    public event Action<string> OrderPlaced;   // event based on Action<T> delegate

    public void PlaceOrder(string orderId)
    {
        Console.WriteLine("Order processing...");
        OrderPlaced?.Invoke(orderId);   // raise the event — '?.' guards against no subscribers
    }
}

var processor = new OrderProcessor();
processor.OrderPlaced += id => Console.WriteLine($"Email: Order {id} confirmed");   // subscribe
processor.OrderPlaced += id => Console.WriteLine($"SMS: Order {id} confirmed");     // subscribe another

processor.PlaceOrder("ORD123");
// Output:
// Order processing...
// Email: Order ORD123 confirmed
// SMS: Order ORD123 confirmed
```

**Intermediate — the standard .NET event pattern (EventHandler):**

```csharp
public class OrderEventArgs : EventArgs
{
    public string OrderId { get; set; }
}

public class OrderProcessor
{
    public event EventHandler<OrderEventArgs> OrderPlaced;

    public void PlaceOrder(string orderId)
    {
        OrderPlaced?.Invoke(this, new OrderEventArgs { OrderId = orderId });
    }
}

var processor = new OrderProcessor();
processor.OrderPlaced += (sender, e) => Console.WriteLine($"Notified: {e.OrderId}");
processor.PlaceOrder("ORD456");
```

**Practical — real-world UI/frontend-adjacent scenario (event-driven pattern you'll recognize from JS/jQuery too):**

```csharp
public class InventoryManager
{
    public event Action<string> LowStockAlert;

    private int stock = 10;

    public void SellItem(string itemName, int quantity)
    {
        stock -= quantity;
        if (stock < 5)
            LowStockAlert?.Invoke(itemName);
    }
}

var inventory = new InventoryManager();
inventory.LowStockAlert += item => Console.WriteLine($"⚠️ Low stock warning: {item}");
inventory.SellItem("Laptop", 7);   // triggers the alert
```

---

## 📊 Delegate vs Event — Key Difference

| Aspect             | Delegate                                             | Event                                                              |
| ------------------ | ---------------------------------------------------- | ------------------------------------------------------------------ |
| Who can invoke it? | Anyone with a reference                              | Only the DECLARING class                                           |
| Who can subscribe? | Anyone                                               | Anyone (via`+=`)                                                 |
| Encapsulation      | Weak — external code could reset/invoke it directly | Strong — external code can only subscribe/unsubscribe, not invoke |

---

## 🚨 Common Mistakes

- ❌ Forgetting the `?.Invoke()` null-check — raising an event with zero subscribers throws `NullReferenceException` without it
- ❌ Not unsubscribing from events when an object is no longer needed — causes **memory leaks** (the publisher holds a reference to the subscriber, keeping it alive) — see `J.Memory_and_Runtime/05_Memory_Leaks_in_NET.md`
- ❌ Using a plain delegate (public field) instead of `event` when external invocation should be prevented

---

## 🎤 Interview Questions

| Question                                               | Key Point                                                                                                        |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| What's the difference between a delegate and an event? | An event restricts invocation to the declaring class; a plain delegate can be invoked by anyone with a reference |
| Why use`?.Invoke()` when raising an event?           | Prevents`NullReferenceException` if there are no subscribers                                                   |
| What's a common memory issue with events?              | Forgetting to unsubscribe causes the publisher to keep the subscriber alive, leading to memory leaks             |

---

## 📝 30-Second Revision Cheat Sheet

- Event = restricted delegate — only declaring class can raise/invoke it
- Publisher/Subscriber pattern — `+=` to subscribe, `-=` to unsubscribe
- Always use `?.Invoke()` when raising an event (null-safety)
- Unsubscribe when done to avoid memory leak

# Events

---

## 📌 Overview

> Write your notes here.

---

## 🔑 Key Concepts

---

## 💻 Code Example

```csharp
```

---

## ❓ Interview Questions

---

## 🔗 Related Topics

---

*Last updated: 2026-03-15*
