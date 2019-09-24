---
title: Aerospike Connect for JMS - Outbound Routing Configuration
description: Configuring Aerospike Connect for JMS Outbound Connector routing section
---

### Routing

The routing section controls how Aerospike records are routed to a JMS destination.
The JMS destination can either be a named queue or a topic.

The following routing modes are available.

 * **static:** Always route to a static queue/topic.
 * **namespace:** Use the namespace of the Aerospike record as the JMS queue/topic.
 * **set:** Use the set of the Aerospike record as the JMS queue/topic.
 * **bin:** Sets the route based on the value of a bin in the record. Only string,
    blob and integer bin types are supported.
 * **path:** Sets the route based on the URL path used to submit the record. The URL is configured in Aerospike's JMS XDR data center section.


#### Static routing

For static routing the configuration options are:

Option | Required | Expected value | Description
--- | --- | --- | ---
mode|yes|static|Selects static routing configuration.
type|yes|queue/topic|The destination type.
destination|yes| |The name of the destination queue/topic.

Sample static routing configuration

```
routing:
  mode: static
  type: queue
  destination: jms-queue

```

#### Set name routing

For record set name routing the configuration options are

Option | Required | Expected value | Description
--- | --- | --- | ---
mode|yes|set|Selects set name routing configuration.
type|yes|queue/topic|The destination type.
default|no| |The default destination queue/topic to use in case the set name is missing in the record or the destination queue/topic is not found.
transforms| no | |List of  transformations to apply to the set name. See [transforms](/docs/connectors/enterprise/jms/outbound/configuration.html#transforms) section for details.

##### Example
```
routing:
  mode: set
  type: queue
  default: default-queue
  transforms:
    - trim
    - regex:
        pattern: '(.*):(.*)'
        replacement: '$2:$1'
    - regex:
        pattern: '$'
        replacement: ':please'
    - uppercase
```

#### Namespace name routing

For record namespace name routing the configuration options are

Option | Required | Expected value | Description
--- | --- | --- | ---
mode|yes|namespace|Selects namespace name routing configuration.
type|yes|queue/topic|The destination type.
default|no| |The default destination queue/topic to use in case the namespace name is missing in the record or the destination queue/topic is not found.
transforms| no | |List of  transformations to apply to the namespace name. See [transforms](/docs/connectors/enterprise/jms/outbound/configuration.html#transforms) section for details.

##### Example

```
routing:
  mode: namespace
  type: queue
  default: default-queue
  transforms:
    - trim
    - regex:
        pattern: '(.*):(.*)'
        replacement: '$2:$1'
    - regex:
        pattern: '$'
        replacement: ':please'
    - uppercase
```

#### Bin value routing

For bin based routing the configuration options are

Option | Required | Expected value | Description
--- | --- | --- | ---
mode|yes|bin|Selects bin based routing.
type|yes|queue/topic|The destination type.
bin|yes| |The name of the bin to pick value from.
default|yes| |The default destination queue/topic to use in case the bin is missing in the record or the destination queue/topic is not found.
transforms| no | |List of  transformations to apply to the namespace name. See [transforms](/docs/connectors/enterprise/jms/outbound/configuration.html#transforms) section for details.

##### Example
```
routing:
  mode: bin
  type: queue
  bin: category
  default: test-queue
  transforms:
    - trim
    - regex:
        pattern: '[^A-Za-z0-9]'
        replacement: '-'
    - lowercase
```

#### Path routing

For XDR data center path routing the configuration options are

Option | Required | Expected value | Description
--- | --- | --- | ---
mode|yes|path|Selects path routing configuration.
type|yes|queue/topic|The destination type.
default|no| |The default destination queue/topic to use in case the destination queue/topic is not found.
transforms| no | |List of  transformations to apply to the namespace name. See [transforms](/docs/connectors/enterprise/jms/outbound/configuration.html#transforms) section for details.

##### Example
```
routing:
  mode: path
  type: queue
  default: default-queue
  transforms:
    - trim
    - regex:
        pattern: '(.*):(.*)'
        replacement: '$2:$1'
    - regex:
        pattern: '$'
        replacement: ':please'
    - uppercase
```

#### Transforms

You can configure a list transforms that will be applied, in order, to the record's set name, namespace path or bin value
in order to derive the destination queue or topic.

Currently the following transforms are supported

  * lowercase - converts the bin value to lowercase.
  * uppercase - converts the bin value to uppercase.
  * trim - trim leading and trailing whitespace.
  * regex - match against a regex pattern and replace all occurrences with a replacement.
  * The regex and replacement uses Java regex syntax.

##### Example
The following transform configuration will first trim the route, replace all non-alphanumeric characters with '-' and convert the result to lowercase.
```
routing:
  mode: bin
  type: queue
  bin: category
  default: test-queue
  transforms:
    - trim
    - regex:
        pattern: '[^A-Za-z0-9]'
        replacement: '-'
    - lowercase
```
