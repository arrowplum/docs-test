---
title: Getting Started
description:  Use the Aerospike C client library with the Aerospike database to build your real-time application.
styles:
  - /assets/styles/ui/steps.css
---

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - INSTALL
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Install Client" markdown=true}}

Install the Aerospike C client library, which includes library (_libaerospike_), header files, examples, and other resourses.

  {{steps-action "Install C Client Library" "/docs/client/c/install"}}

---

You can download the source code for the Aerospike C client on [GitHub](http://github.com/aerospike/aerospike-client-c).

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - SETUP SERVER
  ----------------------------------------------------------------------------}}
{{#steps-step 2 "Setup Aerospike Server" markdown=true}}

Install the Aerospike server.

  {{steps-action "Install Aerospike Server" "/download/server" }}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - RUN EXAMPLES
  ----------------------------------------------------------------------------}}
{{#steps-step 3 "Run Examples" markdown=true}}

Run the examples included with the Aerospike C client.

  {{steps-action "Examples" "/docs/client/c/examples"}}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - RUN BENCHMARK
  ----------------------------------------------------------------------------}}
{{#steps-step 4 "Run Benchmark" markdown=true}}

Use the benchmark tool to simulate a particular application database usage pattern.

  {{steps-action "Benchmarks" "/docs/client/c/benchmarks"}}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP – BEGIN DEVELOPING
  ----------------------------------------------------------------------------}}
{{#steps-step 5 "Begin Developing" markdown=true}}

Read the following sections to understand how to:
- [Connect](/docs/client/c/usage/connect) to an Aerospike cluster. 
- [Read](/docs/client/c/usage/kvs/read.html) and [write](/docs/client/c/usage/kvs/write.html) a single record.
- Create a [secondary index](/docs/client/c/usage/query/sindex.html).
- Run [UDFs](/docs/client/c/usage/udf).


  {{steps-action "C Client Library" "/docs/client/c/usage/connect"}}

---

To use the Aerospike C client library to its full extent, read the [Best Practices](/docs/client/c/best_practices) section.

  {{steps-action "Best Practices" "/docs/client/c/best_practices"}}

---

The API reference is available online at /apidocs/c.

  {{steps-action "API Reference" "/apidocs/c" }}

{{/steps-step}}
{{/steps}}
