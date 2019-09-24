---
title: Benchmarks
description: Use the Aerospike Java client benchmarks tool to generate load on an Aerospike cluster and calculate performance metrics. 
categories:
  - aerospike-client-java
tags:
  - aerospike-client-java
---

The benchmark application generates load on the cluster and calculates performance metrics for the Java client. The benchmark application allows you to:

- Read and write data to the database at a particular read/write ratio.
- Change the number of client threads to simulate client concurrency.
- Look at latency distribution from the client side.

### Build

```bash
$ cd benchmarks
$ mvn package
```

### Run

To start default benchmarks:

```bash
$ ./run_benchmarks -h <host name>

Benchmark: 127.0.0.1:3000, namespace: test, set: testset, threads: 16, workload: READ_UPDATE
read: 50% (all bins: 100%, single bin: 0%), write: 50% (all bins: 100%, single bin: 0%)
keys: 100000, start key: 0, transactions: 0, bins: 1, random values: false, throughput: unlimited
read policy:
    socketTimeout: 0, totalTimeout: 0, maxRetries: 2, sleepBetweenRetries: 0
    consistencyLevel: CONSISTENCY_ONE, replica: SEQUENCE, reportNotFound: false
write policy:
    socketTimeout: 0, totalTimeout: 0, maxRetries: 2, sleepBetweenRetries: 500
    commitLevel: COMMIT_ALL
Sync: connPoolsPerNode: 1
bin[0]: integer
debug: false
2013-04-22 11:40:49.983 INFO Thread main Add node BB933E391211B00 192.168.100.1 3000
2013-04-22 11:40:50.000 INFO Thread main Add node BB9671B9F211B00 192.168.100.2 3000
2013-04-22 11:40:50.001 INFO Thread main Add node BB9C52127DBBAA4 192.168.100.3 3000
2013-04-22 11:40:51.001 write(tps=75712 timeouts=0 errors=0) read(tps=8273 timeouts=0 errors=0) total(tps=83985 timeouts=0 errors=0)
2013-04-22 11:40:52.002 write(tps=76412 timeouts=0 errors=0) read(tps=8517 timeouts=0 errors=0) total(tps=84929 timeouts=0 errors=0)
2013-04-22 11:40:53.002 write(tps=76351 timeouts=0 errors=0) read(tps=8502 timeouts=0 errors=0) total(tps=84853 timeouts=0 errors=0)
2013-04-22 11:40:54.002 write(tps=75599 timeouts=0 errors=0) read(tps=8369 timeouts=0 errors=0) total(tps=83968 timeouts=0 errors=0)
2013-04-22 11:40:55.002 write(tps=77090 timeouts=0 errors=0) read(tps=8757 timeouts=0 errors=0) total(tps=85847 timeouts=0 errors=0)
...
Ctrl C to stop
```

### Usage

