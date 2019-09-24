---
title: Capacity Planning for Google Compute Engine
description: |
  Choosing machine types for different storage models on Google Compute Engine with the Aerospike database
scripts:
  - /assets/scripts/utils/target_blank.js

persistence-name: "Persistent Disks"
cloud-provider: "Google Compute Engine"

---

{{#info}}For details on disk and memory sizing, see the main [capacity planning](/docs/operations/plan/capacity) docs.{{/info}}

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/two-usecases.part.md") }}


### Aerospike as a Hybrid-Memory store

Aerospike has developed a simple to run [Cloud Qualification Process](/docs/operations/plan/cloud) that demonstrates the capabilites of untuned instances. By default it tests our hybrid-memory configuration. This can be used to differentiate what each instance family is more suited towards.

{{md (path-resolve (path-dirname page.src) "../../../operations/plan/cloud/gcp.md") }}


### Aerospike as In-Memory with No Persistence

The [Data In-Memory without Persistence](/docs/operations/configure/namespace/storage/index.html#recipe-for-data-in-memory-without-persistence) storage engine is ideal for this use case.

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/in-memory-desc.part.md") }}

The `n1-standard` and `n1-highmem` machine types should be considered in this case based on the memory requirements for your application. Use the [Capacity Planning Guide](/docs/operations/plan/capacity) to determine how much RAM will be required for data storage along with index storage. For most use-cases, the Aerospike database is not limited by the CPU performance; hence, you may consider a machine type with fewer CPUs to save on [costs](https://cloud.google.com/compute/pricing#machinetypes). For instance, if your memory requirement is 20GB, it may be economically better to use a `n1-highmem-4` machine type that has 26 GB memory rather than using a `n1-standard-8` standard machine type that has 30 GB memory.

Here are some performance numbers to help guide you in your machine type selection. The in-memory configuration was used with 100 byte objects and an 80% read, 20% write workload.

| Instance type  | TPS (max) | Latency<br/>(% reads above 8ms) |
| --             | --:       | --:                           |
| n1-standard-2  | 22 K      |                           15% |
| n1-standard-4  | 40 K      |                            9% |
| n1-standard-8  | 56 K      |                            3% |
| n1-standard-16 | 100 K     |                            1% |
| n1-highmem-8   | 56 K      |                            5% |
| n1-highcpu-4   | 39 K      |                           10% |
| n1-highcpu-8   | 62 K      |                            3% |
| n1-highcpu-16  | 101 K     |                            1% |

{{#note}}The performance numbers above are only indicative. You are encouraged to perform tests based on your average object size and specific read-write workload.{{/note}}

### Aerospike as a Fast Persistent Data Store

Take advantange of local disk performance with the persistence guarantee of persitent disks. An extension of our [SSD Storage Engine](/docs/operations/configure/namespace/storage/index.html#recipe-for-an-ssd-storage-engine), the [Shadow Device Storage Engine](/docs/operations/configure/namespace/storage/index.html#recipe-for-shadow-device) is designed for cloud environments like Google Compute Engine.

Google Compute Engine provides persistent storage in the form of [Persistent Disks](https://cloud.google.com/compute/docs/disks) that are network-attached to virtual machine instances.

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/persistent-desc.part.md") }}

We recommend using the SSD Persistent Disks for storage while configuring your Aerospike data store, as they offer [better performance](https://cloud.google.com/compute/docs/disks#pdperformance).
  - **Create and Attach Local Disks**: You must create the VM with Local disks. It cannot be added later.

  - **Create and Attach an SSD Persistent Disk**: You can use either the Google Compute Engine [web console](https://console.developers.google.com/project)  or the **gcloud** command to [do it programmatically](https://cloud.google.com/compute/docs/disks#create_disk)

  - **Configure Aerospike to use the Local and SSD Persistent Disk**: Once the disk has been attached to the instance, it should appear as a block device (e.g.: `/dev/disk/by-id/google-persistent-disk-1 and /dev/disk/by-id/google-local-disk-0`) on your instance. Next, you should use the [recipe for Shadow Device](/docs/operations/configure/namespace/storage/index.html#recipe-for-shadow-device) to configure an Aerospike namespace to use these devices for storage.

You can read more about our recommendations about local and persisted disks on our [recommendations page](/docs/deploy_guides/gce/recommendations/index.html#persistence)

#### High availability using zones

{{#info}}"Google Cloud Platform resources are hosted in multiple locations world-wide. These locations are composed of `regions` and `zones`. Putting resources in different zones in a region provides isolation for many types of infrastructure, hardware, and software failures. Putting resources in different regions provides an even higher degree of failure independence." *Read more from [Google Compute Engine Regions and Zones documentation](https://cloud.google.com/compute/docs/zones)*
{{/info}}

{{md (path-resolve (path-dirname page.src) "/docs/deploy_guides/common/availability-suggestions.part.md") }}
