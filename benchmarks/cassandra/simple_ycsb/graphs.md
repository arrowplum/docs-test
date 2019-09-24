---
title: Steps to Generate Graphs from YCSB Tests
description: These steps will guide you is creating graphs from the output of the 
styles:
  - /assets/styles/ui/steps.css
---
While this step is optional, it can provide insights into the performance of each database. 

Recent Aerospike blog posts show graphs with results superimposed on each other. Creating graphs like these
often requires choosing colors, planes, and axes manually. Here, we will describe only
the process for drawing simple graphs with one database per graph.

{{#steps}}
{{!---
  ############################################################################
  #
  # STEP 1 - Prerequisites
  #
  ############################################################################
---}}
{{#steps-step 1 "Prerequisites" markdown=true}}
You should already have moved the output from the benchmark tests into a directory of its own.

Now cd into that directory, as follows:

```bash
ycsbhost$ cd [TEST_OUTPUT_DIR]
```
{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 2 - Create File Of Files (FOF) data files
  #
  ############################################################################
---}}
{{#steps-step 2 "Create File Of Files (FOF) data files" markdown=true}}
First, you must create a set of files that lists the files to be analyzed. Since there is only one such file for each type of run (READ v. UPDATE), this step may not seem necessary. However, these instructions are intended to eventually take input from multiple clients for high-performance runs, so we use this more complex process.

The easiest way to create the file is to simply run:

```bash
ycsbhost$ ls READ\* > read.fof
ycsbhost$ more read.fof
READ.json
ycsbhost$ ls UPDATE\* > update.fof
ycsbhost$ more update.fof
UPDATE.json
```
{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 3 - Generate read graphs
  #
  ############################################################################
---}}
{{#steps-step 3 "Generate read graphs" markdown=true}}

Now simply run the script on the read.fof file as follows (DATABASE_NAME - either "Aerospike" or "Cassandra" - refers to the database being tested. This field is only used in the title of the chart):

```bash
ycsbhost$ /opt/aerospike-benchmarks/scripts/plotReadData.py -i read.fof -d [DATABASE_NAME]
```
For 12-hour runs, it may take 30 minutes or longer to generate the output.

The output of the scripts is as follows:
  * ReadLatency.Aerospike.png - A histogram of the latency of the read requests.
  * ReadLogHist.Aerospike.png - A histogram of the latency of the read requests plotted as log-linear. This makes it easier to see the longer latency behavior.
  * ReadThroughput.Aerospike.png - The throughput of the read requests over time.
{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 4 - Generate update graphs
  #
  ############################################################################
---}}
{{#steps-step 4 "Generate update graphs" markdown=true}}

Now run the following script on the update.fof file:

```bash
ycsbhost% /opt/aerospike-benchmarks/scripts/plotUpdateData.py -i update.fof -d [DATABASE_NAME]
```
For 12-hour runs, it may take 30 minutes or longer to generate the output.

The output of the scripts is as follows:
  * UpdateLatency.Aerospike.png - A histogram of the latency of the update requests.
  * UpdateLogHist.Aerospike.png - A histogram of the latency of the update requests plotted as log-linear. This makes it easier to see the longer latency behavior.
  * UpdateThroughput.Aerospike.png - The throughput of the update requests over time.
{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 5 - Next steps
  #
  ############################################################################
---}}
{{#steps-step 5 "Next steps" markdown=true}}

If you have already tested Aerospike and are now ready to test Cassandra, please reconfigure the database servers according to either the [AWS Setup of Cassandra](/docs/benchmarks/cassandra/simple_ycsb/cassandra_aws.html) or the [Manual Installation of Cassandra](/docs/benchmarks/cassandra/simple_ycsb/cassandra_install.html).

If you have completed your database tests, please let us know about your experience on the [Aerospike user forum](https://discuss.aerospike.com/).

{{/steps-step}}
{{/steps}}
