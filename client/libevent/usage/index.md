---
title: Introduction - libevent Client
description: Use the Aerospike libevent client to write applications to store and retrieve data from an Aerospike database cluster.
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

The Aerospike libevent client provides an asynchronous, non-blocking API to store and retrieve data from an Aerospike database cluster.

The Aerospike libevent client is written in C, and uses the libevent library explicitly. It is only intended for use by applications already running libevent. The application must manage the libevent event bases, and must inform the client which event base to use for each database transaction.

{{#note}}
The Aerospike libevent client only supports key-value functionality. It does not yet support scans, queries, user-defined-functions and other complex features.
{{/note}}

### Why libevent?

An asynchronous API allows parallelization of transactions within a single thread. Because new transactions can be initiated before earlier transactions complete, many transactions may simultaneously be in progress on the same thread. This can allow a far higher transaction throughput per thread than synchronous, blocking APIs. 

Achieving parallelization with fewer threads can be advantageous. For example, mutex contention can be reduced or avoided, and using one thread-per-processor core can be optimal for a variety of reasons.

Because the rate at which the application initiates transactions is independent of how long each transaction takes to complete, there is no natural "push-back" by slower transactions. Also, there are no push-back mechanisms in the Aerospike libevent client; the application is responsible for controlling the transaction throughput. If the application fails to properly manage this, bad symptoms could occur (for example, the client machine may run out of sockets or the libevent library itself may become flooded with events and exhibit bad behavior).

With proper use, this client and application model can offer extremely high performance; however developers must be comfortable with libevent and event-based applications. It is less convenient to develop applications using this library, and may be preferable to use one of the Aerospike synchronous clients.

### Supported Platforms

The Aerospike libevent client requires a 64-bit environment and is available as source code. 

It is certified against libevent2 version 2.0.21-stable, for the following platforms:
- Linux
  - Ubuntu 12.04
  - RedHat/CentOS - 5, 6
  - Debian 6
- Windows
  - Windows 7

<div class="text-center">
<a class="button primary" href="/docs/client/libevent/start">GET STARTED</a>
</div>
