---
title: Recommendations for successfully deploying Aerospike on Amazon EC2
description: |
  Here are recommendations for ensuring a successful deployment of Aerospike database on Amazon Web Services.
scripts:
  - /assets/scripts/utils/target_blank.js

---

{{#note}}
Most of the AWS-related information provided below is given with the 
performance requirements of Aerospike in a production environment in mind. For
development purposes, the Aerospike Community Server will work on any of the
instances which have at least 256MB of RAM dedicated to the Aerospike server.
(Enterprise versions need at least 1GB of RAM).
{{/note}}

{{#info}}
Aerospike provides [CloudFormation](https://aws.amazon.com/cloudformation/) templates that are already configured with 
recommended settings. Please see the our [CloudFormation page](/docs/deploy_guides/aws/cloudformation) for more details
on how you can quickly get a cluster up and running.
{{/info}}

# Recommended Minimum Requirements

## OS
For AWS, we recommend using the latest version of
[Amazon Linux](http://aws.amazon.com/amazon-linux-ami/) or [Amazon Linux 2](https://aws.amazon.com/amazon-linux-2/)
for performance
reasons. We have observed Amazon Linux to have the best performance on AWS EC2
instances. The rest of supported OS's also work in AWS, but their
performance might lag as compared to AWS Linux.

{{#info}}
You don't need to make an OS choice when using the Aerospike Marketplace AMI. The Aerospike
AMI is based on Amazon Linux. You need to choose an OS only when building an
Aerospike server by creating an instance using Amazon-provided AMI and
installing Aerospike packages manually.
{{/info}}

{{#note}}
Amazon Linux is RHEL6 compatible (System V).  
Amazon Linux 2 is RHEL7 compatible (systemd).  
Take care to deploy the correct version of Aerospike and Aerospike tools based on the OS chosen.
{{/note}}

## Virtualization Type
We recommend using Hardware Virtual Machine
[(HVM)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/virtualization_types.html)
based AMIs for usage with Aerospike. In our benchmarks, we have seen an approximate 3x performance
gain without any other tuning when using HVM
instances instead of PV instances.

{{md (path-resolve (path-dirname page.src) "/docs/operations/configure/network/heartbeat/cloud_consideration.part.md") }}

## Network Setup
As a prerequisite of using HVM and Enhanced Networking, we recommend you use a
[VPC](http://aws.amazon.com/vpc/) based setup. You cannot use HVM AMIs and
Enhanced networking in Classic EC2 mode.

We suggest you use an AWS
[private IP](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#concepts-private-addresses)
for setup of mesh heartbeat rather than the public IP. Private IPs are allocated
to each EC2 instance by default. You may also need to add a public IP to your
instance if you need direct access to the instance. There are two ways
to add a public IP to your instance:

 - **During instance launch**: You can explicitly set the option to
   [enable a public IP](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-ip-addressing.html#vpc-public-ip)
   for your instance while launching the instance. You can also
   [modify the behavior of your VPC subnet](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-ip-addressing.html#subnet-public-ip)
   to enable it by default.
 - **Attach an Elastic IP after launching instance**: An
   [Elastic IP address](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-ip-addressing.html#vpc-eip-overview)
   is a static, public IP address designed for dynamic cloud computing. You can
   [associate an Elastic IP](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-ip-addressing.html#WorkWithEIPs)
   address with any instance or network interface for your VPC.

{{md (path-resolve (path-dirname page.src) "../configure/network/heartbeat/mesh.part.md") }}

### Network Interface
Each network interface on an amazon Linux HVM instance can handle about 250K packets
per second. If you need higher performance per instance, you need to do one of
the following:
- **Add More NIC/ENI**
  Elastic Network Interfaces (ENI) provide a way of adding multiple (virtual)
  NICs to an instance. A single NIC peaks at around 250k TPS, bottlenecking on
  cores processing interrupts. Adding more interfaces help to process more
  packets per second on the same instance. Using ENIs with private IPs is free
  of cost in AWS.
   
  {{#note}}
  Starting with Aerospike 3.10, you can specify separate network interfaces for service, info and fabric traffic.
  Read more [here](/docs/operations/upgrade/network_to_3_10).

  This will help alleviate both packets per second and bandwidth concerns with individual ENIs.
  {{/note}}

- **Receive Packet Steering**
  {{#note}}
  RPS is only available in kernel version 2.6.35 and above.
  {{/note}}
  Another simpler approach is to distribute IRQ over multiple cores using
  [RPS](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/network-rps.html)
  ```asciidoc
  echo f > /sys/class/net/eth0/queues/rx-0/rps_cpus
  ```

  With Aerospike, this eliminates the need to use multiple NICs/ENIs, making management easier,
  and result in similar TPS. A single NIC with RPS enabled can achieve
  up to 800K TPS with interrupts spread over 4 cores.

  {{#info}}
  AWS has introduced Elastic Network Adapters, or [ENAs](https://aws.amazon.com/blogs/aws/elastic-network-adapter-high-performance-network-interface-for-amazon-ec2/),
  that supports Multi-Queue device interface and Receive-Size Steering on select instance types. This
  makes the above Receive Packet Steering and the addition of more NIC/ENIs redundant. Additional NIC/ENIs can still be beneficial in cases of XDR, Heartbeat and Fabric isolation. 

  ENAs are only supported on select [instance types](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html).
  {{/info}}

## Security Group
Aerospike needs TCP ports **3000-3003** for intra-cluster communication. These
ports need not be open to the rest of Internet.

If using XDR, port **3000** (or the info port for
remote data centers' aerospike) of destination datacenter should be reachable
from the source datacenter.

{{#note}}
Pre Aerospike 3.8, XDR required port **3004** to be open.
{{/note}}

[AMC](/docs/amc) requires port 8081 for access from the Internet.

Additionally, you will need a port for
[SSH access](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
to your instances (default TCP port 22)


## Redundancy Using Availability Zone

To add further redundancy to Aerospike in AWS using
[Availability Zone (AZ)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html),
you can set up one cluster across multiple different availability zones such that there is one set of data in each AZ by leveraging the [Aerospike Rackware feature](/docs/operations/configure/network/rack-aware/). 

{{md (path-resolve (path-dirname page.src) "../../common/availability-suggestions.part.md") }}

## Initializing EBS and Ephemeral disks

Intializing (formerly known as pre-warming) EBS volumes is only required for volumes that were restored from a snapshot. Blank EBS volumes
[do not require initialization](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-initialize.html).

Some ephemeral volumes also needs initializing. Consult [this chart](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html#instance-store-volumes)
on which instance's volumes requires initialization.

{{#note}}
The following command will read every block to initialize a volume
{{/note}}

```asciidoc
sudo dd if=/dev/<deviceID> of=/dev/null bs=1M &
```

## Swapping to device storage

When a raw device is used for storage, it must be either:

* Zeroed to instantiate as an empty device  
**or**
* In a state left by Aerospike

This is because on startup, Aerospike will scan the entire device to discover the state of the data on the device.
If the device was used previously for another purpose, like file storage, the leftover data is essentially corrupt to Aerospike and will have undefined
behavior when scanned.

{{#info}}
Newly provisioned blank EBS volumes and all Ephemeral disks are already zeroed.
{{/info}}

{{#note}}
The following command will zero every block on a device
{{/note}}

```asciidoc
sudo dd if=/dev/zero of=/dev/<deviceID> bs=1M &
```
## i3 NVME SSD instances AMI version

AWS recommends using and Amazon Linux or Ubuntu 16.04 AMI launched after **March 29th, 2017** when using NVME SSD disk instances due to a driver issue.
The Ubuntu Cloud Image that AWS provides for Ubuntu 16.04 has been enabled with the AWS-tuned Ubuntu kernel. The new kernel by default comes enabled with improved I3 instance class support with NVMe storage disks under high IO load, increased I/O performance and improved instance initialization with NVMe backed storage disks.

More information can be found at the following link: 
https://insights.ubuntu.com/2017/04/05/ubuntu-on-aws-gets-serious-performance-boost-with-aws-tuned-kernel/ 


## Shadow device configuration

As noted above, some EC2 instance types have direct-attached SSDs called [Instance Store Volumes](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html), colliquially known as ephemeral drives/volumes. These can be significantly faster than EBS volumes (as EBS volumes are network attached). But, it is recommended by AWS to not rely on instance store volumes for valuable, long-term data, as these volumes are purged when the instance stops.

To take advantage of the fast direct-attached instance store SSDs, Aerospike provides the concept of [shadow device configuration](/docs/operations/configure/namespace/storage#recipe-for-shadow-device) where all writes are also propagated to these shadow devices. This is configured by specifying an additional device name in the **storage engine** block of the namespace configuration.
```
storage-engine device {
	    ...
        device /dev/sdb /dev/sdf
        write-block-size 1024K   # Write block size should be appropriate for object size and disk medium (SSD/HDD)
        ...
    }
```
In this case, ```/dev/sdb``` is the Instance store volume where all reads and writes will be directed to. The other **shadow** device ```/dev/sdf``` is the EBS volume where only the writes are propagated. In this way, we can achieve the high speeds of direct-attached SSDs while not compromising on the durability guarantees of EBS volumes. Note that the write throughput will still be limited by the EBS volume and hence this strategy will give good results when the percentage of writes is low.

{{#note}}
The shadow EBS device should have a size at least equal to the size of the direct-attached SSD.
{{/note}}
{{#note}}
For data-in-memory use cases with persistence, it may also be preferable to use an SSD direct-attached device alongside an EBS volume. In this case, it would be to save on IOPS cost incurred on read during the defragmentation process. The reads would be performed against the SSD device and re-written/defragmented blocks directly mirrored to the EBS volume.
{{/note}}

When partitioning shadow devices please consider the following recommendations
* Increasing partitions increases the number of write queues and defragmentation threads.  Typically, Aerospike recommends 4 partitions for a 900GB drive (r5d/c5d/m5d)  in the 12xl and 24xl sizes.  For smaller 300GB or 400GB drives 3 partitions are recommended.  For larger 1900GB drives on i3.2xl instances, 8 partitions are recommended.
* More partitions translate into a faster recovery time from shadow devices when the local ephemeral device is empty.


### EBS Snapshot Backups

[EBS Snapshots](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html) are an excellent method to create and maintain backups.
Snapshots maintain the state of an EBS volume at a particular point-in-time. Deploying an EBS volume based on a snapshot is essentially restoring the data from the time the snapshot was taken, into a new volume.

This is beneficial as a backup mechanism because:
* snapshots are taken extremely quickly
* snapshots are block-level consistent
* snapshots are [portable](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-modifying-snapshot-permissions.html)

With Aerospike, snapshots guarantee data consistency on a per-volume basis.

See our [Backup and Recovery](/docs/deploy_guides/aws/backup#ebs-snapshots) page for more details.

### Placement Groups

Placement groups are logical grouping of instances within a single AWS Availablility Zone. This provides the lowest latency and highest bandwidth for systems deployed within the same Placement Group. However, Placement Groups are not flexible and you may find yourself running into `insufficient capacity` errors should you try to scale up your cluster later on. More details about Placement Groups can be found in [Amazon's Documentation](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html).

{{#note}}
Aerospike does *not* recommend using Placement Groups in production due to these limitations.
{{/note}}

### Additional Information
* [Tuning for 1 Million TPS on an AWS instance](/docs/deploy_guides/aws/tune)
* [EBS Initialization](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-initialize.html)
* [EC2 Disk Performance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/disk-performance.html)
