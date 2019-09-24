---
title: Aerospike Loader Options
description: Options for the asloader tool.
---

| Options | Description | Default |
|---|---|---|
| -h &lt;hosts&gt; | List of seed hosts, where Aerospike servers are running | 127.0.0.1 |
| -p &lt;port&gt;  | Port to use with the host specified in the -h option. | 3000 |
| -U &lt;user&gt;  | User name |  |
| -P &lt;password&gt; | Password | |
| -n &lt;namespace&gt; | Namespace to load data into. | test  |
| -c &lt;config&gt;  | JSON formatted configuration file specifying parsing attributes and schema mapping. |  |
| -g &lt;max-throughput&gt; | Set a maximum target transactions per second for the loader. | 0 (no throttling) |
| -T &lt;transaction-timeout&gt;  | (In milliseconds) Timeout for a transaction during write operation.  | 0 (no timeout) |
| -e &lt;expiration-time&gt;      | Expiration time of records in seconds. Other valid values are: set it to -1 for records to never expire and 0 to use server default. | -1 |
| -tz &lt;timezone&gt;            | Time zone of data backup source. This value is used when loading data of timestamp datatype. For example, if data backup location timezone is X and that data is to be loaded into server located in Y timezone, then specify X's timezone. Valid values are standard three-letter codes such as PST, EST, etc. |  local timezone |
| -ec &lt;abort-error-count&gt; | Error threshold to determine when the loader should stop loading data. Set it to 0 to ignore the threshold. | 0 |
| -wa &lt;write-action&gt;  | The possible values are: 1) UPDATE - Create or update records. Merge incoming bin values with existing ones, 2) UPDATE_ONLY - Update existing records. Fail if record does not exist. Merge incoming bin values with existing ones, 3) REPLACE - Create or replace existing records, 4) REPLACE_ONLY - Replace existing records. Fail if record does not exist, 5) CREATE_ONLY - Create new records. Fail if record already exists. | UPDATE |
| -tls &lt;tls-enable&gt; | Use TLS/SSL sockets  | False  |                                                                        
| -tp &lt;tls-protocols&gt;  | Allow TLS protocols. Values:  TLSv1,TLSv1.1,TLSv1.2 separated by comma | TLSv1.2)  |
| -tlsCiphers &lt;tls-cipher-suite&gt;  | Allow TLS cipher suites. Values:  cipher names defined by JVM separated by comma | null (default cipher list provided by JVM) |
| -tr &lt;tls-revoke&gt; | Revoke certificates identified by their serial number. Values:  serial numbers separated by comma | null (Do not revoke certificates) |
| -te &lt;tls-encrypt-only&gt; | Enable TLS encryption and disable TLS certificate validation | |
| -uk &lt;send-user-key&gt;  | Send user defined key in addition to hash digest to store on the server. (default: userKey is not sent to reduce meta-data overhead). | |
| -v   | Verbose mode.  If this option is specified, verbose mode is enabled and additional information is displayed on the console. | DISABLED  |
| -u   | Display command usage. |  |
| -V   | Print the asloader version. |  |  |

