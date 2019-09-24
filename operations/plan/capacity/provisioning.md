---
title: Provisioning a Cluster
description: Examples of provisioning an Aerospike cluster based on capacity planning data
---

## Provision for a Flash (SSD) Database

Storing data on flash allows you to have a large database that is still very responsive. For example, if you want your database to handle approximately:

- 125 million objects at 1.5 K bytes/object
- 16K reads/sec and 8K writes/sec

Then you might install the server on a 2-node cluster where each server has:

- 1 Quad-core CPU (2.2+ GHz)
- 16 GB RAM
- 2 × 256 GB SSD
- 1 Gb Ethernet Network card

{{#note}}The SSDs that are used for data cannot be used for the operating system, so you would typically have a rotational drive for system files. In addition if you're planning to use SSD devices over 2 TiB, you would need to split them in multiple partitions of sizes less then 2 TiB (or have different files on them with filesize under 2 TiB chunks - if it is fine to use files and the performance downsides for the specific use case) since the maximum allowed value is 2 TiB per filesize or device. {{/note}}


## Provision for an In-Memory Database

You can also use Aerospike for an in-memory database. For example, let’s say you want your database to handle approximately:

- 50 million objects at 500 bytes/object
- 60K reads/sec and 30K writes/sec

In that case, you’d need to provision enough RAM for both data and index, as described above. So you might provision a 2-node cluster where each server has:

- 1 Quad-core CPU (2.2+ GHz)
- 64 GB RAM
- 1 Gb Ethernet Network card

### Provision Persistence for an In-Memory Database

If a namespace configured to store data in memory uses flash (SSD) for persistance, you can use the data sizing guide above, then factor in a multiple of 2 times to account for additional space needed for defragmentation. Alternatively, you can use a simpler rule-of-thumb of provisioning 4 times the size allocated for RAM to determine the flash-based persistence.

In the example above the RAM used on each server is 27 GB. To support
persistence on SSD you would also provision:

- 110 GB SSD

## Provision for Amazon EC2 (Cloud-Based)

You can use Aerospike with Amazon EC2 in either flash (SSD) or in-memory configurations.

For flash, if you chose a `i3.2xlarge` instance and with two servers, you would be able to support approximately:

250 million objects at 1.5 K bytes/object
40K reads/sec and 20K writes/sec
For an in-memory database, you would consider instances from the `r4`, `c5`, `c5d`, `x1e` families.

See the [Amazon EC2 Deployment Guide](/docs/deploy_guides/aws/index.html).

