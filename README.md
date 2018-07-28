# Confluent Platform

Ansible role to install and configure the [Confluent Platform](https://www.confluent.io/) on RHEL / CentOS 6 and 7.

[Confluent Platform](https://www.confluent.io/) is a complete distribution of an Apache Kafka streaming platform.

[Apache Kafka](http://kafka.apache.org/) is a message bus using publish-subscribe topics. Other components and products can consume these messages by subscribing to these topics. Kafka is extremely fast, handling megabytes of reads and writes per second from thousands of clients. Messages are persisted and replicated to prevent data loss. Data streams are partitioned and can be elastically scaled with no downtime.

The Confluent Platform includes the below additional components that build upon this framework:
* [Schema Registry](https://docs.confluent.io/current/platform.html#confluent-schema-registry) used when publishing and consuming messages in Avro format. Schemas can be registered to ensure that messages conform to a standard format when serializing and deserializing.
* [Kafka REST Proxy](https://docs.confluent.io/current/platform.html#confluent-kafka-rest-proxy) which provides a RESTful service for sending and receiving messages. It can also be used for accessing topic metadata.
* [KSQL](https://docs.confluent.io/current/platform.html#ksql) is a SQL engine for Apache Kafka. It provides the ability to use SQL-like statement to run stream processing operations with Kafka topics, streams, and tables.

## Requirements

Platform: RHEL / CentOS 6 and 7

## Basic Role Variables
    confluent_platform_version: 4.1
    confluent_kafka_broker_id: 0
    confluent_kafka_listener_protocol: PLAINTEXT
    confluent_kafka_listener_hostname: localhost
    confluent_kafka_listener_port: 9092
    confluent_kafka_num_network_threads: 3
    confluent_kafka_log_dirs: /var/lib/kafka
    confluent_kafka_num_partitions: 1
    confluent_kafka_log_retention_hours: 168
    confluent_kafka_auto_create_topics_enable: true
    confluent_kafka_delete_topic_enable: true
    confluent_kafka_default_replication_factor: 1
    confluent_kafka_zookeeper_connect: 'localhost:2181'
    confluent_kafka_zookeeper_connection_timeout: 6000
    confluent_kafka_bootstrap_servers: 'localhost:9092'
    confluent_kafka_consumer_group_id: kafka-consumer-group
    confluent_zookeeper_id: 1

## Starting and stopping services using systemd
* The ZooKeeper service can be started via: `systemctl start zookeeper`
* The ZooKeeper service can be stopped via: `systemctl stop zookeeper`
* The Kafka service can be started via: `systemctl start kafka`
* The Kafka service can be stopped via: `systemctl stop kafka`

## Starting and stopping services using initd
* The ZooKeeper service can be started via: `service zookeeper start`
* The ZooKeeper service can be stopped via: `service zookeeper stop`
* The Kafka service can be started via: `service kafka start`
* The Kafka service can be stopped via: `service kafka stop`

#### Ports

| Port | Description |
|------|-------------|
| 2181 | ZooKeeper listener port |
| 9092 | Kafka listener port |
| 8081 | Schema Registry port |
| 8082 | REST Proxy port |
| 8083 | Kafka Connect port |
| 8088 | KSQL port |
