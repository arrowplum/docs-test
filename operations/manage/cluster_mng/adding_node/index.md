---
title : Adding a new node to an Aerospike cluster
---

Adding a node to an Aerospike cluster is an easy way to add capacity. Aerospike supports adding a single or multiple nodes to a running cluster and does not have specific requirements in the number of nodes that can be added based on the number of existing node. Cluster configured with Rackware would typically add the same number of nodes in each group.

This document provides the steps to follow to add a new node to an existing cluster.

{{#warn}}
**IMPORTANT PREREQUISITES**<br><br>
It is important to prepare the node. To avoid any scenario of pre-existing data, the drives must be initialized. New SSD drives must be prepared/installed before use. This process erases the drives using the dd command as described in the [SSD Initialization](/docs/operations/plan/ssd/ssd_init.html) section. 
<br><br>
The files under the  `/opt/aerospike/smd` folder should be removed if the node was previously in use in a different cluster. The system metadata files in this folder contain details about secondary index definitions, udf modules, user roles and permissions, truncate and truncate-namespace related information, roster information and more. Adding a node in a cluster with existing data from a different cluster in the smd folder would result in potentially disastrous situations.
{{/warn}}

## Method
If configured correctly, the new node will join the cluster automatically upon starting up the Aerospike daemon. This document is then mostly covering how to configure the new node. 

{{#info}}
The configuration file is by default under /etc/aerospike/aerospike.conf
{{/info}}

{{#warn}}
Adding a node to a cluster initiates migrations to rebalance the number of partitions across all the available nodes. Migrations occupy various system resources, so it is advised to add a node during low traffic period. This data rebalancing mechanism ensures that query volume distributes evenly across all cluster nodes, and is persistent during node failure. Please see [Auto Rebalance](/docs/architecture/data-distribution.html#automatic-rebalancing) for more information.
{{/warn}}

### Important points to consider when adding a new node to a cluster

The following must be enforced to ensure the node will properly join the cluster:
* For both in-memory and persistent namespaces, the total memory / disk space allocated for each namespace on each node should be the same (other than transient state when changing capacity).
* If the namespace is persistent the individual devices allocated to the namespace have the same capacity.
* Refer to the [Namespace Storage Configuration](/docs/operations/configure/namespace/storage) page for more information on configuring a namespace.
* [Versions prior to 3.13 only] The number of namespaces are the same on all nodes in the cluster i.e. including the new incoming node.
* [Versions prior to 3.13 only] The order of namespaces in the namespace section of the configuration file is preserved across nodes.

{{#info}}
There is no need to restart the existing nodes in the cluster to add a new node.
{{/info}}

{{#info}}
For the purposes of performing a rolling upgrade, Aerospike supports mixed-version clusters. Aerospike does not, however, support long term use of clusters with nodes running different versions of the Aerospike database. Please see [rolling upgrade](/docs/operations/upgrade/aerospike) for more information.
{{/info}}

{{#warn}}
As a recommendation – do not add multiple nodes simultaneously to avoid corner cases where the new nodes form a cluster on their own before joining the main cluster, adding more partition versions that would cause the subsequent duplicate resolution to be heavier. We recommend, as best practice, to wait for a new node to successfully join the cluster before adding the next one.
{{/warn}}

{{#warn}}
**Important when adding empty nodes:** For server version on the older paxos and heartbeat protocol (prior to 3.13), it may be prefereable to wait for migrations to complete between adding empty nodes. The older cluster protocol would drop redundant partition copies prior to completing the migration for the partition, leaving some partitions (the one that would be owned by the newly added node) with 1 less copy than dictated by the replication factor. This adds risks of data unavailability in the unlikely event of a node leaving the cluster prior to migrations completing. The more new nodes added at once, the more partitions would be losing a copy putting more data at risk in case of a node leaving the cluster.
For server versions running the new cluster protocol (3.13 and above), replica copies of a partition is kept around until the migration of the partition has completed.
{{/warn}}

{{#warn}}
The configuration file options [`node-id`](/docs/reference/configuration#node-id) and [`node-id-interface`](/docs/reference/configuration#node-id-interface) are mutually exclusive.
{{/warn}}

{{#steps}}
{{#steps-step 1 "Configuration setup" markdown=true}}

**Mesh mode**

* Please refer to [configure Aerospike database](/docs/operations/configure) for a detailed description of various sections of the configuration file and their relevant configuration parameters.
* The following steps assume the following:
     * A cluster(at least 1 node) is already setup in mesh mode
     * The Aerospike daemon is installed on the new node. Please see [Install Aerospike](/docs/operations/install) for the Aerospike installation procedure.
* The example used to illustrate this procedure consists of a 2 node cluster to which a 3rd node is added.

Example: 2 node cluster

```asciidoc
Admin> info
                         Node               Node                Ip        Build   Cluster            Cluster     Cluster         Principal   Client     Uptime
                            .                 Id                 .            .      Size                Key   Integrity                 .    Conns          .
10.0.0.101:3000                 BB90A09E3270008    10.0.0.101:3000   E-3.11.0.2         2   F0322B636B7B0FE9   True        BB9AF1F8D270008        5   09:26:10
10.0.0.103:3000                 BB9235677270008    10.0.0.103:3000   E-3.11.0.2         2   F0322B636B7B0FE9   True        BB9AF1F8D270008       10   09:26:14
Number of rows: 2

```

**Configuration setup for node 10.0.0.101 which is already part of the cluster**

```asciidoc
# Aerospike database configuration file for deployments using mesh heartbeats.
service {
        user root
        group root
        paxos-single-replica-limit 1 # Number of nodes where the replica count is automatically reduced to 1.
        pidfile /var/run/aerospike/asd.pid
        service-threads 4
        transaction-queues 4
        transaction-threads-per-queue 4
        proto-fd-max 15000
        node-id-interface eth1
}

logging {
        # Log file must be an absolute path.
        file /var/log/aerospike/aerospike.log {
                context any info
        }
}

network {
	service {
		address any
		access-address 10.0.0.101
		port 3000
	}

	heartbeat {
		mode mesh
		address 10.0.0.101
		port 3002 # Heartbeat port for this node.

		# List one or more other nodes, one ip-address & port per line:
		# Please note that we do not have the address of the incoming node in this list
		mesh-seed-address-port 10.0.0.103  3002
		mesh-seed-address-port 10.0.0.101  3002 
		# Having the node itself as a mesh seed node is allowed 
		# and helps with consistent configuration files across the cluster

		interval 250
		timeout 10
	}

	fabric {
		port 3001
	}

	info {
		port 3003
	}
}

namespace test {
	                              # Data in memory without persistance namespace
	replication-factor 2
	memory-size 4G
	default-ttl 30d               # 30 days, use 0 to never expire/evict.
	storage-engine memory
}

namespace bar {
    memory-size 4G   	     	# Maximum memory allocation for primary and secondary indexes.

    # Warning - legacy data in defined raw partition devices will be erased.
    # These partitions must not be mounted by the file system.

    storage-engine device {       # Configure the storage-engine to use persistence
        	                  # Add raw device(s). Maximum size is 2 TiB.
        device /dev/sdb1  
        device /dev/sdc1  
        device /dev/sdd1   
        device /dev/sde1 
	
        # The 2 lines below optimize for SSD.
	scheduler-mode noop
	write-block-size 128K # adjust block size to make it efficient for SSDs.
    	}
}

```

**Configuration setup for node 10.0.0.103 which is already part of the cluster**

```asciidoc

# Aerospike database configuration file for deployments using mesh heartbeats.
service {
        user root
        group root
        paxos-single-replica-limit 1 # Number of nodes where the replica count is automatically reduced to 1.
        pidfile /var/run/aerospike/asd.pid
        service-threads 4
        transaction-queues 4
        transaction-threads-per-queue 4
        proto-fd-max 15000
        node-id-interface eth1
}

logging {
        # Log file must be an absolute path.
        file /var/log/aerospike/aerospike.log {
                context any info
        }
}

network {
    service {
        address any
        access-address 10.0.0.103
        port 3000
    }

    heartbeat {
        mode mesh
        address 10.0.0.103
        port 3002 # Heartbeat port for this node.

        # List one or more other nodes, one ip-address & port per line:
	# Please note that we do not have the address of the incoming node in this list
        mesh-seed-address-port 10.0.0.101  3002
	mesh-seed-address-port 10.0.0.103  3002

        interval 250
        timeout 10
    }

    fabric {
        port 3001
    }

    info {
        port 3003
    }
}

namespace test {
                                  # Data in memory without persistance namespace
    replication-factor 2
    memory-size 4G
    default-ttl 30d               # 30 days, use 0 to never expire/evict.
    storage-engine memory
}

namespace bar {
    memory-size 4G                # Maximum memory allocation for primary and secondary indexes.

    # Warning - legacy data in defined raw partition devices will be erased.
    # These partitions must not be mounted by the file system.

    storage-engine device {       # Configure the storage-engine to use persistence
                                  # Add raw device(s). Maximum size is 2 TiB
            device /dev/sdb3  
            device /dev/sdc3  
            device /dev/sdd3   
            device /dev/sde3 
            # The 2 lines below optimize for SSD.
            scheduler-mode noop
            write-block-size 128K # adjust block size to make it efficient for SSDs.
        }
}
```

**Configuration for the new incoming node with IP 10.0.0.100**

```asciidoc

# Aerospike database configuration file for deployments using mesh heartbeats.
service {
        user root
        group root
        paxos-single-replica-limit 1 # Number of nodes where the replica count is automatically reduced to 1.
        pidfile /var/run/aerospike/asd.pid
        service-threads 4
        transaction-queues 4
        transaction-threads-per-queue 4
        proto-fd-max 15000
        node-id-interface eth1
}

logging {
        # Log file must be an absolute path.
        file /var/log/aerospike/aerospike.log {
                context any info
        }
}

network {
    service {
        address any
        access-address 10.0.0.100
        port 3000
    }

    heartbeat {
        mode mesh
        address 10.0.0.100
        port 3002 # Heartbeat port for this node.

        # List one or more other nodes, one ip-address & port per line:
        mesh-seed-address-port 10.0.0.100  3002 
        mesh-seed-address-port 10.0.0.101  3002
        mesh-seed-address-port 10.0.0.103  3002

        interval 250
        timeout 10
    }

    fabric {
        port 3001
    }

    info {
        port 3003
    }
}

namespace test {
                                  # Data in memory without persistance namespace
    replication-factor 2
    memory-size 4G
    default-ttl 30d               # 30 days, use 0 to never expire/evict.
    storage-engine memory
}

namespace bar {
    memory-size 4G                # Maximum memory allocation for primary and secondary indexes.
    # Warning - legacy data in defined raw partition devices will be erased.
	# These partitions must not be mounted by the file system.

	storage-engine device {       # Configure the storage-engine to use persistence
                                  # Add raw device(s). Maximum size is 2 TiB
            device /dev/sdb  
            device /dev/sdc  
            device /dev/sdd   
            device /dev/sde 
	    # The 2 lines below optimize for SSD.
	    scheduler-mode noop
	    write-block-size 128K # adjust block size to make it efficient for SSDs. 
        }
}
```
**Multicast mode**
As multicast is another mode of communication for the heartbeat protocol all the other sections of the configuration files remain the same except for the heartbeat section. Please see [Network Heartbeat Configuration](/docs/operations/configure/network/heartbeat) for more information on the heartbeat section.

So, in our case the heartbeat section of the incoming node should be
```asciidoc
...
  heartbeat {
    mode multicast                  # Send heartbeats using Multicast
    multicast-group 239.1.99.2      # multicast address
    port 9918                       # multicast port
    address 10.0.0.100		     # (Optional) (Default any) IP of the NIC to
                                    # use to send out heartbeat and bind
                                    # fabric ports
    interval 150                    # Number of milliseconds between heartbeats
    timeout 10                      # Number of heartbeat intervals to wait
                                    # before timing out a node
  }
...
```
{{/steps-step}}
{{#steps-step 2 "Verify node joined the cluster" markdown=true}}

To verify the node is now a part of the cluster on any of the nodes we can issue an info commad within the asadm tool. For our example the info command outputs the following
```asciidoc
Admin> info
                         Node               Node                Ip        Build   Cluster            Cluster     Cluster         Principal   Client     Uptime   
                            .                 Id                 .            .      Size                Key   Integrity                 .    Conns          .   
10.0.0.101:3000                 BB90A09E3270008    10.0.0.101:3000   E-3.11.0.2         3   F0322B636B7B0FE9   True        BB9AF1F8D270008        5   51:49:48   
10.0.0.103:3000                 BB9235677270008    10.0.0.103:3000   E-3.11.0.2         3   F0322B636B7B0FE9   True        BB9AF1F8D270008       10   51:49:36   
10.0.0.100:3000		  	   *BB9AF1F8D270008   10.0.0.100:3000   E-3.11.0.2         3   F0322B636B7B0FE9   True        BB9AF1F8D270008       10   03:05:39   
Number of rows: 3

```
We can verify the same by searching the server log at /var/log/aerospike/aerospike.log

```asciidoc
grep 'CLUSTER-SIZE' /var/log/aerospike/aerospike.log
```
You should see (from our example):
```asciidoc
Jan 28 2017 01:00:03 GMT: INFO (info): (ticker.c:169) NODE-ID bb9af1f8d270008 CLUSTER-SIZE 3
```

{{/steps-step}}
{{/steps}}

