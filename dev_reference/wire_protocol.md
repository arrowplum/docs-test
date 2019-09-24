---
title: Wire Protocol 
description: The Aerospike wire protocol is a purpose-built, binary-oriented protocol for getting and setting values in a Aerospike cluster.
---

The Aerospike wire protocol is a purpose-built, binary-oriented protocol for getting and setting values in a Aerospike cluster.

While Aerospike provides example clients in a number of languages, the primary supported interface is this protocol. While customers are encouraged to modify the provided clients to suit their needs and environment, Aerospike may extend this protocol to add services and features as necessary. New features, however, will be added in a backward-compatible way.

The protocol is a self-parsing binary protocol layered over TCP. Self-parsing is the method of using length specifiers in such a way that new information – fields, operations – can be added to the protocol, but old clients and servers can still parse the data stream effectively, responding with errors when they see messages they don’t understand.

## Aerospike Protocol Header

The protocol first has a fixed header section, which contains three fields: `type`, `version`, `length`. Two message types are currently supported: "Aerospike Info" and "Aerospike Message". Depending on the `type`, the appropriate message payload follows the fixed header section.

"Aerospike Info" is a simple, string-oriented message-type that allows a client to check the status of a cluster node, get information about the capabilities of the node, and explore the cluster.

"Aerospike Message" (`AS_MSG`) is a high-performance, efficient, binary-oriented message for getting and setting data within the cluster. The protocol supports pipelining, but not out-of-order responses.

Where appropriate, network byte order always applies.

| offset | name | size (bytes) | meaning |
|--------|---------|-------------|
| 0  | version | 1 | current protocol version is `2` |
| 1  | type | 1 | current defined message types are: `1` ("Aerospike Info") and `3` ("Aerospike Message") |
| 2  | sz | 6 | number (in network byte order) of bytes in this message to follow this header |

## Aerospike Info: Type 1 - Info Message

This simple but critical message type allows clients to determine the capabilities of each node in the Aerospike cluster using a simple name-value pattern.

Besides providing version and feature information, `info` request enables a client to discover all nodes in the cluster. This allows a client to continually adjust to cluster members without requiring application-level configuration changes.

The message is comprised of name-value pairs. All values are case-sensitive, and UTF-8 encoded. When possible, names and values are preferably limited to 7-bit ASCII characters (i.e., the 8th bit is `0`.) If the value represents a list, a common separator is semi-colon (`;`) Below list examples of the currently supported Info message name and value pairs.

## Example Info Message Names and Values

| name | value(s) |
|------|----------|
| build | server build version, e.g., "3.5.14" |
| edition | server edition, e.g., "Aerospike Enterprise Edition" |
| node | unique string representing this node's ID as a 64-bit hexadecimal number, e.g., "BB9E68F98290C00" |
| replicas-read | list of read replicas hosted by this node, represented as a list with entries of the form: `<Namespace>:<PartitionID>;` |
| replicas-write | list of write replicas hosted by this node, represented as a list with entries of the form: `<Namespace>:<PartitionID>;` |
| service | this node's `<IPAddress>:<TCPPort>` where it listens for Aerospike protocol message transactions |
| services | semicolon-delimited list of service address and port pairs where the other nodes in this cluster can be found |
| statistics | semicolon-delimited list of statistics regarding this node's current state |
| version | full server edition and build version, e.g., "Aerospike Community Edition build 3.5.14" |

## Aerospike Message: Type 3 – AS_MSG

A Aerospike Message is the basic message structure used to request an object read/write, as well as functionalities such as scan. The client sends a request, and the server sends a response.

The request message contains a small, fixed size header, followed by a variable number of "fields." (See `Fields` section), and then followed by a variable number of operations (See `Operations` section below.) In case of future expansion, the `header_size` field will increase, and extra header values will be added to the end of the fixed header.

## Aerospike Message: Header Section

