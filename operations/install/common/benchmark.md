---
title: Benchmark Aerospike
description: Bernchmark your Aerospike Server installation. 
styles:
  - /assets/styles/ui/steps.css
---

{{#steps}}
{{!--- 
  ############################################################################
  #
  # STEP 1 - Benchmark Aerospike
  #
  ############################################################################ 
---}}
{{#steps-step 1 "Benchmark Aerospike" markdown=true}}

Aerospike provides benchmark applications for C, .Net, Java and Node.js. Here is the direct access for those benchmark applications:

- [Java Benchmark Application](/docs/client/java/benchmarks.html)
- [C Benchmark Application](/docs/client/c/benchmarks)
- [C# Benchmark APplication](/docs/client/csharp/benchmarks.html)
- [Node.js Benchmark Applicaiton](/docs/client/nodejs/benchmarks)

The following instructions are for the Java benchmark application. This application generates load on the cluster and calculates performance metrics for this Java client.  The benchmark application is normally installed into the benchmarks folder during the SDK installation. The benchmark program is an actual application that you can use to test your cluster – the application is not a simplified example. The benchmark application is useful for checking end-to-end latency and optimizing performance in your system.

With the benchmark tool, you can:

- Read and write data into the database at a particular read/write ratio.
- Change the number of client threads to simulate client concurrency.
- Look at latency distribution from the client side.

### Install the Java client, including the benchmark utility.

There are [several paths to installing the Java client](/docs/client/java/install).

### Build

```bash
$ cd benchmarks
$ mvn package
```

### Run

The following command will start the default load:
```bash
$ ./run_benchmarks
```

For further information, see the [Java Client's benchmark tool](/docs/client/java/benchmarks.html).

{{/steps-step}}
{{!--- 
  ############################################################################
  #
  # STEP 2 - See your records using `asadm`
  #
  ############################################################################ 
---}}
{{#steps-step 2 "See your records using **asadm**" markdown=true}}

`asadm` (installed in `/opt/aerospike/bin/asadm`) gets server/cluster statistics and presents them in an easy to read format. The `info` command (in `asadm`) provides a listing of information on your cluster, including node, namespace, object, etc. information.  Complete information on `asadm` can be found in the [TOOLS & UTILITIES documentation here](/docs/tools).

Here is an example of running `asadm` on a one node cluster:

```bash
$ ./asadm.py  -h 127.0.0.1 -p 3000
Aerospike Interactive Shell, version 0.1.1
Found 1 nodes
Online:  172.16.146.135:3000

Admin> info
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
               Node               Node                    Ip                  Build   Cluster            Cluster     Cluster         Principal   Client     Uptime   
                  .                 Id                     .                      .      Size                Key   Integrity                 .    Conns          .   
172.16.146.135:3000   *BB907DF26565000   172.16.146.135:3000   E-3.8.4-110-g97826f6         1   9203DDDCEBEE7D97   True        BB907DF26565000        2   01:43:20   
Number of rows: 1

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace                  Node   Avail%   Evictions     Master   Replica     Repl     Stop     Pending       Disk    Disk     HWM        Mem     Mem    HWM      Stop   
        .                     .        .           .    Objects   Objects   Factor   Writes    Migrates       Used   Used%   Disk%       Used   Used%   Mem%   Writes%   
        .                     .        .           .          .         .        .        .   (tx%,rx%)          .       .       .          .       .      .         .   
test        172.16.146.135:3000   N/E        0.000     95.881 K   0.000     1        false    (0,0)            N/E   N/E     50      7.150 MB   1       60     90        
test                                         0.000     95.881 K   0.000                       (0,0)       0.000 B                    7.150 MB                            
Number of rows: 2
```

{{/steps-step}}
{{/steps}}
