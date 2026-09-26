# 05_Event_Sourcing

> **Event Sourcing** = instead of storing only the **current state** of an entity, you store the **full sequence of events** that led to that state — the current state is then derived by replaying those events.

> The natural companion to `04_CQRS.md` (they're frequently used together) and a deeper application of `03_Event_Driven_Architecture.md`'s "events" concept — here, events aren't just used for notifying other services, they become the **primary source of truth** itself.

## 📌 What is it?

In a normal (CRUD) system, an `UPDATE` statement **overwrites** the previous value — the history is gone. Event Sourcing instead **never overwrites**; every change is stored as a new, immutable event, appended to a log. The "current state" is just a computed result — replay all events in order, and you get the current value.

```
Traditional CRUD (state-based):              Event Sourcing (event-based):

Products table:                              Events table (append-only, immutable):
┌────┬────────┬────────┐                     ┌────┬─────────────────────┬───────────┐
│ Id │ Name   │ Price  │                     │ Id │ EventType            │ Data      │
├────┼────────┼────────┤                     ├────┼─────────────────────┼───────────┤
│  5 │ Widget │ 49.99  │  ← ONLY the CURRENT  │  1 │ ProductCreated       │{price:39} │
└────┴────────┴────────┘     value survives  │  2 │ ProductPriceChanged  │{price:45} │
   (history of how it got                    │  3 │ ProductPriceChanged  │{price:49.99}│
    here is LOST)                            └────┴─────────────────────┴───────────┘
                                                 (FULL HISTORY preserved — current
                                                  state = replay all 3 events)
```

## 🤔 Why do we need it?

| Limitation of state-based storage                                                   | How Event Sourcing helps                                                      |
| ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| No history — you only know the current price, not how/why it changed               | Full audit trail is built in — every change is a permanent, queryable record |
| "What did this record look like last Tuesday?" requires separate audit tables/hacks | Just replay events up to that point in time — trivial by design              |
| Debugging "how did we end up in this bad state?" is hard with only the final value  | You can literally replay the exact sequence of events that led there          |
| Undo/rollback is awkward                                                            | Often as simple as not applying (or compensating for) a later event           |

## 🌍 Real-world analogy

A **bank account ledger** (this is actually where the pattern gets its name/intuition). A bank doesn't just store "current balance = $500" and overwrite it on every transaction. It stores every deposit and withdrawal as a permanent line item. The current balance is simply the **sum of all these events** — and if you ever need to know the balance as of last month, you just don't sum events past that date.

## ⚙️ Internal working

```
1. A command comes in: ChangePrice(productId=5, newPrice=45.00)

2. Instead of UPDATE Products SET Price = 45.00 WHERE Id = 5 ...
   ... an EVENT is appended:

   { eventType: "ProductPriceChanged", productId: 5, newPrice: 45.00, timestamp: ... }

3. This event is stored PERMANENTLY, immutably, in an append-only Event Store

4. To get the CURRENT state of Product #5:
   - Fetch ALL events for productId=5, in order
   - "Replay" them one by one, applying each to build up the current object

     ProductCreated      → { Id: 5, Name: "Widget", Price: 39.00 }
     ProductPriceChanged → { Id: 5, Name: "Widget", Price: 45.00 }
     ProductPriceChanged → { Id: 5, Name: "Widget", Price: 49.99 }

     Final replayed state → { Id: 5, Name: "Widget", Price: 49.99 }  ← matches current reality
```

## 📊 Event Sourcing + CQRS — how they typically pair up

| Component              | Role                                                                                 |
| ---------------------- | ------------------------------------------------------------------------------------ |
| **Command side** | Appends new events to the Event Store (the write model, from`04_CQRS.md`)          |
| **Event Store**  | The append-only log of ALL events — the single source of truth                      |
| **Projection**   | A process that replays events to build a fast, queryable "current state" view        |
| **Query side**   | Reads from the Projection (denormalized, fast) — never replays events on every read |

```
Command ──► Event Store (append-only, source of truth)
                  │
                  │  events replayed/projected
                  ▼
            Read Model / Projection  ◄──── Query side reads THIS (fast, no replay needed per query)
```

> This is exactly why the two patterns are so often paired: Event Sourcing solves "what's the source of truth," and CQRS solves "how do I query it efficiently without replaying history on every single read."

## 🖼 Rebuilding state — the real power (and cost)

```
GOOD: Need a NEW read model shape you didn't originally plan for?
      → Just write a new Projection and replay ALL historical events through it.
      → No data is lost, because you never overwrote anything.

COST: If an entity has 100,000 events, replaying ALL of them on every
      state-rebuild is slow.
      → Fix: periodic SNAPSHOTS (save the computed state every N events,
        so replay only needs to start from the last snapshot forward)
```

## 💻 Code examples

### Basic — defining events and an append-only store

```csharp
// Events are immutable facts — named in past tense, never changed after creation
public abstract record ProductEvent(int ProductId, DateTime Timestamp);
public record ProductCreated(int ProductId, string Name, decimal InitialPrice, DateTime Timestamp) : ProductEvent(ProductId, Timestamp);
public record ProductPriceChanged(int ProductId, decimal NewPrice, DateTime Timestamp) : ProductEvent(ProductId, Timestamp);

public class EventStore
{
    private readonly EventStoreDAL _dal; // wraps an append-only table via ADO.NET

    public void Append(ProductEvent evt)
    {
        // INSERT ONLY — this table NEVER has UPDATE or DELETE statements run against it
        _dal.InsertEvent(evt.ProductId, evt.GetType().Name, JsonSerializer.Serialize(evt), evt.Timestamp);
    }

    public List<ProductEvent> GetEventsForProduct(int productId)
    {
        return _dal.GetEventsByProductId(productId); // ordered by timestamp/sequence
    }
}
```

### Intermediate — rebuilding current state by replaying events

```csharp
public class Product
{
    public int Id { get; private set; }
    public string Name { get; private set; } = "";
    public decimal Price { get; private set; }

    // Replay events, one at a time, to build up current state
    public static Product ReplayFrom(List<ProductEvent> events)
    {
        var product = new Product();

        foreach (var evt in events)
        {
            switch (evt)
            {
                case ProductCreated created:
                    product.Id = created.ProductId;
                    product.Name = created.Name;
                    product.Price = created.InitialPrice;
                    break;

                case ProductPriceChanged priceChanged:
                    product.Price = priceChanged.NewPrice; // apply the change
                    break;
            }
        }

        return product; // this is the CURRENT state, derived entirely from history
    }
}
```

### Practical — handling a command, appending an event, updating the projection

```csharp
public class ChangePriceCommandHandler
{
    private readonly EventStore _eventStore;
    private readonly ProductProjectionDAL _projectionDal; // the fast, queryable read table

    public void Handle(int productId, decimal newPrice)
    {
        // 1. Append the event — this is the PERMANENT record of what happened
        _eventStore.Append(new ProductPriceChanged(productId, newPrice, DateTime.UtcNow));

        // 2. Update the projection (read model) so queries stay fast
        //    — doesn't need to replay ALL history on every write, just apply this one change
        _projectionDal.UpdateCurrentPrice(productId, newPrice);
    }
}
```

## ⚡ Performance considerations

- Replaying thousands of events on every read would be far too slow — this is why Event Sourcing is almost always paired with **Projections** (materialized, pre-computed read views) rather than replaying on demand.
- **Snapshots** (periodically saving computed state) prevent replay time from growing unbounded as an entity accumulates history.
- The Event Store itself is append-only, which is actually a very **write-friendly** pattern (no locking/contention from in-place updates) — but it does grow indefinitely, so storage planning matters.

## 🚨 Common mistakes

- ❌ Using Event Sourcing for simple entities with no real need for history/audit — it's significant added complexity for little benefit in a basic CRUD scenario.
- ❌ Replaying full event history on every single read instead of maintaining a projection — kills performance as history grows.
- ❌ Ever mutating or deleting a past event — the entire model depends on events being permanent, immutable facts; "fixing" history should be done via a new **compensating event**, not by editing the past.

## 💡 Best practices

- ✅ Reserve Event Sourcing for domains where **audit trail, history, or replay/what-if analysis genuinely matter** — financial ledgers, inventory changes, order lifecycles, compliance-heavy domains.
- ✅ Always pair it with a Projection (and CQRS's query side) — never require replaying full history for routine reads.
- ✅ Use snapshots once an entity's event count grows large, to bound replay time.
- ✅ Treat events as immutable, permanent facts — never edit or delete them; correct mistakes with new, compensating events instead.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                 | Answer                                                                                                                        |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| What's the core idea of Event Sourcing?                                  | Store the full sequence of events that happened, rather than only the current state; derive current state by replaying events |
| How do you get the "current state" of an entity?                         | Replay all its events, in order, applying each one to build up the final state                                                |
| Why is Event Sourcing almost always paired with a Projection/read model? | Replaying full event history on every read would be far too slow; a projection keeps a fast, pre-computed current view        |
| What is a "snapshot," and why is it needed?                              | A periodically saved computed state, so replay only needs to start from the last snapshot instead of from the very beginning  |
| What must NEVER happen to a stored event?                                | It must never be edited or deleted — corrections are made via new, compensating events, preserving full historical integrity |

## 📝 30-second Revision Cheat Sheet

- Event Sourcing = store the full history of events, not just current state; replay events to get current state.
- Events are immutable, append-only, permanent facts — never edited or deleted.
- Almost always paired with CQRS: events are the write side's source of truth; a Projection serves fast reads.
- Snapshots prevent replay time from growing unbounded as history accumulates.
- Best fit: audit-heavy domains (finance, inventory, order lifecycles) — overkill for simple CRUD.

---

✅ **H.Architecture_Patterns chapter complete** (01–05). Next up: **I.API_Design**.