| offset | name | size (bytes) | description |
|--------|------|------|-----------|
| 0  | header_sz | 1 | number of bytes in this header (always equals `22` for protocol version `2`.) |
| 1  | info1 | 1 | bitfield of flags (mostly read-related) |
| 2  | info2 | 1 | bitfield of flags (mostly write-related) |
| 3  | info3 | 1 | bitfield of flags (mostly write-related) |
| 4  | unused | 1 | an unused byte |
| 5  | result_code | 1 | on a response, whether the request succeeded or failed; `0` on requests |
| 6  | generation | 4 | on request, apply this transaction only if the generation matches; on response, the current generation of this record |
| 10 | record_ttl | 4 | on request, if `> 0`, set the expiration of this record to this number of seconds in the future; `0` means use the namespace default TTL; `-1` means no expiration (use with caution) |
| 14 | transaction_ttl | 4 | on request, set transaction expiration time to this value; _except_ on new batch request, this field stores the batch index |
| 18 | n_fields | 2 | number of fields to follow in the data payload, which will come first |
| 20 | n_ops | 2 | number of operations to follow in the data payload, which will follow the fields |
| 22 | data[] | `proto.sz - header_sz` (i.e., the protocol header's `sz` - 22) | data contains the fields first, followed by the ops |

The `info1` field is a set of flags specifying the overall type of the operation. These flags refer to the entire transaction, while there may be different operations on each bin.

|value| name | description|
|-----|------|------------|
| 1   | AS_MSG_INFO1_READ | contains a read operation |
| 2   | AS_MSG_INFO1_GET_ALL | get all bins' data |
| 4   | unused | unused |
| 8   | AS_MSG_INFO1_BATCH | new batch protocol |
| 16  | AS_MSG_INFO1_XDR | operation is performed by XDR |
| 32  | AS_MSG_INFO1_NOBINDATA | do not read the bin information |
| 64  | AS_MSG_INFO1_CONSISTENCY_LEVEL_B0 | read consistency level - bit 0 |
| 128 | AS_MSG_INFO1_CONSISTENCY_LEVEL_B1 | read consistency level - bit 1 |

The transactions's requested Read Consistency Level, which can be overridden by the server, is determined by the value of the two consistency level bits, `<B1:B0>`:
<UL> Read Consistency Level `ONE` is `00`.
<UL> Read Consistency Level `ALL` is `01`.
<UL> (All other consistency level bit patterns are reserved.)

The ‘info2′ field is a set of flags specifying the overall type of the operation. These flags refer to the entire transaction, while there may be different operations on each bin.

| value | name | description |
|-------|------|-------------|
| 1   | AS_MSG_INFO2_WRITE | contains a write operation |
| 2   | AS_MSG_INFO2_DELETE | delete record |
| 4   | AS_MSG_INFO2_GENERATION | pay attention to the generation |
| 8   | AS_MSG_INFO2_GENERATION_GT | apply write if new generation >= old, good for restore |
| 16  | unused | unused |
| 32  | AS_MSG_INFO2_CREATE_ONLY | write record only if it doesn't exist |
| 64  | AS_MSG_INFO2_BIN_CREATE_ONLY | write bin only if it doesn't exist |
| 128 | AS_MSG_INFO2_RESPOND_ALL_OPS | all bin ops (read, write, or modify) require a response, in request order |

The `info3` field is a set of flags specifying the overall type of the operation. These flags refer to the entire transaction, while there may be different operations on each bin.

| value | name | description |
|-------|------|-------------|
| 1    | AS_MSG_INFO3_LAST | this is the last of a multi-part message |
| 2    | AS_MSG_INFO3_COMMIT_LEVEL_B0 | write commit level - bit 0 |
| 4    | AS_MSG_INFO3_COMMIT_LEVEL_B1 | write commit level - bit 1 |
| 8    | AS_MSG_INFO3_UPDATE_ONLY | update existing record only, do not create new record |
| 16   | AS_MSG_INFO3_CREATE_OR_REPLACE | completely replace existing record, or create new record |
| 32   | AS_MSG_INFO3_REPLACE_ONLY | completely replace existing record, do not create new record |
| 64   | AS_MSG_INFO3_BIN_REPLACE_ONLY | replace existing bin, do not create new bin |
| 128  | &lt;Unused&gt; | (Reserved.) |

The transactions's requested Write Commit Level, which can be overridden by the server, is determined by the value of the two commit level bits, `<B1:B0>`:
<UL> Write Commit Level `ALL` is `00`.
<UL> Write Commit Level `MASTER` is `01`.
<UL> (All other commit level bit patterns are reserved.)

Note that in many cases the read and write fields must be combined with other fields. For example, if a record is to be deleted, the write bit must also be set. Similarly, for a `get all` operation, the `read` bit must be set.

A record can contain both read and write operations. The writes will be applied before the read operations.

## Aerospike Message: Fields

Fields are per-request information that identifies the record to be operated on.
Typical values are the namespace, the key (or digest), the set, and other
information to determine the record. Generally multiple fields are required to uniquely identify a (list of) record(s.)

| offset | name | size (bytes) | description |
|--------|------|------|-------------|
| 0 | size | 4 | size of data to follow |
| 4 | field_type | 1 | type of data - allowed field types are described below |
| 5 | data | size - 1 | data |

### Allowed Field Types

| value | name | description |
|-------|------|-------------|
| 0 | AS_MSG_FIELD_TYPE_NAMESPACE | namespace |
| 1 | AS_MSG_FIELD_TYPE_SET | a particular `set` within the namespace |
| 2 | AS_MSG_FIELD_TYPE_KEY | the key |
| 3 | AS_MSG_FIELD_TYPE_BIN | (Unused.) |
| 4 | AS_MSG_FIELD_TYPE_DIGEST_RIPE | the RIPEMD160 digest representing the key (20 bytes) |
| 5 | AS_MSG_FIELD_TYPE_GU_TID | (Unused.) |
| 6 | AS_MSG_FIELD_TYPE_DIGEST_RIPE_ARRAY | an array of digests |
| 7 | AS_MSG_FIELD_TYPE_TRID | transaction ID |
| 8 | AS_MSG_FIELD_TYPE_SCAN_OPTIONS | scan operation options |
| 21 | AS_MSG_FIELD_TYPE_INDEX_NAME | secondary index name |
| 22 | AS_MSG_FIELD_TYPE_INDEX_RANGE | secondary index query range |
| 26 | AS_MSG_FIELD_TYPE_INDEX_TYPE | secondary index type |
| 30 | AS_MSG_FIELD_TYPE_UDF_FILENAME | udf file name |
| 31 | AS_MSG_FIELD_TYPE_UDF_FUNCTION | udf function |
| 32 | AS_MSG_FIELD_TYPE_UDF_ARGLIST | udf argument list |
| 33 | AS_MSG_FIELD_TYPE_UDF_OP | udf operation type |
| 40 | AS_MSG_FIELD_TYPE_QUERY_BINLIST | bins to return on a secondary index query |

## Aerospike Message: Operations

Operations describe the actions that are to be taken on the specified bin(s) within the identified record.

Operations are of the following format:

| offset | name | size (bytes) | description |
|--------|------|------|-------------|
| 0 | size | 4 | number of bytes to follow |
| 4 | op | 1 | operation to apply (allowed types described below) |
| 5 | bin data type | 1 | type of data to follow |
| 6 | bin version | 1 | unused (always set to 0) |
| 7 | bin name length (N) | 1 | size of bin name to follow |
| 8 | binname | N | bin name in UTF-8 |
| 8 + N | data | size - (N + 4) | bin data according to the bin_data_type |

### Allowed Operations

| value | name | description |
|-------|------|-------------|
| 1  | AS_MSG_OP_READ | read the value in question |
| 2  | AS_MSG_OP_WRITE | write the value in question |
| 3  | &lt;Unused&gt; | (Reserved.) |
| 4  | &lt;Unused&gt; | (Reserved.) |
| 5  | AS_MSG_OP_INCR | arithmetically add a value to an existing value, works only on integers |
| 6  | &lt;Unused&gt; | (Reserved.) |
| 7  | &lt;Unused&gt; | (Reserved.) |
| 8  | &lt;Unused&gt; | (Reserved.) |
| 9  | AS_MSG_OP_APPEND | append a value to an existing value, works on strings and blobs |
| 10 | AS_MSG_OP_PREPEND | prepend a value to an existing value, works on strings and blobs |
| 11 | AS_MSG_OP_TOUCH | touch a value without doing anything else to it - will increment the generation |

### Memcache-Compatible Operations

| value | name | description |
|-------|------|-------------|
| 129 | AS_MSG_OP_MC_INCR | Memcache-compatible version of the increment command |
| 130 | AS_MSG_OP_MC_APPEND | append the value to an existing value, works only strings for now |
| 131 | AS_MSG_OP_MC_PREPEND | prepend a value to an existing value, works only strings for now |
| 132 | AS_MSG_OP_MC_TOUCH | Memcache-compatible touch - does not change generation |

## Aerospike Message: Response

The response to an Aerospike Message very much mirrors the request. Responses can be chunked. Each chunk is first prefaced with the Aerospike Protocol Header, then followed by multiple Aerospike Message with Operations and Fields, one for each record.

## Aerospike Info: Response

The response to an Aerospike Info request is simply ascii name/value pairs. No extra binary headers.
