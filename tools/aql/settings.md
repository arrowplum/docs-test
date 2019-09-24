---
title: aql – Settings
description: Learn how aql provides the ability to set and get settings, which can be set via command-line arguments, but can be overridden from within aql.
---

`aql` provides the ability to set and get settings for its environment. These settings can be set via command-line arguments, but can be overridden from within `aql`

To get the value of a setting, run:

```sql
aql> get <setting>
```

Where `<setting>` is the name of the setting.

The following is an example of getting the setting for `OUTPUT`:

```sql
aql> get output
OUTPUT = TABLE
```

To set the value of a setting, run:

```sql
aql> set <setting> <value>
```

Where

- `<setting>` is the name of the setting
- `<value>` is the value to change the setting to.

The following is an example of setting `OUTPUT=JSON`.

```sql
aql> set output json
```

The available settings are:

### ECHO

Echo commands.

```sql
ECHO ( TRUE | FALSE )
```

### OUTPUT

The output mode.

```sql
OUTPUT ( TABLE | JSON | MUTE | RAW )
```

Example

```sql
aql> set output table
```

### OUTPUT_TYPES

Disable printing of type of the data value such as GeoJSON, LIST, MAP_KEY_ORDERED etc.

```sql
OUTPUT_TYPES (FALSE | TRUE)
```

### VERBOSE

Enable verbose output.

```sql
VERBOSE ( TRUE | FALSE )
```

### TIMEOUT

The time, in milliseconds, to wait for a query to complete.

```sql
TIMEOUT <milliseconds>
```

### SOCKET_TIMEOUT

Socket idle timeout in milliseconds for a query.

```sql
SOCKET_TIMEOUT <milliseconds>
```

### LUA_USERPATH

The path to the user-managed lua files.

```sql
LUA_USERPATH <path>
```

### RECORD_TTL

The time, in seconds, that subsequently created or updated record will live before being evicted by the server.

```sql
RECORD_TTL <seconds>
```

### RECORD_PRINT_METADATA

Enable printing of metadata (digest, ttl, generation) of the record. Valid only in JSON view.

```sql
RECORD_PRINT_METADATA (TRUE | FALSE)
```

### REPLICA_ANY

Flag for sending the primary key lookup request to all the replicas in round robin manner, not just master.

```sql
REPLICA_ANY (TRUE | FALSE)
```

### KEY_SEND

Flag for sending the key to the server in read/write/delete operations.

```sql
KEY_SEND (TRUE | FALSE)
```

### DURABLE_DELETE

Flag for setting policy value on the underlying C API calls used by DELETE, INSERT, EXECUTE, and SELECT (when doing a scan) operations. Applicable only for server version 3.10+. See [Durable Delete](/docs/guide/durable_deletes.html) for details.

```sql
DURABLE_DELETE (TRUE | FALSE)
```

### FAIL_ON_CLUSTER_CHANGE

Fail scan in case cluster has ongoing data migrations. Default: TRUE.

```sql
FAIL_ON_CLUSTER_CHANGE (TRUE | FALSE)
```


### SCAN_PRIORITY

The parameter to set scan priority. Default: AUTO 

```sql
SCAN_PRIORITY (LOW, MEDIUM, HIGH, AUTO)
```

{{#note}}The setting values set using aql are valid only for the current session. A new invoking of aql would have the default values.{{/note}}

