# Confluent Platform

[![Build Status](https://travis-ci.org/sleighzy/ansible-confluent-platform.svg?branch=master)](https://travis-ci.org/sleighzy/ansible-confluent-platform)

Ansible role to install and configure [Confluent Platform 5.0](https://www.confluent.io/) on RHEL / CentOS 6 and 7.

[Confluent Platform](https://www.confluent.io/) is a complete distribution of an Apache Kafka streaming platform. This distribution ships with Apache Kafka 2.0.

[Apache Kafka](http://kafka.apache.org/) is a message bus using publish-subscribe topics. Other components and products can consume these messages by subscribing to these topics. Kafka is extremely fast, handling megabytes of reads and writes per second from thousands of clients. Messages are persisted and replicated to prevent data loss. Data streams are partitioned and can be elastically scaled with no downtime.

The Confluent Platform includes the below additional components that build upon this framework:
* [Schema Registry](https://docs.confluent.io/current/platform.html#confluent-schema-registry) used when publishing and consuming messages in Avro format. Schemas can be registered to ensure that messages conform to a standard format when serializing and deserializing.
* [Kafka REST Proxy](https://docs.confluent.io/current/platform.html#confluent-kafka-rest-proxy) which provides a RESTful service for sending and receiving messages. It can also be used for accessing topic metadata.
* [KSQL](https://docs.confluent.io/current/platform.html#ksql) is a SQL engine for Apache Kafka. It provides the ability to use SQL-like statement to run stream processing operations with Kafka topics, streams, and tables.

## Requirements

Platform: RHEL / CentOS 6 and 7

## Basic Role Variables
    confluent_platform_version: 5.0
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

#### Ports

| Port | Description |
|------|-------------|
| 2181 | ZooKeeper listener port |
| 9092 | Kafka listener port |
| 8081 | Schema Registry port |
| 8082 | REST Proxy port |
| 8083 | Kafka Connect port |
| 8088 | KSQL port |

## ZooKeeper and Kafka Security

This Ansible role includes the configuration for securing the Kafka cluster.

https://docs.confluent.io/current/security.html

### ZooKeeper Security

By default the ZooKeeper configuration this role uses DIGEST-MD5 for SASL security. Credentials are stored in plaintext within the JAAS configuration file.

The `confluent_zookeeper_sasl_mechanism` property can be set to `GSSAPI` when integrating with Kerberos. This Ansible role does not create keytab files, these will need to be created and supplied separately.

### Kafka Security

This Ansible role configures the Kafka cluster securely using the SASL_SSL protocol, including inter-broker communication.

#### SSL

When using SSL security keystore(s) and truststores will need to be provided, these are not created by this role. The keystore is required for clients connecting to the brokers, including the schema registry, rest proxy, etc. when the `ssl.client.auth` property is set to `required`.

#### SASL

By default the configuration for the Kafka cluster in this role uses SASL_SSL. The `confluent_*_sasl_mechanism` property by default uses `SCRAM-SHA-256` as the authentication mechanism. The [SASL/SCRAM](https://docs.confluent.io/current/kafka/authentication_sasl_scram.html#sasl-scram-overview) mechanism in Kafka stores credentials within ZooKeeper. This Ansible role includes tasks for creating the credentials for each of the components within ZooKeeper when using SASL/SCRAM.

The value of the `confluent_kafka_super_users` property will be used for the `super.users` configuration option in the `server.properties` file. These users have access to read and write to the topic, and other cluster operations, without explicitely needing to create ACLs for them in ZooKeeper. All components in this installation have been added to the super users list so that they can create their required topics on start up etc. If removing these you will need to create their topics upfront and add their ACLs. With both SSL and broker authentication required each broker will use the distinguished name (DN) of their SSL certificate as the user principle. For example,

`super.users=User:CN=broker-1,OU=kafka,O=example.com;User:CN=broker-2,OU=kafka,O=example.com;User:CN=broker-3,OU=kafka,O=example.com`

When integrating with Kerberos (GSSAPI) the super user will be the kafka service name, i.e `super.users=User:kafka`.

The `confluent_*_sasl_mechanism` properties can be set to `GSSAPI` when integrating with Kerberos. This Ansible role does not create keytab files, these will need to be created and supplied separately.

### Secure ACLs

By default this role does not set `zookeeper.secure.acl` to true in the Kafka `server.properties` file. If the ZooKeeper cluster is secured and secure znodes are used then the `confluent_kafka_zookeeper_secure_acl` property can be set to `true`. When using secure znodes all brokers will need to use a single shared user name, this includes the user principal when integrating with Kerberos, when connecting to ZooKeeper.