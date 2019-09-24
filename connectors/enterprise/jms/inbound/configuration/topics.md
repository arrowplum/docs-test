---
title: JMS Queues & Topics Configuration
description: Properties specific to the JMS queues and topics.
---

The JMS Queues & Topics sections configures the properties of the JMS
destinations to consume messages.

## Sections

There are individual sections for queues and topics. Atleast one queue or topic
needs to be configured.

| Option | Required | Description                                                                                             |
| ------ | -------- | ------------------------------------------------------------------------------------------------------- |
| topics | no       | The topics to consume messages. See <a href="#jms-destination-config"> JMS destination</a> for details. |
| queues | no       | The queues to consume messages. See <a href="#jms-destination-config"> JMS destination</a> for details. |

## JMS Destination Config

The configuration of the JMS destination queue/topic.

| Option                      | Required | Default | Description                                                                                                                      |
| --------------------------- | -------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| session-ratio               | no       | 1       | The ratio of total JMS sessions to use for this destination. See <a href="#session-ratio-config"> session ratio</a> for details. |
| jms-transaction-size        | no       | 16      | The number of JMS messages to consume in a single JMS transaction.                                                               |
| jms-message-receive-timeout | no       | 100     | The JMS message receive timeout in milliseconds. A value of zero signifies no timeout.                                           |
| aerospike-operation         | no       | write   | The Aerospike operation. See <a href="#aerospike-operation-config"> aerospike</a> for details.                                   |
| mapping                     | yes      |         | The JMS message to Aerospike record mapping. See <a href="#mapping-config"> mapping</a> for details.                             |
| parsing                     | yes      |         | The JMS message parsing config. See <a href="#parsing-config"> parsing</a> for details                                           |

## Session Ratio Config

The ratio of the total JMS sessions to allocate for this destination.

The calculation is as given below

```
total JMS sessions = max-connections * max-sessions-per-connection
sum of all session ratio = sum session ratio of all destinations
sessions allocated per destination = (destination session-ratio / sum of session ratios) * total JMS sessions

total allocated sessions by ratio = sum of sessions allocated per destination for all destinations
remaining sessions = total JMS sessions - total allocated sessions by ratio

All the remaining sessions are randomly distributed among the destinations.
```

Ex: max-connections = 4, max-sessions-per-connection = 2, total JMS sessions = 8

| Destination | Type  | Session Ratio | Allocated Sessions |
| ----------- | ----- | ------------- | ------------------ |
| Employees   | Topic | 1             | 3                  |
| Offices     | Topic | 1             | 3                  |
| Buildings   | Queue | 1             | 2                  |

Total allocated sessions by ratio = Employees: 2, Offices: 2, Buildings: 2

Remaining Sessions = 2

Randomly allocate the remaining 2 sessions to Employees and Office.

Final session allocation = Employees: 3, Offices: 3, Buildings: 2

## Aerospike Operation Config

The Aerospike operation on the record. It can be a write or a delete.

| Option               | Required | Default           | Description                                                                                                                              |
| -------------------- | -------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| type                 | yes      | write             | Valid values are: write, delete.                                                                                                         |
| max-retries          | no       | 2                 | Maximum number of retries before aborting the transaction.                                                                               |
| total-timeout        | no       | 0 - no time limit | Total transaction timeout in milliseconds.                                                                                               |
| record-exists-action | no       | update            | Record exists action on a write. Valid values are update, update-only, replace, replace-only, create-only. Only applies to type "write". |
| send-key             | no       | false             | Send user defined key in addition to hash digest on write. Only applies to type "write".                                                 |
| durable-delete       | no       | false             | On delete leave a tombstone for the record. Only applies to type "delete".                                                               |

## Mapping Config

The JMS message to Aerospike record mapping.

| Option     | Required | Default | Description                                                                                                                                                      |
| ---------- | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| all-fields | yes      |         | Indicates if all fields from the JMS message should be converted to Aerospike record bins.                                                                       |
| bins       | no       |         | Mapping of JMS message field names to Aerospike bin names.                                                                                                       |
| key-field  | yes      |         | The JMS message field to use as the Aerospike record key.                                                                                                        |
| namespace  | yes      |         | The Field Selection for namespace. See <a href="#field-selection-config">Field Selection</a> for details.                                                        |
| set        | no       |         | The Field Selection for set name.                                                                                                                                |
| ttl        | no       |         | The Field Selection for time to live. See <a href="#field-selection-config">Field Selection</a> and <a href="#time-to-live-config">time to live</a> for details. |

## Time To Live Config

Time to live for the Aerospike record. Valid values -

- -2 do not change ttl when record is updated.
- -1 never expire.
- 0 defaults to namespace configuration on the server.
- &gt;0 Time to live in seconds. Can be suffixed with S(seconds), M(minutes), H(hours), D(days).

## Field Selection Config

The field selection can be either static or dynamic. See <a href="#example">
examples</a>.

#### Static Field Selection Config

Select a static value.

| Option | Required | Expected Value | Description                     |
| ------ | -------- | -------------- | ------------------------------- |
| mode   | yes      | static         | Selects static field selection. |
| value  | yes      |                | The static field value.         |

### Dynamic Field Selection Config

Select the value of a field in a message.

| Option     | Required | Expected Value | Description                                              |
| ---------- | -------- | -------------- | -------------------------------------------------------- |
| mode       | yes      | dynamic        | Selects dynamic field selection.                         |
| field-name | yes      |                | The dynamic field in the message to pick the value from. |

## Parsing Config

The message parsing config. Valid values -

1. json - message is a JMS TextMessage in JSON format.
2. msgpack - message is a JMS BytesMessage in MsgPack format.
3. map - message is a JMS MapMessage.
4. java-serialized - message is a JMS ObjectMessage in Java serialized object format.

### Java Serialized

Message is a JMS ObjectMessage in Java serialized object format. The
serialized object should be in the class path. The JAR containing the class
file and its dependencies should be put in _/opt/aerospike-jms-inbound-1.0.0/usr-lib_

## Example

```
topics:
  employees:
    session-ratio: 1
    jms-transaction-size: 16
    jms-message-receive-timeout: 100
    aerospike-operation:
      type: write
      max-retries: 2
      total-timeout: 0
      record-exists-action: update
      send-key: false
    mapping:
      all-fields: false
      bins:
        field-one: binOne
      key-field: employeeID
      namespace:
        mode: static
        value: east
      set:
        mode: dynamic
        field-name: subLocation
      ttl:
        mode: dynamic
        value: ttl
    parsing: json
```

Ex: The mapping of the JSON to Aerospike records for the above configuration

| JSON                                                                            | Aerospike Namespace | Aerospike Set | Aerospike Key | Aerospike Bin                      | TTL  |
| ------------------------------------------------------------------------------- | ------------------- | ------------- | ------------- | ---------------------------------- | ---- |
| {"field-one":"one","subLocation":"BNG", "employeeID": E123, ttl: "100H"}        | test                | BNG           | E123          | Bin("binOne", one)                 | 100H |
| {"field-one":["one"],"subLocation":"MYS", "employeeID": E456, ttl: "10M"}       | test                | MYS           | E456          | Bin("binOne", List("one"))         | 10M  |
| {"field-one":{"one": "one"},"subLocation":"DLH", "employeeID": E789, ttl: "1D"} | test                | DLH           | E789          | Bin("binOne", Map("one" to "one")) | 1D   |
| {"field-one":"one","subLocation":"MUM", "employeeID": E987, ttl: "100"}         | test                | MUM           | E987          | Bin("binOne", one)                 | 100  |
