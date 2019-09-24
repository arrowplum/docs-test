---
title: Server Hardware Requirements
description: A guide for optimizing hardware for the Aerospike database.
---

All databases have strong reliance on the underlying hardware. It is not always obvious what your priorities should be in determining what to purchase. In some cases your environment is fixed and the choices are limited; in others, you have complete freedom. We discuss below the reliance of Aerospike on different hardware with a list of cautions and recommendations that will guide you to the optimal solution for your environment.

### Minimum Requirements for Production Deployment

**CPU**: One quad core CPU.

**Memory**: 4 GB of RAM is required. As indexes are stored in memory, the amount of memory will limit the number of rows the hardware can store. Aerospike is very memory efficient, and each row (object or record) requires only 64 bytes of memory for the index. This means each GB of memory can index 16M rows, and a 4 GB configuration can index 64 million objects. For development purposes, you may make due with as little as 2 GB of RAM.

### CPU

How Aerospike uses CPU:

- While every software product naturally depends on CPU, Aerospike does not have a strong direct dependency on the CPU performance. Due to the simple nature of the get/set query, CPUs are rarely stressed by the database process.

Recommendations:

- Although there is not a strong direct dependency on CPU, you may find that CPUs are saturated with system interrupts (please see the network section above).
- Also, the amount of memory you can have on your server may be determined by the CPU you use. There are currently some differences between the Intel i-series (e.g. i5 or i7) and the Intel e-series (e.g. e5 or e7). The Intel e-series allow you to use much more memory per CPU than the i-series. This may not be an issue for your use case, but you should check with the system manufacturer on what the memory limits are.
- We recommend turning hyperthreading off when running on older v1 CPU Intel chips. Hyperthreading is reliable from Intel CPU version v2 (ivy bridge) chips onward. In older chips (v1 sandy bridge and older) you can only rely on the physical number of cores (half of the hyperthreaded number). As a general advice, benchmark your deployment to be sure of which option works for you.
- Most customers will not see an improvement in performance using 2 or more sockets. While Aerospike will, in general, make use of more cores, multiple sockets are generally not useful.

### Hard disk

How Aerospike uses rotational hard disk:

- Aerospike makes use of hard disk to store the executables and configuration files.
- When RAM is used for storage, rotational hard disks are also used to persist the data into a file. When used for persistence:
  - Aerospike will store the data in memory and write a copy to disk. During normal operation this data is only read to defragment the persistence file.
  - If a node is taken down, the data that was stored in memory is lost. At startup, the data will be read from the persistence file to reconstruct the index and reload the data into memory. (Note: this can be turned off, if you wish to repopulate the data from the other nodes instead of the persistence file).

Recommendations:

- The recommended size of the persistence file is 4 times the amount of data stored in RAM. So if you want to store 40 GB of data in RAM, the persistence file should be 4 x 40 GB = 160 GB. The large size of the file is to allow for defragmentation of data and to account for [sizing](/docs/operations/plan/capacity) overhead differences between RAM and SSD storage.
- While not required, it is a good idea to put the persistence file on a different disk than the OS.
- Rotational speed is not crucial, so while 10K or 15K RPM disks wouldn’t hurt, we have found very good performance with 7200 RPM disks.

### Memory (RAM)

How Aerospike uses RAM:

- Other than standard RAM required to run the process, Aerospike always stores data for the index in RAM. Note that the index is based on a hash of the key and results in a fixed 64 bytes per record.
- If using SSDs for storage, no additional RAM is required.
- If storing data in RAM, you must also account for the storage of data in RAM. You should multiply this by the replication factor.

Recommendations:

- In our tests, the specific type of memory and even speed has not significantly affected performance.
- The major precaution you should take is with capacity. While it is easy to fill a modern motherboard with 32 GB of RAM, it is also easy to forget the limits and cost of upgrading. Some of our customers have looked at putting 64 GB of RAM on a motherboard and assume that 256 GB will simply be 4 times the cost. However, some motherboards may have much lower limits and even then sometimes the costs of higher density RAM are prohibitive. Please plan accordingly.


&nbsp;

---

<div class="clearfix"></div>
<div class="pull-left">
<a class="btn btn-default" href="/docs/operations/plan/capacity">&lsaquo; Capacity Planning</a>
</div>
<div class="pull-right">
<a class="btn btn-primary" href="/docs/operations/plan/ssd">Flash Storage &rsaquo;</a>
</div>
<div class="clearfix"></div>

&nbsp;
