---
title: Managing Sets in a Namespace
description: Learn how to set and manage Sets in a Namespace.
---


### Overview

An Aerospike “set” is similar to a table in a relational database. One of the big difference is that with Aerospike, you do not need to predefine a schema. Thus, sets are created dynamically and implicitly on first record insertion in set.

{{#info}}
There is a limit of 1023 on the total number of sets in a namespace. Deleting a set metadata (to reclaim space from this 1023 limit) requires durably deleting all the data in the set and cold restarting the node. See the follwing [knowledge base article](https://discuss.aerospike.com/t/how-to-clear-up-set-and-bin-names-when-it-exceeds-the-maximum-set-limit/3122) for details.
{{/info}}

Below are some basic configurations and commands which can be used to manage data in a set.

- [truncate](/docs/reference/info#truncate) introduced in Aerospike Server version 3.12.0, released in March 2017.
- [set-delete](/docs/reference/info#sets) deprecated and works up to Aerospike 3.12.1, released in April 2017. 

The version of the Aerospike Server being maintained will determine which of the following asinfo commands to execute.

### Truncating a Set in a Namespace

{{#warn}}
The truncate related commands should be used with extreme caution. Providing an incorrect command can result in truncating a whole namespace rather than just a set.<br><br>

Server versions 4.3.1.11 (and higher 4.3.1.X), 4.4.0.11 (and higher 4.4.0.X), 4.5.0.6 (and higher 4.5.0.X) and 4.5.1.5 and higher 
provide 2 different commands for truncation:<br>
- [`truncate`](/docs/reference/info/index.html#truncate) for truncating sets.<br>
- [`truncate-namespace`](/docs/reference/info/index.html#truncate-namespace) for truncating namespaces.<br><br>

The AQL command abstracts both in the single TRUNCATE command which takes an optional set parameter.
{{/warn}}

Introduced in Aerospike Server version 3.12, the "truncate" command removes all records belonging to a set or a namespace. To initiate a set truncation, 
the [asinfo](/docs/tools/asadm/index.html) or [aql](/docs/tools/aql/index.html) tools can be used to issue a truncate command to one of the nodes in the cluster.

For truncating a set, use either one of the following commands:

```
asinfo -v "truncate:namespace=namespace_name;set=set_name;"
```

```
aql -c "TRUNCATE <namespace_name>.<set_name>"
```

{{#note}}
For Server versions 4.5.1.5 and above, it is recommended to use the `asinfo` command due to the clear separation between `truncate` for truncating a set and `truncate-namespace` for truncating a namespace. For older versions (prior to 4.5.1.5 unless covered by the hotfixes introduced in 4.3.1.11, 4.4.0.11 and 4.5.0.6) it is safer to use the `aql` command to avoid the risk of a typo in the command truncating a whole namespace inadvertently.
{{/note}}

The command starts a background thread on each of the nodes in the cluster, which removes, from primary index, all records belonging to namespace_name and set_name on the node.  

The command takes an optional historical `lut`(last-update-time) parameter X. The timestamp limits the truncation to records with a last-update-time earlier than time X. If no timestamp is specified, the `now` timestamp of the receiving node is used.

To validate that a set is truncated, issue the following info command to get set truncation information and set object count information.

```
# 3 objects in set 'testset', truncate_lut=0
asinfo -v 'sets/test/testset'

objects=3:tombstones=0:memory_data_bytes=0:truncate_lut=0:deleting=false:stop-writes-count=0:set-enable-xdr=use-default:disable-eviction=false

# Issue a truncate command on 'testset'
asinfo -v 'truncate:namespace=test;set=testset;'

# see that the set now has 0 objects, and a non-zero truncate_lut
asinfo -v 'sets/test/testset'

objects=0:tombstones=0:memory_data_bytes=0:truncate_lut=229115378244:deleting=false:stop-writes-count=0:set-enable-xdr=use-default:disable-eviction=false
```

Set truncation can be issued repeatedly. It updates the effective last-update-time of affected set, and is reflected in the `lut` field of the set information.

For truncation on sets with secondary indexes, each record data is actively read, and any secondary index entries associated with the record are also removed.

In the Enterprise Edition, truncation is durable and preserves record deletions through a cold-restart. In the Community Edition, similar to record deletes, records in previously truncated sets are not durable and deletes can return through a cold-start.

More information on understanding the `truncate` command can be found in [Info Command Reference - Truncate](/docs/reference/info#truncate).

{{#note}}
The [`truncate-undo`](/docs/reference/info/index.html#truncate-undo) and 
[`truncate-namespace-undo`](/docs/reference/info/index.html#truncate-namespace-undo) commands can be used to remove the truncate related entry(ies) in SMD to allow 
for potential recovery of inadvertently truncated records upon a subsequent [Cold Restart](/docs/operations/manage/aerospike/cold_start/). Truncated records that would have been already overwritten on persistent storage would not be recoverable. 
{{/note}}

### Deleting a Set in a Namespace (Deprecated)

This section refers to the deprecated `set-delete` call. For server versions 3.12 and above, please use the `truncate` call.

Some key limitations in using the `set-delete` command, which are addressed in the `truncate`command:
- Cannot be issued during migrations.
- Cannot be issued while new data is being written into the set
- Does not durably delete the records in the Enterprise Edition.
- Dependency on the nsup thread to get to the namespace the set belongs to.
- Length of the procedure if there are a lot of records to delete (waiting for prole side deletes).
- Does not provide an option to delete data only beyond a certain time stamp (`lut`).
- Necessity to be issued on each node in the cluster.

When deleting a set from a namespace, make sure that no writes are being done on that set. Also, note that the deletion of data in the set precedes the deletion of set itself. So, if there is data being written to the set, data deletion will take precedence over the set deletion. Basically, set deletion takes place in the next namespace supervisor (nsup) cycle thread so deletion might take time depending on the size of the set.

The period of the namespace supervisor cycle is set by the `nsup-period` configuration value.

To dynamically delete a set, we need to update a configuration parameter in the namespace context as follows. 
```
asinfo -v "set-config:context=namespace;id=namespace_name;set=set_name;set-delete=true;"
```

To completely eliminate the data and metadata, you should delete the records associated with a set across the cluster. It is recommended to not delete a set when there are ongoing migrations since this might require the set-delete to be issued again to delete records form partitions which were in flux while the set-delete command was issued.

You can confirm if a 'set' is empty by executing the following on `aql`. Information on [Using aql](/docs/tools/aql).

```
aql> SHOW SETS
```
{{#info}}
Note: The name of the set will still be visible until the entire cluster is restarted, however no memory will be taken up by an empty set in the namespace.
{{/info}}

By default `set-delete` is false. When dynamically set to true, it initiates the namespace supervisor cycle, deletes the set
and automatically modifes its value to false after the deletion of the complete set.

{{#info}}
For server versions 3.7.0.1 and above, issuing a set-delete will also trigger a clean-up of the associated secondary indices on the data in the set. Note that the set-delete flag will be shown as true while the secondary index is garbage collected post deletion and will switch to false when completed.
{{/info}}

In order to delete all the objects in the set in a cluster, `set-delete` needs to be dynamically set to true on all nodes. If configured only on a single node, only the objects of which the node is the master will be deleted.

{{#info}}
Note: There is an upper limit of 1023 unique sets in a namespace and a set-name length limit of 63 characters. A set-name cannot contain the ':' or ';' characters.
{{/info}}

Additional Notes:
- We cannot rename a set, only mark it for deletion. 
- Removing the metadata associated with Sets is not supported across a [fast restart](/docs/operations/manage/aerospike/fast_start/index.html).
- In considering the case of a cold start, the index will be rebuilt from persistent storage and hence, deleted data residing on write blocks that have not been defragmented and overwritten with new data will reappear. In case of a [fast restart](/docs/operations/manage/aerospike/fast_start/index.html), deleted data will not reappear.
- In order to delete the set data cleanly, ensure that `set-delete` shows as false after executing the deletion command, and drop any roles with privileges on the set in that namespace prior to deleting the set. Following this, you would need to stop a node and zeroize the drives that may contain that set (the drives associated with the namespace the set is on). Finally, start the node back up, wait for migrations to complete and continue to the next node.


### Managing data in a Set

As of Aerospike 3.6.1, a set can be protected from evictions, and as of Aerospike 3.7.0.1, a set can have a maximum record count cap.

For examples on configuring this, go to [Configure Data Retention](/docs/operations/configure/namespace/retention).

#### `set-disable-eviction`

With this setting turned on, a set will be protected from evictions. If evictions are in effect, records on a set with set-disable-eviction turned on will not be evicted, regardless of their expiration time.

#### `set-stop-writes-count`

If a set exceeds the `set-stop-writes-count` records, further writes will start failing on the set. Basically, this is the number of objects in the set beyond which we stop all writes to that specific set. Note: that once this value is breached, only the master writes are stopped. The replica writes will still occur at that node which might make the total objects cross the `stop-writes-count`. However, once all the nodes have hit the `stop-writes-count`, writes will stop at all nodes.


More information on understanding Sets can be found in [Data Model](/docs/architecture/data-model.html).


