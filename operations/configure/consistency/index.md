---
title: Configuring Strong Consistency
description: Configure for a consistent database which does not lose data
---

This section will guide you through the process of setting up and configuring a namespace with "strong-consistency".

- Make sure you have NTP configured
- Add config file entries to denote strong consistency namespaces
- Configure the initial roster
- Configure Node IDs with SC
- Configuring Rack Awareness in an SC environment

## Install and Configure NTP ( or other clock synchronization )

Strong Consistency mode is sensitive to clock skew and will enter 'stop-writes' clock skew exceeds 27 seconds.

Computer hardware tends to skew from true time, sometimes as much as by seconds a day. Aerospike's gossip clustering protocol
will monitor the amount of skew in a cluster, and alert if the a large amount of skew has been detected. SC namespaces will
stop taking writes if skew is detected at a larger interval - lower than the point where data loss occurs.

In order to avoid any issues, Aerospike recommends using a clock synchronization system. The system does not need
to be very accurate. Second, or 10 second, accuracy is acceptable. It is also acceptable if clocks pause or go through
periods of correction (accelerated clocks), as long as the total skew in the cluster is less than 27 seconds.

NTP will assist in keeping clocks synchronized. The most common method of synchronizing clocks is the common 
NTP protocol, which easily exceeds the granularity that Aerospike demands.

