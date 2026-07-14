# Kafka basics

Kafka is a distributed streaming/messaging platform. Producers publish messages, consumers read them, independently and at their own pace.

## Core components

- **Broker** — a single Kafka server. A cluster of brokers together forms "Kafka". Clients connect via a bootstrap address, which points into the broker cluster.
- **Topic** — a named feed messages are published to (e.g. `in-ocp-audit`). Similar to a folder, but messages are appended in order and read sequentially — not randomly accessed, and not removed when read.
- **Partition** — a topic is split into partitions, which is how Kafka parallelizes writes and reads. Each message gets an offset (its position within a partition).
- **Producer** — anything that publishes messages to a topic.
- **Consumer** — anything that reads messages from a topic.
- **Consumer group** — a set of consumers that share the work of reading a topic, so messages aren't duplicated across them.

## Why Kafka over a simpler protocol (e.g. syslog)

Syslog is fire-and-forward to one destination. Kafka allows **multiple independent consumers** to subscribe to the same stream, each reading at their own pace, without one consumer affecting another. This makes it suited for feeding several downstream systems (data lake, analytics, other teams' tooling) from a single stream.

## Retention

Messages persist in a topic for a configured retention period, not deleted on read. This lets multiple consumers read the same messages independently, and lets a new consumer replay recent history if needed.