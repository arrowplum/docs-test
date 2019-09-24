---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" with Manual Installation
description: How to set up and run the YCSB test
styles:
  - /assets/styles/ui/steps.css
---


The manual installation instructions assume you are using hosts with a clean CentOS/RedHat 6.7 OS installation. You will need to do 3 different installations on the 2 different sets of hosts below: 

  * YCSB (Client) host - The instructions call for a single host. 
  * Database hosts - You will need 3 or more hosts and use the same servers for both Aerospike and Cassandra.

If you are uncertain of the hardware to use, please look at the [Server Requirements in the Simple YCSB page](/docs/benchmarks/cassandra/simple_ycsb/index.html).




Install and configure the software on the hosts using these steps in the following order:

  1. [Install the YCSB (Client) on a Host (1 host only)](/docs/benchmarks/cassandra/simple_ycsb/client_install.html)
  1. [Install and Configure Aerospike (3 hosts recommended)](/docs/benchmarks/cassandra/simple_ycsb/aerospike_install.html)
  1. [Run the YCSB Benchmark on Aerospike](/docs/benchmarks/cassandra/simple_ycsb/ycsb_aerospike.html)
  1. [Install and Configure Cassandra (same hosts as with Aerospike)](/docs/benchmarks/cassandra/simple_ycsb/cassandra_install.html)
  1. [Run the YCSB Benchmark on Cassandra](/docs/benchmarks/cassandra/simple_ycsb/ycsb_cassandra.html)
  1. [Generate Graphs from Your Data](/docs/benchmarks/cassandra/simple_ycsb/graphs.html)
