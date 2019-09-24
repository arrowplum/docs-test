---
title: Microsoft Azure Capacity Planning
description: Various storage models as applicable on Azure
scripts:
 - /assets/scripts/utils/target_blank.js

cloud-provider: Azure

---


{{#info}}For details on disk and memory sizing, see the main
[capacity planning](/docs/operations/plan/capacity) docs.{{/info}}

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/two-usecases.part.md") }}

### Aerospike as a Hybrid-Memory Store

Aerospike has developed a simple to run [Cloud Qualification Process](/docs/operations/plan/cloud) that demonstrates the capabilites of untuned instances. By default it tests our hybrid-memory configuration. This can be used to differentiate what each instance family is more suited towards.

{{md (path-resolve (path-dirname page.src) "../../../operations/plan/cloud/azure.md") }}

### Aerospike as In-Memory with no Persistence

The [Data In-Memory without Persistence](/docs/operations/configure/namespace/storage/index.html#recipe-for-data-in-memory-without-persistence) 
storage engine is ideal for a cache based use-case.

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/in-memory-desc.part.md") }}

### Aerospike as a Fast Persistent Data Store

The storage engine suited for this use case is the 
[SSD Storage Engine](/docs/operations/configure/namespace/storage/index.html#recipe-for-an-ssd-storage-engine).

Azure provides SSD storage in the form of [Premium Storage](https://docs.microsoft.com/en-us/azure/storage/storage-premium-storage).
These are network attached to virtual machine instances.


#### Local SSDs Backed by Premium Storage Persistence

This is the recommended storage engine in cloud environments.

**Pros:**
- RAM requirement is same as persistence model only.
- Provides persistence offered by Storage Disks while surpassing the performance
  bottleneck by making use of local SSDs performance as caching
  layer.
- Provides the best of performance and persistence possible by using local
  SSD as RAM alternative along with Storage Disks for persistence storage.

**Cons:**
- More operational overhead than any other storage models.
- Need to use instances supporting the required amount of local SSD
  instance storage volumes.

### Instance Types
Presently, we suggest using the **Lsv2** instances. These support
[Premium Storage](https://docs.microsoft.com/en-us/azure/storage/storage-premium-storage), which are
SSD based network storage, as well as having local SSDs. See [Certifying Flash Devices](/docs/operations/plan/ssd/ssd_certification.html).

Instance families with 'S' in their designation have Premium Storage (SSD) support. If no persistence is required,
non-'s' families can be used.


### Disk Caching

Azure provides a unique [disk caching](https://docs.microsoft.com/en-us/azure/storage/storage-premium-storage-performance#disk-caching) feature.
This will accelerate your disk access for both local and persisted disks.

If your access pattern is predominately read based, it would be advisable to configure the Disk Caching to `ReadOnly`. `ReadOnly` caching also benefits access patterns that are less random or uniform and more concentrated (eg: zipfian, last-access, hotspot distributions)

{{#warn}}
Aerospike recommends testing your personal workload for any potential benefits of `ReadOnly` disk caching.

Aerospike does not recommend `ReadWrite` disk based caching.
{{/warn}}
