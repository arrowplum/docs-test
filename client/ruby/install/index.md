---
title: Installation
description: Use the Aerospike Ruby client to develop Ruby applications to store and retrieve data in the Aerospike database.
categories:
  - aerospike-client-ruby
tags:
  - aerospike-client-ruby
  - install
---

Use the Aerospike Ruby client to develop Ruby applications to store and retrieve data in the Aerospike database.

### Prerequisites

The Aerospike Ruby client requires [Ruby](http:#ruby-lang.org) version v1.9.3+, and supports MRI and Rubinious.

The Aerospike Ruby client implements the wire protocol. It does not depend on the C client. It is thread-safe and asynchronous.

Supported operating systems:

- Major Linux distributions (Ubuntu, Debian, Redhat)
- Mac OS X
- Windows (untested)

### Installing

1. Install Ruby 2.3.0 or later
2. Install the gem:

```
gem install aerospike
```

### Running Tests

The Aerospike Ruby client uses RSpec for testing.

To run all tests:

```bash
$ bundle exec rspec
```

### Next Steps

- [Read or write data](/docs/client/ruby/examples.html)
- Try the [Benchmark Tool](/docs/client/ruby/benchmarks.html)
