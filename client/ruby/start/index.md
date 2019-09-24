---
title: Getting Started
description: Use the Aerospike Ruby client to build Ruby applications to store and retrieve data in the Aerospike database.
styles:
  - /assets/styles/ui/steps.css
---

Use the Aerospike Ruby client to build Ruby applications to store and retrieve data in the Aerospike database.

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - INSTALL
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Install Client" markdown=true}}

Install the Aerospike Ruby client library, which includes source code, examples, and the benchmark tool.

  {{steps-action "Install Ruby Client Library" "/docs/client/ruby/install"}}

---

You can also start from **<a id="github" href="http://github.com/aerospike/aerospike-client-ruby">GitHub <span class="fa fa-github" style="font-size: 1.5em"></span></a>**

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

Run the provided examples.

  {{steps-action "Examples" "/docs/client/ruby/examples.html" }}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP - RUN BENCHMARK
  ----------------------------------------------------------------------------}}

{{#steps-step 4 "Run Benchmark" markdown=true}}

Run the benchmark tool to place default load against the cluster.

 {{steps-action "Benchmarks" "/docs/client/ruby/benchmarks.html" }}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP – BEGIN DEVELOPING
  ----------------------------------------------------------------------------}}
{{#steps-step 5 "Begin Developing" markdown=true}}

Read these topics to understand how to:
- Connect to an Aerospike cluster. 
- Read and write a single record.
- Create a secondary index.
- Run user-defined functions (UDFs).


  {{steps-action "Ruby Client Library" "/docs/client/ruby/usage/connect"}}

---

Read [Best Practices](/docs/client/ruby/usage/best_practices.html) to understand how to best utilize the client library.

  {{steps-action "Best Practices" "/docs/client/ruby/usage/best_practices.html"}}

---

The API Reference is online.

  {{steps-action "API Reference" "http://www.rubydoc.info/gems/aerospike/"}}

{{/steps-step}}
{{/steps}}

