# 💬 Case Study: Chat System (e.g. WhatsApp, Slack)

## 📌 What is it?

A system enabling **real-time messaging** between users — one-on-one or group chats — with message delivery, presence (online/offline), and history.

## 🤔 Requirements

### Functional

- Send/receive messages in real time.
- Support 1:1 and group chats.
- Message history/persistence.
- Delivery status (sent ✓ / delivered ✓✓ / read ✓✓ blue).
- Online/offline presence indicators.

### Non-Functional

- **Low latency** — messages should feel instant.
- **High availability** — chat is a core, always-on feature.
- **Consistency**: messages should never be lost, and ideally delivered in order.
- Massive scale — billions of messages/day for large apps.

## 🧠 Core Design Question: How do servers push messages to clients in real time?

HTTP is traditionally **request-response** — the server can't "push" data to a client unprompted. Chat needs the opposite: the server must notify a client the instant a new message arrives.

| Approach                           | How it works                                                                    | Tradeoff                                            |
| ---------------------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------- |
| **Polling**                  | Client repeatedly asks "any new messages?" every few seconds                    | Simple but wasteful, adds delay                     |
| **Long Polling**             | Client asks, server holds the request open until a message arrives (or timeout) | Better than polling, still has overhead per request |
| **WebSockets** ⭐            | Persistent, full-duplex connection between client and server                    | Industry standard for real-time chat                |
| **Server-Sent Events (SSE)** | Server pushes to client over a single long-lived HTTP connection                | One-directional only (server→client)               |

**WebSockets** are the standard choice — one persistent connection, low overhead, bidirectional.

## ⚙️ High-Level Architecture

```
┌────────┐   WebSocket    ┌───────────────────┐
│ User A │◀──────────────▶│  Chat Server 1     │
└────────┘                 └─────────┬─────────┘
                                       │
┌────────┐   WebSocket    ┌───────────┴─────────┐
│ User B │◀──────────────▶│  Chat Server 2       │
└────────┘                 └─────────┬───────────┘
                                       │
                            ┌──────────▼───────────┐
                            │   Message Queue        │
                            │   (Kafka/RabbitMQ)     │ ── decouples delivery from persistence
                            └──────────┬───────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                                       ▼
          ┌──────────────────┐                   ┌──────────────────┐
          │  Message Storage   │                   │  Presence Service │
          │  (Cassandra)       │                   │  (Redis)          │
          └──────────────────┘                   └──────────────────┘
```

### Why does this need a Message Queue?

User A and User B might be connected to **different chat servers** (server affinity/load balancing). A message queue (or a pub/sub layer) lets any server publish a message and have it delivered to whichever server the recipient is actually connected to — this is essentially a distributed **Pub/Sub pattern** (see `04_Pub_Sub_Pattern.md`).

## 📊 Database Choice

| Data                            | Store                               | Why                                                                |
| ------------------------------- | ----------------------------------- | ------------------------------------------------------------------ |
| **Messages**              | Cassandra / DynamoDB (wide-column)  | Write-heavy, append-only, naturally partitioned by`chat_id`      |
| **User/Presence**         | Redis                               | Needs to be fast and ephemeral (online/offline is transient state) |
| **User metadata/profile** | Relational DB (SQL Server/Postgres) | Structured, relational data                                        |

### Message Table (simplified)

| Column                                              | Notes                                            |
| --------------------------------------------------- | ------------------------------------------------ |
| `chat_id`                                         | Partition key — all messages for a conversation |
| `message_id` (timestamp-based, e.g. Snowflake ID) | Sort key — keeps messages naturally ordered     |
| `sender_id`                                       |                                                  |
| `content`                                         |                                                  |
| `status`                                          | sent / delivered / read                          |

Using a **timestamp-based ID (Snowflake ID)** as the sort key means messages come back from the DB **already in chronological order** — no separate sort needed.

## 🔀 Message Delivery Flow

```
1. User A sends message via WebSocket → Chat Server 1
2. Chat Server 1 persists message to DB (Cassandra) — for durability/history
3. Chat Server 1 publishes message to Message Queue/Pub-Sub
4. Whichever Chat Server User B is connected to (could be Server 2) receives it
5. Server 2 pushes message to User B via their open WebSocket
6. If User B is offline → message waits in DB; push notification (APNs/FCM) sent instead
```

## ⚡ Scaling Considerations

- **WebSocket connections are stateful** — a given connection lives on one specific server, so load balancers need **sticky sessions** (see `04_Sticky_Sessions.md`) or a shared presence registry so any server knows which server holds which user's connection.
- **Group chats at scale**: broadcasting one message to a group with thousands of members needs **fan-out** handling — often done asynchronously via the message queue rather than synchronously per recipient.
- **Message ordering**: partitioning messages by `chat_id` (not globally) keeps ordering consistent per-conversation without needing a global lock.

## 🚨 Common Mistakes in Interviews

- ❌ Suggesting plain HTTP polling for a "real-time" system without acknowledging the latency cost.
- ❌ Forgetting that WebSocket connections are **stateful**, which breaks naive round-robin load balancing.
- ❌ Not addressing offline delivery (queued messages + push notifications).
- ❌ Ignoring message ordering guarantees within a conversation.

## 💡 Key Takeaways

- WebSockets for real-time bidirectional communication.
- Message Queue/Pub-Sub decouples "who's connected where" from message delivery.
- Wide-column DB (Cassandra) fits write-heavy, partition-friendly chat data well.
- Snowflake-style IDs give free chronological ordering.

## 🎤 Interview Questions

1. Why are WebSockets preferred over polling or long-polling for a chat system?
2. How do you deliver a message to a user connected to a *different* server than the sender?
3. How would you design message storage to support fast retrieval of chat history in order?
4. How do you handle message delivery to an offline user?

## 📝 30-second Revision Cheat Sheet

- Real-time delivery → WebSockets (persistent, bidirectional).
- Message Queue/Pub-Sub routes messages across servers to the right recipient connection.
- Cassandra/DynamoDB for messages, partitioned by `chat_id`.
- Snowflake IDs → naturally ordered messages.
- Offline users → queue message + push notification.
