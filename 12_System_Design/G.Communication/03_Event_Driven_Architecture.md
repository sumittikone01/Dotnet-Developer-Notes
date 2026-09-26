# 03_Event_Driven_Architecture

> **Event-Driven Architecture (EDA)** = a design style where services communicate by broadcasting **events** ("something happened") rather than directly calling each other — interested services react independently, whenever they choose.

> Builds directly on `02_Message_Queues.md`'s Publish/Subscribe pattern. Where a Message Queue is a *tool*, Event-Driven Architecture is the *overall design philosophy* built using that tool (and pub/sub specifically).

## 📌 What is it?

Instead of Service A **directly calling** Service B, C, and D to tell each one what happened, Service A simply **publishes an event** — "OrderPlaced" — and any service that cares can **subscribe** and react. Service A doesn't know or care who's listening.

```
Traditional (direct calls):              Event-Driven:

OrderService                              OrderService
   │  calls Email                             │  publishes "OrderPlaced" event
   │  calls Inventory                         ▼
   │  calls Analytics                    [ Event Bus / Broker ]
   │  calls Shipping                        │    │    │    │
   (OrderService KNOWS about               ▼    ▼    ▼    ▼
    and depends on all 4)              Email  Inv.  Analytics  Shipping
                                        (each independently reacts;
                                         OrderService knows NONE of them)
```

## 🤔 Why do we need it?

| Problem with direct service-to-service calls                                                         | How EDA helps                                                                              |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| OrderService must know about (and call) every interested service                                     | OrderService just publishes one event — total decoupling                                  |
| Adding a new interested service (e.g., a new "SendSmsAlert" feature) requires modifying OrderService | New service just subscribes to the existing event — zero changes to OrderService          |
| If one downstream service (e.g., Analytics) is slow/down, it can block the whole order flow          | Each subscriber processes independently — a slow Analytics service doesn't block Shipping |
| Tight coupling makes services hard to change independently                                           | Services only need to agree on the**event shape**, not on each other's internals     |

## 🌍 Real-world analogy

A **fire alarm system**. When smoke is detected, the alarm doesn't personally call the fire department, the building occupants, and the sprinkler system one by one. It just **sounds the alarm** (publishes an event) — the fire department, the occupants, and the sprinklers each independently react to that same signal, without the alarm needing to know any of them exist.

## ⚙️ Internal working

```
1. Something happens: a customer places an order
2. OrderService publishes an event to an Event Bus/Broker:

   {
     "eventType": "OrderPlaced",
     "orderId": 501,
     "userId": 42,
     "amount": 129.99,
     "timestamp": "2026-09-26T10:15:00Z"
   }

3. The Event Bus delivers a COPY of this event to every subscribed service:

   EmailService      → sends order confirmation email
   InventoryService  → decrements stock levels
   AnalyticsService  → records the sale for reporting
   ShippingService   → schedules a shipment

4. Each subscriber processes the event INDEPENDENTLY, at its own pace,
   with its own success/failure/retry handling — completely isolated from the others.
```

## 📊 Direct Calls vs Event-Driven — full comparison

| Aspect                            | Direct Service Calls                                  | Event-Driven Architecture                                                |
| --------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------ |
| Coupling                          | Tight — caller knows every callee                    | Loose — publisher knows nothing about subscribers                       |
| Adding a new reaction to an event | Requires modifying the publisher's code               | Just add a new subscriber — publisher untouched                         |
| Failure isolation                 | One failing callee can affect the whole chain         | Each subscriber fails/retries independently                              |
| Communication style               | Usually synchronous (REST/gRPC)                       | Usually asynchronous (via a Message Queue/Broker)                        |
| Complexity                        | Simpler to trace/debug (linear call chain)            | Harder to trace — "who's listening to this event?" isn't always obvious |
| Best for                          | Simple systems, few services, need immediate response | Systems with many independent reactions to the same business event       |

## 🖼 Event Notification vs Event-Carried State Transfer

| Style                                  | What the event contains                                    | Trade-off                                                                                                                         |
| -------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Event Notification**           | Just the fact + minimal info (e.g.,`{ "orderId": 501 }`) | Subscribers must call back for details → some coupling remains, but events stay small                                            |
| **Event-Carried State Transfer** | The full relevant data (e.g., entire order object)         | Subscribers have everything they need immediately, no callback — but events are larger and can go stale if not consumed promptly |

## 💻 Code examples

### Basic — publishing a domain event (using a message queue under the hood)

