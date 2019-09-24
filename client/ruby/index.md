---
title: Introduction - Ruby Client
description: Use the Aerospike Ruby client to build Ruby applications to store and retrieve data from the Aerospike database.
categories:
  - aerospike-client-ruby
tags:
  - aerospike-client-ruby
  - Ruby
---

Use the Aerospike Ruby client to build Ruby applications to store and retrieve data from the Aerospike database.

The library is compatible with Ruby 2.3+ and supports Linux, Mac OS X and various other BSDs.

### Code

This example creates a client and makes a cluster connection, creates a key, writes and deletes a record, and closes the cluster connection.

```ruby
require 'rubygems'
require 'aerospike'

include Aerospike

client = Client.new('127.0.0.1')

key = Key.new('test', 'test', 'key value')
bin_map = {
  'bin1' => 'value1',
  'bin2' => 2,
  'bin4' => ['value4', {'map1' => 'map val'}],
  'bin5' => {'value5' => [124, "string value"]},
}

client.put(key, bin_map)
record = client.get(key)
record.bins['bin1'] = 'other value'

client.put(key, record.bins)
record = client.get(key)
puts record.bins

client.delete(key)
puts client.exists(key)

client.close
```

<div class="text-center">
<a class="button primary" href="/docs/client/ruby/start">GET STARTED</a>
</div>
