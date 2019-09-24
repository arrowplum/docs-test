---
title: Benchmarks
description: Use the Aerospike Ruby client benchmark tool to generate load on an Aerospike cluster and calculate performance metrics. 
categories:
  - aerospike-client-ruby
tags:
  - aerospike-client-ruby
---

Use the Aerospike Ruby client benchmark tool to generate load on an Aerospike cluster and calculate performance metrics. The benchmark tool is an actual application for testing your cluster. The application is not a simplified example. The benchmark tool also checks end-to-end latency to help optimize system performance.

Use the benchmark tool to:

- Read and write data to the database at the specified read/write ratio.
- Simulate client concurrency by adding client threads.
- Review latency distribution from the client side.

{{#note}}
To achieve the best possible performance, benchmark and profile your application using production workloads. 
{{/note}}

### Running

To start a default load:

```bash
$ ruby tools/benchmark/benchmark.rb
```

This output returns:

```bash

hosts:		172.16.224.130
port:		3000
namespace:	test
set:		benchmark
keys/records:	100000
object spec:	I, size: 0
random bins:	false
workload:	Initialize 100% of records
concurrency:	4
max throughput:	unlimited
timeout:	- ms
max retries:	2
debug:		false

I, [2015-01-09T17:01:15.378200 #4741]  INFO -- : write(tps=10777 timeouts=0 errors=0 totalCount=10777)
I, [2015-01-09T17:01:16.423014 #4741]  INFO -- : write(tps=11521 timeouts=0 errors=0 totalCount=22298)
I, [2015-01-09T17:01:17.467992 #4741]  INFO -- : write(tps=15905 timeouts=0 errors=0 totalCount=38203)
I, [2015-01-09T17:01:18.512694 #4741]  INFO -- : write(tps=12199 timeouts=0 errors=0 totalCount=50402)
I, [2015-01-09T17:01:19.557792 #4741]  INFO -- : write(tps=16342 timeouts=0 errors=0 totalCount=66744)
I, [2015-01-09T17:01:20.602505 #4741]  INFO -- : write(tps=12064 timeouts=0 errors=0 totalCount=78808)
I, [2015-01-09T17:01:21.633178 #4741]  INFO -- : write(tps=15654 timeouts=0 errors=0 totalCount=94462)
...


Press Ctrl+C to stop
```

### Usage

```bash
$ ./tools/benchmark/benchmark.rb -u
```

To review command descriptions, see the command line help.

### Benchmark Examples

These examples present typical benchmarks.

#### Example 1

This example:

- Connects to localhost:3000 in the *test* namespace.
- Uses 10 million keys and 50 character string values.
- Reads 10% and writes 90% of the time using 64 concurrent threads.

```bash
$ ./tools/benchmark/benchmark.rb -h 127.0.0.1 -p 3000 -n test -k 10000000 -o S:50 -w RU,10 -c 64
```

#### Example 2

This example:

- Connects to localhost:3000 in the *test* namespace.
- Uses 1 million keys and 1400 character string values using a single bin.
- Reads 80% and writes 20% of the time using 32 concurrent threads.
- Times out after 50ms for reads and writes.

```bash
$ ./tools/benchmark/benchmark.rb -h 127.0.0.1 -p 3000 -n test -k 1000000 -o S:1400 -w RU,80 -T 50 -c 32

hosts:		172.16.224.130
port:		3000
namespace:	test
set:		benchmark
keys/records:	10000000
object spec:	S, size: 50
random bins:	false
workload:	Read 10%, Write 90%
concurrency:	64
max throughput:	unlimited
timeout:	- ms
max retries:	2
debug:		false

I, [2015-01-09T17:04:20.027617 #4957]  INFO -- : write(tps=9366 timeouts=0 errors=0) read(tps=1018 timeouts=0 errors=0) total(tps=10384 timeouts=0 errors=0, count=10384)
I, [2015-01-09T17:04:21.114836 #4957]  INFO -- : write(tps=12510 timeouts=0 errors=0) read(tps=1357 timeouts=0 errors=0) total(tps=13867 timeouts=0 errors=0, count=24251)
I, [2015-01-09T17:04:22.185621 #4957]  INFO -- : write(tps=9665 timeouts=0 errors=0) read(tps=1072 timeouts=0 errors=0) total(tps=10737 timeouts=0 errors=0, count=34988)
I, [2015-01-09T17:04:23.253686 #4957]  INFO -- : write(tps=12880 timeouts=0 errors=0) read(tps=1421 timeouts=0 errors=0) total(tps=14301 timeouts=0 errors=0, count=49289)
I, [2015-01-09T17:04:24.293911 #4957]  INFO -- : write(tps=8630 timeouts=0 errors=0) read(tps=977 timeouts=0 errors=0) total(tps=9607 timeouts=0 errors=0, count=58896)
I, [2015-01-09T17:04:25.330217 #4957]  INFO -- : write(tps=8851 timeouts=0 errors=0) read(tps=991 timeouts=0 errors=0) total(tps=9842 timeouts=0 errors=0, count=68738)
I, [2015-01-09T17:04:26.399140 #4957]  INFO -- : write(tps=10558 timeouts=0 errors=0) read(tps=1109 timeouts=0 errors=0) total(tps=11667 timeouts=0 errors=0, count=80405)
I, [2015-01-09T17:04:27.457142 #4957]  INFO -- : write(tps=9229 timeouts=0 errors=0) read(tps=1050 timeouts=0 errors=0) total(tps=10279 timeouts=0 errors=0, count=90684)
I, [2015-01-09T17:04:28.557114 #4957]  INFO -- : write(tps=11481 timeouts=0 errors=0) read(tps=1248 timeouts=0 errors=0) total(tps=12729 timeouts=0 errors=0, count=103413)
I, [2015-01-09T17:04:29.648508 #4957]  INFO -- : write(tps=11440 timeouts=0 errors=0) read(tps=1262 timeouts=0 errors=0) total(tps=12702 timeouts=0 errors=0, count=116115)
I, [2015-01-09T17:04:30.759660 #4957]  INFO -- : write(tps=10912 timeouts=0 errors=0) read(tps=1256 timeouts=0 errors=0) total(tps=12168 timeouts=0 errors=0, count=128283)
I, [2015-01-09T17:04:31.847171 #4957]  INFO -- : write(tps=11600 timeouts=0 errors=0) read(tps=1299 timeouts=0 errors=0) total(tps=12899 timeouts=0 errors=0, count=141182)
```

