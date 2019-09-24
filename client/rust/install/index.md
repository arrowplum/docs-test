---
title: Installation
description: Get started using Aerospike’s Rust client with the Aerospike database for your real-time, big data application.
categories:
  - aerospike-client-rust
tags:
  - aerospike-client-rust
  - install
---

### Prerequisites

- Rust 1.13 and above.

### Build from Source Code

The client library can be built from source code using Cargo. The source code
is available from [Github](http://github.com/aerospike/aerospike-client-rust).
To clone the source code repository and build the library, run the following
commands:

```bash
git clone git@github.com:aerospike/aerospike-client-rust.git
cd aerospike-client-rust
cargo build
```

### Reference Client Library

When building an application using Aerospike, the Rust client library needs to
be added as a dependency in your `Cargo.toml` file:

```
[dependencies]
aerospike = "0.0.1"
```