A guide to installing NTP [can be found at here](https://discuss.aerospike.com/t/how-to-configure-ntp-for-aerospike/3907), excerpt follows:

Ubuntu and Debian systems:
```
sudo apt install ntp
```

Centos and RHEL systems
```
yum -y install ntp
```

Edit config file with local or remote time servers. Typically you should use multiple servers. 
If you do not already have an NTP server system, you should consider using the ntp pool servers, 
such as us.pool.ntp.org for the United States. 

For further information regarding NTP server availability, please see [Selecting Offsite NTP Servers](http://support.ntp.org/bin/view/Support/SelectingOffsiteNTPServers).

To configure NTP servers in a public cloud, there are typically cloud-specific servers. For example, documentation regarding the 
Amazon EC2 NTP servers is found [here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html). Similar documents
exist for Azure, Google Compute Engine, IBM Cloud, and others.

```
vi /etc/ntp.conf
```

Reload the service:
```
sudo systemctl reload ntp.service
```

Check the ntpd status:
```
sudo ntpq -p
```

## Configure Node IDs

By default, Aerospike derives the node-id from the server's mac address (or configured node-id-interface). 

You will see in the subsequent commands for roster maintenance, use of hardware generated node ids, while convenient for Availability
mode's automatic algorithms, can become cumbersome when managing Strong Consistency.

Please consider using configured node-ids for your Strong Consistency cluster.

To allow friendlier node-ids, which simplify roster management, and including IDs that can be stable in the face of 
network interface hardware changes, the user may now optionally 
configure a specific 1 to 16 character hexadecimal node-id for each node. For example, "01" and "B2" and "beef" are valid
node ids, but "fred" is not.

This feature is also desirable with cloud providers such as Amazon AWS.
Specifying a node ID which matches a particular EBS volume allows a node to be taken out,
a new instance created, then Aerospike brought up with the new instance with the previous node ID.
This minimizes data migration.

If you mistakenly reuse a node-id, the cluster will detect the duplicate IDs, and refuse to allow these nodes 
to join the cluster. The log file(s) within the cluster will state that the cluster can't be joined due to 
duplicate cluster IDs.

In order to operationalize specified node IDs, it is best to start with specified node-ids during initial 
cluster formation and installation. 

Please follow the subsequent technique for starting with a set of initial node-ids.

If you wish to convert a cluster from automatic node-ids to specified node-ids, 
you may change the configuration file for a server and restart. For an SC namespace, 
you will have to change the roster. To prevent data unavailability, if you are changing multiple servers, 
update a single server, restart it, modify and commit the new roster, then repeat for the next server. 
You do not have to wait until data migration finishes, but you must validate and apply the new roster.

To configure the node-id, edit "/etc/aerospike/aerospike.conf" and add "node-id <HEX NODE ID>" to the service context.

```
service {
    user root
    group root
    service-threads 4
    transaction-queues 4
    transaction-threads-per-queue 4
    nsup-period 16
    proto-fd-max 15000
    
    node-id a1
}
```

When the server starts, validate that the node-id is as expected, then change the roster of any 
SC namespace intending to include this server.

{{#warn}}
The configuration file options [`node-id`](/docs/reference/configuration#node-id) and [`node-id-interface`](/docs/reference/configuration#node-id-interface) are mutually exclusive.
{{/warn}}

## Install & Configure for Strong Consistency

Add the [feature-key-file](https://www.aerospike.com/docs/reference/configuration#feature-key-file) path to your service stanza. The feature key file contains a digitally signed key to enable the Strong Consistency feature.
Please contact Aerospike Sales for further information on Strong Consistency licenses.


```
service {
    ...
    # If default location kept, no need to add this line
    feature-key-file  /etc/aerospike/features.conf    
    ...
}
```

For each namespace where you desire strong consistency, add `strong-consistency true` and `default-ttl 0` to the namespace stanza.
Although consistency can be used with expiration and TTL, they must be used carefully. 
Please see the section below about consistency, eviction, and expiration.

```
namespace test {
    replication-factor 2
    memory-size 1G
    default-ttl 0
    strong-consistency true
    storage-engine device {
       file /var/lib/aerospike/test.dat
       filesize 4G
       data-in-memory true
    }
}
```


Validate that NTP, or other synchronization mechanisms, is installed and operational.
 
Determine whether you need `commit-to-device`. If you are running in a situation where no data loss is 
acceptable even in the case of simultaneous server hardware failures, 
you can choose that a namespace commits to disk. This will cause performance degradation, and only 
provides benefit where hardware failures are within milliseconds of each other. 
Add `commit-to-device true` within the storage engine scope of the namespaces in question. 

`commit-to-device` requires serializing writes to the output buffer, and will thus flush more data than is strictly necessary. 
Although Aerospike automatically determines the smallest flush increment for a given drive, this can be raised with the 
optional `commit-min-size` parameter.

Start the servers.
```
systemctl start aerospike
```

The cluster forms with all nodes but without a per-roster namespace. In order to use your Strong Consistency namespace, you'll
need to operationally add the roster. Until you do so, client requests will fail. 

```
Admin> show stat -flip like cluster_size
~~~~~~~~~~~~~~~~~~~~~~~~Service Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            cluster_size
node2.aerospike.com:3000                             5
node4.aerospike.com:3000                             5
node5.aerospike.com:3000                             5
node6.aerospike.com:3000                             5
node7.aerospike.com:3000                             5
Number of rows: 5

~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            ns_cluster_size
node2.aerospike.com:3000                             0
node4.aerospike.com:3000                             0
node5.aerospike.com:3000                             0
node6.aerospike.com:3000                             0
node7.aerospike.com:3000                             0
Number of rows: 5
```

This result is expected, since the `test` namespace's roster has not yet been modified.
To make this namespace useful we will need to create a roster, please see below.



## Create the Roster

The roster is the list of nodes which are expected to be in the cluster, for this particular namespace. This list
is stored persistently in a distributed table within each Aerospike server, similar to how index configuration
is stored. In order to change and manipulate this roster, please use the following Aerospike real-time tools.
The tools referenced here are part of the aerospike-tools package, which should be installed.

Rosters are specified using node IDs. With the advent of Aerospike Server 4, you can specify the node ID of each server,
as well as using the automatically generated node ID that was supported in Aerospike Server 3 and previous.
For further information about specifying node IDs, please see that section.

The general process of managing a roster is to first bring together the cluster of nodes, then list the node IDs within
the cluster that have the namespace defined, and finally add the node's IDs to the roster. To commit the changes, execute
a recluster command.

In detail:

**Get a list of nodes** which have been observed to have this namespace definition, 
```
asinfo -v 'roster:namespace=[ns-name]'
```
The `observed_nodes` are the nodes which are in the cluster and have the namespace defined.
Copy the `observed_nodes list`.

```
Admin> asinfo -v "roster:namespace=test"
node7.aerospike.com:3000 (192.168.10.7) returned:
roster=null:pending_roster=null:observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202

node6.aerospike.com:3000 (192.168.10.6) returned:
roster=null:pending_roster=null:observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202

node5.aerospike.com:3000 (192.168.10.5) returned:
roster=null:pending_roster=null:observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202

node4.aerospike.com:3000 (192.168.10.4) returned:
roster=null:pending_roster=null:observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202

node2.aerospike.com:3000 (192.168.10.2) returned:
roster=null:pending_roster=null:observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202
```

Now **set the roster** to the `observed_nodes`

```
roster-set:namespace=[ns-name];nodes=[observed nodes list]
```

Note it is safe to issue this command multiple times, once for each observed node, or create a list. The roster
is not active until we `recluster:`.

```
Admin> asinfo -v "roster-set:namespace=test;nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,
						BB9040016AE4202,BB9020016AE4202" with BB9020016AE4202
node2.aerospike.com:3000 (192.168.10.2) returned:
ok
```

We now have a roster but it isn't active yet. 

**Validate your roster** by using the `roster:` command.

```
Admin> asinfo -v "roster:"
node2.aerospike.com:3000 (192.168.10.2) returned:
ns=test:roster=null:pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:observed_nodes=null

node7.aerospike.com:3000 (192.168.10.7) returned:
ns=test:roster=null:pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:observed_nodes=null

node6.aerospike.com:3000 (192.168.10.6) returned:
ns=test:roster=null:pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:observed_nodes=null

node5.aerospike.com:3000 (192.168.10.5) returned:
ns=test:roster=null:pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:observed_nodes=null

node4.aerospike.com:3000 (192.168.10.4) returned:
ns=test:roster=null:pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:observed_nodes=null
```

Notice that roster is null but pending_roster is set with the provided roster.


**Apply the pending_roster** by issuing the `recluster:` command. Only the principal will respond ok to this command, the 
rest will ignore it. This is expected.

```
Admin> asinfo -v "recluster:"
node2.aerospike.com:3000 (192.168.10.2) returned:
ignored-by-non-principle

node7.aerospike.com:3000 (192.168.10.7) returned:
ok

node6.aerospike.com:3000 (192.168.10.6) returned:
ignored-by-non-principle

node5.aerospike.com:3000 (192.168.10.5) returned:
ignored-by-non-principle

node4.aerospike.com:3000 (192.168.10.4) returned:
ignored-by-non-principle
```

**Verify that the new roster was applied** with the `roster:` command.

```
Admin> asinfo -v "roster:"
node2.aerospike.com:3000 (192.168.10.2) returned:
ns=test:roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:pending_roster=BB9070016AE4202,
				BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202
				
node7.aerospike.com:3000 (192.168.10.7) returned:
ns=test:roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:pending_roster=BB9070016AE4202,
				BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202
				
node6.aerospike.com:3000 (192.168.10.6) returned:
ns=test:roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:pending_roster=BB9070016AE4202,
				BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202

node5.aerospike.com:3000 (192.168.10.5) returned:
ns=test:roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:pending_roster=BB9070016AE4202,
				BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202

node4.aerospike.com:3000 (192.168.10.4) returned:
ns=test:roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202:pending_roster=BB9070016AE4202,
				BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202
```

Notice that both roster and pending_roster are set to the provided roster.


**Validate that the namespace cluster size agrees** with the service cluster size.

The namespace ns_cluster_size should now agree with the service cluster_size 
(assuming all nodes in the service have this namespace). 

Validate this with  
`show stat -flip like cluster_size`

```
Admin> show stat -flip like cluster_size
~~~~~~~~~~~~~~~~~~~~~~~~Service Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            cluster_size
node2.aerospike.com:3000                             5
node4.aerospike.com:3000                             5
node5.aerospike.com:3000                             5
node6.aerospike.com:3000                             5
node7.aerospike.com:3000                             5
Number of rows: 5

~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            ns_cluster_size
node2.aerospike.com:3000                             5
node4.aerospike.com:3000                             5
node5.aerospike.com:3000                             5
node6.aerospike.com:3000                             5
node7.aerospike.com:3000                             5
Number of rows: 5
```

## Configuring Rack Aware

Strong Consistency namespaces cause some changes to rack aware configurations.

For a strong-consistency namespace to be rack-aware, the roster list becomes a series of node-id@rack-id pairs. 
Each entry in the roster list needs to define which rack a node is on (defaulting to rack-id 0 if none is defined). 
The roster can be manually constructed by appending @rack-id to each node-id in a comma separated list. 

Note: the rack-id is specified in each namespace. It is possible to set different rack-ids for different namespaces,
and it is possible to have some namespaces not be rack aware. 

However, management of this list may be simplified by just configuring rack-id on each node. 

These configured rack-ids will automatically appear in the observed_nodes list, which can then be used 
directly as a roster list in the set-roster command. The following will describe configuring Strong Consistency 
with rack aware by using the rack-id configuration to generate the `observed_nodes` list.

To configure rack aware for a namespace configured with strong-consistency, modify the rack-id configuration 
for each *namespace* in the /etc/aerospike/aerospike.conf file.

```
namespace test {
   replication-factor 2
   memory-size 1G
   default-ttl 0
   strong-consistency true
   rack-id 101
   
   storage-engine device {
     file /var/lib/aerospike/test.dat
     filesize 4G
     data-in-memory false
     commit-to-device true
   }
}
```

{{#note}}
In strong consistency mode, the rack-id configuration is actually ignored and is only used to facilitate 
setting the roster by copying and pasting the observed_nodes list. Only the rack-id configured when setting 
the roster will be applied, and it could be different than what has been configured in the config file (or directly 
dynamically through the rack-id configuration parameter).
{{/note}}

This can also be done dynamically (refer to the details at the bottom of this page).

**Start the Aerospike server**
```
systemctl start aerospike
```

**Execute a `roster:` command** to retrieve the current rosters

```
Admin> asinfo -v "roster:" -l
node2.aerospike.com:3000 (192.168.10.2) returned:
ns=test
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9070016AE4202@102,BB9060016AE4202@101,BB9050016AE4202@101,BB9040016AE4202@101,BB9020016AE4202@102
```

**Copy the `observed_nodes` list and issue a `roster-set:`** using this list as the value to the `nodes` parameter.

```
Admin> asinfo -v "roster-set:namespace=test;nodes=BB9070016AE4202@102,BB9060016AE4202@101,BB9050016AE4202@101,BB9040016AE4202@101,BB9020016AE4202@102
node2.aerospike.com:3000 (192.168.10.2) returned:
ok
```

**Execute a `roster:` command** and notice that the pending_roster has been updated.

**Apply the roster using the "recluster:"** command.

```
Admin> asinfo -v "recluster:"
node2.aerospike.com:3000 (192.168.10.2) returned:
ignored-by-non-principal

node7.aerospike.com:3000 (192.168.10.7) returned:
ok

node6.aerospike.com:3000 (192.168.10.6) returned:
ignored-by-non-principal

node5.aerospike.com:3000 (192.168.10.5) returned:
ignored-by-non-principal

node4.aerospike.com:3000 (192.168.10.4) returned:
ignored-by-non-principal
```

The cluster is now configured as rack aware, you can verify this by issuing the `racks:` command and verifying that 
each rack&#95;# matches the expected corresponding roster_rack&#95;#:

### Dynamically configuring rack ids

**Dynamically set rack-id** with the following info command:
```
asinfo -v 'set-config:context=namespace;id=[NAMESPACE];rack-id=[RACK]'
```
with space separated list of nodes IDs in this rack. You will execute this command multiple times,
because you need to set different nodes to different rack IDs, based on your hardware placement.

```
Admin> asinfo -v "set-config:context=namespace;id=test;rack-id=101" with 192.168.10.2 192.168.10.4 192.168.10.5
node2.aerospike.com:3000 (192.168.10.2) returned:
ok
node5.aerospike.com:3000 (192.168.10.5) returned:
ok
node4.aerospike.com:3000 (192.168.10.4) returned:
ok

Admin> asinfo -v "set-config:context=namespace;id=test;rack-id=102" with 192.168.10.6 192.168.10.7
node6.aerospike.com:3000 (192.168.10.6) returned:
ok
node7.aerospike.com:3000 (192.168.10.7) returned:
ok
```

**Issue a `recluster:`**

```
asinfo -v "recluster:"
```

```
Admin> asinfo -v "recluster:namespace=test"
node2.aerospike.com:3000 (192.168.10.2) returned:
ignored-by-non-principal

node7.aerospike.com:3000 (192.168.10.7) returned:
ok

node6.aerospike.com:3000 (192.168.10.6) returned:
ignored-by-non-principal

node5.aerospike.com:3000 (192.168.10.5) returned:
ignored-by-non-principal

node4.aerospike.com:3000 (192.168.10.4) returned:
ignored-by-non-principal
```


## Managing Strong Consistency

In order to add nodes, remove nodes, and manage availability, 
please [see Consistency Management documentation](/docs/operations/manage/consistency/index.html).
