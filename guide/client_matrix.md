---
title: Client Matrix
description: Matrix of client's and their features.
assets: /docs/guide/assets
---

Aerospike provides the following client libraries with various levels of functionality:

Each of these features is explained fully in our individual [feature guides](/docs/guide).


| | Key-Value | SC | Scan | Batch Direct | Batch Index | List Ops | Map Ops | Query | UDF | Aggregation | Geo Query | Access Control | IPV6 | TLS | Pred Filter
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Java | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | 
| C# | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) |![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | 
| C / C++ (*) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | 
| Python | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) |
| Node.js | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) |  | ![]({{book.assets}}/check.png) |
| Go |  ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | 
| REST | ![]({{book.assets}}/check.png) | | | | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | | | | | | | | |
| PHP | ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) |  |  |  |
| Ruby | ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png)| ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) |
| Rust | ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png)| ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | ![]({{book.assets}}/check.png) |  |  |  |
| libevent2 (C) |  ![]({{book.assets}}/check.png) | | ![]({{book.assets}}/check.png) | | | | | | | | | | | | |

(*) C client supports libev, libuv, libevent2 asynchronous frameworks.

For more details, check out the architecture guide or the language-specific client manuals under the [Development](/docs/client) tab.
