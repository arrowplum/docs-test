---
title: Benchmarks
description: Use the Aerospike Node.js client benchmark tool to apply load and test your configuration. 
---

Use the Aerospike Node.js client benchmark tool to apply load and test your configuration. 

The benchmark tool is a collection of example programs that provide an end-to-end example of a more complex Aerospike Node.js program.

### Setup

To use the benchmark tool, install the _aerospike_ module in the _benchmarks_ directory.

From the *benchmarks* directory, run the following to install the dependencies:

```
npm install ../
npm update
```

### Running the main Benchmark

_main.js_ runs multiple batches of operations against an Aerospike cluster. It can run for a specified number of iterations or time frame. 

To run _main.js_: 

```
node main.js
```

The configuration parameters to run this benchmark are specified in _config.json_. A sample _config.json_ is in the _benchmarks_ directory. Modify the following parameters in _config.json_ to run your desired configuration:
 
- **host** &mdash; The Aerospike host node (default = `localhost`).
- **port** &mdash; The port to use to connect to the Aerospike server (default = `3000`).
- **namespace** &mdash; All the operations for benchmark are done on this namespace (default = `test`).
- **set** &mdash; The set name where all benchmark operations are performed (default = `demo`).
- **user** &mdash; The username to use to connect with the secured cluster (default = `null`).
- **password** &mdash; The password for the user connecting to the secured cluster (default = `null`).
- **timeout** &mdash; The global timeout for all read/write operations used in the benchmark (default = `0`; never timeout).
- **ttl** &mdash; The time-to-live value for objects written during the benchmark (default `10000` seconds).
- **log** &mdash; The log level of the client module (default = `INFO`). 
- **operations** &mdash; The number of operations for a single batch of operations (default = `100`).
- **iterations** &mdash; The number of iterations for the benchmark to run (default = `null`; runs indefinitely).
- **processes** &mdash; The number of worker processes. These perform the actual read/write or scan/query operations in the Aerospike cluster (default =  `4`; recommended value is the number of CPUs/cores available).
- **time** &mdash; The time to run the benchmark, specified in seconds/minutes/hours (for example, `30s/30m/30h` runs for the benchmark for 30 seconds/30 minutes/30 hours, respectively (default = `24h`; run for 24 hours).
- **reads** &mdash; The read proportion of the read/write ratio (default = `1`).
- **writes** &mdash; The write proportion of the read/write ratio. (default `1`).
- **keyrange** &mdash; The range of key values to use in the benchmark for read/write operations (default = `0-100000`).
- **binSpec** &mdash; The bin specification for write operations in benchmark, specified using 
  - **name** &mdash; The bin name.
  - **type** &mdash; The bin type (`STRING`, `BYTES`, or `INTEGER`).
  - **size** &mdash; Thew size of data to write to each bin (for integer type bins, the default size is `8`).

### Benchmark Output

The benchmark prints the read/write tps in the following format:

```
info: Fri Oct 02 2015 00:03:55 GMT+0530 (IST) read(tps=14434 timeouts=0 errors=0) write(tps=14350 timeouts=0 errors=0)

info: Fri Oct 02 2015 00:03:56 GMT+0530 (IST) read(tps=14009 timeouts=0 errors=0) write(tps=14119 timeouts=0 errors=0) 

info: Fri Oct 02 2015 00:03:57 GMT+0530 (IST) read(tps=14691 timeouts=0 errors=0) write(tps=14581 timeouts=0 errors=0)

info: Fri Oct 02 2015 00:03:58 GMT+0530 (IST) read(tps=14200 timeouts=0 errors=0) write(tps=14200 timeouts=0 errors=0)
```

It then prints the benchmark run summary in the following format:

**SUMMARY**

- Configuration
- operations  : 100 
- iterations  : undefined 
- processes   : 4
- time        : 30 seconds

- Durations (milliseconds) :  latency histogram of read/write operations.
   
| <=1  | >1  | >2  | >4  | >8  |  >16 | >32  |
|---|---|---|---|---|---|---|
| 8.4%  |11.9%   | 21.7%  |27.0%   |18.4%   |10.6%   |1.9%   |

- Status Codes : histogram for return values of read/write operations.

| 0  |
|---|
|100.0%   |


