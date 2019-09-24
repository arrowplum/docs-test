---
title: Amazon EC2 Capacity Planning
description: Various storage models as applicable on AWS
scripts:
  - /assets/scripts/utils/target_blank.js

persistence-name: EBS
cloud-provider: AWS

---

{{#info}}For details on disk and memory sizing, see the main
[capacity planning](/docs/operations/plan/capacity) docs.{{/info}}

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/two-usecases.part.md") }}

### Aerospike as a Hybrid-Memory store
Aerospike has developed a simple to run [Cloud Qualification Process](/docs/operations/plan/cloud) that demonstrates the capabilites of untuned instances. By default it tests our hybrid-memory configuration. This can be used to differentiate what each instance family is more suited towards.

{{md (path-resolve (path-dirname page.src) "../../../operations/plan/cloud/aws.md") }}

### Aerospike as In-Memory with no Persistence

The [Data In-Memory without Persistence](/docs/operations/configure/namespace/storage/index.html#recipe-for-data-in-memory-without-persistence) storage engine is ideal for a cache based use-case.

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/in-memory-desc.part.md") }}

We have observed the following bandwidth and max TPS on various instance types
with the Data in Memory without Persistence storage engine. The tests were
performed with 100 byte objects and a 100% read workload.

|Instance Type|Client Transactions/s|Peak Observed bandwidth<sup>\*\*</sup>|Object Size|
|-----------------------|------|---------|---|
|m3.xlarge              |87 K  |700 Mbps |100|
|m3.2xlarge             |87 K  |1 Gbps   |100|
|c3.2xlarge<sup>\*</sup>|250 K |1 Gbps   |100|
|r3.large               |80 K  |600 Mbps |100|
|r3.xlarge              |120 K |1 Gbps   |100|
|r3.2xlarge<sup>\*</sup>|250 K |1 Gbps   |100|

<sup>\*</sup> Core Bottleneck : We tried many different combinations of number
of NICs and RPS values in our setup, but in every case there was a high amount
of network interrupt processing (`%hi`) and one of the CPU cores was became
the bottleneck. Overall CPU utilization was only around 50% of peak. For
further details, please see [suggested reading](/docs/deploy_guides/aws/plan/index.html#suggested-reading)

<sup>\*\*</sup> Using iperf

{{#note}}The performance numbers above are only indicative. You are encouraged to
perform tests based on your average object size and kind of read-write
workload.{{/note}}

We have seen a limit of 250k TPS per ENI. You can read more about this on our [blog post](https://www.aerospike.com/blog/boosting-amazon-ec2-network-for-high-throughput/).

You can also read more about tuning for AWS in our [tuning page](/docs/deploy_guides/aws/tune)

### Aerospike as a Fast Persistent Data Store

The storage engine suited for this use case is the 
[SSD Storage Engine](/docs/operations/configure/namespace/storage/index.html#recipe-for-an-ssd-storage-engine).

Amazon EC2 provides storage in the form of [Elastic Block Storage](https://aws.amazon.com/ebs/details/), or EBS.
These are network attached to virtual machine instances.

EBS performance is either set via [Provisioned IOPS or General Purpose](https://aws.amazon.com/ebs/details/#SSDvolumes). Provisioned IOPS (io1) delivers consistent IOPS but are costly. General Purpose (gp2) volumes have variable performance based on size. See the [AWS documentation](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html#EBSVolumeTypes_gp2) on the relationship between volume size and IOPS for gp2 volumes.

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/persistent-desc.part.md") }}

#### High Availability using Availability Zones

{{#info}}"Amazon EC2 is hosted in multiple locations world-wide. These locations are
composed of regions and Availability Zones. Each region is a separate
geographic area. Each region has multiple, isolated locations known as
Availability Zones. Amazon EC2 provides you the ability to place resources,
such as instances, and data in multiple locations. Resources aren't replicated
across regions unless you do so specifically. Amazon operates state-of-the-art,
highly-available data centers. Although rare, failures can occur that affect the
availability of instances that are in the same location." *Read more from the [AWS Regions and Availability Zones documentation](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)*
{{/info}}

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/availability-suggestions.part.md") }}

#### Ephemeral SSD Based Cache Backed by EBS Persistence
Otherwise known as [Shadow Device configuration](/docs/operations/configure/namespace/storage#recipe-for-shadow-device).

**Pros:**
- RAM requirement is same as the EBS persistence model only.
- Provides persistence offered by EBS while surpassing the performance
  bottleneck of EBS by making use of ephemeral SSDs performance as caching
  layer.
- Provides the best of performance and persistence possible by using Ephemeral
  SSD as RAM alternative along with EBS for persistence storage.

**Cons:**
- More operational overhead than any other storage models.
- Need to use instances supporting the required number and amount of ephemeral SSD
  instance storage volumes.

### Instance Types
Presently we suggest using the **R3**, **I2**, and **I3** instance types. These
support [Enhanced Networking](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html)
based on Single Root I/O Virtualization (SR-IOV). Keeping in mind the network
requirements, a minimum of **2xlarge** instances to form the cluster is suggested.

**R3** instances are primary for in-memory workloads with minimal storage requirements.  
**I2/I3** is suggested for storage models including ephemeral SSDs.

Other instance types can be used as well, depending upon storage and performance
requirements.

### Suggested Reading
We performed benchmarks on AWS and have outlined our
[observation](http://highscalability.com/blog/2014/8/18/1-aerospike-server-x-1-amazon-ec2-instance-1-million-tps-for.html)
in external blogs. The following provides helpful suggested steps and
[expectations](http://highscalability.com/blog/2014/8/20/part-2-the-cloud-does-equal-high-performance.html)
before you setup with your AWS.