```bash
$ ./run_benchmarks -u

usage: com.aerospike.benchmarks.Main [<options>]
options:
-a,--async                          Benchmark asynchronous methods instead of synchronous methods.
-auth <arg>                         Authentication mode. Values: [INTERNAL, EXTERNAL,
                                    EXTERNAL_INSECURE]
-B,--batchSize <arg>                Enable batch mode with number of records to process in each
                                    batch get call. Batch mode is valid only for RU (read update)
                                    workloads. Batch mode is disabled by default.
-b,--bins <arg>                     Set the number of Aerospike bins. Each bin will contain an
                                    object defined with -o. The default is single bin (-b 1).
-bns,--batchNamespaces <arg>        Set batch namespaces. Default is regular namespace.
-BT,--batchThreads <arg>            Maximum number of concurrent batch sub-threads for each batch
                                    command.
                                    1   : Run each batch node command sequentially.
                                    0   : Run all batch node commands in parallel.
                                    > 1 : Run maximum batchThreads in parallel.  When a node command
                                    finshes, start a new one until all finished.
-C,--asyncMaxCommands <arg>         Maximum number of concurrent asynchronous database commands.
-commitLevel <arg>                  Desired replica consistency guarantee when committing a
                                    transaction on the server. Values:  all | master.  Default: all
-consistencyLevel <arg>             How replicas should be consulted in a read operation to provide
                                    the desired consistency guarantee. Values:  one | all.  Default:
                                    one
-D,--debug                          Run benchmarks in debug mode.
-e,--expirationTime <arg>           Set expiration time of each record in seconds. -1: Never expire,
                                    0: Default to namespace, >0: Actual given expiration time
-F,--keyFile <arg>                  File path to read the keys for read operation.
-g,--throughput <arg>               Set a target transactions per second for the client. The client
                                    should not exceed this average throughput.
-h,--hosts <arg>                    List of seed hosts in format:
                                    hostname1[:tlsname][:port1],...
                                    The tlsname is only used when connecting with a secure TLS
                                    enabled server. If the port is not specified, the default port
                                    is used.
                                    IPv6 addresses must be enclosed in square brackets.
                                    Default: localhost
                                    Examples:
                                    host1
                                    host1:3000,host2:3000
                                    192.168.1.10:cert1:3000,[2001::1111]:cert2:3000
-k,--keys <arg>                     Set the number of keys the client is dealing with. If using an
                                    'insert' workload (detailed below), the client will write this
                                    number of keys, starting from value = startkey. Otherwise, the
                                    client will read and update randomly across the values between
                                    startkey and startkey + num_keys.  startkey can be set using
                                    '-S' or '-startkey'.
-KT,--keyType <arg>                 Type of the key(String/Integer) in the file, default is String
-l,--keylength <arg>                Not used anymore since key is an integer.
-latency <arg>                      ycsb[,<warmup count>] | [alt,]<columns>,<range shift
                                    increment>[,us|ms]
                                    ycsb: Show the timings in ycsb format.
                                    alt: Show both count and pecentage in each elapsed time bucket.
                                    default: Show pecentage in each elapsed time bucket.
                                    <columns>: Number of elapsed time ranges.
                                    <range shift increment>: Power of 2 multiple between each range
                                    starting at column 3.
                                    (ms|us): display times in milliseconds (ms, default) or
                                    microseconds (us)
                                    A latency definition of '-latency 7,1' results in this layout:
                                    <=1ms >1ms >2ms >4ms >8ms >16ms >32ms
                                    x%   x%   x%   x%   x%    x%    x%
                                    A latency definition of '-latency 4,3' results in this layout:
                                    <=1ms >1ms >8ms >64ms
                                    x%   x%   x%    x%
                                    Latency columns are cumulative. If a transaction takes 9ms, it
                                    will be included in both the >1ms and >8ms columns.
-maxRetries <arg>                   Maximum number of retries before aborting the current
                                    transaction.
-N,--reportNotFound                 Report not found errors. Data should be fully initialized before
                                    using this option.
-n,--namespace <arg>                Set the Aerospike namespace. Default: test
-netty                              Use Netty NIO event loops for async benchmarks
-nettyEpoll                         Use Netty epoll event loops for async benchmarks (Linux only)
-o,--objectSpec <arg>               I | S:<size> | B:<size>
                                    Set the type of object(s) to use in Aerospike transactions. Type
                                    can be 'I' for integer, 'S' for string, or 'B' for Java blob. If
                                    type is 'I' (integer), do not set a size (integers are always 8
                                    bytes). If object_type is 'S' (string), this value represents
                                    the length of the string.
-P,--password <arg>                 Password
-p,--port <arg>                     Set the default port on which to connect to Aerospike.
-prole                              Distribute reads across proles in round-robin fashion.
-R,--random                         Use dynamically generated random bin values instead of default
                                    static fixed bin values.
-r,--replica <arg>                  Which replica to use for reads.
                                    Values:  master | any | sequence.  Default: sequence
                                    master: Always use node containing master partition.
                                    any: Distribute reads across master and proles in round-robin
                                    fashion.
                                    sequence: Always try master first. If master fails, try proles
                                    in sequence.
-readSocketTimeout <arg>            Set read socketTimeout in milliseconds.
-readTotalTimeout <arg>             Set read totalTimeout in milliseconds.
-S,--startkey <arg>                 Set the starting value of the working set of keys. If using an
                                    'insert' workload, the start_value indicates the first value to
                                    write. Otherwise, the start_value indicates the smallest value
                                    in the working set of keys.
-s,--set <arg>                      Set the Aerospike set name. Default: testset
-sleepBetweenRetries <arg>          Milliseconds to sleep between retries if a transaction fails and
                                    the timeout was not exceeded. Enter zero to skip sleep.
-socketTimeout <arg>                Set read and write socketTimeout in milliseconds.
-T,--timeout <arg>                  Set read and write socketTimeout and totalTimeout to the same
                                    timeout in milliseconds.
-t,--transactions <arg>             Number of transactions to perform in read/write mode before
                                    shutting down. The default is to run indefinitely.
-tls,--tlsEnable                    Use TLS/SSL sockets
-tlsCiphers,--tlsCipherSuite <arg>  Allow TLS cipher suites
                                    Values:  cipher names defined by JVM separated by comma
                                    Default: null (default cipher list provided by JVM)
-tlsLoginOnly                       Use TLS/SSL sockets on node login only
-totalTimeout <arg>                 Set read and write totalTimeout in milliseconds.
-tp,--tlsProtocols <arg>            Allow TLS protocols
                                    Values:  TLSv1,TLSv1.1,TLSv1.2 separated by comma
                                    Default: TLSv1.2
-tr,--tlsRevoke <arg>               Revoke certificates identified by their serial number
                                    Values:  serial numbers separated by comma
                                    Default: null (Do not revoke certificates)
-U,--user <arg>                     User name
-u,--usage                          Print usage.
-ufn,--udfFunctionName <arg>        Specify the udf function name that must be used in the udf
                                    benchmarks
-ufv,--udfFunctionValues <arg>      The udf argument values comma separated
-upn,--udfPackageName <arg>         Specify the package name where the udf function is located
-V,--version                        Print version.
-W,--eventLoops <arg>               Number of event loop threads when running in asynchronous mode.
-w,--workload <arg>                 I | RU,<percent>[,<percent2>][,<percent3>] |
                                    RR,<percent>[,<percent2>][,<percent3>], RMU | RMI | RMD
                                    Set the desired workload.
                                    -w I sets a linear 'insert' workload.
                                    -w RU,80 sets a random read-update workload with 80% reads and
                                    20% writes.
                                    100% of reads will read all bins.
                                    100% of writes will write all bins.
                                    -w RU,80,60,30 sets a random multi-bin read-update workload with
                                    80% reads and 20% writes.
                                    60% of reads will read all bins. 40% of reads will read a single
                                    bin.
                                    30% of writes will write all bins. 70% of writes will write a
                                    single bin.
                                    -w RR,20 sets a random read-replace workload with 20% reads and
                                    80% replace all bin(s) writes.
                                    100% of reads will read all bins.
                                    100% of writes will replace all bins.
                                    -w RMU sets a random read all bins-update one bin workload with
                                    50% reads.
                                    -w RMI sets a random read all bins-increment one integer bin
                                    workload with 50% reads.
                                    -w RMD sets a random read all bins-decrement one integer bin
                                    workload with 50% reads.
                                    -w TXN,r:1000,w:200,v:20%
                                    form business transactions with 1000 reads, 200 writes with a
                                    variation (+/-) of 20%
-writeSocketTimeout <arg>           Set write socketTimeout in milliseconds.
-writeTotalTimeout <arg>            Set write totalTimeout in milliseconds.
-Y,--connPoolsPerNode <arg>         Number of synchronous connection pools per node.  Default 1.
-z,--threads <arg>                  Set the number of threads the client will use to generate load.
```

