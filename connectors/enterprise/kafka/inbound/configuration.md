---
title: Configure Aerospike Connect for Kafka - Inbound
description: Configure Aerospike Connect for Kafka - Inbound
---

## Aerospike Connect Kafka Sink Connector

The connector is configured using a number of key/value properties. In
standalone mode, these are defined in a properties file and passed to the
Connect process on the command line. In distributed mode, they will be included
in the JSON payload for the request that creates (or modifies) the connector.
All connectors share some common options which are defined in the Aerospike Connect for Kafka
documentation, including:

| Name              | Required | Description |
|-------------------|----------|-------------|
| `name`            | true     | Unique name for the connector. Attempting to register again with the same name will fail. |
| `connector.class` | true     | The Java class for the connector; for the Aerospike Connector the class name is com.aerospike.kafka.connect.sink.AerospikeSinkConnector. |
| `tasks.max`       | true     | The maximum number of tasks that should be created for this connector. The connector may create fewer tasks if it cannot achieve this level of parallelism. |
| `topics`          | true     | A list of topics to use as input for this connector. |

The following configuration options are specific to the Aerospike Connect for Kafka:

| Name                 | Required | Default | Description |
|----------------------|----------|---------|-------------|
| `cluster.hosts`      | true     | -       | Comma separated list of one or more Aerospike cluster hosts; each host can be specified as a valid IP address or hostname followed by an optional port number (default is 3000). |
| `policy.record_exists_action` | false | `update` | Sets the write policy for the Aerospike client: How to handle writes when the record already exists; valid options are: update, update_only, replace,  replace_only, create_only, which match the Java client's RecordExistsAction value of the same name. |
| `max_async_commands` | false    | 300     | Maximum number of concurrent asynchronous client requests to the Aerospike cluster. |
| `max_command_action` | false    | `block` | How to handle cases when the asynchronous max. number of concurrent connections have been reached; valid options are: block, accept, reject which match the Java client's MaxCommandAction value of the same name. |
| `topic.namespace`    | false    | -       | Default Aerospike namespace to use.|
| `topic.set`          | false    | -       | Default Aerospike set name to use. |
| `topic.key_field`    | false    | -       | Name of the Kafka record field that contains the Aerospike user key. If not specified, the Kafka record key is used instead. |
| `topic.set_field`    | false    | -       | Name of the Kafka record field that contains the Aerospike set name; mutually exclusive with the set option. |
| `topic.bins`         | false    | -       | Kafka to Aerospike record field name mapping; see below for details. |
| `feature_key.path`   | true     | -       | The location of the Aerospike feature key file (e.g. etc/aerospike-kafka/features.conf). The `mesg-kafka-connector` feature must be enabled in the feature key. |
| `aerospike.username` | false    | -       | An authorized user for the Aerospike Database.|
| `aerospike.password` | false    | -    | The password for the user specified with `aerospike.username`. This Must be specified if `aerospike.username` is provided. |

In addition, the record mapping (namespace/set/key/bins) can also be configured
on a per-topic basis. The per-topic options use a key prefix of
`topic.<topic_name>.` where `<topic_name>` is the name of the topic to which
the mapping applies:

| Name                           | Required | Description |
|--------------------------------|----------|-------------|
| `topic.<topic_name>.namespace` | false    | Aerospike namespace to use for the topic. |
| `topic.<topic_name>.set`       | false    | Aerospike set name to use for the topic. |
| `topic.<topic_name>.key_field` | false    | Name of the Kafka record field that contains the Aerospike user key. |
| `topic.<topic_name>.set_field` | false    | Name of the Kafka record field that contains the Aerospike set name; mutually exclusive with the set config property. |
| `topic.<topic_name>.bins`      | false    | Kafka to Aerospike record field name mapping; see below for details. |

## Record Field Mapping

The `topic.bins` and `topic.<topic_name>.bins` configuration options determine
the record field mapping when copying records from the Kafka topic into the
Aerospike database. These properties contain comma separated lists of record field
names. If set, only Kafka record fields included in the list are copied into
the Aerospike record. Each entry of the list can optionally also include the
Aerospike bin name, separated by a colon (":").

For example, to copy the two
Kafka record fields field1 and field2 verbatim and copy the field
veryLongFieldName into a bin named asFieldName in the Aerospike record,
topic.bins can be set as follows:

    topic.myTopic.bins=field1,field2,veryLongFieldName:asFieldName
