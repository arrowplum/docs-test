---
title: Certifying Flash Devices (SSDs)
description: Use the Aerospike Certification Tool (ACT) test framework to evaluate flash device performance by simulating a steady read and write workload, and measuring response time prior to your first production release.
---

<style>
.certified-drives em {
  color: red;
  font-style: italic;
}
</style>

The Aerospike database is highly optimized to run on Flash and SSD devices, and is capable - through Hybrid Memory index use,
and direct device access - of providing unique throughput and low latency on Flash.

Over the years, we have found the exact characteristics of different Flash devices are of crucial interest to Aerospike customers.
The most important device characteristics are not captured by any specification sheet - the performance of read latency under sustained write load.

Of course, Aerospike can be run in a variety of other configurations, including pure in-memory mode, memory backed with devices for persistance,


For this purpose, we wrote the Aerospike Certification Tool ([ACT](https://github.com/aerospike/act)), and started gathering and publishing results.

We have published a blog in 2012 about the justification for this test, and we have published the source code,
allowing device manufacturers to include [ACT](https://github.com/aerospike/act) in their internal engineering. Both the justification and source code remain unchanged.

This page transmits the results we have observed and collected. We update this page with new drives, we archive information
for drives that are no longer available. With these results, you will have a very specific result to a
question of "whether this drive will work in my environment" - although your environment might be different from the data points we have collected.

A profile has three fundamental characteristics:
- Read / write ratio
- Object size
- Latency requirements of read operations (detailed below)

The test is run by increasing throughput and determining whether the latency requirement is
met --- and increasing throughput again, if the device is within latency requirements, until
the latency SLA is no longer met.

Please be careful regarding using these results if your intended workload varies from a published workload.
We make available the tools to allow you to test your intended workload, and testing may be required in your individual case.

Finally, we have also found the results themselves are only a starting point for making a purchase decision.

Two fundamental factors must be considered: the price of the device, and the wear rating ( DWPD ).

A device that wears fast may not be a bargain in a high-write environment.

A slow but very inexpensive device may
outperform a faster but more expensive device --- by purchasing more drives. As individuals have different available discounts, 
you may need to explore pricing of different drives before making a final decision.

An important calculation must be made: speed at a given capacity. It should not be assumed that a larger drive is faster. In general
more Flash capacity should result in a faster drive, but internal controller and datapath bottlenecks will present themselves.
For many manufacturers, 1.6T drives are the \"sweet spot\" of performance, but we are starting to see 3.2T drives be the maximum
performance per price ratio. We have perfomance measurements for some drives in different capacities.

While we make our raw data and findings available, Aerospike's solutions architects are available to
help you size your system. With our years of experience helping deploy clusters both large and small,
we'll help you through the process of choosing your next set of hardware.

## Guide to these numbers

Aerospike allows manufacturer-supplied numbers. In those cases, we receive log files from a manufacturer, and do our best
to validate. The cases where we have received values from a manufacturer, we will detail the configuration information
they have supplied, and are often also supplied drives, which we are in process of testing. In the appendix, we will detail what
the manufacturer has told us about test environment.

ACT has two fundamental versions: version 3.0 and version 4.0. We believe the core case, properly configured, results
in very similar results regardless of version, and have validated similar numbers between ACT 3.0 and 4.0. 
However, we have called out the version number for completeness.

Values for endurance are taken from manufacturer's spec sheets and have not been independantly verified.

## Operational Database Devices

Aerospike is often used as an operational database for transactions and user data.

This [ACT](https://github.com/aerospike/act) workload has the following characteristics:
66 % write to 33 % read ratio
1.5KB object size
< 5% exceed 1 ms ; < 1% exceed 8 ms ; < 0.1% exceed 64 ms

The tables below show the results from the Aerospike Certification Tool ([ACT](https://github.com/aerospike/act)) on some popular flash devices that we have tested and that our customers are using in production with the Aerospike database. The test results are split into 6 different categories:

- PCIe/NVMe-based 3D XPoint
- PCIe/NVMe-based flash
- SATA/SAS-based flash
- M.2-based flash
- Networked storage
- Cloud-based flash

To read the results of the test, the devices are run at a constant rate over a period of 24 hours. This is done to remove the effect of any caching and represents the long-term (rather than burst) performance of the disk. The steady state results - which represent the latency histogram after a number of hours and where the results appear to be stable after a number of hours,  appear below. 

Each column represents a time threshold in milliseconds. The numbers beneath are the percentage of transactions that exceed that threshold. For instance, in the table below, the Intel DC s3710 had 1.6% of transactions in excess of 1 millisecond.

These criteria may be relaxed by users depending on the need. In some cases, a manufacturer may have sacrificed a little performance to achieve greater consistency.

Your results may vary from these published numbers due to differences in the server, differences in the RAID controller, or even variations between devices of the same model.

<a name="approved_xpoint">
### PCIe/NVMe-based 3D XPoint
</a>

3DXpoint is a next-generation memory technology that is positioned between DRAM and flash.
Practical devices have been brought to market by Intel under the Optane brand. These devices should be considered
when low latency under sustainted write load is required, as average latency is substantially lower under write load.

While transactions per second are high, the extraordinary low latency can easily be compared to NAND Flash drives below.

**These devices were tested at the specified speed with a 67% read/33% write ratio of 1.5 KB objects over 24 hours.**

| 3D XPoint NVMe Device            | Speed (tps) | >1ms   | >8ms  | >64ms | Endurance | ACT | Source    |
| ---                              | ---         | ---    | ---   | ---   | ---       | --- | ---       |
| Intel SSD DC P4800X 375G         | 435,000     | 0.10%  | 0.01% | 0.00% | 30 DWPD   | 3.1 | Aerospike |
| Intel SSD DC P4800X 750G         | 435,000     | 0.06%  | 0.00% | 0.00% | 30 DWPD   | 4   | Aerospike |

<a name="approved_nvme">
### PCIe/NVMe-Based Flash
</a>

**These devices were tested at the specified speed with a 67% read/33% write ratio of 1.5 KB objects over 24 hours.**

| Flash Device                 | Speed (tps) | >1ms   | >8ms  | >64ms | Endurance | ACT | Source    |
| ---                          | ---         | ---    | ---   | ---   | ---       | --- | ---       |
| SmartIOPS DataEngine 6.4 TB  | 825,000     | 0.30%  | 0.00% | 0.00% | 3 DWPD    | 3   | SmartIOPS |
| SmartIOPS DataEngine 3.2 TB  | 630,000     | 1.5%   | 0.01% | 0.00% | 3 DWPD    | 3   | SmartIOPS |
| Huawei ES3600P V5 3.2 TB     | 384,000     | 2.72%  | 0.02% | 0.00% | 3 DWPD    | 4   | Aerospike |
| ScaleFlux CSS1000 6.4 TB     | 324,000     | 4.12%  | 0.00% | 0.00% | 5 DWPD    | 3   | Aerospike |
| Micron 9200 Max 1.6 TB       | 316,500     | 4.64%  | 0.13% | 0.00% | 3 DWPD    | 3   | Aerospike |
| Intel P4610 1.6 TB           | 300,000     | 4.56%  | 0.85% | 0.00% | 3 DWPD    | 4   | Aerospike |
| Intel P4610 6.4 TB           | 300,000     | 4.98%  | 0.14% | 0.00% | 3 DWPD    | 4   | Aerospike |
| ScaleFlux CSS1000  3.2 TB    | 300,000     | 4.59%  | 0.00% | 0.00% | 5 DWPD    | 4   | ScaleFlux |
| Micron 9200 Pro 3.84 TB      | 283,500     | 4.54%  | 0.13% | 0.00% | 1 DWPD    | 3   | Micron    |
| Huawei ES3600P V5 1.6 TB     | 180,000     | 4.47%  | 0.25% | 0.00% | 3 DWPD    | 4   | Aerospike |
| Toshiba CM5 3.2 TB    | 150,000     | 3.70%  | 0.00% | 0.00% | 3 DWPD   | 3   | Toshiba   |
| Toshiba PX04PMB320 3.2 TB    | 135,500     | 4.73%  | 0.00% | 0.00% | 10 DWPD   | 3   | Toshiba   |
| Intel P4510 1 TB             | 120,000     | 4.31%  | 0.00% | 0.00% | 1 DWPD    | 4   | Aerospike |
| Samsung PM983 1.92 TB        | 108,000     | 0.09%  | 0.00% | 0.00% | 1.3 DWPD  | 3.1 | Aerospike |



<a name="approved_sata">
### SATA/SAS-Based Flash
</a>

The flash device sizes listed below are *after* any overprovisioning that was done to the drive (by setting Host Protected Area using hdparm).

**These devices were tested at the specified speed with a 67% read/33% write ratio of 1.5 KB objects over 24 hours. _Any flash devices marked in red are not recommended_.**

<div class="certified-drives">
{{#markdown}}

| Flash Device                 | Speed (tps) | >1ms   | >8ms  | >64ms | Endurance | ACT | Source    |
| ---                          | ---         | ---    | ---   | ---   | ---       | --- | ---       |
| Micron 5200 MAX              | 33,000      | 4.51   | 0.02% | 0.0%  | 5 DWPD    | 3   | Micron    |
| Micron 5200 PRO              | 15,000      | 4.75   | 0.01% | 0.0%  | 5 DWPD    | 3   | Micron    |
{{/markdown}}
</div>

### M.2-Based Flash

M.2 is a new and emerging standard for SSDs. While M2 is not as performant as its PCIe cousin, the small form factor allows for a much higher density of storage at a more moderate price.

**These devices were tested at the specified speed with a 67% read/33% write ratio of 1.5 KB objects over 24 hours.**

| Flash Device                    | Speed (tps) | >1ms   | >8ms  | >64ms |
| ---                             | ---         | ---    | ---   | ---   |
| LiteOn EP1-KB480 (OP to 400 GB) | 9,000       | 4.05%  | 0.01% | 0.00% |


### Networked Storage

Storage technology is continually evolving to provide innovative solutions for the demands of databases like Aerospike.  Networked storage utilizes disaggregation to provide efficiencies in resource utilization and flexibility.
 
To keep Aerospike community informed about the options available for storage, we are adding a section to the ACT results for networked storage solutions. Depending on the use case and SLA requirements these solutions may provide the right performance, efficiency, and cost for your deployment.

| Storage Solution            | Speed (tps) | >1ms   | >8ms  | >64ms | Device| Protocol | ACT |  Source    |
| ---                              | ---         | ---    | ---   | ---   | ---       | ---       | --- | ---       |
| Drivescale Composer        | 300,000     | 3.85%  | 0.01% | 0.00% | HSGT Ultrastar SN200 3.2 TB    | iSCSI | 4 | Drivescale |
| Drivescale Composer      | 300,000     | 2.60%  | 0.00% | 0.00% | HSGT Ultrastar SN200 3.2 TB    | RoCEv2 | 4 | Drivescale |
| Toshiba Kumoscale | 150,000     | 3.97%  | 0.01% | 0.00% | Toshiba CM5 3.2 TB             | RoCEv2| 3.1 | Toshiba |


### Cloud-Based Flash

Cloud servers with flash devices are becoming increasingly common. The results below are for a **single** flash device (unless mentioned otherwise). 
Multiple flash devices may sometimes be attached to each instance in order to increase throughput linearly. The recorded times are for the best throughput.

**These devices were tested at the specified speed with a 67% read/33% write ratio of 1.5 KB objects over 24 hours.**

| Cloud Provider | Instance Name       | Speed (tps) | >1ms  | >8ms  | >64ms |
| ---            | ---                 | ---         | ---   | ---   | ---   |
| Google         | n1-standard-2\*     | 120,000     | 0.29% | 0.09% | 0.00% |
| Amazon         | c5d.18xlarge        | 114,000     | 2.26% | 0.00% | 0.00% |
| Amazon         | r5d.24xlarge        |  90,000     | 0.25% | 0.04% | 0.00% |
| Amazon         | r5d.12xlarge        |  90,000     | 0.98% | 0.00% | 0.00% |
| Amazon         | c5d.9xlarge         |  90,000     | 1.30% | 0.12% | 0.00% |
| Amazon         | r5d.2xlarge         |  30,000     | 0.63% | 0.03% | 0.02% |
| Azure          | L16sv2              |  15,000     | 3.37% | 0.11% | 0.10% |
| Amazon         | i3.metal            |  12,000     | 1.06% | 0.00% | 0.00% |
| Amazon         | i3.2xlarge          |  12,000     | 1.24% | 0.00% | 0.00% |

{{#info markdown=true}}\* Google local SSDs (SCSI) are independant of instance type. These results are 'gen2' drives which are available primarily on 'Intel Cascade Lake or later', with linear performance gain through 4 devices per instance.{{/info}}

{{#warn markdown=true}}Cloud-based flash devices show great variability. We recommend that you test each new instance. The best numbers tested for a single flash device are provided above.{{/warn}}



### Historical Flash Devices

The devices listed below are older (deemed "historical").  Therefore, we don't recommend they be used.

**SATA/SAS-Based Flash:** The flash device sizes listed in the table below are after any overprovisioning that was done to the drive (by setting Host Protected Area using hdparm). These devices were tested at the specified speed with a 67% read/33% write ratio of 1.5 KB objects over 24 hours.

| Flash Device                    | Speed (tps) | >1ms   | >8ms  | >64ms |
| ---                             | ---         | ---    | ---   | ---   |
| Intel DC s3700 + OP (318GB)             | 18,000      | 1.60%    | 0.00%    | 0.00%   |
| Samsung 843T + OP (370GB)               | 9,000       | 2.31%    | 0.00%    | 0.00%   |
| Micron M500DC 480 GB + OP (300GB)       | 9,000       | 2.95%    | 0.20%    | 0.02%   |
| Seagate 600 Pro + OP (240GB)            | 9,000       | 5.52%    | 0.00%    | 0.00%   |
| Intel DC s3500 + OP (240GB)             | 9,000       | 8.12%    | 0.00%    | 0.00%   |
| _Micron P400e (200 GB)_                 | _9,000_     | _12.30%_ | _4.86%_  | _3.89%_ |

**PCIe/NVMe-Based Flash:** The performance numbers below are to the highest levels passed. None of these drives were overprovisioned. These devices were tested at the specified speed with a 67% read/33% write ratio of 1.5 KB objects over 24 hours.

| Flash Device                    | Speed (tps) | >1ms   | >8ms  | >64ms |
| ---                             | ---         | ---    | ---   | ---   |
| Micron P320h 700GB           | 450,000     | 3.32%  | 0.04% | 0.00% |
| Intel P3700 400 GB *         | 210,000     | 2.20%  | 0.18% | 0.00% |
| Samsung SM1715 3.2 TB        | 192,000     | 3.68%  | 0.00% | 0.00% |
| Micron P420m 1400 GB         |  96,000     | 3.21%  | 0.00% | 0.00% |
| Intel P3608                  |  84,000     | 4.37%  | 0.00% | 0.00% |
| Huawei es3000 2400 GB        |  72,000     | 1.18%  | 0.00% | 0.00% |

{{#info markdown=true}}\* These tests were run on a Dell R720 with dual E5-2690v2 @ 3GHz using RedHat 6.5 kernel 2.6.0.{{/info}}

**Cloud-Based Flash:** The performance of devices on cloud instances that might
not be available for provisioning any longer, or have better alternatives now.

| Cloud Provider | Instance Name       | Speed (tps) | >1ms  | >8ms  | >64ms |
| ---            | ---                 | ---         | ---   | ---   | ---   |
| Azure          | L8s\*               | 30,000      | 1.26% | 0.05% | 0.00% |
| Azure          | GS2                 | 30,000      | 1.30% | 0.21% | 0.13% |
| Amazon         | r3.4xlarge          | 18,000      | 3.74% | 0.01% | 0.00% |
| Amazon         | m3.xlarge (HVM)     | 18,000      | 2.49% | 0.00% | 0.00% |
| Amazon         | r3.2xlarge          | 15,000      | 4.71% | 0.10% | 0.02% |
| Amazon         | r3.xlarge           | 15,000      | 4.90% | 0.45% | 0.35% |
| Amazon         | r3.large            | 12,000      | 4.12% | 0.24% | 0.00% |
| Amazon         | c3.2xlarge          | 9,000       | 2.10% | 0.10% | 0.01% |
| Amazon         | m3.xlarge           | 9,000       | 4.14% | 0.23% | 0.01% |
| Rackspace      | Performance 1       | 9,000       | 1.14% | 0.00% | 0.00% |

{{#info markdown=true}}\* Azure Ls performance is for the single drive on the instance. The Azure
Lsv2 instances have multiple drives equivalent to 5x rating each, so in
aggregate they perform much better than the first generation Ls instances.{{/info}}