```csharp
public class OrderService
{
    private readonly IEventPublisher _eventPublisher; // wraps a message broker (RabbitMQ/Kafka/Azure Service Bus)
    private readonly OrderDAL _dal;

    public int PlaceOrder(OrderViewModel model)
    {
        int orderId = _dal.InsertOrder(model); // 1. Persist via BAL → DAL → Stored Procedure

        // 2. Publish the event — OrderService has NO IDEA who (if anyone) is listening
        _eventPublisher.Publish(new OrderPlacedEvent
        {
            OrderId = orderId,
            UserId = model.UserId,
            Amount = model.Amount,
            Timestamp = DateTime.UtcNow
        });

        return orderId; // OrderService's job is done here
    }
}
```

### Intermediate — multiple independent subscribers reacting to the same event

```csharp
// Subscriber 1 — completely separate service/process
public class EmailEventHandler : IEventSubscriber<OrderPlacedEvent>
{
    public async Task HandleAsync(OrderPlacedEvent evt)
    {
        await _emailService.SendOrderConfirmationAsync(evt.UserId, evt.OrderId);
    }
}

// Subscriber 2 — also completely separate, knows nothing about EmailEventHandler
public class InventoryEventHandler : IEventSubscriber<OrderPlacedEvent>
{
    public async Task HandleAsync(OrderPlacedEvent evt)
    {
        await _inventoryService.DecrementStockForOrderAsync(evt.OrderId);
    }
}

// Subscriber 3 — added LATER, requires ZERO changes to OrderService
public class SmsAlertEventHandler : IEventSubscriber<OrderPlacedEvent>
{
    public async Task HandleAsync(OrderPlacedEvent evt)
    {
        await _smsService.SendOrderAlertAsync(evt.UserId, evt.OrderId);
    }
}
```

> Notice: adding `SmsAlertEventHandler` required **touching zero lines of `OrderService`** — this is the central benefit of EDA in practice.

## ⚡ Performance / design considerations

- EDA scales well for **fan-out** scenarios (one event, many independent reactions) — much cleaner than the publisher calling each one directly.
- Debugging is genuinely harder: tracing "what happened after this order was placed" now means checking multiple independent subscribers instead of reading one linear function call chain — Distributed Tracing (`L.Logging_and_Monitoring/03_Distributed_Tracing.md`) exists specifically to address this.
- Events are typically delivered via the Pub/Sub pattern (`04_Pub_Sub_Pattern.md`) on top of a message broker (Kafka, RabbitMQ, Azure Service Bus, AWS SNS/SQS).

## 🚨 Common mistakes

- ❌ Overusing EDA for simple, tightly-related operations where a direct call would be simpler and easier to trace/debug.
- ❌ Publishing overly large "Event-Carried State Transfer" events for data that changes frequently — subscribers may act on stale data if they process the event late.
- ❌ Not versioning event schemas — changing an event's shape without a versioning strategy can silently break every subscriber at once.

## 💡 Best practices

- ✅ Use EDA when a single business event genuinely needs to trigger **multiple, independent reactions** across different services.
- ✅ Keep events reasonably small and focused (lean toward Event Notification + a follow-up query, unless the full payload is genuinely needed everywhere).
- ✅ Version your event schemas from day one — treat them like a public API contract between services.
- ✅ Invest in distributed tracing/logging early — EDA's decoupling comes at the cost of harder end-to-end visibility.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                           | Answer                                                                                                                                        |
| ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the core idea of Event-Driven Architecture?                                 | Services publish events describing "what happened"; interested subscribers react independently, with the publisher unaware of who's listening |
| What's the main benefit over direct service-to-service calls?                      | Loose coupling — new subscribers can be added without any changes to the publisher                                                           |
| What's a real trade-off/downside of EDA?                                           | Harder to trace/debug the full flow of what happens after an event, since reactions are spread across independent subscribers                 |
| What's the difference between Event Notification and Event-Carried State Transfer? | Notification sends minimal info (subscriber must fetch details); State Transfer sends the full relevant data in the event itself              |
| What underlying technology typically delivers events in EDA?                       | A message broker using the Publish/Subscribe pattern (e.g., Kafka, RabbitMQ, Azure Service Bus)                                               |

## 📝 30-second Revision Cheat Sheet

- EDA = publish events ("X happened"); subscribers react independently — publisher doesn't know who's listening.
- Built on top of Message Queues' Publish/Subscribe pattern (`02_Message_Queues.md`).
- Big win: adding a new reaction to an event needs zero changes to the publisher.
- Big cost: harder to trace/debug the full end-to-end flow.
- Two event styles: Notification (minimal data) vs Event-Carried State Transfer (full data).
- Version your event schemas — they're a contract between independently-deployed services.
