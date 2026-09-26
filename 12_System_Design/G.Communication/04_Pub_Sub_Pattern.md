# 04_Pub_Sub_Pattern

> **Publish/Subscribe (Pub/Sub)** = a messaging pattern where **Publishers** send messages to a **Topic** without knowing who's listening, and **Subscribers** receive messages from topics they've chosen, without knowing who published them.

> This is the exact mechanism referenced throughout `02_Message_Queues.md` and `03_Event_Driven_Architecture.md` — this chapter formalizes it as its own named pattern with its own vocabulary (Topic, Publisher, Subscriber) and precisely distinguishes it from plain point-to-point queuing.

## 📌 What is it?

Pub/Sub introduces a middle concept — the **Topic** — that decouples publishers from subscribers completely. Neither side has a direct reference to the other; they only both know about the topic's name/address.

```
Publisher(s)              Topic               Subscriber(s)
     │                      │                       │
     │──── publish ────────►│                       │
     │                      │──── deliver copy ────►│ Subscriber A
     │                      │──── deliver copy ────►│ Subscriber B
     │                      │──── deliver copy ────►│ Subscriber C
     │                      │                       │

Publisher knows: "I publish to topic 'order-events'"
Subscriber knows: "I subscribe to topic 'order-events'"
NEITHER knows the other exists.
```

## 🤔 Why do we need it?

- Complete **decoupling** — publishers and subscribers can be added, removed, deployed, or scaled entirely independently.
- One message → **many** independent recipients, without the publisher writing any fan-out logic itself.
- New subscribers can start listening at any time **without requiring any code change on the publisher side** — exactly what made `03_Event_Driven_Architecture.md`'s "add SmsAlertEventHandler with zero OrderService changes" example possible.

## 🌍 Real-world analogy

A **radio broadcast**. The radio station (publisher) broadcasts on a specific frequency (topic) without knowing how many people are tuned in, or who they are. Anyone with a radio (subscriber) can tune into that frequency and receive the broadcast — and the station's broadcast doesn't change one bit whether 10 people or 10 million are listening.

## 🚨 Pub/Sub (Topic) vs Plain Queue (Point-to-Point) — the precise distinction

> This exact comparison table appeared briefly in `02_Message_Queues.md` — here it is fully explained, since confusing the two is a very common interview stumble.

| Aspect                     | **Queue (Point-to-Point)**                                                                        | **Pub/Sub (Topic)**                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Who receives each message? | Exactly**ONE** consumer                                                                           | **EVERY** subscriber gets a copy                                       |
| Real-world example         | A single task ("charge this credit card") — must happen exactly once                                   | A broadcast fact ("OrderPlaced") — many parts of the system care            |
| If you add a 2nd listener  | Messages get split between the two (competing consumers) — each message still goes to only ONE of them | The new listener gets a full independent copy of every message going forward |
| Typical use case           | Distributing work/tasks across a worker pool                                                            | Notifying multiple independent services about the same event                 |

```
QUEUE (Point-to-Point):                 TOPIC (Pub/Sub):

Producer → [Queue] ─┬─► Worker 1        Publisher → [Topic] ─┬─► Subscriber A (gets EVERY message)
                     ├─► Worker 2  (each                     ├─► Subscriber B (gets EVERY message)
                     └─► Worker 3   message goes              └─► Subscriber C (gets EVERY message)
                          to only ONE
                          worker — load
                          is SPLIT)
```

## ⚙️ Internal working — how a Topic tracks subscribers

```
1. Subscriber A calls: Subscribe("order-events")
2. Subscriber B calls: Subscribe("order-events")
   → Broker now maintains a subscriber list for "order-events": [A, B]

3. Publisher calls: Publish("order-events", { orderId: 501, ... })

4. Broker looks up subscriber list for "order-events" → [A, B]
5. Broker delivers an independent copy of the message to A AND to B

6. Later, Subscriber C calls: Subscribe("order-events")
   → Subscriber list becomes [A, B, C]
   → Future publishes now also reach C — PAST messages are NOT retroactively delivered
     (unless the broker supports message replay/retention, e.g., Kafka)
```

## 📊 Delivery Guarantees (a subtle but important detail)

| Guarantee               | Meaning                                                                  | Trade-off                                                                      |
| ----------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| **At-most-once**  | Message delivered 0 or 1 times — might be lost                          | Fastest, but risk of silent message loss                                       |
| **At-least-once** | Message delivered 1 or more times — never lost, but could be duplicated | Most common default — requires subscribers to handle duplicates (idempotency) |
| **Exactly-once**  | Message delivered exactly 1 time, no loss, no duplicates                 | Ideal, but hardest/most expensive to implement correctly                       |

> Because **at-least-once** is the most common real-world guarantee, subscriber logic should generally be **idempotent** — processing the same message twice should have the same effect as processing it once (e.g., check "have I already sent this confirmation email?" before sending).

