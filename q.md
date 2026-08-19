# Messaging & Event-Driven Systems Reference

Comparison of:

- Amazon SQS
- Amazon SNS
- Amazon EventBridge
- Amazon Kinesis Data Streams
- Apache Kafka
- RabbitMQ

---

## Оглавление

### Core model

- [1. Основная модель](#1-основная-модель)
- [2. Количество получателей](#2-количество-получателей)
- [3. Fan-out](#3-fan-out)
- [4. Типичный use case](#4-типичный-use-case)
- [5. Ближайший аналог](#5-ближайший-аналог)

### Message delivery

- [6. Метод получения](#6-метод-получения)
- [7. Гарантия доставки](#7-гарантия-доставки)
- [8. Failure / redelivery semantics](#8-failure--redelivery-semantics)
- [9. Повторное чтение](#9-повторное-чтение)
- [10. Replay](#10-replay)
- [11. DLQ](#11-dlq)
- [12. Poison message handling](#12-poison-message-handling)
- [13. Порядок сообщений](#13-порядок-сообщений)
- [14. Ordering scope](#14-ordering-scope)

### Storage

- [15. Хранение сообщений](#15-хранение-сообщений)
- [16. Retention](#16-retention)
- [17. Consumer state / offset](#17-consumer-state--offset)

### Performance & scaling

- [18. Масштабирование](#18-масштабирование)
- [19. Throughput](#19-throughput)
- [20. Latency](#20-latency)
- [21. Backpressure / buffering](#21-backpressure--buffering)
- [22. Streaming](#22-streaming)
- [23. Максимальный размер сообщения](#23-максимальный-размер-сообщения)

### Routing & integrations

- [24. Умный routing](#24-умный-routing)
- [25. Filtering](#25-filtering)
- [26. Интеграция с внешними сервисами](#26-интеграция-с-внешними-сервисами)
- [27. Email / SMS / Push](#27-email--sms--push)
- [28. Cross-account / cross-region](#28-cross-account--cross-region)
- [29. AWS service integrations / Targets](#29-aws-service-integrations--targets)
- [30. Event sources / Triggers](#30-event-sources--triggers)

### Reliability & operations

- [31. Transactional semantics](#31-transactional-semantics)
- [32. Security / Access control](#32-security--access-control)
- [33. Schema / event format](#33-schema--event-format)
- [34. Observability](#34-observability)

### Infrastructure

- [35. Протокол взаимодействия](#35-протокол-взаимодействия)
- [36. Managed / self-managed](#36-managed--self-managed)
- [37. Cost model](#37-cost-model)

### Decision guide

- [Quick Decision Guide](#quick-decision-guide)
- [Главная mental model](#главная-mental-model)

---

# Core model

## 1. Основная модель

| Сервис | Ответ |
|---|---|
| **SQS** | Message queue |
| **SNS** | Pub/Sub notification service |
| **EventBridge** | Event bus + event routing |
| **Kinesis** | Event/data stream |
| **Kafka** | Distributed event streaming platform |
| **RabbitMQ** | Message broker |

### Key concept

- **SQS** — «у меня есть работа, которую нужно обработать».
- **SNS** — «у меня есть событие, которое должны получить несколько подписчиков».
- **EventBridge** — «у меня произошло событие; реши, кому его отправить».
- **Kinesis** — «у меня непрерывный поток данных».
- **Kafka** — distributed event log / streaming platform.
- **RabbitMQ** — message broker с routing и очередями.

---

## 2. Количество получателей

| Сервис | Ответ |
|---|---|
| **SQS** | Несколько consumers могут читать queue, но конкретное сообщение обрабатывается одним consumer |
| **SNS** | Несколько subscribers получают одно сообщение |
| **EventBridge** | Несколько targets могут получить одно событие |
| **Kinesis** | Несколько независимых consumers могут читать один stream |
| **Kafka** | Несколько consumer groups могут независимо читать один topic |
| **RabbitMQ** | Несколько consumers могут читать queue; конкретное сообщение получает один consumer |

### Главное различие

```text
SQS / RabbitMQ:

Message → Consumer A
```

против:

```text
SNS / EventBridge:

Message → Consumer A
        → Consumer B
        → Consumer C
```

---

## 3. Fan-out

| Сервис | Ответ |
|---|---|
| **SQS** | ❌ Нет native broadcast |
| **SNS** | ✅ Основной сценарий |
| **EventBridge** | ✅ Через rules → multiple targets |
| **Kinesis** | ✅ Multiple consumers |
| **Kafka** | ✅ Consumer groups |
| **RabbitMQ** | ✅ Через exchanges / multiple queues |

**SNS** — наиболее прямой AWS-пример классического fan-out.

```text
                 SNS
              /   |   \
             ↓    ↓    ↓
           SQS  Lambda  HTTP
```

---

## 4. Типичный use case

| Сервис | Ответ |
|---|---|
| **SQS** | Асинхронная обработка jobs, buffering, decoupling |
| **SNS** | Notification и fan-out |
| **EventBridge** | Event-driven integration и routing |
| **Kinesis** | High-throughput real-time data streams |
| **Kafka** | Event streaming, distributed event log, data pipelines |
| **RabbitMQ** | Messaging между приложениями и сложный broker-level routing |

---

## 5. Ближайший аналог

| Сервис | Ближайший аналог |
|---|---|
| **SQS** | RabbitMQ Queue |
| **SNS** | Google Cloud Pub/Sub / notification pub-sub |
| **EventBridge** | Event bus / event router |
| **Kinesis** | Kafka |
| **Kafka** | Kinesis Data Streams |
| **RabbitMQ** | SQS + routing capabilities |

> Аналоги приблизительные: архитектурные модели сервисов не идентичны.

---

# Message delivery

## 6. Метод получения

| Сервис | Ответ |
|---|---|
| **SQS** | Pull |
| **SNS** | Push |
| **EventBridge** | Push |
| **Kinesis** | Pull |
| **Kafka** | Pull |
| **RabbitMQ** | Push / consumer-driven delivery |

### SQS

Consumer сам получает сообщения:

```text
Consumer → SQS
```

### SNS

SNS отправляет сообщение subscriber'у:

```text
SNS → Consumer
```

### Kafka / Kinesis

Consumer читает stream:

```text
Consumer → Stream
```

---

## 7. Гарантия доставки

| Сервис | Ответ |
|---|---|
| **SQS Standard** | At-least-once |
| **SQS FIFO** | Exactly-once processing semantics при соблюдении условий FIFO deduplication |
| **SNS Standard** | At-least-once |
| **SNS FIFO** | Exactly-once publishing/deduplication semantics в поддерживаемых сценариях |
| **EventBridge** | At-least-once |
| **Kinesis** | At-least-once processing обычно реализуется consumer'ом |
| **Kafka** | Зависит от producer/consumer configuration; возможен exactly-once processing |
| **RabbitMQ** | Зависит от acknowledgements и publisher confirms; обычно at-least-once |

> **Важно:** exactly-once в distributed systems почти всегда означает конкретную семантику конкретного компонента, а не гарантию того, что бизнес-операция во всей системе выполнится ровно один раз.

---

## 8. Failure / redelivery semantics

| Сервис | Ответ |
|---|---|
| **SQS** | Message становится видимым снова после Visibility Timeout, если не удалён |
| **SNS** | Retry delivery согласно delivery policy; после исчерпания retries возможен DLQ |
| **EventBridge** | Automatic retries для failed target delivery + DLQ |
| **Kinesis** | Consumer сам управляет checkpoint/reprocessing |
| **Kafka** | Consumer offset позволяет повторно обработать сообщения |
| **RabbitMQ** | `nack/reject` может привести к requeue |

### SQS

```text
Receive
   ↓
Visibility Timeout
   ↓
Success → Delete
Failure → Message снова доступно
```

---

## 9. Повторное чтение

| Сервис | Ответ |
|---|---|
| **SQS** | ⚠️ Только через redelivery до удаления / visibility timeout |
| **SNS** | ❌ Не является persistent replay log |
| **EventBridge** | ⚠️ Ограниченные возможности archive/replay |
| **Kinesis** | ✅ Пока record находится в retention window |
| **Kafka** | ✅ Основная возможность |
| **RabbitMQ** | ⚠️ Пока message существует / не удалён |

---

## 10. Replay

| Сервис | Ответ |
|---|---|
| **SQS** | ❌ Не предназначен для replay |
| **SNS** | ❌ Не является event log |
| **EventBridge** | ✅ Archives + Replay |
| **Kinesis** | ✅ Через retention + consumer position |
| **Kafka** | ⭐ Один из основных сценариев |
| **RabbitMQ** | ❌ Не предназначен для event replay |

---

## 11. DLQ

| Сервис | Ответ |
|---|---|
| **SQS** | ✅ Native DLQ |
| **SNS** | ✅ Subscription DLQ |
| **EventBridge** | ✅ Target DLQ |
| **Kinesis** | ❌ Нет классического DLQ; реализуется consumer/application layer |
| **Kafka** | ⚠️ Обычно отдельный topic |
| **RabbitMQ** | ✅ Через DLX / dead-letter exchange |

---

## 12. Poison message handling

| Сервис | Ответ |
|---|---|
| **SQS** | DLQ + `maxReceiveCount` |
| **SNS** | Retry policy + DLQ |
| **EventBridge** | Retry policy + DLQ |
| **Kinesis** | Application-level handling |
| **Kafka** | Обычно retry/DLQ topics |
| **RabbitMQ** | DLX / retry queues / TTL |

**Poison message** — сообщение, которое постоянно вызывает ошибку consumer'а.

---

## 13. Порядок сообщений

| Сервис | Ответ |
|---|---|
| **SQS Standard** | ❌ Не гарантируется |
| **SQS FIFO** | ✅ |
| **SNS Standard** | ❌ |
| **SNS FIFO** | ✅ |
| **EventBridge** | ❌ |
| **Kinesis** | ✅ В пределах shard |
| **Kafka** | ✅ В пределах partition |
| **RabbitMQ** | ✅ В рамках queue при соблюдении условий; competing consumers могут влиять на фактический порядок обработки |

---

## 14. Ordering scope

| Сервис | Scope |
|---|---|
| **SQS FIFO** | Message Group |
| **SNS FIFO** | Message Group |
| **EventBridge** | Нет гарантированного ordering |
| **Kinesis** | Shard |
| **Kafka** | Partition |
| **RabbitMQ** | Queue / delivery order |

### Kafka

```text
Topic
 ├── Partition A → A1 → A2 → A3
 └── Partition B → B1 → B2 → B3

A1 vs B1
↑
global ordering отсутствует
```

---

# Storage

## 15. Хранение сообщений

| Сервис | Ответ |
|---|---|
| **SQS** | ✅ Да |
| **SNS** | ⚠️ Не queue storage; сообщение хранится в рамках delivery/retry semantics |
| **EventBridge** | ⚠️ Event retention ограничена; archives позволяют хранить события дольше |
| **Kinesis** | ✅ Да |
| **Kafka** | ✅ Да |
| **RabbitMQ** | ✅ Да, при соответствующей durable/persistent configuration |

---

## 16. Retention

| Сервис | Ответ |
|---|---|
| **SQS** | 1 минута – 14 дней |
| **SNS** | Ограничено delivery/retry window |
| **EventBridge** | Operational retention + Archive для длительного хранения/replay |
| **Kinesis** | Обычно 24 часа – 365 дней |
| **Kafka** | Конфигурируемая retention policy |
| **RabbitMQ** | Зависит от queue/message TTL и конфигурации |

---

## 17. Consumer state / offset

| Сервис | Ответ |
|---|---|
| **SQS** | ❌ Нет offset; message удаляется после обработки |
| **SNS** | ❌ Нет consumer offset |
| **EventBridge** | ❌ Нет consumer offset |
| **Kinesis** | ✅ Consumer checkpoint |
| **Kafka** | ✅ Consumer offsets |
| **RabbitMQ** | ❌ Нет stream-style offset |

### Ключевой принцип

```text
Queue:

message → consume → delete


Stream:

message → consume
          ↑
       offset
```

---

# Performance & scaling

## 18. Масштабирование

| Сервис | Как масштабируется |
|---|---|
| **SQS** | AWS автоматически масштабирует queue; consumers масштабируются отдельно |
| **SNS** | AWS автоматически масштабирует delivery; subscribers масштабируются независимо |
| **EventBridge** | AWS управляет масштабированием event bus; targets масштабируются независимо |
| **Kinesis** | Через shards / capacity modes |
| **Kafka** | Через partitions и brokers |
| **RabbitMQ** | Через consumers, queues, nodes/clusters |

### Kafka

```text
Topic
 ├── Partition 1 → Consumer 1
 ├── Partition 2 → Consumer 2
 └── Partition 3 → Consumer 3
```

---

## 19. Throughput

| Сервис | Характеристика |
|---|---|
| **SQS** | Высокий throughput; Standard масштабируется автоматически |
| **SNS** | Очень высокий throughput |
| **EventBridge** | Высокий managed throughput, subject to service quotas |
| **Kinesis** | Предназначен для high-throughput streaming |
| **Kafka** | Очень высокий throughput; масштабируется partitions/brokers |
| **RabbitMQ** | Высокий throughput, зависит от topology и workload |

---

## 20. Latency

| Сервис | Типичная модель |
|---|---|
| **SQS** | Low latency, asynchronous |
| **SNS** | Low-latency push |
| **EventBridge** | Near-real-time event delivery |
| **Kinesis** | Near-real-time streaming |
| **Kafka** | Low-latency streaming |
| **RabbitMQ** | Очень низкая latency для messaging workloads |

---

## 21. Backpressure / buffering

| Сервис | Ответ |
|---|---|
| **SQS** | ⭐⭐⭐⭐⭐ Отлично |
| **SNS** | ⭐⭐ Ограниченно |
| **EventBridge** | ⭐⭐ Ограниченно |
| **Kinesis** | ⭐⭐⭐⭐ |
| **Kafka** | ⭐⭐⭐⭐⭐ |
| **RabbitMQ** | ⭐⭐⭐⭐⭐ |

### SQS

```text
Producer
   ↓
████████████ Queue
   ↓
Consumer
```

Producer может временно работать значительно быстрее consumer.

---

## 22. Streaming

| Сервис | Ответ |
|---|---|
| **SQS** | ❌ |
| **SNS** | ❌ |
| **EventBridge** | ⚠️ Event-driven, но не stream processing platform |
| **Kinesis** | ✅ Основное назначение |
| **Kafka** | ✅ Основное назначение |
| **RabbitMQ** | ⚠️ Возможен streaming-like workload, но это не основная модель |

---

## 23. Максимальный размер сообщения

| Сервис | Максимальный размер |
|---|---:|
| **SQS** | 1 MiB |
| **SNS** | 256 KB |
| **EventBridge** | 256 KB |
| **Kinesis** | 1 MB per record |
| **Kafka** | Зависит от configuration |
| **RabbitMQ** | Зависит от configuration / resources |

### Большие payloads

```text
Large object
     ↓
    S3
     ↓
SQS / SNS / EventBridge
     │
     └── S3 object key / URL
```

---

# Routing & integrations

## 24. Умный routing

| Сервис | Ответ |
|---|---|
| **SQS** | ❌ |
| **SNS** | ⚠️ Ограниченный filtering |
| **EventBridge** | ⭐⭐⭐⭐⭐ Основная функция |
| **Kinesis** | ❌ |
| **Kafka** | ⚠️ Обычно application/stream-processing layer |
| **RabbitMQ** | ⭐⭐⭐⭐⭐ Exchanges + routing keys |

---

## 25. Filtering

| Сервис | Ответ |
|---|---|
| **SQS** | ⚠️ Не является основной routing mechanism |
| **SNS** | ✅ Subscription filter policies |
| **EventBridge** | ⭐⭐⭐⭐⭐ Event patterns |
| **Kinesis** | ❌ Application-level |
| **Kafka** | ⚠️ Consumer/application-level |
| **RabbitMQ** | ✅ Routing keys / exchange types |

### EventBridge

```text
Event
  │
  ├── type = OrderCreated → Lambda
  ├── type = PaymentFailed → SQS
  └── type = UserCreated → Step Functions
```

---

## 26. Интеграция с внешними сервисами

| Сервис | Ответ |
|---|---|
| **SQS** | Через consumers / Lambda / integration layer |
| **SNS** | ✅ HTTP/S subscriptions |
| **EventBridge** | ⭐⭐⭐⭐⭐ AWS + SaaS integrations |
| **Kinesis** | Через consumers/connectors |
| **Kafka** | Через Kafka Connect / consumers |
| **RabbitMQ** | Через consumers/plugins/integration components |

---

## 27. Email / SMS / Push

| Сервис | Ответ |
|---|---|
| **SQS** | ❌ |
| **SNS** | ✅ Email / SMS / mobile push |
| **EventBridge** | ❌ Direct delivery не является основной функцией |
| **Kinesis** | ❌ |
| **Kafka** | ❌ |
| **RabbitMQ** | ❌ |

---

## 28. Cross-account / cross-region

| Сервис | Ответ |
|---|---|
| **SQS** | ✅ |
| **SNS** | ✅ |
| **EventBridge** | ⭐⭐⭐⭐⭐ Сильная поддержка |
| **Kinesis** | ⚠️ Через дополнительные архитектурные решения |
| **Kafka** | ✅ Через replication / MirrorMaker и т.п. |
| **RabbitMQ** | ✅ Federation / Shovel / clustering patterns |

---

## 29. AWS service integrations / Targets

| Сервис | Примеры |
|---|---|
| **SQS** | Lambda, ECS и другие consumers |
| **SNS** | Lambda, SQS, HTTP/S, email, SMS |
| **EventBridge** | Lambda, Step Functions, SQS, SNS, ECS, Kinesis, API Destinations и AWS/SaaS targets |
| **Kinesis** | Lambda, Firehose, analytics/consumers |
| **Kafka** | Lambda/MSK integrations, Kafka Connect, consumers |
| **RabbitMQ** | Consumers, plugins, integration components |

### EventBridge

```text
EventBridge
    │
    ├──→ Lambda
    ├──→ Step Functions
    ├──→ SQS
    ├──→ SNS
    ├──→ ECS
    └──→ API Destination
```

---

## 30. Event sources / Triggers

| Сервис | Кто/что может инициировать события |
|---|---|
| **SQS** | Applications, AWS services, producers через API/SDK |
| **SNS** | Applications и AWS services |
| **EventBridge** | AWS services, custom applications, SaaS, scheduled events |
| **Kinesis** | Applications, producers, agents/connectors |
| **Kafka** | Applications, producers, connectors |
| **RabbitMQ** | Applications, producers, integrations |

### Примеры

```text
S3
 ↓
EventBridge
 ↓
Lambda
```

```text
SQS
 ↓
Lambda
```

```text
SNS
 ↓
Lambda
```

```text
Kinesis
 ↓
Lambda
```

---

# Reliability & operations

## 31. Transactional semantics

| Сервис | Ответ |
|---|---|
| **SQS** | Нет general distributed transactions; FIFO/deduplication semantics |
| **SNS** | Нет general distributed transactions |
| **EventBridge** | Нет distributed transaction semantics |
| **Kinesis** | Нет general distributed transactions |
| **Kafka** | ✅ Transactions / exactly-once processing возможны |
| **RabbitMQ** | ⚠️ Publisher confirms / acknowledgements; transactions возможны |

> Message acknowledgement ≠ distributed transaction.

---

## 32. Security / Access control

| Сервис | Ответ |
|---|---|
| **SQS** | IAM + resource policies + encryption |
| **SNS** | IAM + resource policies + encryption |
| **EventBridge** | IAM + resource policies + encryption |
| **Kinesis** | IAM + encryption + resource policies |
| **Kafka** | SASL / TLS / ACLs; зависит от deployment |
| **RabbitMQ** | Users / permissions / TLS; зависит от deployment |

---

## 33. Schema / event format

| Сервис | Ответ |
|---|---|
| **SQS** | Arbitrary message payload |
| **SNS** | Arbitrary message payload |
| **EventBridge** | Structured event envelope + schema capabilities |
| **Kinesis** | Arbitrary records |
| **Kafka** | Arbitrary bytes; Schema Registry часто используется отдельно |
| **RabbitMQ** | Arbitrary message payload |

### EventBridge

```json
{
  "source": "my.application",
  "detail-type": "OrderCreated",
  "detail": {
    "orderId": "123"
  }
}
```

---

## 34. Observability

| Сервис | Ответ |
|---|---|
| **SQS** | CloudWatch metrics, logs/tracing через consumers |
| **SNS** | CloudWatch metrics, delivery status |
| **EventBridge** | CloudWatch metrics/logging/tracing integrations |
| **Kinesis** | CloudWatch metrics, consumer monitoring |
| **Kafka** | Consumer lag, broker metrics, logs, tracing |
| **RabbitMQ** | Queue depth, consumer metrics, management UI/plugins |

### Ключевые metrics

**SQS**

```text
ApproximateNumberOfMessagesVisible
AgeOfOldestMessage
```

**Kafka**

```text
Consumer Lag
```

---

# Infrastructure

## 35. Протокол взаимодействия

| Сервис | Протокол / API |
|---|---|
| **SQS** | HTTPS / AWS API |
| **SNS** | HTTPS / AWS API |
| **EventBridge** | HTTPS / AWS API |
| **Kinesis** | HTTPS / AWS API |
| **Kafka** | Kafka protocol |
| **RabbitMQ** | AMQP |

---

## 36. Managed / self-managed

| Сервис | Ответ |
|---|---|
| **SQS** | ⭐ Fully managed |
| **SNS** | ⭐ Fully managed |
| **EventBridge** | ⭐ Fully managed |
| **Kinesis** | ⭐ Fully managed |
| **Kafka** | Self-managed или managed, например Amazon MSK |
| **RabbitMQ** | Self-managed или managed |

---

## 37. Cost model

| Сервис | Основной принцип |
|---|---|
| **SQS** | Requests + data transfer |
| **SNS** | Requests + deliveries + data transfer |
| **EventBridge** | Events ingested / processed + additional features |
| **Kinesis** | Throughput/capacity + retention/storage и другие компоненты |
| **Kafka** | Infrastructure + storage + network; managed Kafka — по модели провайдера |
| **RabbitMQ** | Infrastructure + storage/network |

### Важный принцип

```text
No servers to manage
        ≠
No cost when idle
```

Даже serverless architecture может иметь постоянные расходы на:

- networking;
- NAT Gateway;
- VPC endpoints;
- storage;
- provisioned capacity;
- retention;
- data transfer;
- API requests.

---

# Quick Decision Guide

| Если тебе нужно... | Смотри в сторону |
|---|---|
| Простая asynchronous queue | **SQS** |
| Decouple producer и consumer | **SQS** |
| Один event → много subscribers | **SNS** |
| Event routing по содержимому события | **EventBridge** |
| AWS/SaaS event integration | **EventBridge** |
| Trigger Lambda asynchronously | **SQS / SNS / EventBridge / Kinesis** |
| Оркестрация workflow | **Step Functions** |
| High-throughput stream | **Kinesis / Kafka** |
| Replay событий | **Kafka / Kinesis / EventBridge Archive** |
| Сложный broker routing | **RabbitMQ** |
| Email/SMS/mobile push | **SNS** |
| Buffering bursts | **SQS** |
| Event log | **Kafka / Kinesis** |

---

# Главная mental model

```text
                         ┌───────────────┐
                         │   EventBridge │
                         │ routing/bus   │
                         └───────┬───────┘
                                 │
                    ┌────────────┼────────────┐
                    ↓            ↓            ↓
                  Lambda     Step Functions   SQS
                    │                         │
                    ↓                         ↓
                 Business                  Workers


Producer → SQS → Consumer
          queue


Producer → SNS → SQS
              → Lambda
              → HTTP
              → SMS


Producer → Kinesis → Consumer
                  → Consumer
                  → Consumer


Producer → Kafka → Consumer Group A
                → Consumer Group B
                → Consumer Group C


Producer → RabbitMQ → Exchange → Queue → Consumer
```

## Самая короткая формула

```text
SQS         = "сделай эту работу"
SNS         = "сообщаю всем заинтересованным"
EventBridge = "вот событие — реши, кому оно нужно"
Kinesis     = "вот поток данных"
Kafka       = "вот распределённый event log"
RabbitMQ    = "вот message broker с мощным routing"
```
