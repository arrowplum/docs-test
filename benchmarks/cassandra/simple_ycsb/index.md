---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real"
description: How to set up and run the benchmark used in the blog Comparing NoSQL Databases - Aerospike and Cassandra - Benchmarking for Real
styles:
  - /assets/styles/ui/steps.css
---
# Intro
Many benchmarks cite great performance numbers over short periods of time. In the blog post [Comparing NoSQL Databases - Aerospike and Cassandra: Benchmarking for Real](/blog/comparing-nosql-databases-aerospike-and-cassandra) we describe the results of testing on a simple large-scale use case for over 12 hours. Rather than simply present the results, we wanted to openly share how to recreate the benchmark steps. 

As you begin planning your own benchmark project, it is very important for you to consider your test environment. There are several very important factors to consider. Naturally, changing the values in the benchmark will lead to different expected results.

# Server Requirements
## Database Servers 
| Component | Hardware Used in Blog | Recommendation | Notes |
| --------- | ----- | --- | --- |
| CPU       | Dual Intel(R) Xeon(R) CPU E5-2665 0 @ 2.4GHz (8 cores) | Dual quad core 2.0GHz | For most testing, Aerospike generally does not need a fast CPU. It is only needed when the per node speeds exceed about 500ktps. For Cassandra, CPU is often the biggest bottleneck, so it will benefit from a faster CPU. |
| RAM       | 32 GB 1333 MHz | 32 GB RAM | The most important factor for RAM will the be the actual volume of RAM. While both databases will naturally benefit from faster RAM, the primary concern should be that you have enough RAM to store the index, data, and cache. |
| SSD       | 4 x 200 GB Intel s3700 | 4 x 200 GB SSD | Choose SSDs that have good performance. Aerospike publishes a set of performance benchmarks for SSDs (including those of cloud vendors). These are available on the [Aerospike SSD Certification Page](/docs/operations/plan/ssd/ssd_certification.html). | 
| Network   | Intel Corporation 82599ES 10-Gigabit |  10 Gb | One factor that can impact performance is the number of network queues on the NIC. Most 10Gb NICs today have at least 8 network queues and are generally good for benchmarks. In the best case scenario, the NIC adapts the number of queues to the number of cores on the server. There are 1Gb NICs that also support multiple queues. Note that in VMs, such as AWS, you may be limited to only 1 or 2 queues. To find the number of queues on your NIC, first identify the NIC's device id (e.g., eth2). Then, look at the file "/proc/interrupts". Transactions queues will have the format "[DEVICEID]-TxRx-[#]". The one with the highest number for the device will denote the total number of queues - starting with zero. Thus, if you see 16 queues with the highest being "eth2-TxRx-15", you have 16 queues for that NIC.|
| Operating System | CentOS 6.7 | CentOS 6.7 | These instructions and the scripts created for this benchmark were created on Linux CentOS 6.7 and will not work on another flavor or major version without modifications. |
  
## Client Server
| Component | Hardware Used in Blog | Recommendation | Notes |
| --------- | ----- | --- | --- |
| CPU       | Dual Intel(R) Xeon(R) CPU E5-2665 0 @ 2.40GHz (8 cores) | Dual 2.4GHz (8+ cores)| Generally speaking, it is best to use roughly the same caliber of CPU on the clients as on the database servers. |
| RAM       | 32 GB 1333 MHz | 16 GB RAM | The clients should have at least 16 GB of RAM to get the most out of them. Using less may result in unstable client behavior. |
| Network   | Intel Corporation 82599ES 10-Gigabit | 10 Gb | The networking here should match the networking on the server. |
| Operating System | CentOS 6.7 | CentOS 6.7 | These instructions and the scripts created for this benchmark were created on Linux CentOS 6.7 and will not work on another flavor or major version without modifications. |

# Next Steps
Choose one of the following installation methods for your own tests:

  * [Amazon AWS with Aerospike Benchmark AMI](/docs/benchmarks/cassandra/simple_ycsb/aws_install.html)
  * [Manual installation of software from a clean CentOS/RedHat installation](/docs/benchmarks/cassandra/simple_ycsb/manual_install.html)
  
{{#note markdown=true}}
Convention: To distinguish between commands that need to be run on the YCSB host vs. those that need to be run on the database hosts, we have used different command prompts. Commands intended for the YCSB host begin with a `ycsbhost $` prompt; those intended for the database hosts begin with a `dbhost $` prompt.

{{/note}}

