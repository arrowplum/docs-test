---
title: Aerospike Benchmarks (asbenchmark)
description: Use the Aerospike benchmarks tool to test your cluster.
---

The Aerospike Benchmarks tool is a precompiled version of the
Java client benchmarks app, which is included in the
[aerospike/aerospike-client-java](https://github.com/aerospike/aerospike-client-java/tree/master/benchmarks)
Git repository. It is provided as part of the tools package.

## Prerequisites

 * Java 1.8 or greater


## Usage

    asbenchmark -u

#### Options

```
$ asbenchmark -u
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
-c,--clusterName <arg>              Set expected cluster name.
-commitLevel <arg>                  Desired replica consistency guarantee when committing a
                                    transaction on the server.
                                    Values:  all | master.  Default: all
-D,--debug                          Run benchmarks in debug mode.
-e,--expirationTime <arg>           Set expiration time of each record in seconds.
                                    -1: Never expire
                                    0: Default to namespace expiration time
                                    >0: Actual given expiration time
-F,--keyFile <arg>                  File path to read the keys for read operation.
-g,--throughput <arg>               Set a target transactions per second for the client. The client
                                    should not exceed this average throughput.
-h,--hosts <arg>                    List of seed hosts in format: hostname1[:tlsname][:port1],...
                                    The tlsname is only used when connecting with a secure TLS
                                    enabled server. If the port is not specified, the default port
                                    is used. IPv6 addresses must be enclosed in square brackets.
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
                                    Values:  master | any | sequence | preferRack.  Default:
                                    sequence
                                    master: Always use node containing master partition.
                                    any: Distribute reads across master and proles in round-robin
                                    fashion.
                                    sequence: Always try master first. If master fails, try proles
                                    in sequence.
                                    preferRack: Always try node on the same rack as the benchmark
                                    first. If no nodes on the same rack, use sequence.
                                    Use 'rackId' option to set rack.
-rackId <arg>                       Set Rack where this benchmark instance resides.  Default: 0
-readModeAP <arg>                   Read consistency level when in AP mode.
                                    Values:  one | all.  Default: one
-readModeSC <arg>                   Read consistency level when in SC (strong consistency) mode.
                                    Values:  session | linearize | allow_replica |
                                    allow_unavailable.  Default: session
-readSocketTimeout <arg>            Set read socketTimeout in milliseconds.
-readTotalTimeout <arg>             Set read totalTimeout in milliseconds.
-S,--startkey <arg>                 Set the starting value of the working set of keys. If using an
                                    'insert' workload, the start_value indicates the first value to
                                    write. Otherwise, the start_value indicates the smallest value
                                    in the working set of keys.
-s,--set <arg>                      Set the Aerospike set name. Default: testset
-sendKey                            Send key to server
-sleepBetweenRetries <arg>          Milliseconds to sleep between retries if a transaction fails and
                                    the timeout was not exceeded. Enter zero to skip sleep.
-socketTimeout <arg>                Set read and write socketTimeout in milliseconds.
-T,--timeout <arg>                  Set read and write socketTimeout and totalTimeout to the same
                                    timeout in milliseconds.
-t,--transactions <arg>             Number of transactions to perform in read/write mode before
                                    shutting down. The default is to run indefinitely.
-timeoutDelay <arg>                 Set read and write timeoutDelay in milliseconds.
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

### Example 1
- Connect to localhost:3000 using test namespace.
- Read 10% and write 90% of the time using 20 concurrent threads.
- Use 100000000 integer keys (starting at "1") and 50 character string values.
- Benchmark synchronous methods.

```bash
asbenchmark -h 127.0.0.1 -p 3000 -n test -k 100000000 -S 1 -o S:50 -w RU,10 -z 20
```

### Example 2
- Connect to localhost:3000 using test namespace.
- Read 80% and write 20% of the time using 8 concurrent threads.
- Use 10000000 integer keys and 1400 bytes values using a single bin.
- Timeout after 50ms for reads and writes.
- Restrict transactions/second to 2500.
- Benchmark synchronous methods.

```bash
asbenchmark -h 127.0.0.1 -p 3000 -n test -k 10000000 -b 1 -o B:1400 -w RU,80 -g 2500 -T 50 -z 8
```

### Example 3
- Benchmark asynchronous methods using a single producer thread and 4 selector threads.
- Limit the maximum number of concurrent commands to 200.
- Use and 50% read 50% write pattern.

```bash
asbenchmark -h 127.0.0.1 -p 3000 -n test -k 100000000 -S 1 -o S:50 -w RU,50 -z 1 -async -asyncMaxCommands 200 -asyncSelectorThreads 4
```

