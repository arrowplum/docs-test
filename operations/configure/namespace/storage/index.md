---
title: Namespace Storage Configuration
description: Configure namespace storage on the Aerospike Database to ensure durability and price performance of your cluster. 
---

Aerospike determines where to store data based on the Storage engine
configured for the namespace. These engines determine if the data will be
persisted to disk, reside in memory, or both. These decisions will affect
the durability, cost, and performance of your cluster.

## Comparing Storage Engines

| Storage engine                     | Device <br/>(data not in memory)   | All Flash | Device <br/>(data in memory) | Memory only <br/> (no persistence) |
|------------------------------------|----------|-----------------------------|-----------------------|----------------------------|
| Index In Memory                    | &#10003; | &#10007;                    |&#10003;                    | &#10003;                   |
| [Fast Restarts](/docs/operations/manage/aerospike/fast_start)<sup>\*</sup>          |  &#10003;                    |&#10003;                  | &#10003;<sup>\*\*</sup>              | &#10007;                   |
| Survive a Power Outage             | &#10003; | &#10003;                    |&#10003;                    |  &#10007;                   |

<sub><sup>\*</sup>Available on Aerospike Enterprise Edition.</sub><br/>
<sub><sup>\*\*</sup>Primary index persisted upon restart for [`data-in-memory`](/docs/reference/configuration/?show-removed=1#data-in-memory) with persistence as of Enterprise Edition version 3.15.1.3.</sub><br/>
<sub><sup>\*\*</sup>[Fast Restarts](/docs/operations/manage/aerospike/fast_start) also supported for [`data-in-index`](/docs/reference/configuration/?show-removed=1#data-in-index) configuration.</sub>


## Storage Engine Configuration Recipes
The following recipes require modifying the Aerospike Server configuration
file which is located `/etc/aerospike/aerospike.conf`. Each recipe describes
the minimal configuration necessary to enable a particular storage engine as
well as the storage sizing parameters used by that engine. To get started,
open the configuration file in your preferred editor and make the appropriate
changes.

```
sudo $EDITOR /etc/aerospike/aerospike.conf
```

### Recipe for an SSD Storage Engine
The minimal configuration for an SSD namespace requires setting [storage-engine](/docs/reference/configuration/index.html#storage-engine)
to `device` and adding a [device](/docs/reference/configuration/index.html#device) parameter for each SSD to be used by this
namespace. The maximum size of a `device` is 2 TiB, for larger devices, partiton into multiple equally sized partitions that are less than 2 TiB each.
In addition, [memory-size](/docs/reference/configuration/index.html#memory-size) may need to be changed from the default of
4 GiB to a size appropriate for the expected primary index size.
For assistance in sizing the primary index, please refer to the
[Sizing Guide](/docs/operations/plan/capacity).
For performance, we recommend, in general, reducing the [write-block-size](/docs/reference/configuration/index.html#write-block-size) from the
default of 1 MiB to 128 KiB on SSD backed namespaces. This may vary based on the specific workload and record average size, and the best way to find the right setting would be to run some benchmarks with different values.

{{md (path-resolve (path-dirname page.src) "ssd_config.part.md") }}

### Recipe for an HDD Storage Engine with Data in Memory
The minimal configuration for an HDD with Data-in-Memory namespace involves
setting [storage-engine](/docs/reference/configuration/index.html#storage-engine) to `device`,
setting [data-in-memory](/docs/reference/configuration/index.html#data-in-memory) to true, and
finally providing a list of [file](/docs/reference/configuration/index.html#file) parameters to indicate where data will
be persisted. Also, [filesize](/docs/reference/configuration/index.html#filesize) needs to be large enough to support the size of the data on disk (with a maximum allowed value of 2 TiB). For common use cases, this should roughly be 4 times the `memory-size`. Lastly, [memory-size](/docs/reference/configuration/index.html#memory-size) may need to be adjusted from the default
of 4GiB to a size appropriate to handle the expected primary index size and the expected size of the data in memory. For assistance sizing `filesize` or `memory-size` please refer to our [Sizing Guide](/docs/operations/plan/capacity).

```bash
namespace <namespace-name> {
	memory-size <SIZE>G             # Maximum memory allocation for secondary indexes (if any).
	storage-engine device {         # Configure the storage-engine to use
									# persistence. Maximum size is 2 TiB
	    file /opt/aerospike/<filename>  # Location of data file on server.
	    # file /opt/aerospike/<another> # (optional) Location of data file on server.
	    filesize <SIZE>G                # Max size of each file in GiB.
	    data-in-memory true             # Indicates that all data should also be
		    							# in memory.
	}
}
```

### Recipe for a HDD Storage Engine with Data in Index Engine
A data-in-index configuration is a highly specialized namespace for a very niche
use case. If your data is [single-bin](/docs/reference/configuration/index.html#single-binB) and fits in 8 bytes and you need to
performance of an in-memory namespace but do not want to lose the **[fast restart](/docs/operations/manage/aerospike/fast_start/index.html)**
capability provided in Aerospike Enterprise Edition, then data-in-index is it.

The minimal configuration for a data-in-index namespace involves setting
`single-bin` to true, [data-in-index](/docs/reference/configuration/index.html#data-in-index) to true,
and [data-in-memory](/docs/reference/configuration/index.html#data-in-memory) to true.
In addition, [storage-engine](/docs/reference/configuration/index.html#storage-engine) must be `device`
and [file](/docs/reference/configuration/index.html#file) or [device](/docs/reference/configuration/index.html#device) parameters
need to be configured to map to the persisted storage device to be used by this
namespace. Finally, [memory-size](/docs/reference/configuration/index.html#memory-size) needs to be adjusted from it's default of 4 GiB to size that can accomodate the primary index, and [filesize](/docs/reference/configuration/index.html#filesize)
from it's 16 GiB defaults to the size of the data on disk (with a maximum allowed value of 2 TiB).
For assistance sizing `filesize` or `memory-size` please refer to our [Sizing Guide](/docs/operations/plan/capacity).

```bash
namespace <namespace-name> {
	memory-size <N>G                # Maximum memory allocation for data and
									# primary and secondary indexes.
	single-bin true                 # Required true by data-in-index.
	data-in-index true              # Enables in index integer store.
	storage-engine device {         # Configure the storage-engine to use
									# persistence.
	file /opt/aerospike/<filename>  # Location of data file on server.
	# file /opt/aerospike/<another> # (optimal) Location of data file on server.
	# device /dev/<device>          # Optional alternative to using files.

	filesize <SIZE>G               # Max size of each file in GiB. Maximum size is 2TiB
	data-in-memory true            # Required true by data-in-index.
	}
}
```

### Recipe for Data in Memory Without Persistence
The minimal configuration for a namespace without persistence is to set
[storage-engine](/docs/reference/configuration/index.html#storage-engine) to `memory`. If your namespace requires more than the default
4 GiB [memory-size](/docs/reference/configuration/index.html#memory-size) allocation for the primary index and data in memory then it
is also necessary to adjust `memory-size` accordingly. For assistance sizing
`memory-size` please refer to our
[Sizing Guide](/docs/operations/plan/capacity).

```bash
namespace <namespace-name> {
	memory-size <SIZE>G   # Maximum memory allocation for data and primary and
						  # secondary indexes.
	storage-engine memory # Configure the storage-engine to not use persistence.
}
```

### Recipe for Shadow Device

The shadow device storage model introduced in 3.5.12 is tailored for cloud environments where you might have extremely high performance SSDs that are ephemeral (not persistent). Whereas the persisted devices are not as performant as one would like.

All writes are duplicated to another (shadow) device. This shadow device will act as the persisted store. The primary device still receives all operations as normal.
This results in a persisted data volume with lower IOPS requirements, while still gaining the IOPS benefit of the non-persisted volume. All without using large amounts of RAM. The Shadow Device only needs to satisfy the write IOPS requirements of your workload, not reads.

{{#note}}
This is an extension of the SSD Storage Engine.
{{/note}}

{{#note}}
When leveraging network attached shadow devices (for example EBS on AWS) or re-assigning shadow devices to a different instance, when running version 3.16.0.1 or above, 
it is recommended to have initially configured the [`node-id`](/docs/reference/configuration/#node-id) across the nodes in the cluster in order to preserve it on a potential new instances that would be re-attached to the shadow device of an instance and avoid redistribution of the partitions in the cluster.
{{/note}}

To utilize Shadow Devices, simply add the persisted volume after the declaration of the non-persisted volume on the same line.

{{#info}}
Shadow devices must be greater than or equal to the size of the primary device.
{{/info}}

```
namespace <namespace-name> {
	...
	storage-engine device {
		device /dev/sdb /dev/sdf  # sdb is the fast ephemeral volume, while sdf is the slower persisted volume
		...
	}
}
```

In the above example, /dev/sdb is the fast, non-persisted device. /dev/sdf is the persisted device. Order is important. Devices must be listed on the same line for Shadow Device configuration.

Shadow Device configuration can be combined with multiple devices. Note the 1-to-1 mapping:

```
	storage-engine device{
		device /dev/sdb /dev/sdf
		device /dev/sdc /dev/sdg
		...
	}
```
{{#note}}
When configuring a namespace to use persistence of any form, care should be taken that a given file or device partition
be associated with a single namespace only.  Two namespaces cannot share the same file or partition.  Configuration of the same file or partition for multiple namespaces could cause issues with the node starting and/or damage to existing data in that file or partition.
{{/note}}


## Where to Next?
- Configure [Data Retention Policy](/docs/operations/configure/namespace/retention) which determines how long
  records are kept after it is initially written.
- Configure [Data Durability Policy](/docs/operations/configure/namespace/durability) which determines how many
  replica copies of a record to keep in the cluster.
- Or return to [Configure Page](/docs/operations/configure).
