# 02_Message_Queues

> **Message Queue** = a middleman service that lets one part of a system send a message **now** and let another part process it **later**, without the sender waiting for the receiver to be ready.

> Builds on `01_REST_vs_gRPC.md` — both were **synchronous**: caller waits for a direct response. A Message Queue introduces **asynchronous** communication: fire-and-forget, decoupled in time.

## 📌 What is it?

A queue sits between a **Producer** (sends messages) and a **Consumer** (processes messages). The producer drops a message in the queue and moves on immediately — it doesn't wait for the consumer to actually do the work.

```
Producer ──► [ Message Queue ] ──► Consumer
  (fast,        (holds messages       (processes at its
   fire-and-      until picked up)      own pace)
   forget)
```

## 🤔 Why do we need it?

| Problem with direct/synchronous calls                                                       | How a Message Queue helps                                                 |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Caller must wait for the whole operation to finish (e.g., sending an email takes 3 seconds) | Caller queues the "send email" task and returns instantly                 |
| A slow/down consumer blocks the producer                                                    | Producer keeps working; queue holds messages until consumer catches up    |
| Sudden traffic spike overwhelms a downstream service                                        | Queue absorbs the burst; consumer processes at a steady, sustainable rate |
| Producer and consumer must both be available at the same time                               | Queue decouples them — consumer can even be offline temporarily          |

## 🌍 Real-world analogy

A **restaurant order ticket rail**. The waiter (producer) writes an order and clips it to the rail — they don't stand at the kitchen window watching it cook. The kitchen (consumer) works through tickets at its own pace, in order. If the kitchen gets slammed, tickets just wait on the rail — the waiter isn't blocked and can keep taking new orders.

## ⚙️ Internal working

```
1. Producer creates a message: { "action": "SendWelcomeEmail", "userId": 42 }
2. Producer pushes it to the queue → gets an immediate ACK (message accepted)
3. Producer's job is DONE — it moves on to other work

4. Consumer(s) continuously poll/listen to the queue
5. Consumer picks up the next message
6. Consumer processes it (sends the actual email)
7. Consumer acknowledges completion → message removed from queue
   (if consumer crashes before ACK, message goes back for another consumer to retry)
```

```
        push                    pull
Producer ───► ┌──────────────┐ ◄─── Consumer 1
              │ Message Queue │ ◄─── Consumer 2   (multiple consumers = parallel processing)
              │  [msg][msg]   │ ◄─── Consumer 3
              └──────────────┘
```

## 📊 Synchronous (REST/gRPC) vs Asynchronous (Message Queue)

| Aspect                   | Synchronous (REST/gRPC)                                 | Asynchronous (Message Queue)                                                       |
| ------------------------ | ------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Caller waits for result? | Yes                                                     | No — fire-and-forget                                                              |
| Coupling in time         | Both sides must be available simultaneously             | Decoupled — consumer can process later                                            |
| Best for                 | Needing an immediate answer (e.g., "get product price") | Background work (emails, notifications, image processing, order fulfillment steps) |
| Failure handling         | Caller sees the error immediately                       | Message can be retried/requeued automatically                                      |

## 📊 Queue delivery patterns

| Pattern                             | Description                                               | Example use                                                                                                                                                                            |
| ----------------------------------- | --------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Point-to-Point (Queue)**    | Each message is consumed by exactly**one** consumer | Order processing — only one worker should charge the card once                                                                                                                        |
| **Publish/Subscribe (Topic)** | Each message is delivered to**every** subscriber    | "OrderPlaced" event notifying Email service, Inventory service, and Analytics service all at once — covered fully in`03_Event_Driven_Architecture.md` and `04_Pub_Sub_Pattern.md` |

```
Point-to-Point:                    Publish/Subscribe:

Producer → [Queue] → Consumer A    Producer → [Topic] ─┬─► Subscriber A
                      (only ONE                        ├─► Subscriber B
                       consumer gets                   └─► Subscriber C
                       each message)                  (ALL subscribers get every message)
```

## 💻 Code examples

### Basic — conceptual producer (enqueue a background job)

```csharp
public class OrderController : ControllerBase
{
    private readonly IMessageQueueClient _queue;

    [HttpPost]
    public IActionResult PlaceOrder(OrderViewModel model)
    {
        // 1. Save the order synchronously (must be immediately consistent)
        int orderId = _orderService.CreateOrder(model);

        // 2. Queue the SLOW, non-critical follow-up work — don't make the user wait for these
        _queue.Enqueue(new SendOrderConfirmationEmailMessage { OrderId = orderId });
        _queue.Enqueue(new UpdateInventoryMessage { OrderId = orderId });

        // 3. Respond to the user IMMEDIATELY — the queue handles the rest in the background
        return Ok(new { orderId, status = "Order placed!" });
    }
}
```

