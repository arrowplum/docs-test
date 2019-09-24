---
title: Benchmarks
description: Use the Aerospike Go client benchmark tool to generate load on an Aerospike cluster and calculate performance metrics.
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
---

Use the Aerospike Go client benchmark tool to generate load on an Aerospike cluster and calculate performance metrics. The benchmark tool is an actual application that you can use to test your cluster, not a simplified example. The benchmark tool checks end-to-end latency and optimizes system performance.

The benchmark tool allows you to:

- Read and write data to the database at a particular read/write ratio.
- Change the number of client goroutines to simulate client concurrency.
- Look at latency distribution from the client side.

### Build

To build the Aerospike Go client benchmark tool:

```bash
go get github.com/aerospike/aerospike-client-go
cd $GOPATH/src/github.com/aerospike/aerospike-client-go/tools/benchmark
go build .
```

### Run

To start the default load:

```bash
go run benchmark.go
```

Example output:

```bash

2014/09/09 18:11:04 hosts:    127.0.0.1
2014/09/09 18:11:04 port:   3000
2014/09/09 18:11:04 namespace:    test
2014/09/09 18:11:04 set:    testset
2014/09/09 18:11:04 keys/records: 1000000
2014/09/09 18:11:04 object spec:  I, size: 0
2014/09/09 18:11:04 random bin values false
2014/09/09 18:11:04 workload:   Initialize 100% of records
2014/09/09 18:11:04 concurrency:  32
2014/09/09 18:11:04 max throughput  unlimited
2014/09/09 18:11:04 timeout   0 ms
2014/09/09 18:11:04 max retries   2
2014/09/09 18:11:04 debug:    false
2014/09/09 18:11:05 write(tps=16310 timeouts=0 errors=0 totalCount=16608)
2014/09/09 18:11:06 write(tps=19523 timeouts=0 errors=0 totalCount=36367)
2014/09/09 18:11:07 write(tps=19043 timeouts=0 errors=0 totalCount=55556)
2014/09/09 18:11:08 write(tps=19837 timeouts=0 errors=0 totalCount=75563)
2014/09/09 18:11:09 write(tps=17342 timeouts=0 errors=0 totalCount=93261)
2014/09/09 18:11:10 write(tps=20317 timeouts=0 errors=0 totalCount=113583)

...
Press Ctrl+C to stop
```

### Usage

See the command line help for command usage information:

```bash
./benchmark -u
```

### Benchmark Examples

These examples run typical benchmarks.

#### Example 1

- Connect to localhost:3000 using namespace _test_.
- Use 10 million keys and 50 character string values.
- Read 10% and write 90% of the time using 64 concurrent goroutines.

```bash
./benchmark -h 127.0.0.1 -p 3000 -n test -k 10000000 -o S:50 -w RU,10 -c 64
```

#### Example 2

- Connect to localhost:3000 using namespace _test_.
- Use 1 million keys and 1400 character string values using a single bin.
- Read 80% and write 20% of the time using 32 concurrent goroutines.
- Timeout after 50ms for reads and writes.

```bash
./benchmark -h 127.0.0.1 -p 3000 -n test -k 1000000 -o S:1400 -w RU,80 -T 50 -c 32
```

#### Example 3

- Connect to localhost:3000 using namespace _test_.
- Use 10 million keys and 2000 character string values.
- Read 50% and write 50%
- Print latency information.

```bash
./benchmark -h 127.0.0.1 -p 3000 -n test -k 10000000 -o S:2000 -w RU,50 -c 64 -L 5,1
```

