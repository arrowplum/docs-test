---
title: Getting Started
description: Build real-time applications using the Aerospike C# client APIs with the Aerospike database.
styles:
  - /assets/styles/ui/steps.css

---

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - INSTALL
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Install Client" markdown=true}}

Install the Aerospike C# client, which includes the source, libraries, and examples.

  {{steps-action "Install C# Client" "/docs/client/csharp/install"}}

---

Download the source code for the Aerospike C# client on **<a id="github" href="http://github.com/aerospike/aerospike-client-csharp">GitHub <span class="fa fa-github" style="font-size: 1.5em"></span></a>**

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - SETUP SERVER
  ----------------------------------------------------------------------------}}
{{#steps-step 2 "Setup Aerospike Server" markdown=true}}

Download and install the Aerospike Server.

  {{steps-action "Install Aerospike Server" "/download/server"}}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - RUN EXAMPLES
  ----------------------------------------------------------------------------}}
{{#steps-step 3 "Run Examples" markdown=true}}

Examples demonstrating Aerospike C# client usage are located in the AerospikeDemo project.

{{steps-action "Examples" "/docs/client/csharp/examples"}}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP - RUN BENCHMARKS
  ----------------------------------------------------------------------------}}
{{#steps-step 4 "Run Benchmarks" markdown=true}}

Use the Aerospike benchmark tool to generate load on a cluster and calculate performance metrics. The benchmark tool is in the AerospikeDemo project.

{{steps-action "Benchmarks" "/docs/client/csharp/benchmarks.html"}}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP – BEGIN DEVELOPING
  ----------------------------------------------------------------------------}}
{{#steps-step 5 "Begin Developing" markdown=true}}

Read the following sections to understand how to:
- [Connect](/docs/client/csharp/usage/connect_sync.html) to an Aerospike cluster.
- [Read](/docs/client/csharp/usage/kvs/read.html) and [write](/docs/client/csharp/usage/kvs/write.html) a single record.
- [Create a secondary index](/docs/client/csharp/usage/query/sindex.html).
- [Query on a secondary index](/docs/client/csharp/usage/query/query.html).
- [Aggregate records](/docs/client/csharp/usage/query/aggregate.html).
- [Scan records](/docs/client/csharp/usage/scan/scan.html).
- [Run user-defined functions](/docs/client/csharp/usage/udf).
 
{{steps-action "C# Client Library" "/docs/client/csharp/usage/connect_sync.html"}}

---

To use the Aerospike C client library to its full extent, read the [Best Practices](/docs/client/csharp/best_practices.html) section.

{{steps-action "Best Practices" "/docs/client/csharp/best_practices.html"}}

---

The API reference is available online.

{{steps-action "API Reference" "https://www.aerospike.com/apidocs/csharp/Index.html"}}

{{/steps-step}}
{{/steps}}
