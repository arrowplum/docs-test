---
title: Scan Records
description: Use the Aerospike Ruby client to scan all records in a specified namespace and set.
---

Use the Aerospike Ruby client to scan all records in a specified namespace and set.

### Example

This example counts the number of records returned:

```ruby
require 'rubygems'
require 'aerospike'

include Aerospike

client = Client.new("127.0.0.1")
 
spolicy = ScanPolicy.new
spolicy.concurrent_nodes = true
spolicy.priority = Aerospike::Priority::LOW
spolicy.include_bin_data = false

recs = client.scan_all('test', 'demoset', [], spolicy)

record_count = 0
recs.each do |r|
    record_count += 1
end

puts record_count

client.close
```

The scan policy is initialized, and if
- `concurrent_nodes = true` (default), queries nodes in parallel for records using threads. Otherwise, queries each node sequentially. 
- `include_bin_data= true` (default), specifies bins (all bins by default) to return with each record.

The scan can now execute. On error, the scan stops and an error returns as an exception to the original executing thread.