**Example 1**

- Run synchronous benchmarks.
- Connect to localhost:3000 using _test_ Namespace.
- Use 10 million integer keys (starting at "1") and 50 character string values.
- Read 10% and write 90% of the time using 20 concurrent threads.

```bash
$ ./run_benchmarks -h 127.0.0.1 -p 3000 -n test -k 10000000 -S 1 -o S:50 -w RU,10 -z 20
```

**Example 2**

- Run synchronous benchmarks.
- Connect to localhost:3000 using test namespace.
- Use 10 million integer keys (starting at "1") and 1400 bytes values using a single bin.
- Read 80% and write 20% of the time using 8 concurrent threads.
- Restrict transactions/second to 2500.
- Timeout after 50ms for reads and writes.

```bash
$ ./run_benchmarks -h 127.0.0.1 -p 3000 -n test -k 10000000 -b 1 -o B:1400 -w RU,80 -g 2500 -T 50 -z 8
```

**Example 3**

- Run asynchronous benchmarks.
- Connect to localhost:3000 using test namespace.
- Use 10 million integer keys (starting at "1") and 50 character string values.
- Read 50% and write 50% of the time.
- Limit the number of concurrent database commands to 300.
- Use 8 event loops with non-blocking connections to process the commands.

```bash
$ ./run_benchmarks -h 127.0.0.1 -p 3000 -n test -k 10000000 -S 1 -o S:50 -w RU,50 -z 1 -async -asyncMaxCommands 300 -eventLoops 8
```
