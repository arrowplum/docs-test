---
title: Aerospike Connect for JMS - Outbound Format Configuration
description: Configuring Aerospike Connect for JMS Outbound Connector format section
---

## Output format

Configures the format of the outbound messages delivered to the message broker.

The following output formats are supported

 * [JSON](/docs/connectors/enterprise/kafka/outbound/json-serialization-format.html) - The default format with JSON formatted messages.
 * [Message Pack](/docs/connectors/enterprise/kafka/outbound/messagepack-serialization-format.html) - Message Pack formatted messages.

### Example

For Json output:
```
format: json
```

For Message pack output:

```
format: msgpack
```
