---
title: Introduction - C Client
description: Use the Aerospike C client to build C applications to store and retrieve data from the Aerospike cluster.
---

The Aerospike C client allows you to build C applications to store and retrieve data from the Aerospike cluster.  The C client is a smart client that periodically pings nodes for cluster status and manages interactions with the Aerospike cluster.  The C client supports both synchronous and asynchronous command models.

### Supported Platforms

The C client is currently available on the following 64-bit platforms:
- Linux
- MacOS
- Windows

Aerospike provides certified packages for the following platforms:
- Redhat/CentOS 6, 7
- Debian 8, 9, 10
- Ubuntu 14.04, 16.04, 18.04
- MacOS 10.9+
- Windows 7+

These certified packages are available for download in [Released Packages]({{book.vars.download-url}}). It is also available in source code form on [Aerospike C Client Github](https://github.com/aerospike/aerospike-client-c).

### Code
The following example writes a record to the Aerospike Server:

```cpp
as_error err;
as_config config;
as_config_init(&config);
 
config.hosts[0] = { .addr = "127.0.0.1", .port = 3000 };
 
aerospike as;
aerospike_init(&as, &config);
 
aerospike_connect(&as, &err);

as_key key;
as_key_init(&key, "test", "demo_set", "test_key");
  
as_record rec;
as_record_inita(&rec, 2);
as_record_set_int64(&rec, "test-bin-1", 1234);
as_record_set_str(&rec, "test-bin-2", "test-bin-2-data");
  
aerospike_key_put(&as, &err, NULL, &key, &rec);

aerospike_close(&as); 
aerospike_destroy(&as);
```

<div class="text-center">
<a class="button primary" href="/docs/client/c/start">GET STARTED</a>
</div>
