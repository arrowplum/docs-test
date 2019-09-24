---
title: Benchmarks
description: Use the Aerospike C client library benchmark with the Aerospike database.
---

The Aerospike C client source code [repository](https://github.com/aerospike/aerospike-client-c) 
includes a benchmark tool. The benchmark tool allows you to:

- Read and write data into the database at a particular read/write ratio.
- Change the number of client threads to simulate client concurrency.
- Examine latency distribution from the client side.

## Linux/MacOS

### Build

To build the benchmark tool on Linux/MacOS, get the source code from  [git repo](https://github.com/aerospike/aerospike-client-c) and run `make` from the benchmarks directory:

```bash
$ git clone https://github.com/aerospike/aerospike-client-c.git
$ cd aerospike-client-c
$ git submodule update --init
$ make
$ cd benchmarks 
$ make
```


### Start the Benchmark Tool

Start the benchmark tool:

```bash
$ make run
```

This places a default workload on the localhost cluster:

```bash
$ make run
./target/benchmarks -h 127.0.0.1 -p 3000
hosts:          127.0.0.1
port:           3000
namespace:      test
set:            testset
keys/records:   1000000
object spec:    int
random values:  false
minimum number of transactions:  -1
workload:       read 50% write 50%
threads:        16
max throughput: unlimited
read timeout:   0 ms
write timeout:  0 ms
max retries:    1
debug:          false
latency:        false
shared memory:  false
read replica:   master
read consistency level: one
write commit level: all
2014-11-06 11:43:29 INFO Read/write using 1000000 records
2014-11-06 11:43:29 INFO Start 16 generator threads
2014-11-06 11:43:30 INFO write(tps=10847 timeouts=0 errors=0) read(tps=10772 timeouts=0 errors=0) total(tps=21619 timeouts=0 errors=0)
2014-11-06 11:43:31 INFO write(tps=11477 timeouts=0 errors=0) read(tps=11271 timeouts=0 errors=0) total(tps=22748 timeouts=0 errors=0)
2014-11-06 11:43:32 INFO write(tps=11051 timeouts=0 errors=0) read(tps=10698 timeouts=0 errors=0) total(tps=21749 timeouts=0 errors=0)

```

In this example:

- The number of keys randomly read/updated are 1000000.
- 50% of the requests are reads; 50% of requests are writes.

Call the following for details on the attributes of the benchmark tool:

```bash
$ ./target/benchmarks -u
```

### Benchmark Examples

#### Example 1

This example:

- Connects to 127.0.0.1:3000 using the namespace _test_.
- Uses 100 million keys (integer key value in the range 1 to 100000000) and 50 character string values as the bin value in _testbin_.
- Reads 90% and writes 10% of the time using 20 concurrent threads.

```bash
$ ./target/benchmarks -h 127.0.0.1 -p 3000 -n test -k 100000000 -o S:50 -w RU,90 -z 20
```

#### Example 2

This example:

- Connects to 127.0.0.1:3000 using the namespace _test_ in set _demoset_.
- Uses 1 million keys and 1400 character string values.
- Reads 80% and writes 20% of the time using 8 concurrent threads.
- Restricts transactions per second to 2500.
- Times out reads and writes after 50ms.
- Shows client-side latency percentages in the <=1ms, >1ms, >8ms, >64ms buckets.

```bash
$ ./target/benchmarks -h 127.0.0.1 -p 3000 -n test -s demoset -k 1000000 -o S:1400 -g 2500 -w RU,80 -z 8 -L 4,3
```

## Windows

The benchmarks project is included the aerospike solution in the source code 
[repository](https://github.com/aerospike/aerospike-client-c).

- Double-click `vs/aerospike.sln`
- Click `Build -> Build Solution`
- Right-click on `benchmarks` project and choose `Set as StartUp Project`
- Right-click on `benchmarks` project and choose `Properties`
- Click `Debugging`
- Enter `-h {your server hostname}` in `Command Arguments` field.
- Click `OK`
- Press `{control}{F5}`
- Press `{control}{c}` to stop benchmarks.

The benchmarks should run in a console window.  `Command Arguments` are identical to Linux/MacOS
benchmark arguments.
