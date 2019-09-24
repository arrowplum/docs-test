---
title: Getting Started
description: Use  the Aerospike Go client to build Go applications to store and retrieve data in the Aerospike database.
styles:
  - /assets/styles/ui/steps.css
---

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - INSTALL
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Install Client" markdown=true}}

Install the Aerospike Go client library, which includes source code, examples, and the benchmark tool.

  {{steps-action "Install Go Client Library" "/docs/client/go/install"}}

---

You can also start from **<a id="github" href="http://github.com/aerospike/aerospike-client-go">GitHub <span class="fa fa-github" style="font-size: 1.5em"></span></a>**

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - SETUP SERVER
  ----------------------------------------------------------------------------}}
{{#steps-step 2 "Setup Aerospike Server" markdown=true}}

Download and install the Aerospike server.

  {{steps-action "Install Aerospike Server" "/download/server" }}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - RUN EXAMPLES
  ----------------------------------------------------------------------------}}

{{#steps-step 3 "Run Examples" markdown=true}}

Run the provided examples.
  {{steps-action "Examples" "/docs/client/go/examples.html" }}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP - RUN BENCHMARK
  ----------------------------------------------------------------------------}}

{{#steps-step 4 "Run Benchmark" markdown=true}}

Run the benchmark tool to place default load against the cluster, and optimize system performance.

 {{steps-action "Benchmarks" "/docs/client/go/benchmarks.html" }}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP – BEGIN DEVELOPING
  ----------------------------------------------------------------------------}}
{{#steps-step 5 "Begin Developing" markdown=true}}

Read these topics to understand how to:

- Connect to the Aerospike cluster. 
- Read and write a single record.
- Create a secondary index.
- Run UDFs.


  {{steps-action "Go Client Library" "/docs/client/go/usage/connect"}}

---

Read [Best Practices](/docs/client/go/usage/best_practices.html) to understand how to best utilize the client library.

  {{steps-action "Best Practices" "/docs/client/go/usage/best_practices.html"}}

---

Read the API reference for command descriptions.

  {{steps-action "API Reference" "https://godoc.org/github.com/aerospike/aerospike-client-go"}}

{{/steps-step}}
{{/steps}}