### Intermediate — a background consumer processing the queue

```csharp
public class EmailQueueConsumer : BackgroundService
{
    private readonly IMessageQueueClient _queue;
    private readonly EmailService _emailService;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var message = await _queue.DequeueAsync<SendOrderConfirmationEmailMessage>(stoppingToken);

            if (message != null)
            {
                try
                {
                    await _emailService.SendOrderConfirmationAsync(message.OrderId);
                    await _queue.AcknowledgeAsync(message); // success — remove from queue
                }
                catch (Exception ex)
                {
                    // Don't acknowledge — message becomes available again for a retry
                    // (ties into Retry patterns from J.Reliability_Patterns later)
                    LogError(ex);
                }
            }
        }
    }
}
```

### Practical — using a real message broker (RabbitMQ-style, conceptual)

```csharp
// Producer side
var message = JsonSerializer.Serialize(new { OrderId = orderId, Action = "SendConfirmation" });
var body = Encoding.UTF8.GetBytes(message);

channel.BasicPublish(exchange: "", routingKey: "email-queue", basicProperties: null, body: body);

// Consumer side
var consumer = new EventingBasicConsumer(channel);
consumer.Received += (model, ea) =>
{
    var body = ea.Body.ToArray();
    var message = Encoding.UTF8.GetString(body);
    ProcessEmailMessage(message);

    channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false); // acknowledge success
};
channel.BasicConsume(queue: "email-queue", autoAck: false, consumer: consumer);
```

## ⚡ Performance considerations

- Message queues let you **absorb traffic spikes** — 10,000 orders in a burst can all be queued instantly; consumers process them at a sustainable rate instead of the system falling over.
- **Multiple consumers** can process the same queue in parallel to increase throughput — similar in spirit to adding read replicas, but for background work.
- Adds a small amount of **end-to-end latency** for the queued work (it's no longer instant) — acceptable for background tasks, not for things the user is actively waiting to see.

## 🚨 Common mistakes

- ❌ Queuing something the user needs to see the result of immediately (e.g., "is this coupon code valid?") — that belongs in a synchronous REST/gRPC call, not a queue.
- ❌ Not handling failed messages — without retry/dead-letter handling, a message that fails processing can silently vanish or endlessly retry forever.
- ❌ Assuming message order is always guaranteed — many queue systems don't guarantee strict ordering by default unless specifically configured (e.g., single-partition ordering in Kafka).

## 💡 Best practices

- ✅ Use queues for anything that's **slow, non-critical-path, or can tolerate a short delay**: emails, notifications, report generation, image/video processing, inventory updates.
- ✅ Always acknowledge messages only **after** successful processing — never before, or a crash mid-processing silently loses work.
- ✅ Set up a **dead-letter queue** for messages that repeatedly fail, so they don't loop forever or get silently dropped.
- ✅ Keep the synchronous, user-facing path fast — save the queue for everything that can happen "after" the user gets their response.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                         | Answer                                                                                                                           |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| What's the core difference between a synchronous call and using a message queue? | Synchronous: caller waits for the result. Queue: caller fires the message and moves on; processing happens later, asynchronously |
| Why are message queues good for handling traffic spikes?                         | They absorb bursts of incoming messages and let consumers process them at a sustainable, controlled rate                         |
| What's the difference between Point-to-Point and Publish/Subscribe delivery?     | Point-to-Point: exactly one consumer gets each message. Pub/Sub: every subscriber gets a copy of each message                    |
| What happens if a consumer crashes before acknowledging a message?               | The message isn't removed from the queue — it becomes available again for another consumer to process (retry)                   |
| Give two real-world examples of tasks well-suited to a message queue.            | Sending confirmation emails, updating inventory after an order, generating reports, processing uploaded images/videos            |

## 📝 30-second Revision Cheat Sheet

- Message Queue = producer sends a message and moves on; consumer processes it later, asynchronously.
- Solves: slow tasks blocking the caller, traffic spikes overwhelming downstream services.
- Two patterns: Point-to-Point (one consumer per message) vs Publish/Subscribe (all subscribers get every message).
- Only acknowledge a message AFTER successful processing — crash-safety depends on this.
- Use for background/non-critical-path work: emails, notifications, inventory updates, media processing.
- Keep the user-facing synchronous path fast; defer everything else to the queue.
