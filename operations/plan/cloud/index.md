---
title: Qualifying Cloud Instances
description: Use the Aerospike Cloud Qualifying Process to evaluate instance performance by simulating a steady read and write workload, and measuring response time prior to your first production release.
---

The Aerospike Cloud Qualifying Process evaluates cloud providers' instance and device performance by simulating a steady read and write workload, and measuring response time. These are tests that should be performed to get an idea of the level of performance you can expect from cloud instances.

If your instance is not one that has been tested by Aerospike, you can test it yourself by downloading the open sourced [Aerospike Cloud Qualification](https://github.com/aerospike/cloud-qualification) package to test your instances, or by utilizing YCSB directly.

### Qualified Instance Types

Cloud instance types will give you a certain amount of resources (CPU, Memory, Disk and Networking), but it can be hard to translate that into expected performance. The Cloud Qualification Process will take these instances and qualify it capable of a certain operational level, as an overall metric. It does not focus on individual resource components or optimizations. 

{{#info}}
This page will hopefully provide a starting point into what you can expect from each instance and how the instance class/family is more suitable toward different object sizes and use cases.
{{/info}}

The Cloud Qualification Process is done using Aerospike's fork of the [Yahoo! Cloud Serving Benchmark](https://github.com/aerospike/YCSB).

The testing methodology is as follows:

| Workload | Access Pattern | Record Count\* | Record Size\* | Cluster Size | YCSB Duration | 
| -------- | -------------- | ------------ | ------------ | ----------- | ----------------- |
| 50/50 R/U | Uniform       | 60% of memory | 50% of disk | 3           | 3 \* Record Count |

\* Based on a 2 node cluster

The testing client is a single instance of of a top-end system focused on compute power and network. 

The entire test is repeated at least 3 times for verification

{{#warn}}
This workload is chosen as a worst-case scenario for Aerospike, as a uniform access pattern will result in all blocks entering defragmentation at nearly the same time.
{{/warn}}

Aerospike is configured to utilize local SSD storage (when available), but is otherwise left as default, as are the instances that Aerospike is deployed on. You may see improved performance from additional tuning. You may also choose to alter these numbers and percentages in your own run of YCSB.

An instance is considered passing at an operational level (eg: 30,000 ops/s) if the average read latency does not exceed 1 millisecond from the client. 95 and 99 percentiles, as well as object size and count, are provided as reference.

Larger operational levels are better.

Your results may vary from these published numbers due to differences in server, topology, and even variation between systems of the same type.

The table below shows the results from the Aerospike Cloud Qualification Process on some popular instance types.

#### Qualified Instances

{{md (path-resolve (path-dirname page.src) "aws.md") }}

{{md (path-resolve (path-dirname page.src) "azure.md") }}

{{md (path-resolve (path-dirname page.src) "gcp.md") }}

{{md (path-resolve (path-dirname page.src) "ibm.md") }}

### See also

* [Aerospike Cloud Qualification](https://github.com/aerospike/cloud-qualification)
* [SSD Certification](/docs/operations/plan/ssd/ssd_certification.html)
* [Cloud Qualification Details](/docs/operations/plan/cloud/details.html)