```
2014/11/25 09:18:35 hosts:        127.0.0.1
2014/11/25 09:18:35 port:        3000
2014/11/25 09:18:35 namespace:        test
2014/11/25 09:18:35 set:        test
2014/11/25 09:18:35 keys/records:    1000000
2014/11/25 09:18:35 object spec:    S, size: 2000
2014/11/25 09:18:35 random bin values    false
2014/11/25 09:18:35 workload:        Read 50%, Write 50%
2014/11/25 09:18:35 concurrency:    64
2014/11/25 09:18:35 max throughput    unlimited
2014/11/25 09:18:35 timeout        0 ms
2014/11/25 09:18:35 max retries        2
2014/11/25 09:18:35 debug:        false
2014/11/25 09:18:35 latency:        1:5
2014/11/25 09:18:35 Nodes Found: [BB9A5E34ABAE290]
2014/11/25 09:18:36 write(tps=117385 timeouts=0 errors=0) read(tps=117797 timeouts=0 errors=0) total(tps=235182 timeouts=0 errors=0, count=235182)
2014/11/25 09:18:36         Min(ms)    Avg(ms)    Max(ms)    |<=   1 ms    |>   1 ms    |>   2 ms    |>   4 ms    |>   8 ms    |>  16 ms
2014/11/25 09:18:36     READ    0    0.038    6    | 117309/99.58%    |    488/0.41%    |     93/0.08%    |     11/0.01%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:36     WRITE    0    0.035    6    | 116908/99.59%    |    477/0.41%    |     73/0.06%    |      5/0.00%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:37 write(tps=132547 timeouts=0 errors=0) read(tps=133146 timeouts=0 errors=0) total(tps=265693 timeouts=0 errors=0, count=500875)
2014/11/25 09:18:37         Min(ms)    Avg(ms)    Max(ms)    |<=   1 ms    |>   1 ms    |>   2 ms    |>   4 ms    |>   8 ms    |>  16 ms
2014/11/25 09:18:37     READ    0    0.034    5    | 132703/99.67%    |    443/0.33%    |    102/0.08%    |     10/0.01%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:37     WRITE    0    0.030    5    | 132145/99.70%    |    402/0.30%    |     87/0.07%    |      7/0.01%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:38 write(tps=133343 timeouts=0 errors=0) read(tps=132160 timeouts=0 errors=0) total(tps=265503 timeouts=0 errors=0, count=766378)
2014/11/25 09:18:38         Min(ms)    Avg(ms)    Max(ms)    |<=   1 ms    |>   1 ms    |>   2 ms    |>   4 ms    |>   8 ms    |>  16 ms
2014/11/25 09:18:38     READ    0    0.034    6    | 131725/99.67%    |    435/0.33%    |     72/0.05%    |      2/0.00%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:38     WRITE    0    0.031    5    | 132917/99.68%    |    426/0.32%    |     66/0.05%    |      1/0.00%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:39 write(tps=133148 timeouts=0 errors=0) read(tps=132351 timeouts=0 errors=0) total(tps=265499 timeouts=0 errors=0, count=1031877)
2014/11/25 09:18:39         Min(ms)    Avg(ms)    Max(ms)    |<=   1 ms    |>   1 ms    |>   2 ms    |>   4 ms    |>   8 ms    |>  16 ms
2014/11/25 09:18:39     READ    0    0.034    5    | 131855/99.62%    |    496/0.37%    |    102/0.08%    |      6/0.00%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:39     WRITE    0    0.032    5    | 132728/99.68%    |    420/0.32%    |     75/0.06%    |      4/0.00%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:40 write(tps=131913 timeouts=0 errors=0) read(tps=131953 timeouts=0 errors=0) total(tps=263866 timeouts=0 errors=0, count=1295743)
2014/11/25 09:18:40         Min(ms)    Avg(ms)    Max(ms)    |<=   1 ms    |>   1 ms    |>   2 ms    |>   4 ms    |>   8 ms    |>  16 ms
2014/11/25 09:18:40     READ    0    0.035    10    | 131477/99.64%    |    476/0.36%    |    131/0.10%    |     14/0.01%    |      3/0.00%    |      0/0.00%
2014/11/25 09:18:40     WRITE    0    0.033    10    | 131507/99.69%    |    406/0.31%    |     91/0.07%    |      8/0.01%    |      4/0.00%    |      0/0.00%
2014/11/25 09:18:41 write(tps=132274 timeouts=0 errors=0) read(tps=132981 timeouts=0 errors=0) total(tps=265255 timeouts=0 errors=0, count=1560998)
2014/11/25 09:18:41         Min(ms)    Avg(ms)    Max(ms)    |<=   1 ms    |>   1 ms    |>   2 ms    |>   4 ms    |>   8 ms    |>  16 ms
2014/11/25 09:18:41     READ    0    0.034    6    | 132484/99.63%    |    497/0.37%    |     91/0.07%    |      3/0.00%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:41     WRITE    0    0.031    6    | 131819/99.66%    |    455/0.34%    |     92/0.07%    |      1/0.00%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:42 write(tps=127077 timeouts=0 errors=0) read(tps=126670 timeouts=0 errors=0) total(tps=253747 timeouts=0 errors=0, count=1814745)
2014/11/25 09:18:42         Min(ms)    Avg(ms)    Max(ms)    |<=   1 ms    |>   1 ms    |>   2 ms    |>   4 ms    |>   8 ms    |>  16 ms
2014/11/25 09:18:42     READ    0    0.042    9    | 125937/99.42%    |    733/0.58%    |    156/0.12%    |     22/0.02%    |      2/0.00%    |      0/0.00%
2014/11/25 09:18:42     WRITE    0    0.038    9    | 126363/99.44%    |    714/0.56%    |    143/0.11%    |     17/0.01%    |      4/0.00%    |      0/0.00%
2014/11/25 09:18:43 write(tps=132952 timeouts=0 errors=0) read(tps=133070 timeouts=0 errors=0) total(tps=266022 timeouts=0 errors=0, count=2080767)
2014/11/25 09:18:43         Min(ms)    Avg(ms)    Max(ms)    |<=   1 ms    |>   1 ms    |>   2 ms    |>   4 ms    |>   8 ms    |>  16 ms
2014/11/25 09:18:43     READ    0    0.034    6    | 132567/99.62%    |    503/0.38%    |    104/0.08%    |      6/0.00%    |      0/0.00%    |      0/0.00%
2014/11/25 09:18:43     WRITE    0    0.031    5    | 132492/99.65%    |    460/0.35%    |     97/0.07%    |      1/0.00%    |      0/0.00%    |      0/0.00%
```

