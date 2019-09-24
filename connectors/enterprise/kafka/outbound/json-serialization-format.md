---
title: JSON Format for Aerospike Connect for Kafka - Outbound
description: JSON Serialization Format for Aerospike Connect for Kafka outbound connector
---
## JSON Serialization Format

This document specifies the JSON data serialization format used by Aerospike
Connect for Kafka. It describes how Aerospike notification messages are represented as
JSON objects. Each message represents an event that occurred in an Aerospike
database cluster. Different kinds of messages are used to describe events such
as record creation/update and record deletions.

JSON (JavaScript Object Notation) is a light-weight data serialization format.
It is a text-based format that is easy for humans to read and write, as well as
easy for computers to parse and generate. JSON implementations exist for a wide
variety of popular programming languages. The official JSON specification can
be found at https://json.org/.

### JSON Schema

An informal description of the JSON message format can be found in the sections below. Alternatively, a formal
definition using [JSON Schema](http://json-schema.org) is provided in the
[`json-message-schema.json`](json-message-schema.json) document.

### Message Format

Each message consists of a JSON object. The type of the message is identified
by the value of the `msg` property. Further properties of the message object are
dependent on the message type, which is defined below.

| `msg`    | Message Type   | Description |
|----------|----------------|-------------|
| `write`  | Write Message  | Record creation or update; messages include the record key, record meta data as well as the values of all record bins (post update). |
| `delete` | Delete Message | Record deletion; messages include on the record key. |

#### Write Message

A Write Message object contains the following properties:

| Key    | Format | Name                | Description |
|--------|--------|---------------------|-------------|
| `msg`  | String | Message Format      | Always `write`. |
| `key`  | Array  | Record Key          | See below for definition of Key format. The record key in a Write message always contains the namespace and key digest, and may optionally contain the set name and user key components. |
| `gen`  | Number | Record Generation   | A record's generation value changes with each record update. |
| `exp`  | Number | Expiration Time     | Time when the record will expire, in seconds since the Unix epoch. Zero means the record will not expire. |
| `lut`  | Number | Last-Update Timestamp | Time when the record was last updated, in seconds since the Unix epoch. May be zero, if the last-update time is not known. [1] |
| `bins` | Array  | Record Bins         | List of record bin values. See below for definition of Bin format. |

[1] At present, Aerospike change notifications do not include the last-update
timestamp. Hence, the `lut` property will be zero for all Write message.
Last-update time will be populated once a future Aerospike server update
enables this feature.

#### Delete Message

A Delete Message object contains the following properties:

| Key       | Format  | Name           | Description |
|-----------|---------|----------------|-------------|
| `msg`     | String  | Message Format | Always `delete`. |
| `key`     | Array   | Record Key     | See below for definition of Key format. The record key in a Delete message only contains the namespace and key digest, but not the set name or user key components. |
| `durable` | Boolean | Durable Delete Flag | Indicates whether a tombstone was written for the deleted record. See the [Durable Deletes](/docs/guide/durable_deletes.html) documentation for further info. |

#### Key Format

The record key is an array with four elements: Namespace, set name, record digest and user key. Each key contains a namespace and digest; the set name and user key are optional and may be `null`:

```
+--------------------+-------------------+-----------------+-------------------------------+
| Namespace [string] | Set [string|null] | Digest [string] | User Key [string|number|null] |
+--------------------+-------------------+-----------------+-------------------------------+
```

* **Namespace**: Aerospike namespace.
* **Set**: Optional set name; may be `null`.
* **Digest**: Base64-encoded 160-bit record digest.
* **User Key**: The integer, string or bytes value used by the application to uniquely address the record. Will be included in the message only if it is stored on the server. Binary user keys use Base64 encoding.

#### Bins Format

Each recod bin is represented as a JSON object with the following properties:

| Key     | Format | Description |
|---------|--------|-------------|
| `name`  | String | Name of the bin |
| `type`  | String | Data type - see below |
| `value` | varies - see below | Value of the bin |

Aerospike supports many data types for bin values, including complex data types such as [Lists](/docs/guide/cdt-list.html), [Maps](/docs/guide/cdt-map.html) and [Geospatial](/docs/guide/geospatial.html) data (using GeoJSON). The data type of a bin in the JSON representation is specified in the `type` property of the Bin object. Depending on the specified data type, the `value` is represented as a JSON string, number, array or object. Depending on the data type, the Bin object may have additional properties.

| Bin `type` | Data Type | JSON format | Details & Additional properties |
|------------|-----------|-------------|---------------------------------|
| `str`      | String    | String      | - |
| `int`      | Integer   | Number      | - |
| `float`    | Double    | Number      | - |
| `blob`     | Bytes     | String      | Base64-encoded |
| `list`     | List      | Array       | `ordered` (boolean): Whether the list is ordered or not |
| `map`      | Map       | Object      | `order` (string, optional): One of `key`, `key-value` |
| `geojson`  | GeoJSON   | Object      | - |

Map and list elements can be of any allowed data type, including nested lists and maps.

### Examples

#### Example "Write"-type message

```json
{
  "msg": "write",
  "key": ["ns", "set", "YWJjZGVmZ2hpamtsbW5vcHFyc3Q=", null],
  "gen": 123,
  "exp": 0,
  "lut": 1523859494,
  "bins": [
    {
      "name": "myString",
      "type": "str",
      "value": "a string value"
    },
    {
      "name": "myBlob",
      "type": "blob",
      "value": "QUJDREVGR0hJSktMTU5PUFFSU1RVVldYWVo="
    },
    {
      "name": "myList",
      "type": "list",
      "value": ["abc", "def", "ghi", "jkl"],
      "ordered": true,
    },
    {
      "name": "myMap",
      "type": "map",
      "value": {
        "i": 42,
        "f": 3.1415,
        "l": [3, 2, 1, 0]
      },
      "order": "key-value"
    },
    {
      "name": "myGeo",
      "type": "geojson",
      "value": { "type": "Point", "coordinates": [1.30824, 103.91327] }
    }
  ]
}
```

#### Example "Delete"-type message

```json
{
  "msg": "delete",
  "key": ["ns", null, "YWJjZGVmZ2hpamtsbW5vcHFyc3Q=", null],
  "durable": true
}
```
