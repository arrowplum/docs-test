{{!-- This page is used at /docs/deploy_guides/aws/ --}}
{{!-- This page is used at /docs/deploy_guides/gcp/ --}}

In this storage engine, the index and data is stored in RAM. It's advisable to use this model with replication factor 3 to get the reliability offered by virtue of multiple copies.

**Pros:**
* Faster in-memory access
* Lower storage overhead as compared to persistent storage

**Cons:**

* Requires larger amount of RAM
* Correspondingly higher number of instances required to support required RAM size
* Minimum of 3 nodes required for using replication factor 3
* Need to wait for migrations to complete on each node during any rolling restarts/upgrade

&nbsp; {{!-- Hack to not mess up the formatting later in the page --}}