## 💻 Code examples

### Basic — publisher, unaware of any subscriber

```csharp
public class OrderService
{
    private readonly ITopicPublisher _publisher; // e.g., wraps Azure Service Bus Topic, or a Kafka producer

    public void PlaceOrder(Order order)
    {
        _dal.InsertOrder(order);

        // Publish to the TOPIC — the publisher has no idea who (if anyone) is subscribed
        _publisher.Publish("order-events", new OrderPlacedEvent
        {
            OrderId = order.Id,
            UserId = order.UserId
        });
    }
}
```

### Intermediate — two independent subscribers to the same topic

```csharp
// Subscriber registration (each service does this independently, at startup)
_subscriber.Subscribe("order-events", async (OrderPlacedEvent evt) =>
{
    await _emailService.SendConfirmationAsync(evt.OrderId); // Email service's own logic
});
```

```csharp
// A COMPLETELY separate service/codebase, subscribing to the SAME topic
_subscriber.Subscribe("order-events", async (OrderPlacedEvent evt) =>
{
    await _inventoryService.DecrementStockAsync(evt.OrderId); // Inventory service's own logic
});
```

### Practical — idempotent handling for "at-least-once" delivery

```csharp
public async Task HandleAsync(OrderPlacedEvent evt)
{
    // Guard against duplicate delivery (at-least-once semantics)
    if (await _emailService.WasConfirmationAlreadySentAsync(evt.OrderId))
    {
        return; // already handled — safely ignore the duplicate
    }

    await _emailService.SendConfirmationAsync(evt.OrderId);
    await _emailService.MarkConfirmationSentAsync(evt.OrderId);
}
```

## ⚡ Performance considerations

- Pub/Sub scales fan-out extremely well — publishing once and having the broker handle delivery to N subscribers is far more efficient than the publisher looping and calling each one directly.
- Adding subscribers has **no performance cost to the publisher** — the broker absorbs the extra delivery work, not the publisher's request path.
- Message replay/retention (e.g., Kafka's log-based storage) allows a **new** subscriber to optionally reprocess historical messages — not all Pub/Sub systems support this (many, like simple broker topics, only deliver messages published *after* you subscribe).

## 🚨 Common mistakes

- ❌ Confusing Pub/Sub (every subscriber gets every message) with a plain Queue (each message goes to only one consumer) — a very common interview and design mistake.
- ❌ Assuming "exactly-once" delivery when the broker only guarantees "at-least-once" — leads to bugs like duplicate emails/charges if handlers aren't idempotent.
- ❌ Expecting a new subscriber to automatically receive messages published *before* it subscribed — most brokers only deliver going forward, unless message replay is explicitly supported and configured.

## 💡 Best practices

- ✅ Use a Topic (Pub/Sub) when a message represents a **fact/event** multiple independent parties care about; use a Queue when a message represents a **task** that should be done exactly once by exactly one worker.
- ✅ Design all subscriber handlers to be **idempotent** — assume "at-least-once" delivery unless you've specifically verified and configured exactly-once semantics.
- ✅ Document topic names and event schemas centrally — since publishers and subscribers never talk directly, the topic/schema definition is the only real contract between them.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                               | Answer                                                                                                                             |
| -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| What's the core difference between a Queue and a Topic (Pub/Sub)?                      | A Queue delivers each message to exactly one consumer; a Topic delivers a copy of each message to every subscriber                 |
| Why does adding a new subscriber require no changes to the publisher?                  | The publisher only knows about the topic, never about individual subscribers — full decoupling                                    |
| What does "at-least-once" delivery mean, and what must subscribers do about it?        | A message may be delivered more than once; subscribers should be idempotent to handle duplicates safely                            |
| Does a brand-new subscriber typically receive messages published before it subscribed? | No, generally not — unless the broker specifically supports message replay/retention (e.g., Kafka)                                |
| Give a real-world example that fits Pub/Sub better than a plain Queue.                 | Notifying Email, Inventory, and Analytics services about the same "OrderPlaced" event — all three need their own independent copy |

## 📝 30-second Revision Cheat Sheet

- Pub/Sub = Publishers send to a Topic; every Subscriber gets its own copy — full decoupling, no direct knowledge of each other.
- Queue (Point-to-Point) ≠ Topic (Pub/Sub): Queue = one consumer per message; Topic = every subscriber per message.
- Delivery guarantees: at-most-once (may lose), at-least-once (may duplicate — most common), exactly-once (ideal, hardest).
- Design subscriber handlers to be **idempotent** — assume at-least-once delivery.
- New subscribers usually only get FUTURE messages, not past ones, unless replay is supported.

---

✅ **G.Communication chapter complete** (01–04). Next up: **H.Architecture_Patterns**.
