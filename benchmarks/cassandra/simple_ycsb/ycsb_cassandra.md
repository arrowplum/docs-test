---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" - Running the Benchmark on Cassandra
description: How to run the YCSB test.
styles:
  - /assets/styles/ui/steps.css
---

Prior to starting these steps, you should have already done a basic installation of Cassandra on 3 database hosts and YCSB. If not, please go to one of the following links to do the basic installation:

  * YCSB (client) - [AWS installation](/docs/benchmarks/cassandra/simple_ycsb/ycsb_aws.html) or [manual installation](/docs/benchmarks/cassandra/simple_ycsb/client_install.html)
  * Cassandra - [AWS installation](/docs/benchmarks/cassandra/simple_ycsb/cassandra_aws.html) or [manual installation](/docs/benchmarks/cassandra/simple_ycsb/cassandra_install.html)

We recommend running short (10-minute) tests prior to running longer-term tests. This will let you determine good targets for the long-term test.

_(These benchmarks use the Aerospike fork of YCSB. Even though the only change we made to the standard YCSB distribution was to add a metric, the scripting for the graphs will not work with the standard YCSB distribution. Please make sure to follow the instructions below.)_


{{#steps}}



{{!--- 
  ############################################################################
  #
  # STEP 1 - Start Cassandra
  #
  ############################################################################ 
---}}
{{#steps-step 1 "Start Cassandra" markdown=true}}

First, make sure that all Aerospike nodes have stopped. Run the following command on each database host in your cluster:

```bash
dbhost$ service aerospike status
asd is stopped
```
On each database server in your cluster, start the nodes in turn:

```bash
dbhost$ cd /opt/cassandra/apache-cassandra-3.5/bin
dbhost$ sudo nohup ./cassandra -R &
```

{{/steps-step}}

{{!--- 
  ############################################################################
  #
  # STEP 2 - Load data into Cassandra
  #
  ############################################################################ 
---}}
{{#steps-step 2 "Local data into Cassandra" markdown=true}}
These steps are very similar to the ones for Aerospike: simply load the data and run the short tests. 

##Load Data
Now, execute the load workload file (replace "[NODE\_IP\_ADDR]" with the IP address of any node in your cluster):  
```bash
ycsbhost$ cd /opt/YCSB
ycsbhost$ nohup bin/ycsb load cassandra2-cql -s -threads 200 -P workloads/workload-shortload -p hosts=[NODE_IP_ADDR] > run.out 2> run.err &
```

As with the Aerospike tests, inspect the file `run.out`. The calculated "Throughput(ops/sec)" is a useful guide for a good starting point for Cassandra. However, since Cassandra tends to do much better on inserts than on mixed workloads, we have found that you may want to start benchmarking at 1/5th the level of the inserts for the benchmark run below. This is your target level.

##Clear filesystem cache
We recommended that you clear the filesystem cache after loading the data. To clear the cache, run the following command on each database server:

```bash
dbhost$ sudo sh -c 'echo 1 >/proc/sys/vm/drop_caches'
dbhost$ sudo sh -c 'echo 2 >/proc/sys/vm/drop_caches'
dbhost$ sudo sh -c 'echo 3 >/proc/sys/vm/drop_caches'
```



{{/steps-step}}

{{!--- 
  ############################################################################
  #
  # STEP 3 - Run the Benchmark to Determine Performance
  #
  ############################################################################ 
---}}
{{#steps-step 3 "Run the Benchmark to Determine Performance" markdown=true}}
Now that you have your target level, you can try running a short-term test.

##Run Benchmark Scripts

```bash
ycsbhost$ nohup bin/ycsb run cassandra2-cql -s -threads 200 -P workloads/workload-shortrun -p hosts=[NODE_IP_ADDR] -p target=[TARGET_LEVEL] -p maxexecutiontime=[RUNTIME_SEC] > run.out 2> run.err &
```
The output of the test results will be in the current directory. As YCSB is generally intended to run out of the installation directory, the output will be in `/opt/YCSB`.


{{/steps-step}}

{{!--- 
  ############################################################################
  #
  # STEP 4 - Repeat Cassandra Benchmark With New Levels
  #
  ############################################################################ 
---}}
{{#steps-step 4 "Repeat Cassandra Benchmark With New Levels" markdown=true}}

As with the Aerospike tests, you can now start another test at new levels. If the test fails to keep up, reduce TARGET_LEVEL and retry until your test is successful. If the test succeeds, increase the target level and retry.

To run the benchmark test, run the following, where:
  * [NODE\_IP\_ADDR] is the IP address of any node in your Cassandra cluster
  * [TARGET_LEVEL] is the target throughput rate
  * [SECONDS] is the number of second in the test, such as 21600 for a six hour test

To load data:
```bash
ycsbhost$ cd /opt/YCSB
ycsbhost$ nohup bin/ycsb load cassandra2-cql -s -threads 200 -P workloads/workload-shortload -p hosts=[NODE_IP_ADDR] > run.out 2> run.err &
```

To run the benchmark:
```bash
ycsbhost$ nohup bin/ycsb run cassandra2-cql -s -threads 275 -P workloads/workload-shortrun -p hosts=[NODE_IP_ADDR] -p target=[TARGET_LEVEL] -p maxexecutiontime=[RUNTIME_SEC] > run.out 2> run.err &
```

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 5 - Move Output Files
  #
  ############################################################################
---}}
{{#steps-step 5 "Move Output Files" markdown=true}}

When the test is complete, move the results to their own directory. The following command will create a directory based on the current time, and copy the `run.\*` and `\*.json` files.

The test run does not have to be complete in order to copy the files and generate graphs.

```bash
ycsbhost$ D=results/\`date +%F-%T\` ; mkdir -p $D ; cp run.out run.err $D; cp **.json $D
```

{{/steps-step}}

{{!--- 
  ############################################################################
  #
  # STEP 6 - Next Steps
  #
  ############################################################################ 
---}}
{{#steps-step 6 "Next Steps" markdown=true}}

The final step is to generate graphs from your test run. Go to the [Generating Graphs](/docs/benchmarks/cassandra/simple_ycsb/graphs.html) page for instructions on how to do this.
{{/steps-step}}


{{/steps}}
