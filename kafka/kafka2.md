# Kafka 2.0

Apache Kafka is a distributed streaming platform capable of handling trillions of events a day. Initially conceived as a messaging queue, Kafka is based on an abstraction of a distributed commit log. Since being created and open sourced by LinkedIn in 2011, Kafka has quickly evolved from messaging queue to a full-fledged streaming platform.

As a streaming platform, Apache Kafka provides low-latency, high-throughput, fault-tolerant publish and subscribe pipelines and is able to process streams of events. Kafka provides reliable, millisecond responses to support both customer-facing applications and connecting downstream systems with real-time data.

# Useful links

- Kafka documentation
  - [Kafka manual](https://kafka.apache.org/documentation/)
  - Primer on [Kafka client rebalancing](https://tomlee.co/2019/03/the-unofficial-kafka-rebalance-how-to/#retriable-vs-non-retriable-errors)
  - [Kafka Controller Internals](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Controller+Internals)
  - [Topic/Partition Design Principals](https://www.confluent.io/blog/how-to-choose-the-number-of-topicspartitions-in-a-kafka-cluster/)
  - [Apache Kafka source code](https://github.com/apache/kafka)
- Links to dashboards, build and release pipelines.

# Package management and release

# Kafka offset exporter
[Kafka offset exporter](https://github.com/echojc/kafka-offset-exporter) can be used to export consumer statistics to a metrics backend like Prometheus. 

## Kafka Manager UI
[Kafka manager UI](https://github.com/yahoo/CMAK) can be used to manage the Kafka clusters. 

# Runbook

