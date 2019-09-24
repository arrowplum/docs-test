---
title: Datacenter Replication Management
description: Managing Aerospike Database
---

{{#todo markdown=true}}
- Section to explain file and directories used by XDR.
{{/todo}}

## Versions 3.8 and above

As of Aerospike Server Enterprise Edition version 3.8, the Cross Datacenter Replication 
(XDR) service is built in within the main Aerospike service (asd). As such it is 
always started and stopped when the Aerospike server is started and stopped.

{{#todo markdown=true}}
Review following with Sunil before publishing:

asinfo -v "xdr-command:resume" [-Udata_admin -Pdata_admin]
STATUS : XDR_COMING_UP

asinfo -v "xdr-command:noresume" [-Udata_admin -Pdata_admin]
STATUS : XDR_COMING_UP

asinfo -v "xdr-command:status" [-Udata_admin -Pdata_admin]
STATUS : XDR_UP

asinfo -v "xdr-command:stop" [-Udata_admin -Pdata_admin]
STATUS : XDR_GOING_DOWN

asinfo -v "xdr-command:status" [-Udata_admin -Pdata_admin]
STATUS : XDR_DOWN

{{/todo}}

## Versions up to 3.7.5
The XDR daemon can be controlled using the SysV init script located at
`/etc/init.d/aerospike_xdr` which supports the following options:
- resume [default]
- resume-nofailover
- noresume
- stop
- status

### Resume XDR from a Previous Execution [Default]
Starts XDR and will resume of shipping of the log from start of the log. Along
with shipping data of current node it also takes responsibility of shipping of
data for the nodes which are not present in the cluster currently. It will be
able to ship data of a node which did not come up only if the digest log on this
node has the shadow logs of it. This should be used when multiple nodes have
gone down and all could not be brought back online. If the master and replica
nodes corresponding to some data cannot be brought back online for any reason,
obviously the data cannot be shipped because both the copies of the
corresponding log will not be processed.
```
/etc/init.d/aerospike_xdr resume
```

### Resume XDR but don't Ship for Others
Starts XDR with resumption of shipping of log from the point the last failure
happened. However, the XDR will not take the responsibility of shipping on
behalf of the nodes that did not come up. This should be used when a certain
node(s) goes down and is brought up again and it is known that the shipping for
the replica need not be done. If all the nodes are restarted each of them takes
care of its own logs, all the log records should be covered. This is useful when
doing a rolling upgrade or planned shutdown of the entire cluster when it is
sure that all the nodes that are taken down can be brought up again.

XDR needs to be fault tolerant. So, when a machine is shutdown (or has a system
failure), it should not lead to loss of data that is supposed to be shipped to
the remote cluster. To handle this case, XDR writes the corresponding log
records in the digest log (shadow log) at the replica site also along with the
data. i.e, XDR records the information about what data has to be shipped to the
remote cluster at the replica site also. But XDR does not ship using the shadow
log unless the master node failed.

Once a node failed, its replica nodes will take over the responsibility of
shipping the data that this node is earlier responsible for. This way XDR
ensures that there is no data loss during a single node failure.
```
/etc/init.d/aerospike_xdr resume-nofailover
```

### Start XDR with a Clear Backlog
Starts XDR afresh. Please note that once XDR starts with noresume all the old
log records in the digest log are lost. This should be done when doing a fresh
start. This can also be used in the event of a single node failure when that
failed node is being restarted. When a single node fails, the responsibility of
shipping the records belonging to it is taken over by XDR on a different node in
the cluster. So, the XDR on the failed node, when it restarts, need not resume
from the point at which it left. It can ignore them and continue to ship only
the freshly written data.
```
/etc/init.d/aerospike_xdr noresume
```

### Stop XDR
To shutdown the XDR service use the `stop` command:
```
/etc/init.d/aerospike_xdr stop
```

### Get Running Status of XDR
To determine if the XDR service is currently running, use the `status` command:
```
/etc/init.d/aerospike_xdr status
```
