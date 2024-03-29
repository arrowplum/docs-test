---
title: Benchmarks
description: Use the Aerospike C# client benchmark tool to generate load on an Aerospike cluster and calculate performance metrics. 
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

The Aerospike C# benchmark tool generates load on a cluster and calculates performance metrics, which are stored in the _AerospikeDemo_ project. 

### Executing

To execute the Aerospike C# benchmark tool:
1. Ensure that Aerospike server(s) is up and running.
1. Open _Aerospike.sln_ in Visual Studio.
1. Press **F5** to start _AerospikeDemo_.
1. Enter the server host IP address and port (typically 3000) of a node in the Aerospike cluster at the top of the window.
1. Enter the **namespace** (default is _test_) configured on the cluster
1. Enter a **set** name.
1. In the left pane, select **Initialize**. 
	The benchmark configuration panel appears.
1. Click **Start**.
	The bottom half of the window displays the results generated by the benchmark tool.
1. Click **Stop** to close the benchmark tool.

### Initializing

The Aerospike benchmark tool seeds the database and measures write performance. This benchmark is used to prepare for the Read/Write benchmark. This calculates and displays the writes per second.

```
2014-01-29 12:01:10 INFO BenchmarkInitialize Begin
2014-01-29 12:01:10 INFO Add node BB9A2ECDC290C00 127.0.0.1:3000
2014-01-29 12:01:10 INFO Initialize 100000 records
2014-01-29 12:01:10 INFO Start 8 generator threads
2014-01-29 12:01:11 INFO write(tps=14932 timeouts=0 errors=0 total=14970))
2014-01-29 12:01:12 INFO write(tps=15331 timeouts=0 errors=0 total=30592))
2014-01-29 12:01:13 INFO write(tps=15650 timeouts=0 errors=0 total=46398))
2014-01-29 12:01:14 INFO write(tps=15618 timeouts=0 errors=0 total=62157))
2014-01-29 12:01:15 INFO write(tps=15429 timeouts=0 errors=0 total=77864))
2014-01-29 12:01:16 INFO write(tps=15565 timeouts=0 errors=0 total=93803))
2014-01-29 12:01:17 INFO write(tps=6197 timeouts=0 errors=0 total=100000))
2014-01-29 12:01:17 INFO BenchmarkInitialize End
```

### Read/Write

The Aerospike benchmark tool measures random read/write performance, according to the benchmark configuration variables:

- Configure load generating threads to increase the number of threads. This increases transaction performance up to an unspecified limit. 
	Every platform has a limit where increasing threads does not increase performance. When you increase threads beyond this limit, the client can freeze and become unresponsive.
	Thread count is not relevant in asynchronous mode because the C# runtime uses its own thread pool to process asynchronous socket calls. Database commands return immediately after placing the asynchronous command on the queue, so only one thread is usually required to generate load.
- In asynchronous mode, configure the maximum number of concurrent commands. This indicates the maximum commands to  concurrently process at any point in time. 
	If the command queue is full, new commands are blocked until a processing slot becomes available. Increasing max commands usually increases performance up to an unspecified limit. Performance degrades if you exceed this limit. 

```
2014-01-29 12:19:41 INFO BenchmarkReadWrite Begin
2014-01-29 12:19:41 INFO Add node BB9A2ECDC290C00 127.0.0.1:3000
2014-01-29 12:19:41 INFO Read/write using 1000000 records
2014-01-29 12:19:41 INFO Start 8 generator threads
2014-01-29 12:19:42 INFO write(tps=3220 timeouts=0 errors=0) read(tps=13084 timeouts=0 errors=0) total(tps=16304 timeouts=0 errors=0)
2014-01-29 12:19:43 INFO write(tps=3454 timeouts=0 errors=0) read(tps=13891 timeouts=0 errors=0) total(tps=17345 timeouts=0 errors=0)
2014-01-29 12:19:44 INFO write(tps=3506 timeouts=0 errors=0) read(tps=14244 timeouts=0 errors=0) total(tps=17750 timeouts=0 errors=0)
2014-01-29 12:19:45 INFO write(tps=3525 timeouts=0 errors=0) read(tps=13953 timeouts=0 errors=0) total(tps=17478 timeouts=0 errors=0)
2014-01-29 12:19:46 INFO write(tps=3145 timeouts=0 errors=0) read(tps=12575 timeouts=0 errors=0) total(tps=15720 timeouts=0 errors=0)
2014-01-29 12:19:47 INFO write(tps=3188 timeouts=0 errors=0) read(tps=12764 timeouts=0 errors=0) total(tps=15952 timeouts=0 errors=0)
2014-01-29 12:19:48 INFO write(tps=3533 timeouts=0 errors=0) read(tps=14191 timeouts=0 errors=0) total(tps=17724 timeouts=0 errors=0)
2014-01-29 12:19:50 INFO write(tps=3325 timeouts=0 errors=0) read(tps=14032 timeouts=0 errors=0) total(tps=17357 timeouts=0 errors=0)
2014-01-29 12:19:51 INFO write(tps=3445 timeouts=0 errors=0) read(tps=14139 timeouts=0 errors=0) total(tps=17584 timeouts=0 errors=0)
2014-01-29 12:19:52 INFO write(tps=3375 timeouts=0 errors=0) read(tps=14108 timeouts=0 errors=0) total(tps=17483 timeouts=0 errors=0)
2014-01-29 12:19:53 INFO write(tps=3566 timeouts=0 errors=0) read(tps=13860 timeouts=0 errors=0) total(tps=17426 timeouts=0 errors=0)
2014-01-29 12:19:54 INFO write(tps=3371 timeouts=0 errors=0) read(tps=13511 timeouts=0 errors=0) total(tps=16882 timeouts=0 errors=0)
2014-01-29 12:19:55 INFO write(tps=3615 timeouts=0 errors=0) read(tps=13791 timeouts=0 errors=0) total(tps=17406 timeouts=0 errors=0)
2014-01-29 12:19:56 INFO write(tps=3551 timeouts=0 errors=0) read(tps=13908 timeouts=0 errors=0) total(tps=17459 timeouts=0 errors=0)
2014-01-29 12:19:57 INFO write(tps=3455 timeouts=0 errors=0) read(tps=14275 timeouts=0 errors=0) total(tps=17730 timeouts=0 errors=0)
2014-01-29 12:19:58 INFO write(tps=3373 timeouts=0 errors=0) read(tps=13965 timeouts=0 errors=0) total(tps=17338 timeouts=0 errors=0)
2014-01-29 12:19:59 INFO write(tps=3470 timeouts=0 errors=0) read(tps=13879 timeouts=0 errors=0) total(tps=17349 timeouts=0 errors=0)
2014-01-29 12:20:00 INFO write(tps=3320 timeouts=0 errors=0) read(tps=13279 timeouts=0 errors=0) total(tps=16599 timeouts=0 errors=0)
2014-01-29 12:20:01 INFO write(tps=3361 timeouts=0 errors=0) read(tps=13313 timeouts=0 errors=0) total(tps=16674 timeouts=0 errors=0)
2014-01-29 12:20:01 INFO Stop requested. Waiting for thread to end.
2014-01-29 12:20:02 INFO BenchmarkReadWrite End
```

