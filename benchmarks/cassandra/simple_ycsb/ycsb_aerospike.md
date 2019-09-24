---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" - Running the Benchmark on Aerospike
description: How to run the YCSB test.
styles:
  - /assets/styles/ui/steps.css
---

You should have already completed both of the following:

  * Installed YCSB (client) on [AWS](/docs/benchmarks/cassandra/simple_ycsb/ycsb_aws.html) or [manually](/docs/benchmarks/cassandra/simple_ycsb/client_install.html). 
  * Installed Aerospike on [AWS](/docs/benchmarks/cassandra/simple_ycsb/aerospike_aws.html) or [manually](/docs/benchmarks/cassandra/simple_ycsb/aerospike_install.html).

We recommend running short (10 minute) tests prior to running longer-term tests. This will let you determine good target rates for the long-term test.

_(These benchmarks use the Aerospike fork of YCSB. Even though the only change we made to the standard YCSB distribution was to add a metric, the scripting for the graphs will not work with the standard YCSB distribution. Please make sure to follow the instructions below.)_


In order to run the benchmark tests, do the following steps:

{{#steps}}

{{!---
  ############################################################################
  #
  # STEP 1 - Preparing to Start Aerospike
  #
  ############################################################################
---}}
{{#steps-step 1 "Preparing to start serospike" markdown=true}}

Prepare each Aerospike host as follows (this must be done on each Aerospike server):

  1. Make sure that Cassandra is not running. Run the following command (there should be no process with "cassandra" in it):
     ```bash
     dbhost$ ps -ef | grep java
     30386  30364  0 04:05 pts/0    00:00:00 grep --color=auto java
     ```
     The `grep` line simply refers to the search you just ran.

  1. Make sure your data devices are NOT mounted. Execute the `mount` command below; the results should not show your data drives, which are typically `/dev/sdb` or similar. The results below show that `/dev/sdb` is not mounted. If the device is mounted, you will need to execute `umount` or edit the `/etc/fstab` file and restart the host.

     ```bash
     dbhost$ mount
     proc on /proc type proc (rw,relatime)
     sysfs on /sys type sysfs (rw,relatime)
     /dev/xvda1 on / type ext4 (rw,noatime,data=ordered)
     devtmpfs on /dev type devtmpfs (rw,relatime,size=15701048k,nr_inodes=3925262,mode=755)
     devpts on /dev/pts type devpts (rw,relatime,gid=5,mode=620,ptmxmode=000)
     tmpfs on /dev/shm type tmpfs (rw,relatime)
     none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
     ```
  1. Make sure that Aerospike is not running. Run the following command to see its status:

     ```bash
     dbhost$ sudo service aerospike status
     ```
     If it is still up, you may stop it by running this command:
     ```bash
     dbhost$ sudo service aerospike stop
     ```


{{/steps-step}}
{{!---
  ############################################################################
  #
  # STEP 2 - Start Aerospike
  #
  ############################################################################
---}}
{{#steps-step 2 "Start Aerospike" markdown=true}}

To start Aerospike, you must do the following on each of the database servers:

  1. If you have previously used this SSD for testing, delete any old data that may be on the SSD by running the following command on each SSD device. Run this command in the background in case you become disconnected. The common EC2 device is `/dev/xvdb`. If this is the first time you are running either database on an Amazon instance, you may skip this step, as Amazon zeroes the drives during the launching of the instance.

     ```bash
     dbhost$ sudo dd if=/dev/zero of=/dev/[DEVICE_ID] bs=1M &
     ```

  1. Start Aerospike by running the following command:

     ```bash
     dbhost$ sudo service aerospike start
     ```
     You should allow a minute of time to make sure all the servers are clustered properly.

  1. Check that the `asd` processes have started and are functioning. Tailing the log file with `sudo tail -F /var/log/aerospike/aerospike.log` is prudent.

  1. Validate that the client can connect to the cluster by using the `asadm` command from a database node. Log into any database node and execute the following command, which checks the status of the Aerospike database (you can enter any of the nodes' internal IP addresses as NODE_IP):

     ```bash
     dbhost$ asadm -e info -h [NODE_IP]
     ~~~~~~~~~~~Network Information~~~~~~~~~~~
     Node                              Node                     Ip       Build   Cluster            Cluster     Cluster         Principal   Client     Uptime
        .                                Id                      .           .      Size                Key   Integrity                 .    Conns          .
     10.144.130.101:3000    BB96F82900A0022    10.144.130.101:3000   C-3.8.2.3         3   2F69E8DC370A6FF8   True        BB9D0CEFE0A0022      569   03:00:41
     10.144.156.102:3000    BB9159CC60A0022    10.144.131.102:3000   C-3.8.2.3         3   2F69E8DC370A6FF8   True        BB9D0CEFE0A0022      569   03:00:41
     10.144.130.103:3000    BB9D0CEFE0A0022    10.144.132.103:3000   C-3.8.2.3         3   2F69E8DC370A6FF8   True        BB9D0CEFE0A0022      569   03:00:41
     Number of rows: 3

     ~~~~~~~~~~~Namespace Information~~~~~~~~~~~
     Namespace              Node   Avail%      Evictions       Master     Replica   Repl      Stop     Pending         Disk    Disk     HWM         Mem     Mem    HWM      Stop
     .                         .        .               .     Objects     Objects Factor    Writes    Migrates         Used   Used%   Disk%        Used   Used%   Mem%   Writes%
     .                         .        .               .           .           .      .         .   (tx%,rx%)            .       .       .           .       .      .         .
     ycsb    10.144.130.101:3000       48               0   131.843 M   130.062 M      2     false       (0,0)   374.658 GB      51       75  15.611 GB      61     95        90
     ycsb    10.144.130.102:3000       46               0   134.468 M   134.492 M      2     false       (0,0)   384.750 GB      52       75  16.031 GB      62     95        90
     ycsb    10.144.130.103:3000       46               0   133.690 M   135.446 M      2     false       (0,0)   385.002 GB      52       75  16.042 GB      62     95        90
     ycsb                                               0   400.000 M   400.000 M                        (0,0)     1.118 TB                   47.684 GB
     Number of rows: 41
     ```

    The field "Cluster Size" should match the number of nodes in your cluster.

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 3 - Load data into Aerospike for short tests
  #
  ############################################################################
---}}
{{#steps-step 3 "Load data into Aerospike for short tests" markdown=true}}

Now that the database cluster is up, it is time to move to the client host and begin running tests using YCSB. 

We have provided some sample configuration files for short duration tests. While not perfectly accurate in terms of how a long duration test will run, they provide useful information on reasonable starting values. Starting with these values will, in a short amount of time, let you validate that the system works. 

The parameters for the short duration test are:
  * 1M records. 1 million records can be loaded in a short amount of time for both Aerospike and Cassandra, but is large enough to avoid hotkey issues.
  * 1,000 byte record size. This is the same size as the published benchmark values (10 fields of 100 bytes each)
  

## Change directory to the working directory

To load data into Aerospike, you will need to be in the main YCSB directory of the client host:
```bash
ycsbhost$ cd /opt/YCSB
```

## Start the sample YCSB test in the load phase

For the first runs of the test, start with the sample short runs. Execute the load workload file as follows, replacing "[NODE\_IP\_ADDR]" with the IP address of any host in your cluster:  
```bash
ycsbhost$ nohup bin/ycsb load aerospike -s -threads 200 -P workloads/workload-shortload -p as.host=[NODE_IP_ADDR] > run.out 2> run.err &
```


## Validate that data is loading correctly

Within a few minutes, you will see the following messages printed in the run.err file:

```bash
ycsbhost$ sudo tail -f run.err
2016-05-31 22:33:15:329 5180 sec: 396265670 operations; 66103 current ops/sec; est completion in 49 seconds [INSERT: Count=661025, Max=218111, Min=345, Avg=3021.97, 90=8279, 99=21791, 99.9=31247, 99.99=43039]
2016-05-31 22:33:25:329 5190 sec: 396916663 operations; 65099.3 current ops/sec; est completion in 41 seconds [INSERT: Count=650989, Max=78463, Min=357, Avg=3068.43, 90=8415, 99=19983, 99.9=28495, 99.99=34271]
2016-05-31 22:33:35:329 5200 sec: 397583493 operations; 66683 current ops/sec; est completion in 32 seconds [INSERT: Count=666838, Max=111679, Min=353, Avg=2995.15, 90=8223, 99=21551, 99.9=30175, 99.99=37983]
2016-05-31 22:33:45:329 5210 sec: 398244665 operations; 66117.2 current ops/sec; est completion in 23 seconds [INSERT: Count=661164, Max=87423, Min=342, Avg=3020.34, 90=8311, 99=20687, 99.9=28559, 99.99=35903]
```

In the log file `run.err`, you will see errors as shown below; these are known and not a problem:

```asciidoc
Starting test.
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
2016-05-24 22:41:27:905 0 sec: 0 operations; est completion in 0 seconds
Connected to cluster: YCSB Cluster
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
```

## Wait until data load is complete

When data load is complete, the YCSB process will exit, and a final log entry will be added to `run.err` as follows:

```asciidoc
[CLEANUP: Count=173, Max=174, Min=56, Avg=84.23, 90=120, 99=151, 99.9=174, 99.99=174] [INSERT: Count=449439, Max=96895, Min=349, Avg=1814.9, 90=4159, 99=13167, 99.9=22511, 99.99=33951]
```

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 4 - Inspect Aerospike Data Load Results
  #
  ############################################################################
---}}
{{#steps-step 4 "Inspect Aerospike data load results" markdown=true}}

Once loading has completed, inspect the file "run.out", as follows:

```bash
ycsbhost$ more run.out
[OVERALL], RunTime(ms), 10712.0
[OVERALL], Throughput(ops/sec), 93353.24869305451
[CLEANUP], Operations, 200.0
[CLEANUP], AverageLatency(us), 92.915
[CLEANUP], MinLatency(us), 49.0
[CLEANUP], MaxLatency(us), 514.0
[CLEANUP], 95thPercentileLatency(us), 155.0
[CLEANUP], 99thPercentileLatency(us), 264.0
[INSERT], Operations, 1000000.0
[INSERT], AverageLatency(us), 1788.039379
[INSERT], MinLatency(us), 42.0
[INSERT], MaxLatency(us), 507391.0
[INSERT], 95thPercentileLatency(us), 4651.0
[INSERT], 99thPercentileLatency(us), 8551.0
[INSERT], Return=OK, 1000000
```
Look at the value for "Throughput(ops/sec)" in the third line above. You'll want this throughput value as a starting point for your performance runs. The rate for a 50/50 workload will likely be between 50-100% of this value; this is a good initial range for the performance of this benchmark. For the above example, the throughput was "93353.24869305451", so you might select a level of 75,000 operations per second as the target for the actual benchmark.

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 5 - Run the Benchmark to Determine Performance
  #
  ############################################################################
---}}
{{#steps-step 5 "Run the benchmark to determine performance" markdown=true}}

## Run the benchmark for 10 minutes to determine initial performance

In order to get a good picture of operational latency, you don't want to overload a cluster. Approximating the correct performance will lead to more reasonable long-term test results, especially with databases of widely varying performance.

Run the benchmark program with short, 10-minute tests until you find a throughput level that can be maintained for the entirety of the 10 minutes. 

To run the benchmark test, run the following, where:
  * [NODE\_IP\_ADDR] is the IP address of any node in your Aerospike cluster
  * [TARGET_LEVEL] is the target throughput rate (start with the load write performance).


```bash
ycsbhost$ nohup bin/ycsb run aerospike -s -threads 275 -P workloads/workload-shortrun -p as.host=[NODE\_IP\_ADDR] -p target=[TARGET_LEVEL] -p maxexecutiontime=600 > run.out 2> run.err &
```

The output of the test results will be in the current directory. As YCSB is generally intended to run out of the install directory, this means you will be in the directory`/opt/YCSB`.

## Monitor the log files to see if the proposed performance level is being provided

If the short test provides the requested throughput, increase the target rate until you find a maximum. Then you can run the benchmark at a sustainable high level for an extended test, or decrease the target rate if the performance can't be met for the full 10 minutes. For Aerospike, 10-minute tests are a fair indicator of the rate that the test will be able to withstand. For Cassandra, you will generally need to be more conservative in your long-term throughput tests, since its compaction effects can take many hours to show.

In `run.err`, the current performance is provided, as follows:

```bash
ycsbhost$ tail -f run.err
2016-06-01 17:38:38:314 84 sec: 3327340 operations; 39992 current ops/sec;
2016-06-01 17:38:39:314 85 sec: 3367354 operations; 40014 current ops/sec;
2016-06-01 17:38:40:314 86 sec: 3407347 operations; 39993 current ops/sec;
2016-06-01 17:38:41:314 87 sec: 3447355 operations; 40008 current ops/sec;
```

If the number of operations per second is reached almost exactly, then the performance level is being met. Increase the target rate in the above command and try again to see how it performs at a higher load.

If the number of operations per second varies wildly (more than 20%), this indicates the cluster is struggling to provide that performance. Decrease the requested target rate and try again.

## Validate that servers are being stressed

In this set of instructions, we require a single client. This client's CPU performance needs to exceed that of the database server nodes. This simplifies running the tests, as it eliminates the need to coordinate multiple clients and combine their data output.

In general, a fully loaded Aerospike cluster will use all available CPU or disk throughput.

However, you may find that it is the client node that becomes a bottleneck. Check your database server statistics to verify.

Check to see if your database server CPU cores are being used with `top` (or the CPU monitoring tool of your choice).

```bash
dbhost$ top
```

The following example is reasonable for a 4 CPU server:

```asciidoc
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
 3067 root      20   0 20.0g  18g 4296 S 370.0 60.4   1084:54 asd
 2491 root      20   0  4380   88    0 S  1.0  0.0   1:18.59 rngd
 1794 root       0 -20     0    0    0 S  0.3  0.0   1:34.77 kworker/0:1H
```

Validate that server data devices are fully loaded with `iostat`. The following test shows the load on all devices at a two-second interval:

```bash
dbhost$ iostat -x -d 2
 Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await  svctm  %util
xvda              0.00     0.00    0.00    1.50     0.00    12.00     8.00     0.00    0.00   0.00   0.00
xvdb              0.00     0.00 26929.00  328.00 80787.00 83968.00     6.04     7.32    0.27   0.04  99.20
```
For a system under stress, the `%util` field value should be close to 100% for the SSDs in use. If the client is not fast enough, you may have to reconsider your hardware choice and choose more powerful hosts for the client.


## Terminate poorly running tests

If a particular test is not running well, or if you'd like to try a faster run, terminate the running test on the client host.

```bash
ycsbhost$ pkill java
```

## Repeat Aerospike Benchmark with New Levels

It may take you several attempts to find the optimal levels for the database. If you find that a test run did not complete successfully, you may need to decrease the target throughput rate and try again. You will not need to empty and reload the database in-between runs during the initial tests.


## Examine Other Log Files

Optionally, you may want to look at the other files that YCSB provides. These will give you a more detailed picture of whether this test setting is the one you want to proceed with.

| File | Description |
| --- | --- |
| run.out | Raw output of the run. As this file is only written at the end of the test, the file may be empty until the test has completed. |
| run.err | The file logs any errors that may occur. Note that you will see errors that start with "DBWrapper: report latency for each error is false"; these are not an issue. |
| UPDATE.json | Output for updates. |
| READ.json | Output for reads. |
| Intended-UPDATE.json | Data on intended updates. |
| Intended-READ.json | Data on intended reads. |
| Intended-CLEANUP.json | Cleanup log for intended data. |
| CLEANUP.json | Cleanup data for the test run. |


{{#note markdown=true}}
While not required, it is good practice to copy these files into a new directory between each run. This will give you better information on which tests succeeded, or failed. You can follow the instructions in step 7 below.{{/note}}

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 6 - Run a long-term test
  #
  ############################################################################
---}}
{{#steps-step 6 "Run a long-term test" markdown=true}}
Once you are comfortable with short-term tests, you can begin the longer-term ones.

The basic parameters of the published benchmark test are as follows:
  * 400 M records
  * 1,000-byte record size
  * 50% read / 50% update
  * Uniform (random) data access pattern


Alter the parameters to better suit your benchmark. Radically changing the parameters will result in very different hardware requirements. You must check to ensure that any saturation you see is actually occurring on the server, as opposed to the client. Aerospike is working on additional tooling to allow the use of multiple clients in parallel for these tests.

There are 2 files that are used in the benchmark. These are both based off the standard `workloada`:
  * Load - This file controls the loading of data. The information on performance during the load will help you determine the best starting point for running the actual test.
  * Run - This file controls the actual benchmark.

## Workload file 1: Load - /opt/YCSB/workloads/workload-asload

| Parameter   | Default Value | Notes |
| ---         | ---           | ---   |
| recordcount | 400,000,000   | Changing this value will linearly change the data storage requirements on SSDs, as well as the amount of memory needed for the primary index. |
| operationcount | 400,000,000 | Must be the same as recordcount. |
| fieldcount | 10 | This is the total number of fields (analogous to columns) for every record. |
| fieldlength | 100 | The size (in bytes) of each field. 10 fields of 100 bytes is roughly equivalent to 1000 bytes per record.  |


## Workload file 2: Run - /opt/YCSB/workloads/workload-asrun

Workload file 2: Run

| Parameter   | Default Value | Notes |
| ---         | ---           | ---   |
| recordcount | 400,000,000   | Changing this value will linearly change the data storage requirements on SSDs. |
| fieldcount | 10 | This is the total number of fields (analogous to columns) for every record. |
| fieldlength | 100 | The size (in bytes) of each field. 10 fields of 100 bytes is roughly equivalent to 1000 bytes per record.  |
| readproportion | 0.5 | The proportion of operations that is made up of reads. |
| updateproportion | 0.5 | The proportion of operations that is made up of updates. |
| requestdistribution | uniform | 'Uniform' means random distribution across all the data.  |

After having run the benchmark program using 10-minute tests, proceed with a longer-term test, such as 3 hours, 6 hours, or longer. 

To run the benchmark test, run the following, where:
  * [NODE\_IP\_ADDR] is the IP address of any node in your Aerospike cluster.
  * [TARGET_LEVEL] is the target throughput rate.
  * [SECONDS] is the number of seconds in the test, such as 21,600 for a six-hour test.


```bash
ycsbhost$ nohup bin/ycsb run aerospike -s -threads 275 -P workloads/workload-asrun -p as.host=[NODE\_IP\_ADDR] -p target=[TARGET_LEVEL] -p maxexecutiontime=[SECONDS] > run.out 2> run.err &
```

## Examine `run.out` frequently

Remember to periodically examine `run.out` to make sure that the test is running accurately.

Even if you pass the short-term testing stage with flying colors, you could encounter problems (especially with Amazon services) when running a long-term test. Make sure your machine can keep up.

In particular, if you see the following error line:
```asciidoc
java.lang.ArrayIndexOutOfBoundsException
```
this is a limitation in YCSB. You might come across this error when latencies are so high that integers become negative; if you do, restart your test at a lower throughput.


{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 7 - Move Output Files
  #
  ############################################################################
---}}
{{#steps-step 7 "Move output files" markdown=true}}

When the test is complete, move the results to their own directory. The following command will create a directory based on the current time, and copy the `run.*` and `*.json` files:


```bash
ycsbhost$ D=results/\`date +%F-%T\` ; mkdir -p $D ; cp run.out run.err $D; cp **.json $D
```

The test run does not have to be complete in order to copy the files and generate graphs.

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 8 - Generate graphs
  #
  ############################################################################
---}}
{{#steps-step 8 "Generate graphs" markdown=true}}

The final step is to generate graphs from your test run. Go to the [Generating Graphs](/docs/benchmarks/cassandra/simple_ycsb/graphs.html) page.
{{/steps-step}}

{{/steps}}
