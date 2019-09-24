---
title: Aerospike Session Handler
description: Use the Aerospike PHP client custom session handler to store user sessions.
---

Use the Aerospike PHP client custom session handler to store user sessions. 
- The application must specify the set and namespace to use as the container for user sessions. 
- The session ID is the primary key of the record that contains the session. 
- The key-value pairs in the `$\_SESSION` object are stored in the matching record. 
- The value of `session.gc_maxlifetime` is the record time-to-live (TTL or expiration).
- `session.save_handler string` &mdash; Must be set to *aerospike* to enable sessions support.
- `session.save_path string` &mdash; A string formatted as `*ns*|*set*|*addr*:*port*\[,*addr*:*port*\[,...\]\]` (for example, `test|sess|127.0.0.1:3000`). 
  As with the `$config` of the constructor, only the host info of one cluster node is required.

