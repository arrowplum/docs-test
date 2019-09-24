---
title: MessagePack Format for Aerospike Connect for Kafka - Outbound
description: MessagePack Serialization Format for Aerospike Connect for Kafka outbound connector
---

## MessagePack Serialization Format

This document specifies the MessagePack data serialization format used by the
Aerospike Connect for Kafka. It describes how Aerospike change notifications are
represented using MessagePack. Each message represents an event that occurred
in an Aerospike database cluster. Different kinds of messages are used to
describe events such as record creation/update and record deletions.

MessagePack is a binary data serialization format. It's compact and simple form
is designed for efficient transmission of data over the wire. MessagePack parser
implementations exist for a wide variety of popular programming languages. The
official MessagePack specification can be found at https://msgpack.org/. The
message format used by the Aerospike Connect for Kafka is compliant with the
MessagePack spec as published on 2017-08-09.

### Message format

Each message consists of a MessagePack array with 3 elements:

```
+---------------+------------+------------------+
| Version [int] | Type [int] | Payload [varies] |
+---------------+------------+------------------+
```

* **Version**: 1
* **Message Types**:
    * WRITE: 1
    * DELETE: 2
* **Payload**: Message payload depending on message type - see below.

#### Write Message Type Payload

A WRITE message is a MessagePack array with 4 parts:

```
+-----+------------------+-------------------+-----------+--------------+
| Key | Generation [int] | Expiry Time [int] | LUT [int] | Bins [array] |
+-----+------------------+-------------------+-----------+--------------+
```

* **Key**: Record key - see below for MessagePack format. The record key in WRITE
  messages always contains the namespace and key digest, and *may* contain the
  set name and user key components.
* **Generation**: Record generation, changed on each record update.
* **Expiry Time**: Time when the record will expire in seconds since Unix epoch;
  zero means record will not expire.
* **LUT**: Last update timestamp of the record in seconds since Unix epoch. May
  be zero, if the last-update time is not known. [1]
* **Bins**: List of record bins - see below for MessagePack format for each bin.

[1] At present, Aerospike change notifications do not include the last-update
timestamp. Hence, the `lut` property will be zero for all Write message.
Last-update time will be populated once a future Aerospike server update
enables this feature.

#### Delete Message Type Payload

A DELETE message is a MessagePack array with 2 parts:

```
+-----+--------------+
| Key | Flags [int]  |
+-----+--------------+
```

* **Key**: Record key - see below for MessagePack format. The record key in DELETE
  messages only contains the namespace and key digest, but not the set name or
  user key components.
* **Flags**:
    * DURABLE_DELETE: 0x01

### Key Format

A record key consists of a MessagePack array with 4 elements. Only the namespace
and digest are included in every message. The set name and user key are
optional and may be `nil`.

```
+-----------------+---------------+--------------+----------------------------+
| Namespace [str] | Set [str|nil] | Digest [bin] | User Key [str|int|bin|nil] |
+-----------------+---------------+--------------+----------------------------+
```

* **Namespace**: Aerospike namespace
* **Set**: Set name within the namespace (optional)
* **Digest**: 160-bit record digest.
* **User Key**: The integer, string or bytes value used by the application to
  uniquely address the record. Will be included in the message only if it is
  stored on the server.

### Bins Format

Record bins are sent as a MessagePack array; each bin has the following format:

```
+------------+------------+-------------+----------------+
| Name [str] | Type [int] | Flags [int] | Value [varies] |
+------------+------------+-------------+----------------+
```

* **Name**: Name of the bin.
* **Type**: Type of the bin's value:
    * INTEGER: 1
    * DOUBLE: 2
    * STRING: 3
    * BLOB: 4
    * JAVA OBJ: 7 (Serialized Java Objects)
    * MAP: 19
    * LIST: 20
    * GEOJSON: 23
* **Flags**: Type specific flags:
    * MAP: Map Order:
        * Unordered: 0
        * Key-Ordered: 1
        * Key-Value-Ordered: 3
    * LIST: List Order:
        * Unordered: 0
        * Ordered: 1
    * Other: zero (0)
* **Value**: The MessagePack format of the value depends on the bin's type:
    * INTEGER: int
    * DOUBLE: float
    * STRING: str
    * BLOB: bin
    * JAVA OBJ: bin
    * MAP: map
    * LIST: array
    * GEOJSON: str

Map and list elements can be of any allowed bin data type, including nested
lists and maps. Within maps and lists only, serialized Java Objects and GeoJSON
strings are represented using the MessagePack ext format type, with the ext
type being the same as the corresponding bin data type.
