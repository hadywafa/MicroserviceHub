# 🧭 RabbitMQ + .NET Mastery Roadmap

## 🚀 LEVEL 1: 🧱 Fundamentals – Getting Started

### 🔹 1. What is RabbitMQ & Messaging Patterns?

- Pub/Sub, Work Queue, RPC
- Differences from Kafka and when to use it

### 🔹 2. Install RabbitMQ (Locally or via Docker)

- Docker + Management UI
- Access via `http://localhost:15672` (web)
- User/pass: `guest / guest`

```bash
docker run -d --hostname rabbit \
  -p 5672:5672 -p 15672:15672 \
  --name rabbitmq rabbitmq:3-management
```

### 🔹 3. First Message with Raw .NET (AMQP.Client)

- Send/Receive with official `RabbitMQ.Client`
- Create queue, publish, and consume manually

---

## 🧰 LEVEL 2: 🔄 Messaging Patterns in .NET

### 🔹 4. Work Queues (Task Distribution)

- Use a single queue, multiple consumers
- Manual ack vs auto ack
- Use case: email worker system

### 🔹 5. Publish/Subscribe (Fanout Exchange)

- One producer → many consumers
- All receive a copy
- Use case: notifications, logging

### 🔹 6. Direct Exchange (Routing)

- Use routing keys like `order.created`, `order.cancelled`
- Each queue binds to specific keys

### 🔹 7. Topic Exchange (Wildcard Routing)

- Routing keys with `.` and `*`, `#` wildcards
- Use case: logs → `logs.error`, `logs.info`

### 🔹 8. Headers Exchange

- Route by custom headers
- Useful when routing key logic is too rigid

---

## 🧠 LEVEL 3: ✅ Delivery Guarantees & Reliability

### 🔹 9. Message Durability & Persistence

- `durable: true` queues
- `persistent` delivery mode for messages
- RabbitMQ survives restarts

### 🔹 10. Manual Acknowledgements

- `BasicAck`, `BasicNack`, `BasicReject`
- Handle message retries, fail fast

### 🔹 11. Dead Letter Exchange (DLX)

- Unprocessable messages go to a **dead-letter queue**
- Use case: poison messages, retry backoff

---

## 🧩 LEVEL 4: Abstractions with Libraries

### 🔹 12. Using MassTransit with RabbitMQ

- Simplifies bus setup, message contracts, DI
- Built-in retry, saga state machine, observability

```bash
dotnet add package MassTransit.RabbitMQ
```

### 🔹 13. Using EasyNetQ (Simple Wrapper)

- Friendly, opinionated API
- Perfect for small apps or services

```bash
dotnet add package EasyNetQ
```

---

## 🛠 LEVEL 5: Infrastructure & HA

### 🔹 14. RabbitMQ Clustering & Quorum Queues

- Set up 3-node cluster using Docker or Kubernetes
- Use **quorum queues** for replication

### 🔹 15. Retry Policies (Immediate vs Delayed Retry)

- Retry with `MassTransit`, `EasyNetQ`, or manually
- Use delayed exchange plugin for timed retry

### 🔹 16. Monitoring & Observability

- RabbitMQ Management UI
- Prometheus + Grafana
- MassTransit Diagnostic Source

---

## 🧪 LEVEL 6: Advanced Messaging

### 🔹 17. RPC (Remote Procedure Calls over RabbitMQ)

- Use `reply_to` and `correlation_id` headers
- Useful for microservice-style APIs

### 🔹 18. Message Batching & Prefetch

- Increase throughput, lower latency
- Configure `prefetchCount` wisely

### 🔹 19. Shovel & Federation Plugins

- Move messages between clusters
- Multi-data-center reliability

---

## 📦 LEVEL 7: Project-Based Learning

### 🔹 20. Build Microservices Using RabbitMQ & MassTransit

- ✅ OrderService: publishes order events
- ✅ EmailService: consumes events and sends mail
- ✅ InvoiceService: handles billing events

Use **MassTransit** to wire them with RabbitMQ for:

- Saga patterns
- Retry policies
- Message contracts
- Tracing/logging with OpenTelemetry

---

## 🔄 Bonus Topics

| 🔥 Topic                        | 💡 Why Learn It                               |
| ------------------------------- | --------------------------------------------- |
| ✅ RabbitMQ in Kubernetes       | For cloud-native deployment                   |
| 🧼 Message Deduplication        | Prevents double-processing                    |
| ⏱ Delayed Messaging Plugin      | Schedule messages (like reminders)            |
| 👮 Secure RabbitMQ (TLS + Auth) | Production-grade security                     |
| 📋 CI/CD & Integration Testing  | Use TestContainers for RabbitMQ in .NET tests |

---

## 🧠 Summary: Where to Start?

| You Are...                   | Start Here      |
| ---------------------------- | --------------- |
| 🆕 New to RabbitMQ           | Level 1 + 2     |
| 🧑‍💻 .NET Developer         | Level 1 → 3 → 4 |
| 🏗️ Building Microservices    | Level 4 → 5 → 7 |
| 🧪 Obsessed with reliability | Level 3 → 5 → 6 |
| ☁️ Going cloud-native        | Level 5 → Bonus |
