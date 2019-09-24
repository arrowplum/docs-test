---
title: Large Lists
description: Aerospike's Large Data Types (LDT) have been removed in server version 3.15. Alternatives include using the native List and Map data types.
---

{{#warn}}
Aerospike has **removed** the Large Data Type feature as of server version 3.15, after deprecating this functionality 12 months earlier. Refer to the [removal notice](https://www.aerospike.com/blog/aerospike-removed-large-data-type-ldt-feature/) and [deprecation notice](https://www.aerospike.com/blog/aerospike-ldt) documents.
{{/warn}}

{{#note}}
Alternatives to Large List include:
<ul>
 <li>Using native [Lists](/docs/guide/cdt-list.html), and [Maps](/docs/guide/cdt-map.html). Both data types include an extensive API and have long term support.</li>
 <li>As of [server version 4.2.0.2](https://www.aerospike.com/download/server/notes.html#4.2.0.2) the [`write-block-size`](/docs/reference/configuration/index.html#write-block-size) configuration parameter can be set up to 8MB.</li>
</ul>
{{/note}}

