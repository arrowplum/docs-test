---
title: Known Limitations
description: Known Limitations
---

The following is a summary of the limitations within the Aerospike database:


| Item | Limit | Current Limitation |
| ---  | ---   | ---             |
| Set  | Number of sets	| There is currently a limit of 1023 sets per namespace. | 
| Set  | Characters | A set name cannot contain the ':' or ';' characters. |
| Sindex | Bytes size | A bin will be eligible for secondary indexing if its value size is <= 2048 Bytes for a STRING sindex type or <= 1MB for GEOJSON sindex type.  |
| Bin  | Number of bins	| There is a maximum number of 32,767 bins per namespace.  This is hard-coded and cannot be altered. |
| Bin  | Name Length | 15 characters of any single byte.  Double-byte characters are not allowed. Limit is 14 characters for server versions prior to 4.2. |
| Namespace	| Number | Maximum of 32 for Enterprise, 2 for Community (as of version 4.0). A set-name cannot contain the ':' or ';' characters. |
| Namespace | Storage | Namespaces are like tablespaces for traditional SQL databases.  Note that you can have more than one SSD associated with a namespace, but any SSD used as a raw device can only be associated with a single namespace. Each namespace is limited to a maximum of 128 devices (Refer to 'Devices' description below for details). |
| Namespace | Characters | A namespace name cannot contain the ':' or ';' characters. |
| Record | Number | Actual number limited by RAM and storage.  Every record takes 64 bytes for the index entry.  The index entry is ONLY stored in RAM.  The key itself is not actually stored in the index, but the hash of the key (using RIPE-MD 160 algorithm) is.  This hash with overhead takes exactly 64 bytes. The maximum number of records per namespace on a given node is limited to 4,294,967,296 on the Community Edition (2^32 due to 4 bytes used for storing references), and 34,359,738,368 on the Enterprise Edition (2^35). This represents 2TiB of RAM.|
| Record | Total value size | Every record value size is limited to the [write-block-size](/docs/reference/configuration/index.html#write-block-size), which is typically set in the configuration to 128KB for Flash devices, but can be increased up to 8MiB as of version 4.2 (for prior versions, maximum is 1MiB which is also the default if not specified in the configuration file).|
| Record | Number of bins | There is no built-in limit to the number of bins in a record, but there is a limit to the number of bins overall in a namespace.  Currently this is 32,767. |
| Record | Sets | A record can only belong to one set.  Note that the set and key are both put into the hash, so in order to get the value for a key with a set, you have to know the set. |
| Cluster Size | Number of nodes | For Enterprise Edition, the maximum number of nodes in a cluster is 128. For versions prior to 3.14 running the heartbeat protocol v2, the limit is defined by the configuration [paxos-max-cluster-size](/docs/reference/configuration/index.html#paxos-max-cluster-size) (with a default of 31). Refer to this page regarding the process to increase the default for an already running cluster: [Increase Maximum Cluster Size](https://discuss.aerospike.com/t/how-to-add-more-than-31-nodes-to-a-cluster/2031). For Community Edition, the maximum number of nodes is limited to 31 for versions prior to 4.0 and limited to 8 for versions 4.0 and above. |
| Devices | Number of devices | The maximum number of devices (or files) per namespace per node is 128 as of version 4.2, 64 for versions down to 3.12.1, and 32 for versions prior to that. |
| XDR | Number of Data Centers(DCs) | For Enterprise Edition, a maximum number of 32 Data Centers can be configured for Cross Data Center Replication(XDR). |
| Batch Reads | Record size limit | For versions prior to 4.5.3.4, there is a size limit check of 10MB per record in a batch result. So if the batch read transaction contains a record > 10MB, the transaction will fail with `AEROSPIKE_ERR_RECORD_TOO_BIG(13)` error. This is valid only for `storage-engine memory` namespaces. |

The following limits also apply:


| Aerospike server item | limits |
| ---                   | ---    |
| bin-names | <= 14 characters |
| set-names | <= 63 characters (':' or ';' not allowed in set-name) |
| namespace-names | <= 31 characters  (':' or ';' not allowed in namespace-name) |
| Total bins across all keys in a namespace	| < 32 K |
| Total secondary indices in a namespace	| <= 256 indices |
| Bins per key | < 32 K |
| Sindex name | <= 255 characters (':' or ';' not allowed in index name)|
| Sindex Key size | <= 2048 Bytes (STRING) or <= 1MB (GEOjSON)|
| hist-track-back | 86400 seconds (slice of 10 seconds) |
| replication-factor | = number of nodes is cluster |
| Maximum configurable size of each file or disk for a namespace | = 2 TiB |
| Security user name (Enterprise only) | <= 63 characters  (':' or ';' not allowed in user name) |
| Security role name (Enterprise only) | <= 63 characters |
| Security password (Enterprise only) | [Valid password contructs](https://www.aerospike.com/docs/guide/security/access-control.html#passwords) |
| Number of Interfaces | <= 500 (For server versions prior to 3.15, the limit is 50) |
 
