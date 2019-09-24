---
title: Getting Started
description: Use the Aerospike Node.js client to build your real-time application for the Aerospike database.
styles:
  - /assets/styles/ui/steps.css
---

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - INSTALL
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Install Client" markdown=true}}

Install the Aerospike Node.js client library, which includes the library (libaerospike), header files, examples, and other resourses.


The source code is available on **<a id="github" href="https://github.com/aerospike/aerospike-client-nodejs">GitHub <span class="fa fa-github" style="font-size: 1.5em"></span></a>**

  {{steps-action "Install Node.js Client Library" "/docs/client/nodejs/install"}}

---

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

Run the examples bundled with the Aerospike Nodejs client.

  {{steps-action "Examples" "/docs/client/nodejs/examples"}}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - RUN BENCHMARK
  ----------------------------------------------------------------------------}}
{{#steps-step 4 "Run Benchmark" markdown=true}}

To simulate application database usage patterns, run the benchmark tool.

  {{steps-action "Benchmarks" "/docs/client/nodejs/benchmarks"}}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP – BEGIN DEVELOPING
  ----------------------------------------------------------------------------}}
{{#steps-step 5 "Begin Developing" markdown=true}}

Read the following topics to understand how to:
- Connect to an Aerospike cluster. 
- Read and write a single record.


  {{steps-action "Node.js Client Library" "/docs/client/nodejs/usage"}}

---

The API reference is available online.

  {{steps-action "API Reference" "https://www.aerospike.com/apidocs/nodejs/"}}

{{/steps-step}}
{{/steps}}

