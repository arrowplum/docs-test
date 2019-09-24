---
title: Recommendations for Openstack
description: |
  Follow these easy steps to achieve a successful deployment of the Aerospike database on Openstack
scripts:
  - /assets/scripts/utils/target_blank.js

---

Due to Openstack's modularity, recommendations would be given on a per-component basis. Overall, Openstack should be treated no different than any other virtualized environment.


## Openstack Components/Projects

At a minimum, we require the following Openstack components to deploy Aerospike:

| Component | Purpose 	|
| --------- | --------- |
| Nova		| Compute	|

The following additional components are highly suggested to enhance or augment your Openstack deployment.

| Component | Purpose	|
| --------- | --------- |
| Neutron	| Networking/VPC	|
| Glance	| VM Imaging	|
| Cinder	| Block Storage	|
| Keystone	| Identity	|
| Horizon	| Dashboard	|
| Heat		| Resource Templating |

### OS

We recommend using the latest Debian or Redhat based OSs for your base install.

For help on installing, please see our [install page](/docs/operations/install/linux).

{{#note}}
The Glance service would aid in rapid deployment of multiple VMs after obtaining a working system.
{{/note}}

### Network Setup

{{#note}}
The Neutron service is highly recommended for network configuration.
{{/note}}

Private, self-service networks are recommended for intra-cluster networking.

In addition, due to the relatively high levels of traffic expected within a busy cluster, it is also recommended to utilize separate physical NICs and a separate network.

### Storage 

{{#note}}
The Cinder service is recommended for flexible storage configurations
{{/note}}

Cinder provides flexible block storage. We highly suggest utilizing SSDs for Cinder block stores.

{{#info}}
If you are familiar with AWS, Cinder providers the EBS and Ephemeral Storage features.
{{/info}}

** Cinder + Ceph **

Ceph is a distributed object store which can be used as the underlying block device that powers Cinder. Being distributed, each disk operation entails a subsequent network operation. This introduces latency.

As such, we do not recommend Ceph deployments with regular HDDs or high IO demands.

{{#warn}}
We do not recommend adding an SSD cache pool into Ceph. Contrary to expectations, an SSD cache would most likely degrade performance due to the additional complexity, cache-misses, and requirement to warm up the cache.

However, if your read workload mostly consists of recently written objects, then an SSD cache pool would be of some benefit.

Read more on [Ceph Cache Tiering](http://docs.ceph.com/docs/master/rados/operations/cache-tiering/).
{{/warn}}

[Data-in-memory](/docs/reference/configuration#data-in-memory) configurations are a good compromise to provide increased read performance without hitting the storage subsystem.

{{#note}}
It is doubly important to have dedicated NICs and separate networks between Openstack administative and user networks with the additional network load of Ceph.
{{/note}}

### Deployment and Configuration Management

Due to the modular nature of Openstack, there is not a universal deployment and configuration management tool within Openstack. 

Until the Openstack ecosystem has a widely adopted module for configuration management, we recommend using an external 3rd party tool like Ansible, Puppet, Chef, Salt, etc al... to manage your cluster configurations.

{{#note}}
HEAT provides deployment management via resource orchestration, but is otherwise incapable of configuration management. Notably, there is no peer-discovery mechanism available within deployed VMs.
{{/note}}
